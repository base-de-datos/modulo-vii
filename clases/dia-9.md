# Procedimientos almacenados

## Contenido

* Archivos CSV
* Procedimientos almacenados
* Ejercicios

## Archivos CSV

* Importar un archivo CSV

```sql
\copy zip_codes FROM '/path/to/csv/ZIP_CODES.txt'
DELIMITER ',' CSV;
```

* Especificando las columnas

```sql
\copy zip_codes(ZIP,CITY,STATE) FROM '/path/to/csv/ZIP_CODES.txt'
DELIMITER ',' CSV;
```
* Exportar un archivo CSV

```sql
COPY (SELECT * FROM country WHERE country_name LIKE 'A%')
TO '/usr1/proj/bray/sql/a_list_countries.copy';
```

* A la salida estándar

```sql
COPY country TO STDOUT (DELIMITER '|');
```

## Procedimientos almacenados

* PostgreSQL permite crear funciones definidas por el usuario
* Puede estar escrito en otros lenguajes como SQL o C
* Estos lenguajes se denominan Procedural Languages (PLs)
* Actualmente hay cuatro PLs en PostgreSQL:
  - PL/pgSQL
  - PL/Tcl
  - PL/Perl
  - PL/Python

### PL/pgSQL

* Es un lenguaje que se carga para PostgreSQL
* Puede ser usado para crear funciones y triggers
* Adicionar estructuras de control al lenguaje SQL
* Puede mejorar complejos programas de computación
* Hereda todos los tipos definidos por el usuario, funciones y operadores
* Es fácil de usar

### Estructura de PL/pgSQL

* El texto completo de la función debe ser un bloque

```sql
[ <<etiqueta>> ]
[ DECLARE
    declaraciones ]
BEGIN
    sentencias
END [ etiqueta ];
```

* Ejemplo de procedimiento almacenado

```sql
CREATE FUNCTION somefunc() RETURNS integer AS $$
<< outerblock >>
DECLARE
    quantity integer := 30;
BEGIN
    RAISE NOTICE 'Quantity here is %', quantity;  -- Prints 30
    quantity := 50;
    --
    -- Create a subblock
    --
    DECLARE
        quantity integer := 80;
    BEGIN
        RAISE NOTICE 'Quantity here is %', quantity;  -- Prints 80
        RAISE NOTICE 'Outer quantity here is %', outerblock.quantity;  -- Prints 50
    END;

    RAISE NOTICE 'Quantity here is %', quantity;  -- Prints 50

    RETURN quantity;
END;
$$ LANGUAGE plpgsql;
```

### Declaraciones

* Todas las variables usadas deben ser declaradas
* Pueden ser tipo de dato SQL: enteros, cadenas, y caracteres

```sql
user_id integer;
quantity numeric(5);
url varchar;
myrow tablename%ROWTYPE;
myfield tablename.columnname%TYPE;
arow RECORD;
```

* Ejemplos de declaración de variables

```sql
quantity integer DEFAULT 32;
url varchar := 'http://mysite.com';
user_id CONSTANT integer := 10;
```

### Declaración parámetros de función

* Los parámetros son identificados con `$1`, `$2`, etc
* Dos maneras para crear un alias:

```sql
CREATE FUNCTION sales_tax(subtotal real) RETURNS real AS $$
BEGIN
    RETURN subtotal * 0.06;
END;
$$ LANGUAGE plpgsql;
```

* La segunda manera es realizarlo de manera explícita

```sql
CREATE FUNCTION sales_tax(real) RETURNS real AS $$
DECLARE
    subtotal ALIAS FOR $1;
BEGIN
    RETURN subtotal * 0.06;
END;
$$ LANGUAGE plpgsql;
```

* Parámetro de entrada la fila de una tabla

```sql
CREATE FUNCTION concat_selected_fields(in_t sometablename) RETURNS text AS $$
BEGIN
    RETURN in_t.f1 || in_t.f3 || in_t.f5 || in_t.f7;
END;
$$ LANGUAGE plpgsql;
```

### Parámetros de salida

* Sumar y restar dos números

```sql
CREATE FUNCTION sum_n_product(x int, y int, OUT sum int, OUT prod int) AS $$
BEGIN
    sum := x + y;
    prod := x * y;
END;
$$ LANGUAGE plpgsql;
```

* Ejecutar el procedimiento

```sql
SELECT sum_n_product(1,2);
sum_n_product
---------------
(3,2)
(1 fila)
```

```sql
SELECT * FROM sum_n_product(1,2);
sum | prod
-----+------
  3 |    2
(1 fila)
```

### Procedimiento para retornar una tabla

* Función PL/pgSQL que retorna una tabla

```sql
CREATE FUNCTION extended_sales(p_itemno int)
RETURNS TABLE(quantity int, total numeric) AS $$
BEGIN
    RETURN QUERY SELECT s.quantity, s.quantity * s.price FROM sales AS s
                 WHERE s.itemno = p_itemno;
END;
$$ LANGUAGE plpgsql;
```

