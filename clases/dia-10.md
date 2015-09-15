# Triggers

## Contenido

* Triggers
* Transacciones
* Copias de seguridad y recuperación
* Ejercicios

## Triggers

* Utiliza el lenguaje PL/pgSQL para definir sus procedimientos
* Es creado con el comando `CREATE FUNCTION`
* Es una función sin argumentos
* Retorna el tipo `trigger`

### Variables especiales

* `NEW` tipo de dato `RECORD`. Es `NULL` en triggers de tipo `DELETE`
* `OLD` tipo de dato `RECORD`. Es `NULL` en triggers de tipo `INSERT`
* `TG_NAME` contiene el nombre del trigger
* `TG_WHEN` muestra el tipo del trigger
* `TG_LEVEL`, `TG_OP`, `TG_RELID`, `TG_RELNAME`, `TG_TABLE_NAME`
* `TG_TABLE_SCHEMA`, `TG_NARGS`, `TG_ARGV[]`

### Ejemplo de trigger

* Definición de la tabla `emp`

```sql
CREATE TABLE emp (
  empname text,
  salary integer,
  last_date timestamp,
  last_user text
);
```

* Definición del procedimiento almacenado para el trigger

```sql
CREATE FUNCTION emp_stamp() RETURNS trigger AS $emp_stamp$
  BEGIN
    -- Check that empname and salary are given
    IF NEW.empname IS NULL THEN
        RAISE EXCEPTION 'empname cannot be null';
    END IF;
    IF NEW.salary IS NULL THEN
        RAISE EXCEPTION '% cannot have null salary', NEW.empname;
    END IF;

    -- Who works for us when she must pay for it?
    IF NEW.salary < 0 THEN
        RAISE EXCEPTION '% cannot have a negative salary', NEW.empname;
    END IF;

    -- Remember who changed the payroll when
    NEW.last_date := current_timestamp;
    NEW.last_user := current_user;
    RETURN NEW;
  END;
$emp_stamp$ LANGUAGE plpgsql;
```

* Definición del trigger

```sql
CREATE TRIGGER emp_stamp BEFORE INSERT OR UPDATE ON emp
  FOR EACH ROW EXECUTE PROCEDURE emp_stamp();
```

### Trigger para auditoría

* Definición de las tablas

```sql
CREATE TABLE emp (
  empname           text NOT NULL,
  salary            integer
);
```

```sql
CREATE TABLE emp_audit(
  operation         char(1)   NOT NULL,
  stamp             timestamp NOT NULL,
  userid            text      NOT NULL,
  empname           text      NOT NULL,
  salary integer
);
```

```sql
CREATE OR REPLACE FUNCTION process_emp_audit() RETURNS TRIGGER AS $emp_audit$
  BEGIN
    --
    -- Create a row in emp_audit to reflect the operation performed on emp,
    -- make use of the special variable TG_OP to work out the operation.
    --
    IF (TG_OP = 'DELETE') THEN
        INSERT INTO emp_audit SELECT 'D', now(), user, OLD.*;
        RETURN OLD;
    ELSIF (TG_OP = 'UPDATE') THEN
        INSERT INTO emp_audit SELECT 'U', now(), user, NEW.*;
        RETURN NEW;
    ELSIF (TG_OP = 'INSERT') THEN
        INSERT INTO emp_audit SELECT 'I', now(), user, NEW.*;
        RETURN NEW;
    END IF;
    RETURN NULL; -- result is ignored since this is an AFTER trigger
  END;
$emp_audit$ LANGUAGE plpgsql;
```

```sql
CREATE TRIGGER emp_audit
AFTER INSERT OR UPDATE OR DELETE ON emp
  FOR EACH ROW EXECUTE PROCEDURE process_emp_audit();
```

## Transacciones

* Es un concepto fundamental en los gestores de base de datos
* Ejecuta múltiples pasos en uno simple
* Las operaciones son del tipo *todo o nada*
* Los estados intermedios no son visibles en concurrencia
* Si una falla ocurre en la transacción
  - Ninguno de los pasos afecta a la base de datos

* Considerar una base de datos de un banco

