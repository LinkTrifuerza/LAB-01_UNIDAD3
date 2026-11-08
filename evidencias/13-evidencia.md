# Evidencia 13 — Intentos de Manipulación de la Auditoría

---

## 1. Intento de Manipulación por Parte de la Aplicación

Comando ejecutado por app_ventas:
psql "postgresql://app_ventas:Ventas_Practica_2026@127.0.0.1/comercial" -c "DELETE FROM auditoria.cambios_clientes" -c "ALTER TABLE clientes DISABLE TRIGGER trg_auditoria_clientes"

Resultados:

* ERROR: permission denied for schema auditoria
* ERROR: must be owner of table clientes

---

## 2. Manipulación Realizada por el Administrador

El administrador ejecutó:
ALTER TABLE clientes DISABLE TRIGGER trg_auditoria_clientes;
UPDATE clientes SET rfc = 'GAMA780912GHI' WHERE id = 3;
ALTER TABLE clientes ENABLE TRIGGER trg_auditoria_clientes;

Resultado en la tabla de auditoría: No se generó ninguna fila adicional.
Resultado en la bitácora del servidor:

* LOG: statement: ALTER TABLE clientes DISABLE TRIGGER trg_auditoria_clientes
* LOG: statement: UPDATE clientes SET rfc = 'GAMA780912GHI' WHERE id = 3
* LOG: statement: ALTER TABLE clientes ENABLE TRIGGER trg_auditoria_clientes

---

## 3. Respuestas a Preguntas de Análisis

1. ¿Qué evidencia permite detectar que hubo un cambio sin auditar?
   La bitácora del servidor. Aunque la tabla de auditoría no registró la fila, la bitácora registró las sentencias DDL de desactivación y activación del disparador.
2. ¿Qué control administrativo reduciría este riesgo?
   Un esquema de custodia externa de bitácoras en tiempo real a donde el administrador no tenga acceso para modificar ni borrar archivos.
3. Relaciona esta actividad con el principio de separación de funciones de la Unidad II.
   Demuestra que quien opera y administra la base de datos no debe ser la misma entidad encargada de custodiar las evidencias y los registros de auditoría.
