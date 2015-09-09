## Contenido

* Replicación Master-Slave
* Ejercicio

## Replicación Master-Slave

* Es muy utilizado en términos de Seguridad de Datos
* Soluciones fail-over
* Backup de base de datos en el slave
* Aumentar la redundancia de datos para mejorar la tolerancia a fallos

### Requisitos

* **Master** y **Slave** sobre `debian jessie`
* IP del Master es: **172.17.0.1**
* IP del Slave es: **172.17.0.2**
* Master y Slave están en la misma red **LAN**
* Master permite acceso remoto en el puerto **3306**

### Configuración

* **Fase I**: Se configurará el servidor `Master`
* **Fase II**: Se configurará el servidor `Slave`

### Fase I: Configuración Master (172.17.0.1)

* Instalar MySQL
* Configurar la **Replicación**
* Crear un usuario para la replicación
* Verificar la replicación
* Backup de todas las bases de datos
* Enviar el backup el servidor slave

### Instalar MySQL en el servidor Master

* Procedemos la instalación con el comando `apt-get`

```sh
sudo apt-get install mysql-server
```

### Configurar la replicación en el servidor Master

* Abrir el archivo `sudo nano /etc/mysql/my.cnf`
* Adicionar las siguientes entradas en la sección `[mysqld]`

```conf
[mysqld]
...
server-id        = 1
log_bin          = /var/log/mysql/mysql-bin.log
expire_logs_days = 10
max_binlog_size  = 100M
binlog_do_db     = elmer_mendoza # BD que se va replicar
```

* Reiniciar el servicio

```sh
sudo service mysql restart
```

### Crear un usuario de replicación

```sh
mysql -u root -p
```

```sql
mysql> GRANT REPLICATION SLAVE ON *.* TO 'slave_user'@'%'
     > IDENTIFIED BY 'slave_user';
mysql> FLUSH PRIVILEGES;
```

### Verificar la replicación

```sql
mysql> FLUSH TABLES WITH READ LOCK;
mysql> SHOW MASTER STATUS;

+------------------+----------+---------------+------------------+
| File             | Position | Binlog_Do_DB  | Binlog_Ignore_DB |
+------------------+----------+---------------+------------------+
| mysql-bin.000001 | 11128001 | elmer_mendoza |                  |
+------------------+----------+---------------+------------------+
1 row in set (0.00 sec)

mysql> exit;
```

### Exportar todas las bases de datos

* Exportar todas las bases de datos y la información de la base de datos master con el siguiente comando:

```sh
mysqldump -u root -p --all-databases --master-data > /tmp/dbdump.db
```

* Después de exportar, conectarse a la base de datos y ejecutar

```sql
mysql> UNLOCK TABLES;
mysql> quit;
```

### Subir el backup al servidor slave

* Con el comando `scp` se puede enviar archivos entre servidores

```bash
scp /tmp/dbdump.db elmer@172.17.0.2:/tmp/dbdump.db
```

* Eso es todo con relación a la configuración del servidor **Master**

### Fase II: Configuración Slave (172.17.0.2)

* Instalar MySQL
* Configurar MySQL
* Restaurar las bases de datos
* Configurar la replicación
* Verificar la replicación

### Instalar MySQL en el servidor Slave

* Procedemos la instalación con el comando `apt-get`

```sh
sudo apt-get install mysql-server
```

### Configurar MySQL

* Editar la configuración de MySQL

```bash
sudo vim /etc/mysql/my.cnf
```

* Modificar la sección `[mysqld]`

```
[mysqld]
...
server-id               = 2
replicate_do_db         = elmer_mendoza # BD a replicarse
log_bin                 = /var/log/mysql/mysql-bin.log
```

### Restaurar las bases de datos

* Restaurar la base de datos

```bash
mysql -u root -p < /tmp/dbdumb.db
```

* Reiniciar el servidor MySQL

```bash
sudo service mysql restart
```

### Configurar la replicación en el Slave

```bash
mysql -u root -p
```

```sql
mysql> SLAVE STOP;
mysql> CHANGE MASTER TO MASTER_HOST='172.17.0.1',
     > MASTER_USER='slave_user', MASTER_PASSWORD='slave_user',
     > MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=11128001;
mysql> SLAVE START;
mysql> SHOW SLAVE STATUS\G;
```
* Verificar IP Master `Master_Host: 172.17.0.1`
* Verificar BD `Replicate_Do_DB: elmer_mendoza`

### Verificar la replicación en Master y Slave

* En el servidor Master:

```sql
mysql> CREATE DATABASE elmer_mendoza;
mysql> USE elmer_mendoza;
mysql> CREATE TABLE employee (c int);
mysql> INSERT INTO employee (c) VALUES (1);
mysql> SELECT * FROM employee;
```

```bash
+------+
|  c  |
+------+
|  1  |
+------+
1 row in set (0.00 sec)
```

* En el servidor Slave:

```sql
mysql> USE elmer_mendoza;
mysql> SELECT * FROM employee;
```

```bash
+------+
|  c  |
+------+
|  1  |
+------+
1 row in set (0.00 sec)
```

### Sitios sobre replicación en MySQL

* [How to Setup MySQL (Master-Slave) Replication in RHEL, CentOS, Fedora](http://www.tecmint.com/how-to-setup-mysql-master-slave-replication-in-rhel-centos-fedora/)
* [Setting the Replication Master Configuration](http://dev.mysql.com/doc/refman/5.1/en/replication-howto-masterbaseconfig.html)

### Ejercicios

1. Configurar un servidor Master para replicación
2. Configurar el usuario `replicacion` para realizar la replicación en Master
3. Configurar un servidor Slave para replicación

Sobre el servidor Master realizar:

1. Crear la base de datos `movimientos`
4. Crear la tabla `usuarios` con los campos `id`, `inicios_de_sesion`, `institucion_id`, `paginas_gratis`
5. Crear la tabla `cuentas` con los campos `id`, `saldo`, `usuario_id`, `creado_el`
6. Crear la tabla `transacciones` con los campos `id`, `usuario_id`, `carga`, `accion`, `saldo`, `creado_el`
7. Importar el `CSV` de nombre `usuarios.csv` a la tabla `usuarios`
8. Importar el `CSV` de nombre `cuentas.csv` a la tabla `cuentas`
9. Importar el `CSV` de nombre `transacciones.csv` a la tabla `transacciones`
