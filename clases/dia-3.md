## Contenido

* Procedimientos almacenados
* Herramientas para backup y restauración
* Ejercicios

## Procedimientos almacenados

* Un procedimiento es un conjunto de instrucciones en el servidor MySQL
* Se ejecutan frecuentemente
* Permite seguridad: no muestra nombres de tablas
* Permite reutilización de código
* En el desarrollo es más rápido agarrar un procedimiento que todas las instrucciones a la vez

### Estructura de un procedimiento almacenado

* Uno o más parámetros de entrada, o también ninguno
* El cuerpo del procedimiento es un bloque de instrucciones

```sql
CREATE PROCEDURE nombre ([parámetro1,parámetro2,...])
[Atributos de la rutina]
BEGIN
  instrucciones
END
```

### Parámetros de entrada y salida

* Es un dato importante para el funcionamiento de un procedimiento
* Parámetros de entrada (`IN`)
* Parámetros de salida (`OUT`)
* Parámetros de entrada/salida (`INOUT`)
* Deben tener definido un tipo
* Por defecto un parámetro es de entrada (`IN`)
* `[{IN|OUT|INOUT} ] nombre_campo tipo_dato`

### Ejemplo 1 de procedimiento almacenado

* Imprimir los números del `1` hasta `n`, donde `n` está dado por el usuario

```sql
DELIMITER //

DROP PROCEDURE IF EXISTS numeros_1_hasta_n;
CREATE PROCEDURE numeros_1_hasta_n(IN n INT)
BEGIN
  DECLARE contador INT DEFAULT 1;
  WHILE true DO
    SELECT contador;
    SET contador = contador + 1 ;
  END WHILE;
END//

DELIMITER ;
```

### Ejecución de procedimientos almacenados

* Se utiliza el comando `CALL`

```sql
CALL numeros_1_hasta_n(5)
```

* Mostrar las características del procedimiento

```sql
SHOW CREATE PROCEDURE numeros_1_hasta_n;

SHOW PROCEDURE STATUS LIKE 'numeros_1_hasta_n'\G
```

### Ejemplo 2 de procedimiento almacenado

* Insertar un cliente en la base de datos

```sql
DELIMITER //

CREATE PROCEDURE insertar(id_cliente INT, nombre_cliente VARCHAR(100), apellido_cliente VARCHAR(100))
COMMENT 'Procedimiento que inserta un cliente a la base de datos'
BEGIN
  IF NOT EXISTS ( SELECT C.ID
  FROM CLIENTE AS C
  WHERE C.ID = id_cliente) THEN
    INSERT INTO CLIENTE(ID, NOMBRE, APELLIDO)
    VALUES ( id_cliente,nombre_cliente,apellido_cliente);
  ELSE
    SELECT 'Este cliente ya existe en la base de datos!';
  END IF;
END//

DELIMITER ;
```

### Ejemplo 3 de procedimiento con salida

* Actores cuyo nombre comienza con una letra dada y cantidad de los mismos

```sql
DELIMITER //

CREATE PROCEDURE pa_actores_buscar(IN letra CHAR(2), OUT actores INT)
BEGIN
  SELECT * FROM actor WHERE nombre LIKE letra;
  SELECT COUNT(*) INTO actores FROM actor
  WHERE nombre LIKE letra;
END//

DELIMITER ;
```

```sql
CALL pa_actores_buscar('h%', @cantidad);
SELECT @cantidad;
```

### Funciones almacenadas

* Tienen un valor de retorno

```sql
DELIMITER //

CREATE FUNCTION fa_actores_cantidad() RETURNS INT
BEGIN
  DECLARE actores INT;
  SELECT COUNT(*) INTO actores FROM actor;

  RETURN actores;
END//

DELIMITER ;
```

```sql
SELECT fa_actores_cantidad();
```

## Herramientas para backup y restauración

