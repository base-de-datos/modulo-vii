# Tablespaces y Schemas

## Contenido

* Tablespaces
* Schemas
* Ejercicios

## Tablespaces

* Permiten al administrador de base de datos definir el lugar de almacenamiento
* Pueden ser referidos por un nombre asignado en la creación
* Hay un mejor control en el uso del disco
  - Creados en diferentes particiones
* Son utilizados en optimización del rendimiento
  - Un índice usado constantemente puede estar en un disco com mayor velocidad de lectura
  - Datos archivados y raramente accesados pueden estar en un disco lento

### Creación de tablespaces

* Ejecutar la siguiente sentencia SQL

```sql
CREATE TABLESPACE fastspace LOCATION '/ssd1/postgresql/data';
```

* La locación debe ser una carpeta vacía y el dueño debe ser el usuario `postgres`
* Los objetos creados dentro del tablespace se guardarán en ese directorio
* La locación no debe ser removible

### Lista de tablespaces

* Para listar se puede utilizar el catálogo de sistema `pg_tablespace`

```SQL
SELECT spcname FROM pg_tablespace;
```

* Se puede usar el metacomando `\db` para listar tablespaces

### Creación de objetos en tablespaces

* Se pueden crear, tablas, índices, y bases de datos enteras
* Creación de una tabla en el tablespace `space1`

```sql
CREATE TABLE foo(i int) TABLESPACE space1;
```

* Tablespace por defecto

```SQL
SET default_tablespace = space1;
CREATE TABLE foo(i in
```

### Eliminación de un tablespace

* No se puede eliminar el tablespace si aún tiene objetos creados en él
* Para eso es necesario eliminar todos sus objetos
* Para la eliminación usar:

```sql
DROP TABLESPACE space1;
```

## Schemas

* Un schema es esencialmente un espacio de nombres
* Contiene el nombre de objetos
  - tablas, tipos de datos, funciones, operadores, ...
  - sus nombres pueden duplicar al de otros esquemas
* El acceso a los nombres debe contener el nombre del schema
* El comando `CREATE` crea un objeto en el schema actual

### Funcionalidad básica

* Crear un schema de nombre `elmer`

```SQL
prueba=# CREATE SCHEMA elmer;
CREATE SCHEMA
```

* Crear o acceder a un objeto en un schema

```sql
nombre_schema.nombre_de_objeto
```

```SQL
base_de_datos.nombre_schema.nombre_tabla
```

* Crear una tabla en un schema

```SQL
prueba=# CREATE TABLE elmer.mendoza (id INT, txt TEXT);
CREATE TABLE
```

* Crear otro schema de nombre `freddy`

```SQL
prueba=# CREATE SCHEMA freddy;
CREATE SCHEMA
```

* Crear una tabla en un schema

```SQL
prueba=# CREATE TABLE freddy.mendoza (id INT, txt TEXT);
CREATE TABLE
```

* Insertar registros

```SQL
prueba=# INSERT INTO elmer.mendoza VALUES(1, 'Ejemplo schema');
INSERT 12345 1
```

### Eliminar un schema

* Utilizando la sentencia `DROP SCHEMA`

```sql
DROP SCHEMA nombre_de_schema;
```

* Eliminar incluyendo todos sus objetos

```sql
DROP SCHEMA mi_schema CASCADE;
```

### Dueño de un schema

* Para establecer el dueño a un schema

```sql
CREATE SCHEMA elmer AUTHORIZATION elmer;

CREATE SCHEMA AUTHORIZATION elmer
```

### Schema `public`

* El esquema por defecto es `public`

```sql
CREATE TABLE productos (...);
```

es equivalente a:

```sql
CREATE TABLE public.productos (...);
```

### Schemas y privilegios

* Por defecto no hay permiso de ingresar a otros schemas sin ser dueños
* El dueño del schema debe asignar el privilegio `USAGE`
* Para que los usuarios hagan uso de los objetos del schema es necesario privilegios adicionales
* Todos por defecto tienen los permisos `CREATE` y `USAGE` en schema `public`
* Remover el acceso el privilegio `CREATE` del schema `public` a todos los usuarios:

```sql
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
```

### Ejercicios

Se tiene parte de un sistema de administración de usuarios, en el cual los funcionarios tienen una cuenta asignada para iniciar sesión. Además existen ciertos cargos en la institución que corresponden a un funcionario. Realizar las siguientes actividades:

1. Crear un usuario `rrhh` en el sistema operativo
2. Crear el usuario `rrhh` en Postgres con método de autenticación `peer`
3. Crear un tablespace en `/home/rrhh/administracion` de nombre `administracion`
4. Crear la base de datos `recursos_humanos` en el tablespace `administracion`
5. Crear el schema `personas`
6. Crear la tabla `funcionarios` en el schema `personas` con los campos:
  - `id` de tipo entero auto-incrementable
  - `nombres` de tipo cadena
  - `fecha_nacimiento` de tipo fecha
  - `creado_el` de tipo fecha y hora
7. Crear el schema `sesiones`
8. Crear la tabla `cuentas` en el schema `sesiones` con los campos:
  - `id` de tipo entero auto-incrementable
  - `usuario` de tipo cadena
  - `contrasena` de tipo cadena
  - `funcionario_id` que hace referencia al `id` de `funcionarios`
9. Crear la tabla `cargos` en el schema `personas` con los campos:
  - `id` de tipo entero auto-incrementable
  - `nombre` de tipo cadena, que indica el nombre del cargo
  - `funcionario_id` que hace referencia al `id` de `funcionarios`
10. Insertar los datos de prueba en las siguientes tablas:
  - Insertar 3 registros en la tabla `funcionarios`
  - Insertar 2 registros en la tabla `cuentas`:
    * el primer registro correspondiente al primer funcionario
    * el segundo registro correspondiente al último funcionario
  - Insertar 4 registros en la tabla `cargos`:
    * 3 de los registros insertados relacionarlo con los funcionarios
11. Listar todos los `funcionarios`, con sus `cargos`, y sus `cuentas` de usuario.
