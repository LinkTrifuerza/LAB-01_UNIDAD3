# Evidencia 07 — Modificaciones de Datos: DDL frente a MOD

---

## 1. Comparación de Niveles de Registro

Con log_statement = 'ddl':

* La ejecución de UPDATE clientes SET rfc = 'GAMA780912XYZ' WHERE id = 3 NO quedó registrada en la bitácora. Solo aparecieron la conexión y la desconexión.

Con log_statement = 'mod':

* La ejecución de UPDATE clientes SET telefono = '6679990000' WHERE id = 3 SÍ quedó registrada en la bitácora con la línea: LOG: statement: UPDATE clientes SET telefono = '6679990000' WHERE id = 3.
* La sentencia SELECT nombre, rfc FROM clientes WHERE id = 3 ejecutada en el mismo comando NO apareció en la bitácora.

---

## 2. Respuestas a Preguntas de Análisis

1. ¿Qué nivel habría registrado la lectura del RFC? ¿Qué costo tendría en un servidor con miles de consultas por minuto?
   El nivel log_statement = 'all'. En un servidor con alto tráfico, este nivel generaría un volumen masivo de datos en disco, afectando sensiblemente el rendimiento de entrada y salida del servidor.
2. ¿Qué le falta a esta bitácora para revertir el cambio con certeza?
   Le falta almacenar el valor anterior que tenía el campo antes de ser modificado. La bitácora solo muestra la sentencia ejecutada, no los datos previos.