* La herramienta encargada de hacer el backup es `mysqldump`
* Utilizada para hacer backup a una base de datos
* El backup no es necesariamente para otro servidor MySQL
* El backup contiene sentencias SQL: crear tabla y/o insertar datos
* Puede ser creado para generar archivos CSV

### Backup de una base de datos

```sh
mysqldump -u elmer -p activos_fijos > activos_fijos.sql
```

### Restauración de una base de datos

```sh
mysqldump -u elmer -p recursos_humanos < recursos_humanos.sql
```

### Exportación en CSV

* Se puede exportar la salida de una consulta

```sql
SELECT nombres, cargo
FROM usuarios
INTO OUTFILE '/tmp/usuarios.csv'
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```

> Nota: No se adiciona el encabezado

### Importar archivos CSV

* Comando para importar un archivo

```sql
LOAD DATA INFILE '/tmp/usuarios.csv'
INTO TABLE usuarios
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES;
```

> Si no tiene encabezado quitar `IGNORE 1 LINES`

### Exportar a un archivo con timestamp

```sql
SET @TS = DATE_FORMAT(NOW(),'_%Y_%m_%d_%H_%i_%s');

SET @FOLDER = '/tmp/';
SET @PREFIX = 'usuarios';
SET @EXT    = '.csv';

SET @CMD = CONCAT("SELECT * FROM usuarios INTO OUTFILE '",
  @FOLDER, @PREFIX, @TS, @EXT,
  "' OPTIONALLY ENCLOSED BY '\"' FIELDS TERMINATED ',' ESCAPED BY '\"'",
  "  LINES TERMINATED BY '\n';"
);

PREPARE statement FROM @CMD;

EXECUTE statement;
```

## Ejercicios

1. Crear la base de datos `movimientos`
2. Crear el usuario `listar`
3. Dar permisos de `SELECT` al usuario `listar` sobre todas las tablas
4. Crear la tabla `usuarios` con los campos `id`, `inicios_de_sesion`, `institucion_id`, `paginas_gratis` con engine `InnoDB`
5. Crear la tabla `cuentas` con los campos `id`, `saldo`, `usuario_id`, `creado_el` con engine `InnoDB`
6. Crear la tabla `transacciones` con los campos `id`, `usuario_id`, `carga`, `accion`, `saldo`, `creado_el` con engine `InnoDB`
7. Importar el `CSV` de nombre `usuarios.csv` a la tabla `usuarios`
8. Importar el `CSV` de nombre `cuentas.csv` a la tabla `cuentas`
9. Importar el `CSV` de nombre `transacciones.csv` a la tabla `transacciones`
10. Realizar backup a la base de datos en el archivo `movimientos.inicial.sql`
11. Crear el usuario `registrar`
12. Dar permisos para crear, y ejecutar procedimientos almacenados a `registrar`
13. Crear procedimiento almacenado para adicionar `saldo` a una cuenta
  * Debe haber los parámetros de entrada: `monto`, `usuario_id`
  * El valor de entrada de `monto` debe ser positivo
  * Debe actualizar el campo `saldo` de la tabla `cuentas`
  * Debe adicionar un registro en la tabla `transacciones` como `deposit`
14. Crear procedimiento almacenado para restar `saldo` de una cuenta
  * Debe haber dos parámetros de entrada: `monto`, `usuario_id`
  * El valor de entrada de `monto` debe ser negativo
  * No debe permitir restar más del `saldo` que tiene disponible el usuario
  * Debe actualizar el campo `saldo` de la tabla `cuentas`
  * Debe adicionar un registro en la tabla `transacciones` como `withdraw`
15. Crear el usuario `administrador`
  * Con permisos sobre toda la base de datos `movimientos`
16. Crear un procedimiento almacenado con un bucle infinito
  * El procedimiento almacenado debe ser creado por el usuario `registrar`
  * El procedimiento almacenado debe ser ejecutado por el usuario `registrar`
  * Con el usuario `administrador` listar todos los procesos
  * Matar el proceso que está ejecutando el bucle infinito
