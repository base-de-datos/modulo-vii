## Contenido

* Historia de MySQL
* Instalación en GNU/Linux
* Creación de usuarios
* Creación de base de datos
* Ejercicios

### Historia de MySQL

* Es un SGBDR
* Licenciamiento dual
  - GNU GPL
  - Licencia específica
* MySQL es patrocinado por Oracle Corporation (04/2009)
  - Posee el copyright de la mayor parte del código
* Sitios web: Wikipedia, Google, Facebook, Twitter, Flickr, y YouTube
* Interfaces de programación de aplicaciones
  - C, C++, C#, Pascal, Delphi, Eiffel, Smalltalk, Java, Lisp, Perl, PHP, Python, Ruby, Gambas,...
* Interfaz ODBC (MyODBC)

### Instalación en GNU/Linux

* Comando para instalar:

```bash
sudo apt-get install mysql-server
```

* Comando para iniciar el demonio:

```bash
sudo service mysql start
```

* Comando para ingresar a la base de datos

```bash
mysql -u root -p
```

### Creación de usuarios

* Comando para crear un usuario

```sql
CREATE USER elmer IDENTIFIED BY 'elmer';
```

* Listar usuarios en MySQL

```sql
SELECT User, Password FROM mysql.user;
```

* Comando eliminar usuarios

```sql
DROP USER elmer;
```

* Privilegios

```sql
GRANT ALL PRIVILEGES ON *.* TO 'elmer';
```

* Refrescar los privilegios

```sql
FLUSH PRIVILEGES;
```

### Creación de base de datos

* Comando para crear una base de datos

```sql
CREATE DATABASE elmer_mendoza;
```

* Comando para eliminar una base de datos

```sql
DROP DATABASE elmer_mendoza;
```

### Ejercicios

* Crear la base de datos `activos_fijos`
* Crear la tabla `usuarios (nombre, cargo)`
* Crear la tabla `activos (código, descripcion)`
* Generar registros de prueba
* Relacionar un `usuario` con sus respectivos `activos`
* Generar un archivo SQL con todas las consultas anteriores
