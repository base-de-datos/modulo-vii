## Contenido

* Más sobre usuarios y privilegios
* Los storage engines en MySQL
* Ejercicios

### Más sobre usuarios y privilegios

* Los privilegios en MySQL son una combinación de nombres de usuarios y host de los usuarios
* El usuario `root` tiene privilegios en todo el `localhost`

#### Información sobre usuarios

* Lista de usuarios

```sql
SELECT User, Host, Password FROM mysql.user;
```

* El usuario actual

```sql
SELECT current_user();
```

* Lista de procesos

```sql
SHOW PROCESSLIST;
```

* Matar procesos

```sql
KILL [id]
```

#### Contraseñas

* Modificar los passwords en la base de datos

```sql
SET PASSWORD FOR 'elmer'@'127.0.0.1' = PASSWORD('freddy');
```

* Cambiar el password de un usuario `root`:

```sh
mysqladmin -u root -p flush-privileges password "nueva contraseña"
```

#### Consola

* Ejecutar comandos desde la consola:

```sh
mysql -u root -p -e "SELECT User,Host FROM mysql.user;"
```

* Mostrar los privilegios de un usuario

```console
mysql -u root -p -e "SHOW GRANTS FOR 'elmer@'localhost' \G"
```
#### Privilegios

* Los privilegios pueden realizarse de manera global: `*.*`
* Privilegios a nivel de base de datos: `[nombre de base de datos].*`
* Privilegios a nivel de tablas: `[nombre de la base de datos].[tabla]`
* Privilegios a nivel de columnas: `GRANT SELECT (nombres), INSERT (nombres,cargo) ON activos_fijos.usuarios TO 'elmer'@'localhost';`
* Privilegios de procedimientos almacenados: 

```sql
GRANT CREATE ROUTINE ON activos_fijos.* TO 'elmer'@'localhost';
GRANT EXECUTE ON PROCEDURE activos_fijos.mi_procedimiento TO 'elmer'@'localhost';
```

El comando para asignar privilegios:

```console
GRANT [permiso] ON [nombre de bases de datos].[nombre de tabla] TO ‘[nombre de usuario]’@'localhost’;
```

El comando para revocar privilegios:

```console
REVOKE [permiso] ON [nombre de base de datos].[nombre de tabla] FROM ‘[nombre de usuario]’@‘localhost’;
```
Lista de algunos privilegios:

* `ALL PRIVILEGES`: como mencionamos previamente esto permite a un usuario de MySQL acceder a todas las bases de datos asignadas en el sistema.
* `CREATE`: permite crear nuevas tablas o bases de datos.
* `DROP`: permite eliminar tablas o bases de datos.
* `DELETE`: permite eliminar registros de tablas.
* `INSERT`: permite insertar registros en tablas.
* `SELECT`: permite leer registros en las tablas.
* `UPDATE`: permite actualizar registros seleccionados en tablas.
* `GRANT OPTION`: permite remover privilegios de usuarios.
 
una lista completa de privilegios está en la documentación de mysql ([ver documentación](https://dev.mysql.com/doc/refman/5.1/en/grant.html))

Algunos ejemplos

```sql
CREATE USER 'elmer'@'localhost' IDENTIFIED BY 'elmer';
GRANT ALL ON activos_fijos.* TO 'elmer'@'localhost';
GRANT SELECT ON almacenes.usuarios TO 'elmer'@'localhost';
GRANT USAGE ON *.* TO 'elmer'@'localhost' WITH MAX_QUERIES_PER_HOUR 90;
```

### Storage Engines en MySQL

* MySQL soporta múltiples motores de almacenamiento
* Utilizadas para diferentes tipos de tablas
* Para tablas con transacciones seguras
* Para tablas con transacciones no seguras

Listar los motores de almacenamiento que soporta:

```console
mysql> SHOW ENGINES\G
*************************** 1. row ***************************
      Engine: PERFORMANCE_SCHEMA
     Support: YES
     Comment: Performance Schema
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 2. row ***************************
      Engine: CSV
     Support: YES
     Comment: CSV storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 3. row ***************************
      Engine: MRG_MYISAM
     Support: YES
     Comment: Collection of identical MyISAM tables
Transactions: NO
          XA: NO
  Savepoints: NO
...
```

#### Los engines

##### MyISAM

* Es el motor por defecto en MySQL
* Es el más utilizado en la web, data warehousing, y otros entornos de aplicación. 

##### InnoDB

* Es un motor para transacciones seguras
* Tiene capacidades de commit, rollback, y crash-recovery
* Tiene bloqueo a nivel de filas
* Incrementa la concurrencia multiusuario y el rendimiento
* Tiene soporte de integridad referencial de datos mediante FOREIGN KEY constraints.

##### Memory

* Almacena todo los datos en RAM
* El acceso a la informacińo es muy rápido

##### Merge

* Grupo lógico de tablas de tipo MyISAM
* Se los referencia como un objeto
* Bueno para entornos tales como data warehousing

##### Archive

* Solución para almacenar y recuperar largas cantidades de información
* Información histórica, archivados, o auditorías de seguridad

##### Federated

* Ofrece la posibilidad de enlazar servidores separados
* Crear una base de datos lógica de muchos servidores físicos
* Muy bueno para entornos distribuidos

##### CSV

* Almacena en archivos de texto utilizando el formato de valores separados por coma
* Se usa para fácil intercambio con otros software que puede importar/exportar en CSV

##### Blackhole

* Acepta pero no almacena datos
* Las recuperaciones siempre devuelven un conjunto vacío
* En el diseño de bases de datos distribuidas donde los datos son automáticamente replicados, pero no almacenados localmente

##### Example

* Es un motor "tonto" que no hace nada
* Se puede crear tablas pero no almacena/recupera datos
* Su propósito es servir como un ejemplo para desarrollar nuevos engines
* Es de interés para los desarrolladores

Es importante recordar que no estamos restringidos a utilizar un mismo motor de almacenamiento para un servidor o esquema, se puede utilizar diferentes motores para cada tabla en un esquema.

#### Utilizar un Storage Engine

Al crear una tabla:

```sql
CREATE TABLE t (i INT) ENGINE = INNODB;
```

Establecer el ENGINE en una sesión actual:

```sql
SET storage_engine=MYISAM;
```

Convertir una tabla de una engine a otro, usando `ALTER TABLE`:

```sql
ALTER TABLE t ENGINE = MYISAM;
```

#### Información de engines

La tabla que tiene la información sobre los engines es: `information_schema.TABLES`

### Ejercicios

* Crear el usuario `rrhh` en base al host `127.0.0.1`
* Crear un segundo usuario llamado `rrhh2` en base al host `localhost`
* Crear una base de datos llamada `recursos_humanos`
* Dar permisos sobre toda la base de datos al usuario `rrhh`
* Crear la tabla `funcionarios` con el engine `MyISAM`
* Crear la tabla `cargos` con el engine `InnoDB`
* Dar permisos sobre esta tabla al usuario `rrhh2`
* Listar los engines de la base de datos `recursos_humanos` con el nombre de tabla y el engine
