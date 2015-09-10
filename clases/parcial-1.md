# Parcial 1: MySQL/MariaDB

Se necesita implementar un sistema que permita realizar el seguimiento de actividades de los funcionarios de una institución, para lo cual es necesario definir la base de datos con las siguientes tablas: `funcionarios`, `tareas`, y `reportes`. Donde:

* `funcionarios` tiene la lista de todos los funcionarios de la institución
* `tareas` tiene la lista de tareas para cada funcionario
* `reportes` tiene la lista de reportes para cada tarea

Lista de `funcionarios`:

```csv
1;"Cameron Galloway";"eget@porttitorscelerisque.co.uk";"1998-12-05"
2;"Alec Fox";"tristique.aliquet@vitae.org";"1989-12-17"
3;"Alfonso Cole";"penatibus.et.magnis@tempusnon.co.uk";"1996-10-03"
4;"Belle Gilmore";"eu.eleifend.nec@dolor.edu";"1987-02-12"
5;"Dorothy Underwood";"sit@egestasFusce.edu";"1983-08-05"
6;"Colin Campbell";"Aliquam.rutrum.lorem@diam.co.uk";"1982-08-12"
7;"Nash Mcintosh";"nonummy@volutpatNulla.net";"1998-01-03"
8;"Timothy Farmer";"Nulla.aliquet@cursusvestibulumMauris.edu";"1999-02-25"
9;"Lael Vinson";"velit.dui@ametconsectetuer.co.uk";"1987-01-12"
10;"Maris Mcfadden";"vel.turpis@tinciduntaliquamarcu.ca";"1982-11-15"
11;"Dara Jones";"Suspendisse@suscipitnonummyFusce.net";"1999-06-15"
12;"Xantha Frederick";"ligula.Aliquam.erat@lorem.net";"1988-09-21"
13;"Zane Barlow";"consectetuer.ipsum@aliquam.co.uk";"1989-11-14"
14;"Rahim Rojas";"facilisis.non.bibendum@pellentesque.net";"1997-04-09"
15;"Echo Sanders";"magna.nec.quam@facilisisfacilisis.co.uk";"1988-11-09"
16;"Athena Dawson";"sed.libero@Nuncmauris.co.uk";"1994-07-10"
17;"Juliet Thornton";"vel@Suspendissecommodo.co.uk";"1985-02-18"
18;"Ramona Combs";"lacus.Aliquam@Morbi.edu";"1982-11-20"
19;"Darius Gray";"Nulla.eu.neque@necmollisvitae.com";"1994-06-02"
20;"Leah Mann";"massa@erosNam.net";"1987-12-22"
```

**PARA CADA UNO DE LOS SIGUIENTES PUNTOS GENERAR UN ARCHIVO SQL CON LOS COMANDOS UTILIZADOS**

Realizar lo siguiente:

1. Crear la base de datos `parcial_mysql`
2. Crear el usuario `parcial-1` del host `127.0.0.1`
3. Asignarle los permisos a `parcial-1` para:
  - administrar todos los objetos de la base de datos `parcial_mysql`
4. Crear la tabla `funcionarios` con los campos:
  - `id` de tipo entero, auto-incrementable, y llave primaria
  - `nombres` de tipo cadena
  - `email` de tipo cadena
  - `fecha` de tipo fecha
5. Importar la lista de `funcionarios` anterior a la tabla de `funcionarios`
6. Adicionar el campo `tareas_cantidad` a la tabla `funcionarios`:
  - tipo de dato entero
  - valor por defecto 0 (cero)
7. Adicionar el campo `reportes_cantidad` a la tabla `funcionarios`:
  - tipo de dato entero
  - valor por defecto 0 (cero)
8. Crear la tabla `tareas` con los campos:
  - `id` de tipo entero, auto-incrementable, y llave primaria
  - `descripción` de tipo cadena
  - `funcionario_id` de tipo entero (corresponde al `id` de `funcionarios`)
  - `reportes_cantidad` de tipo entero y por defecto valor 0 (cero)
  - `creado_el` de tipo fecha y hora (fecha y hora de creación)
9. Crear la tabla `reportes` con los campos:
  - `id` de tipo entero, auto-incrementable, y llave primaria
  - `reporte` de tipo cadena
  - `tarea_id` de tipo entero (corresponde al `id` de `tareas`)
  - `creado_el` de tipo fecha y hora (fecha y hora de creación)
10. Con un procedimiento almacenado generar el reporte de la cantidad de registros para cada tabla: (nombre: `reporte_tablas`)

```bash
+--------------+----------+
| Tabla        | Cantidad |
+--------------+----------+
| funcionarios | 20       |
| tareas       | 0        |
| reportes     | 0        |
+--------------+----------+
```

11. Crear un trigger que permita Actualizar el campo `tareas_cantidad` al INSERTAR nuevas `tareas` para un funcionario
12. Crear un trigger que permita Actualizar el campo `tareas_cantidad` al ELIMINAR una `tareas` para un funcionario
13. Crear un trigger que permita Actualizar el campo `funcionarios.reportes_cantidad` al INSERTAR nuevos `reportes` para un funcionario
13. Crear un trigger que permita Actualizar el campo `funcionarios.reportes_cantidad` al ELIMINAR `reportes` para un funcionario
13. Crear un trigger que permita Actualizar el campo `tareas.reportes_cantidad` al INSERTAR nuevos `reportes` para una tarea
14. Crear un trigger que permita Actualizar el campo `tareas.reportes_cantidad` al ELIMINAR `reportes` para una tarea
15. Crear una función almacenada que permita realizar el conteo de `tareas` para un usuario
16. Crear una función almacenada que permita realizar el conteo de `reportes` para un usuario
17. [opcional] Crear un procedimiento almacenado para generar el siguiente reporte:
  - Nombre funcionario
  - Cantidad de tareas (conteo sobre la tabla `tareas`)
  - `tareas_cantidad` (mostrar el campo de la tabla `funcionarios`)
  - Cantidad de reportes (conteo sobre la tabla `reportes`)
  - `reportes_cantidad` (mostrar el campo de la tabla `funcionarios`)