### Alias

* Se puede declarar alias para alguna variable
* Bastante utilizado para NEW y OLD en triggers

```sql
DECLARE
  prior ALIAS FOR old;
  updated ALIAS FOR new;
```

### Copiar el tipo de una variable

* `%TYPE` provee el tipo de dao de una variable o columna de una tabla

```sql
user_id users.user_id%TYPE;
```

### Filas de una tabla

* Una variable compuesta es llamado variable *fila*

```sql
CREATE FUNCTION merge_fields(t_row table1) RETURNS text AS $$
DECLARE
    t2_row table2%ROWTYPE;
BEGIN
    SELECT * INTO t2_row FROM table2 WHERE ... ;
    RETURN t_row.f1 || t2_row.f3 || t_row.f5 || t2_row.f7;
END;
$$ LANGUAGE plpgsql;

SELECT merge_fields(t.*) FROM table1 t WHERE ... ;
```

### Tipos `RECORD`

* Son similares a las variables *fila*
* No tienen predefinido una estructura
* Toman la estructura actual de una fila

### Excepciones

```sql
SELECT * INTO myrec FROM emp WHERE empname = myname;
IF NOT FOUND THEN
    RAISE EXCEPTION 'employee % not found', myname;
END IF;
```

```sql
BEGIN
  SELECT * INTO STRICT myrec FROM emp WHERE empname = myname;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN
      RAISE EXCEPTION 'employee % not found', myname;
    WHEN TOO_MANY_ROWS THEN
      RAISE EXCEPTION 'employee % not unique', myname;
END;
```

### Sentencias dinámicas

* Pueden ser construidos usando la función `format`

```sql
EXECUTE format('UPDATE tbl SET %I = %L WHERE key = %L', colname, newvalue, keyvalue);
```

* Una forma más eficiente a la anterior es:

```sql
EXECUTE format('UPDATE tbl SET %I = $1 WHERE key = $2', colname)
  USING newvalue, keyvalue;
```

### Retornando `NULL`

* Para sentencias de IF/THTN/ELSE que no retornan nada

```sql
BEGIN
    y := x / 0;
EXCEPTION
    WHEN division_by_zero THEN
        NULL;  -- ignore the error
END;
```

```sql
BEGIN
    y := x / 0;
EXCEPTION
    WHEN division_by_zero THEN  -- ignore the error
END;
```

* Retornando filas de una tabla

```sql
CREATE TABLE foo (fooid INT, foosubid INT, fooname TEXT);
INSERT INTO foo VALUES (1, 2, 'three');
INSERT INTO foo VALUES (4, 5, 'six');
```

```sql
CREATE OR REPLACE FUNCTION get_all_foo() RETURNS SETOF foo AS
$BODY$
DECLARE
    r foo%rowtype;
BEGIN
    FOR r IN
        SELECT * FROM foo WHERE fooid > 0
    LOOP
        -- can do some processing here
        RETURN NEXT r; -- return current row of SELECT
    END LOOP;
    RETURN;
END
$BODY$
LANGUAGE plpgsql;
```

```sql
SELECT * FROM get_all_foo();
```

## Ejercicios

1. Realizar un procedimiento almacenado para calcular el fibonacci de un número
2. Realizar un procedimiento almacenado para calcular el factorial de un número
3. Crear el usuario `administrador`
4. Crear la base de datos `movimientos` con dueño el usuario `administrador`
5. Crear la tabla `usuarios` con los campos `id`, `inicios_de_sesion`, `institucion_id`, `paginas_gratis`
6. Crear la tabla `cuentas` con los campos `id`, `saldo`, `usuario_id`, `creado_el`
7. Crear la tabla `transacciones` con los campos `id`, `usuario_id`, `carga`, `accion`, `saldo`, `creado_el`
8. Importar el `CSV` de nombre `usuarios.csv` a la tabla `usuarios`
9. Importar el `CSV` de nombre `cuentas.csv` a la tabla `cuentas`
10. Importar el `CSV` de nombre `transacciones.csv` a la tabla `transacciones`
11. Crear procedimiento almacenado para adicionar `saldo` a una cuenta
  * Debe haber los parámetros de entrada: `monto`, `usuario_id`
  * El valor de entrada de `monto` debe ser positivo
  * Debe actualizar el campo `saldo` de la tabla `cuentas`
  * Debe adicionar un registro en la tabla `transacciones` como `deposit`
12. Crear procedimiento almacenado para restar `saldo` de una cuenta
  * Debe haber dos parámetros de entrada: `monto`, `usuario_id`
  * El valor de entrada de `monto` debe ser negativo
  * No debe permitir restar más del `saldo` que tiene disponible el usuario
  * Debe actualizar el campo `saldo` de la tabla `cuentas`
  * Debe adicionar un registro en la tabla `transacciones` como `withdraw`
13. Crear el usuario `administrador`
  * Con permisos sobre toda la base de datos `movimientos`
