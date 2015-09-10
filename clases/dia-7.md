# PostgreSQL

## Contenido

* Historia de PostgreSQL
* Características
* Instalación en GNU/Linux
* Creación de usuarios
* Creación de base de datos
* Ejercicios

## Historia de PostgreSQL

* Es un SGBDR Orientado a Objetos y Libre
* Licencia PostgreSQL, similar a la BSD o la MIT
* No es manejado por una empresa o persona
  - Está dirigido por una comunidad de desarrolladores
  - El nombre PGDG (PostgreSQL Global Development Group)
  - Apoyados por organizaciones comerciales
* El nombre está entre `PostgreSQL` o `Postgres`

## Características

* Alta concurrencia
  - Mientras un proceso escribe en una tabla
  - otros acceden a la misma tabla sin necesidad de bloqueos
* Amplia variedad de tipos nativos
  - Números de precisión arbitraria
  - Texto de largo ilimitado
  - Figuras geométricas
  - Direcciones IP (IPv4 e IPv6)
  - Bloques de direcciones estilo CIDR
  - Direcciones MAC
  - Arrays
  - Los usuarios pueden crear sus **propios tipos de datos**

### Otras características

* Foreign keys
* Triggers
* Vistas
* Integridad transaccional
* Herencia de tablas
* Tipos de datos y operaciones geométricas
* Transacciones distribuidas

### Funciones

* Bloques de código que se ejecutan en el servidor
* Soporta varios lenguajes:
  - PL/PgSQL, C, C++, Java PL/Java web, PL/Perl, plPHP, PL/Python, PL/Ruby, ...
* Soporte para retornar filas

### Ventajas

* Seguridad en términos generales
* Integridad en BD
* Integridad referencial
* Afirmaciones
* Triggers
* Autorizaciones
* Conexión a DBMS
* Transacciones y respaldos

### Alternativas comerciales

* Gracias a la licencia BSD, se permite comercializar el código
  - Por ej. Enterprise DB (PostgreSQL Plus)
  - Tiene varios agregados
  - Interfaz de desarrollo basado en Java

### GIS

* Para sistemas de información geográfica
* Extensión `PostGIS`
  - Soporte para objetos geográficos
  - Consultas SQL espaciales

### Replicación

* `PgCluster`: replicación multi maestro
* `Slony-I`: replicación maestro-esclavo
* `PyReplica`: replicación maestro-esclavo, y multi maestro asíncrona

### Herramientas de administración

* `PgAdmin3`: entorno de escritorio visual. Soporte en Linux, Windows, Mac OSX.
* `PgAccess`: entorno de escritorio visual
* `PhpPgAdmin`: entorno web
* `psql`: cliente de consola
* `Database Master`: entorno de escritorio visual

### Búsqueda de texto

* `Full text search`
  - incluido en el núcleo a partir de la versión 8.3

### Usuarios destacados

* .org, .info, .mobi y .aero registros de dominios por Afilias.3
* La American Chemical Society.
* BASF.
* IMDb.
* Skype.
* TiVo.
* Penny Arcade.
* Sony Online.4
* U.S. Departamento de Trabajo.
* USPS.
* VeriSign.
* Pictiger.com
* Wisconsin Circuit Court Access con 6 * 180GB DBs replicados en tiempo real.
* OpenACS y .LRN.
* INEGI.
* IFE.
* CartoCiudad.5 del IGN de España.

## Instalación en GNU/Linux

* Comando para instalar:

```bash
sudo apt-get install postgresql
```

### Ficheros

* `/etc/postgresql/9.4/main/postgresql.conf`
  - fichero de configuración principal
  - asignación de parámetros de configuración
* `/etc/postgresql/9.4/main/pg_hba.conf`
  - configuración de autenticación de clientes
* Y otros como `pg_ident.conf`, `PG_VERSION`, ...

### Conceptos

* **Cluster de base de datos**
  - Conjunto de bases de datos
  - Conjunto de usuarios que se conectan al cluster
* **Base de datos*** es una agrupación de `esquemas`
* **Tablespaces** ubicaciones alternativas al del cluster
* **Roles** engloba el concepto de `usuarios` y `grupos`

### Iniciar el demonio de PostgreSQL


* Comando para iniciar el demonio:

```bash
sudo service postgresql start
```

### Configurar el usuario `postgres`

* Iniciar sesión como el usuario `postgres` en el SO

```sh
sudo su - postgres
```

* Ingresar a la base de datos

```sh
psql

postgres=#
```

* Establecer una contraseña al usuario `postgres`

```sql
postgres=# ALTER USER postgres WITH PASSWORD 'postgres';
postgres=# \q
```

* Modificar el archivo `pg_hba.conf`

```sh
vim /etc/postgresql/9.4/main/pg_hba.conf
```

```conf
...
# Database administrative login by Unix domain socket
local   all       postgres                md5

# TYPE  DATABASE  USER      ADDRESS       METHOD
local   all       all                     md5
host    all       all       127.0.0.1/32  md5
...
```

* Reiniciar el servidor de PostgreSQL

```sh
service postgresql restart
```

* Conectarse a PostgreSQL

```bash
psql -U postgres
```

### Creación de usuarios

* Desde sistema operativo

```bash
createuser -U postgres -P elmer
```

* Mediante la consola de Postgres

```sql
CREATE USER elmer WITH PASSWORD 'elmer';
```

* Listar usuarios

```sql
SELECT * FROM pg_user;
```

* Comando eliminar usuarios

```sql
DROP USER elmer;
```

### Creación de base de datos

* Desde el sistema operativo

```bash
createdb -U postgres -O elmer mi_base_de_datos
```

* Mediante la consola de Postgres

```sql
CREATE DATABASE mi_base_de_datos OWNER elmer;
```

* Listar las bases de datos

```sql
SELECT * FROM pg_database;
```

* Comando para eliminar una base de datos

```sql
DROP DATABASE mi_base_de_datos;
```

### Ejercicios

1. Crear la base de datos `activos_fijos`
2. Crear la tabla `usuarios (nombre, cargo)`
3. Crear la tabla `activos (codigo, descripcion)`
4. Generar registros de prueba
5. Relacionar un `usuario` con sus respectivos `activos`
6. Generar un archivo SQL con todas las consultas anteriores
