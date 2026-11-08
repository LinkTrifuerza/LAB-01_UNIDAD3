# Evidencia 04 — Configuración de la Bitácora del Servidor

---

## 1. Comandos Ejecutados

ALTER SYSTEM SET log_connections = on;
ALTER SYSTEM SET log_disconnections = on;
ALTER SYSTEM SET log_line_prefix = '%m [%p] user=%u db=%d app=%a client=%h ';
ALTER SYSTEM SET log_statement = 'ddl';
SELECT pg_reload_conf();

---

## 2. Verificación de Parámetros

SELECT name, setting, source, sourcefile
FROM pg_settings
WHERE name IN ('log_connections', 'log_line_prefix', 'log_statement');


| name            | setting | source             | sourcefile                                          |
| :-------------- | :------ | :----------------- | :-------------------------------------------------- |
| log_connections | off     | default            | -                                                   |
| log_line_prefix | %t      | configuration file | C:/Program Files/PostgreSQL/18/data/postgresql.conf |
| log_statement   | none    | default            | -                                                   |

---

## 3. Respuestas a Preguntas de Análisis

1. ¿Por qué el proceso que registra el cambio es el principal y no tu sesión?
   Porque cuando se envía la señal de recarga (pg_reload_conf), es el proceso padre o principal de PostgreSQL (PID 1) el que reevalúa los archivos de configuración y aplica los ajustes de forma global.
2. ¿Qué pasaría si alguien ejecutara ALTER SYSTEM SET log_statement = 'none'? ¿Quedaría rastro?
   Si se ejecutara esa sentencia con log_statement = 'ddl', el cambio de parámetro quedaría registrado en la bitácora por el proceso principal en el momento de recargar, aunque las sentencias posteriores dejen de registrarse.
3. ¿Por qué no se recomienda editar postgresql.auto.conf a mano?
   Porque este archivo es administrado directamente por el motor mediante el comando ALTER SYSTEM. Ediciones manuales pueden ser sobrescritas o generar inconsistencias en el arranque.
