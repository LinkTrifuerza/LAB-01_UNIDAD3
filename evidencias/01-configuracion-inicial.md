## 1. Comando Ejecutado

SELECT name, setting, source
FROM pg_settings
WHERE name IN ('log_destination', 'logging_collector', 'log_directory', 'log_filename',
               'log_connections', 'log_disconnections', 'log_line_prefix',
               'log_statement', 'log_min_error_statement', 'log_timezone')
ORDER BY name;

---

## 2. Resultado Obtenido

| Parámetro              | Valor                          | Origen              |
| ----------------------- | ------------------------------ | ------------------- |
| log_connections         | off                            | default             |
| log_destination         | stderr                         | configuration file  |
| log_directory           | log                            | default             |
| log_disconnections      | off                            | default             |
| log_filename            | postgresql-%Y-%m-%d_%H%M%S.log | default             |
| log_line_prefix         | %t                             | configuration file  |
| log_min_error_statement | error                          | default             |
| log_statement           | none                           | default             |
| log_timezone            | America/Chihuahua              | configuration file  |
| logging_collector       | on                             | configuration file  |

---

## 3. Respuestas a Preguntas de Análisis

1. Con esta configuración, ¿se registra quién se conecta?
   No. El parámetro log_connections está desactivado (off), por lo que las conexiones exitosas o fallidas no quedan registradas.
2. ¿Se registran las sentencias UPDATE?
   No. El parámetro log_statement tiene el valor none, omitiendo el registro de cualquier sentencia SQL.
3. ¿La línea de bitácora incluye el usuario o la dirección IP del cliente?
   No. El prefijo log_line_prefix solo incluye la marca de tiempo (%m) y el identificador de proceso (%p).
4. ¿En qué zona horaria se registran los eventos? ¿Por qué importa al reconstruir una cronología?
   Se registran en la zona horaria UTC (Etc/UTC). Es importante porque al correlacionar eventos con otros sistemas se deben unificar los horarios para no tergiversar el orden de los sucesos.
5. Con logging_collector = off, ¿quién decide dónde termina la bitácora: PostgreSQL o el mecanismo con que se desplegó el servidor?
   El mecanismo con el que se desplegó el servidor (como Docker o el gestor de servicios del sistema operativo), ya que PostgreSQL solo envía sus mensajes a la salida de error estándar (stderr).
