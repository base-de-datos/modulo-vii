# Parcial 2: PostgreSQL

Implementar una base de datos para auditoría de las transacciones de un banco:

1. Crear el usuario `egpp`
2. Crear la base de datos `banco_egpp` con dueño el usuario `egpp`
3. Crear la tabla `cuentas`
  - `id` de tipo entero, autoincrementable y llave primaria
  - `nombres` de tipo cadena
  - `apellidos` de tipo cadena
  - `saldo` de tipo entero
4. Crear la tabla `movimientos`
  - `id` de tipo entero, autoincrementable y llave primaria
  - `monto` de tipo entero, que pueden ser montos de ingreso o salida
  - `cuenta_id` de tipo entero, hace referencia a la tabla `cuentas`
5. Crear la tabla `auditoria`
  - `id` de tipo entero, autoincrementable y llave primaria
  - `nombre_completo` de tipo cadena, que es `nombres` y `apellidos` del cliente
  - `monto` de tipo entero
  - `saldo` de tipo entero  
  - `usuario_bd` de tipo cadena, usuario actual de la base de datos
  - `movimiento_id` de tipo entero, hace referencia a la tabla `movimientos`

Es necesario que la base de datos responda a lo siguiente:

1. Cuando se crear un registro en la tabla `cuentas`, debe crear en `movimientos` un registro, de tal manera que tenga un `monto` cero (0) para el cliente.
2. Con cada inserción en la tabla de `movimientos`, se debe actualizar el `saldo` de la cuenta correspondiente.
3. Con cada inserción en la tabla de `movimientos`, se debe registrar lo que se está realizando con los campos requeridos en la tabla de `auditoria`.

Implementar el/los procedimiento(s) almacenado(s) o el/los trigger(s) que solucionen los enunciados anteriores.