```sql
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
UPDATE branches SET balance = balance - 100.00
    WHERE name = (SELECT branch_name FROM accounts WHERE name = 'Alice');
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Bob';
UPDATE branches SET balance = balance + 100.00
    WHERE name = (SELECT branch_name FROM accounts WHERE name = 'Bob');
```

* Hay muchas actualizaciones
* La idea es asegurarse que toda las actualizaciones salgan bien
  - o en su defecto no se ejecute ninguna
* Por ej. `Bob` puede recibir $100 sin debitar de `Alice`

### Transaction block

* Se necesita garantizar que no haya error en la operación
* Para eso lo podemos agrupar en una `transaction`
  - que brinda garantía de la operación
* En PostgreSQL una transacción está entre los SQL:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100.00
  WHERE name = 'Alice';
  -- etc etc
COMMIT;
```

* Si vemos que el balance de `Alice` es negativo
  - podemos ejecutar `ROLLBACK` en vez de `COMMIT`
  - todas las actualizaciones serán cancelados

### Savepoints

* Permiten descartar partes de una transacción
  - Mientras el restro del código está `COMMIT`
* Se utiliza el comando `SAVEPOINT`
  - Se puede revertir hacia ese punto con `ROLLBACK TO`
* Se puede revertir muchas veces

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Bob';
-- oops ... forget that and use Wally's account
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Wally';
COMMIT;
```

## Copias de seguridad y recuperación

### `pg_dump`

* Es una utilidad para backups
* Extrae una base de datos PostgreSQL en un archivo de texto
* Realiza backups consistentes incluso en concurrencia
* Dumps son scripts o archivo con formato
* Son archivos de texto plano conteniendo comandos SQL

#### Estructura de `pg_dump`

```bash
pg_dump [opciones-de-conexión] [opciones] [base-de-datos]
```

* Más opciones

```bash
pg_dump --help
```

* Ejemplo: Realizar un backup de la base de datos `movimientos`

```bash
pg_dump -U postgres movimientos > movimientos.sql
```

```bash
pg_dump -Fc activos_fijos > activos_fijo.dump
```

```bash
pg_dump -t transacciones movimientos > transacciones.sql
```

### `pg_dumpall`

* Realizar backup a toda la base de datos

```bash
pg_dumpall [connection-option...] [option...]
```

* Realizar un backup de todas las bases de datos

```bash
pg_dumpall > db.out
```

### `pg_restore`

* Restaura la base de datos de un backup genarado por `pg_dump`
* Estos backups no están en formato de texto plano

```bash
pg_restore [connection-option...] [option...] [filename]
```

```bash
pg_restore --help
```

* Ejemplos de restauración del archivo `db.dump`

```bash
pg_dump -Fc mydb > db.dump

dropdb mydb
pg_restore -C -d postgres db.dump
```

```bash
pg_restore -d newdb db.dump
```

* Restaurar con `psql`

```bash
psql -U posgres -d movimientos -f movimientos.sql
```

```bash
psql -U posgres movimientos < movimientos.sql
```

## Ejercicios

En base a un sistema de facturación, se quiere mantener la integridad de una base de datos:

1. Crear una base de datos llamada `ventas`
2. Crear la tabla `pedidos` con los campos `id`, `cliente_id`, y `monto`
3. Crear la tabla `total_ventas` con los campos `id`, `cliente_id`, y `total`

La tabla `total_ventas` almacena las ventas totales realizados a cada cliente, por ej. si un cliente en una ocasión compró 1000 dolares, luego compró 1250 dolares y hace poco ha vuelto a comprar 2000 dolares, entonces el total vendido a este cliente es de 4250 dolares.

Utilizar los triggers necesarios para mantener los siguientes casos:

1. Si insertamos un nuevo pedido para un cliente existente/nuevo, entonces actualizar la tabla `total_ventas` actualizando el `total` comprado.
2. Si actualizamos el `monto` de un `pedido`, entonces descontar el monto anterior y adicionar el nuevo monto.
3. Si eliminamos un `pedido` de un cliente simplemente descontamos del `total` acumulado el `monto` que se había acumulado con anterioridad.
