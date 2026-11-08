# Evidencia 08 — Errores y Operaciones Rechazadas

---

## 1. Sentencias Rechazadas y Registro

1. DELETE FROM clientes WHERE id = 5 (ejecutado por soporte_ana):

   * Bitácora: ERROR: permission denied for table clientes
   * Bitácora: STATEMENT: DELETE FROM clientes WHERE id = 5
2. DROP ROLE soporte_ana (ejecutado por postgres):

   * Bitácora: ERROR: role "soporte_ana" cannot be dropped because some objects depend on it
   * Bitácora: DETAIL: privileges for table clientes

---

## 2. Respuestas a Preguntas de Análisis

1. ¿El intento de soporte_ana causó algún daño? ¿Por qué debe auditarse de todos modos?
   No causó daño directo porque el motor bloqueó la operación por falta de permisos. Debe auditarse porque representa un intento no autorizado de manipulación de datos o un posible abuso de funciones.
2. ¿Qué diferencia hay entre un error de autorización y uno de autenticación?
   Un error de autenticación ocurre al intentar ingresar al servidor con credenciales inválidas. Un error de autorización ocurre cuando un usuario ya conectado intenta ejecutar una acción para la que no tiene permisos.
3. ¿Qué indica el error de DROP ROLE sobre el orden correcto para dar de baja a un usuario?
   Indica que primero deben revocarse o reasignarse todos los privilegios y objetos dependientes del rol antes de poder eliminarlo del motor.
