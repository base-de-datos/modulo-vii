# Administración e Implementación de Bases de Datos

## MySQL

### Historia de MySQL

completar

### Instalación en GNU/Linux

completar

### Creación de usuarios

completar

### Creación de Base de Datos

completar

### Ejercicios 1

completar

### Sistema de Archivos



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
