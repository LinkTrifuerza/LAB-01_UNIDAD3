# Evidencia 02 — Activación del Colector de Bitácora

---

## 1. Comandos Ejecutados

ALTER SYSTEM SET logging_collector = on;

-- Tras reiniciar el contenedor/servicio:
SELECT name, setting, source FROM pg_settings WHERE name = 'logging_collector';
SELECT pg_current_logfile();

-- Creación de la vista de lectura:
CREATE SCHEMA dba;
REVOKE ALL ON SCHEMA dba FROM PUBLIC;

CREATE VIEW dba.bitacora AS
SELECT t.n AS num, t.linea
FROM regexp_split_to_table(pg_read_file(pg_current_logfile()), E'\n')
     WITH ORDINALITY AS t(linea, n)
WHERE t.linea <> '';

---

## 2. Resultado Obtenido

* logging_collector: on (origen: configuration file)
* pg_current_logfile: log/postgresql-2026-10-25_224920.log

Primeras líneas capturadas por la vista dba.bitacora:


| num | linea                                                                                                  |
| :-: | :----------------------------------------------------------------------------------------------------- |
|  1  | 2026-11-08 09:20:14 CST LOG: el sistema de bases de datos fue apagado en 2026-11-08 00:11:30 CST       |
|  2  | 2026-11-08 09:25:14 CST LOG: empezando checkpoint: time                                                |
|  3  | 2026-11-08 09:25:14 CST LOG: checkpoint completo: escritos 0 búfers (0.0%); ...                       |
|  4  | 2026-11-08 09:58:47 CST FATAL: la autentificación password falló para el usuario «postgres»        |
|  5  | 2026-11-08 09:58:47 CST DETALLE: La conexión coincidió con el archivo «.../pg_hba.conf» línea 117 |
|  6  | 2026-11-08 09:58:47 CST FATAL: la autentificación password falló para el usuario «postgres»        |
|  7  | 2026-11-08 09:58:47 CST DETALLE: La conexión coincidió con el archivo «.../pg_hba.conf» línea 115 |
|  8  | 2026-11-08 10:00:14 CST LOG: empezando checkpoint: time                                                |
|  9  | 2026-11-08 10:01:55 CST LOG: checkpoint completo: escritos 1004 búfers (6.1%); ...                    |
| 10 | 2026-11-08 10:19:57 CST ERROR: ALTER SYSTEM cannot run inside a transaction block                      |

---

## 3. Respuestas a Preguntas de Análisis

1. ¿Por qué logging_collector requiere reinicio y log_statement solo recarga?
   Porque el colector inicia un proceso secundario a nivel de infraestructura del servidor para capturar archivos de disco. log_statement solo cambia la forma en que las sesiones existentes interpretan las consultas.
2. ¿Qué ventaja tiene que la bitácora se lea con SQL y no con una herramienta del sistema operativo o del contenedor? ¿Qué riesgo introduce?
   La ventaja es que permite consultar e inspeccionar los logs directamente con sentencias SQL sin depender de accesos a la terminal del servidor. El riesgo es que si se otorga acceso a la función de lectura a usuarios no autorizados, estos podrían leer archivos del sistema operativo.
3. ¿Quién debería poder consultar dba.bitacora?
   Únicamente el rol de administrador del sistema (superusuario o el equipo de auditoría/DBA).
