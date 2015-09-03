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

### Usuario y privilegios

* Cambiar el password de un usuario `root`:

```console
mysqladmin -u root -p flush-privileges password "new_pwd"
```

* Los privilegios en MySQL son una combinación de nombres de usuarios y host de los usuarios
* El usuario `root` tiene privilegios en todo el `localhost`
* Ejecutar comandos desde la consola:

```console
mysql -u root -p -e "SELECT User,Host FROM mysql.user;"
```

* Eliminar los usuarios que no sirven
* Modificar los passwords en la base de datos

```console
SET PASSWORD FOR 'elmer'@'127.0.0.1' = PASSWORD('freddy');
```

* Mostrar los privilegios de un usuario

```console
mysql -u root -p -e "SHOW GRANTS FOR 'elmer@'localhost' \G"
```

### Herramientas y utilidades para la administración

[TODO]

### Ejercicios 2

[TODO]
