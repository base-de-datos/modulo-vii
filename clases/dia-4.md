## Contenido

* Conexión a servidor remoto
* Expresiones regulares
* Triggers
* Ejercicios

## Conexión a servidor remoto

* Por defecto MySQL no permite la conexión remota para `root`
* Es necesario deshabilitar en el archivo `/etc/mysql/my.cnf` la línea:
  - `bind-address = 127.0.0.1` a `# bind-address = 127.0.0.1`
* Habilitar los permisos al usuario `root`

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'
IDENTIFIED BY 'password' WITH GRANT OPTION;
```
* Conectarse al servidor remoto

```sh
mysql -u root -p -h ip.host.remoto
```

## Expresiones regulares

* Permiten especificar un patrón para una búsqueda compleja
* Una expresión regular describe una serie de cadenas
* Una expresión regular simple no contiene caracteres especiales
* Una expresión regular más compleja: `B[an]*s`
* En MySQL se utiliza el operador `REGEXP`
* Otra alternativa es con la palabra clave `RLIKE`

### Seleccionar campos

* Para seleccionar los campos de la tabla `usuarios` cuyo campo `nombre` empieza por una vocal

```sql
SELECT * FROM usuarios WHERE nombre REGEXP '^[aeiouAEIOU]';
```

### Caracteres especiales

* El caracter `^` indica el comienzo de una cadena
* El caracter `$` indica el final de una cadena

* Nombres que terminan con una vocal

```sql
SELECT * FROM usuarios WHERE nombre REGEXP '[aeiouAEIOU]$';
```

### Coincidencia con cualquier caracter

* Se utiliza el caracter especial `.` (punto)
* Coincide con cualquier caracter que aparezca en una cadena

* Seleccionar todos los nombres de tres caracteres

```sql
SELECT * FROM usuarios WHERE nombre REGEXP '^...$';
```

### Indicadores de repetición

* `?` indica que el caracter que precede puede aparecer cero o una veces
* `+` indica que el caracter que precede puede aparecer una o más veces
* `*` indica que el caracter que precede puede aparecer cero, una o más veces

* Seleccionar nombres que empiecen en `A` y terminen en `o` o en `os`

```sql
SELECT * FROM usuarios WHERE nombre REGEXP '^A.*os?$';
```

### Indicador genérico de repetición

* Con `{n}` indica que el caracter precedente debe aparecer `n` veces consecutivas
* Con `{n,m}` indica que el caracter precente debe aparecer un mínimo de `n` veces y un máximo de `m` veces consecutivas.
  - Si no se especifica `m` el número máximo no está limitado
* Algunas equivalencias:
  - `a?` es equivalente a `a{0,1}`
  - `a+` es equivalente a `a{1,}`
  - `a*` es equivalente a `a{0,}`

### Coincidencia de caracteres

* Se utiliza el caracter `|` (pipe)
* Cuando la expresión regular cmple de entre dos o más secuencias de caracteres

* Seleccionar nombres que contienen `Juan`, `Pedro`, o `Luis`

```sql
SELECT * FROM usuarios WHERE nombre REGEXP 'Juan|Pedro|Luis';
```

### Secuencia de caracteres

* Una secuencia de caracteres se puede encerrar entre paréntesis para indicar que un caracter especial que le sigue afecta a toda la secuencia

* Verificar si en la secuencia `Pérez` aparece dos o más veces

```sql
(Pérez){2,}
```

### Coincidencia y no coindicencia

* Un conjunto de caracteres encerrados entre corchetes, indica que la coincidencia se cumple si en la cadena aparece cualquiera de ellos
* `[abc]` es equivalente a `a|b|c`
* Un rango de caracteres se especifica con guión: `[a-zA-Z]`
* Para adicionar a un conjunto el caracter guión debe colacarse al principio o al final: `[abc-]`
* Para adicionar a un conjunto el caracter `]` debe colacarse inmediatamente después de `[`: `[]abc]`
* El caracter `^` después de `[` indica que los caracteres a continuación no deben aparecer en la cadena: `[^a-z]`

* Más información sobre expresiones regulares en la documentación de MySQL ([ver aquí](https://dev.mysql.com/doc/refman/5.0/en/regexp.html))

## Triggers

* Es un programa almacenado
* Se ejecuta automáticamente de acuerdo a eventos
* Eventos generados por `INSERT`, `UPDATE`, y `DELETE`
* Los privilegios necesario son `SUPER` y `TRIGGER`
* Casos de uso:
  - Validación de información
  - Calcular atributos derivados
  - Seguimiento de movimientos en la base de datos, etc.

### Estructura de un Trigger

* Es similar a la definición de procedimientos y funciones

```sql
CREATE [DEFINER={usuario|CURRENT_USER}]
TRIGGER nombre_del_trigger {BEFORE|AFTER} {UPDATE|INSERT|DELETE}
ON nombre_de_la_tabla
FOR EACH ROW
<bloque_de_instrucciones>
```

> Nombre de trigger: [tabla]_[evento]_TRIGGER
> -- BEFORE INSERT
> clientes_BI_TRIGGER

### Identificadores `NEW` y `OLD`

* Para relacionar con columnas específicas de una tabla
* `OLD` indica el valor antiguo de la columna
* `NEW` indica el valor nuevo que puede tomar

```sql
OLD.paginas_gratis
NEW.paginas_gratis
```

* `UPDATE`: `OLD` y `NEW`
* `INSERT`: `NEW`
* `DELETE`: `OLD`


### Triggers `BEFORE` y `AFTER`

* Indican si el trigger se ejecuta antes o después del evento
* Hay ciertos eventos que son incompatibles con estas sentencias:
  - `AFTER UPDATE`: no se puede editar valores nuevos `NEW`
  - `AFTER INSERT`: no se puede referenciar valores con `NEW`

### Información de un trigger

* Ver las especificaciones del trigger

```sql
SHOW CREATE TRIGGER logs_AI_TRIGGER;
```

* Ver los triggers de una base de datos

```sql
SHOW TRIGGERS;
```

### Eliminación de un trigger

* La sintaxis para eliminar un trigger

```sql
DROP TRIGGER [IF EXISTS] nombre_de_trigger;
```

### Ejemplo actualizar una variable

```sql
CREATE TABLE cuenta (nro INT, monto DECIMAL(10,2));
CREATE TRIGGER cuenta_BI_TRIGGER BEFORE INSERT ON cuenta
FOR EACH ROW SET @suma = @suma + NEW.monto;
```

```sql
SET @suma = 0;
INSERT INTO cuenta VALUES(137, 14.98),(141,1937.50),(97,-100.00);
SELECT @suma AS 'Monto total insertado';

+-----------------------+
| Monto total insertado |
+-----------------------+
| 1852.48               |
+-----------------------+
```

### Ejemplo de trigger `BEFORE` en `UPDATE`

* Validar la edad de un cliente antes de actualizar

```sql
DELIMITER //

CREATE TRIGGER cliente_BU_TRIGGER
BEFORE UPDATE ON cliente FOR EACH ROW
BEGIN
  -- La edad es negativa?
  IF NEW.edad < 0 THEN
    SET NEW.edad = NULL;
  END IF;
END//

DELIMITER ;
```

### Ejemplo de trigger `AFTER` en `UPDATE`

* Log de auditoría

```sql
DELIMITER //

CREATE TRIGGER detalle_factura_AU_Trigger
AFTER UPDATE ON detalle_factura FOR EACH ROW
BEGIN
  INSERT INTO log_updates(idusuario, descripcion)
  VALUES (user( ),
    CONCAT('Se modificó el registro ','(',
    OLD.iddetalle,',', OLD.idfactura,',',OLD.idproducto,',',
    OLD.precio,',', OLD.unidades,') por ',
    '(', NEW.iddetalle,',', NEW.idfactura,',',NEW.idproducto,',',
    NEW.precio,',', NEW.unidades,')')
  );
END//

DELIMITER ;
```

## Ejercicios

En base a un sistema de facturación, se quiere mantener la integridad de una base de datos:

1. Crear una base de datos llamada `ventas`
2. Crear la tabla `pedidos` con los campos `id`, `cliente_id`, y `monto`
3. Crear la tabla `total_ventas` con los campos `id`, `cliente_id`, y `total`

La tabla `total_ventas` almacena las ventas totales realizados a cada cliente, por ej. si un cliente en una ocasión compró 1000 dolares, luego compró 1250 dolares y hace poco ha vuelto a comprar 2000 dolares, entonces el total vendido a este cliente es de 4250 dolares.

Utilizar los triggers necesarios para mantener los siguientes casos:

1. Si insertamos un nuevo pedido para un cliente nuevo, crear el registro correspondiente en la tabla `total_ventas`.
2. Si insertamos un nuevo pedido para un cliente existente, entonces actualizar la tabla `total_ventas` actualizando el `total` comprado.
3. Si actualizamos el `monto` de un `pedido`, entonces descontar el monto anterior y adicionar el nuevo monto.
4. Si eliminamos un `pedido` de un cliente simplemente descontamos del `total` acumulado el `monto` que se había acumulado con anterioridad.
