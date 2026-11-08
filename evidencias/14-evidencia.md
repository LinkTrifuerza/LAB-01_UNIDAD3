# Evidencia 14 — Límites: Lecturas y Volumen

---

## 1. Resultados de las Pruebas

1. Consulta SELECT sobre rfc ejecutada por soporte_ana:

   * Bitácora con log_statement = 'mod': 0 registros.
   * Tabla de auditoría: 0 registros.
2. Sentencia UPDATE clientes SET segmento = 'regular':

   * Bitácora del servidor: 1 línea registrada.
   * Tabla de auditoría: 5 filas agregadas (una por cada registro modificado).

---

## 2. Respuestas a Preguntas de Análisis

1. Si clientes tuviera un millón de registros, ¿qué ocurriría con la auditoría tras un UPDATE sin WHERE?
   La tabla de auditoría intentaría insertar un millón de filas en una sola transacción, lo que podría saturar el espacio en disco y ralentizar el servidor.
2. Propón quién podría autorizar la depuración de auditoria.cambios_clientes.
   Únicamente el Oficial de Seguridad de la Información o el equipo de auditoría de la empresa.
3. ¿Por qué log_statement = 'all' puede convertir la bitácora en un nuevo depósito de datos sensibles?
   Porque registraría todas las consultas SELECT, incluyendo los datos filtrados y leídos por los usuarios en texto claro dentro de los logs.
