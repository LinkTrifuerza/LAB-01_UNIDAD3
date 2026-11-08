# Evidencia 09 — Sesiones Activas con pg_stat_activity

---

## 1. Consulta Ejecutada

SELECT pid, usename, application_name, client_addr, backend_start, state, left(query, 30) AS query
FROM pg_stat_activity
WHERE backend_type = 'client backend';

---

## 2. Resultado Obtenido



|  pid  | usename  | application_name             | client_addr | backend_start           | state | query                                    |
| :---: | :------- | :--------------------------- | :---------: | :---------------------- | :----: | :--------------------------------------- |
| 2224 | postgres | pgAdmin 4 - CONN:6499244     |     ::1     | 2026-11-08 09:59:13.890 | active | SELECT pid, usename, application_name... |
| 15444 | postgres | pgAdmin 4 - DB:lab01_db      |     ::1     | 2026-11-08 09:59:06.500 |  idle  | SELECT COUNT(*) FROM...                  |
| 19432 | postgres | pgAdmin 4 - DB:practicas_gus |     ::1     | 2026-11-08 09:58:52.059 |  idle  | SELECT nsp.nspname as sche...            |
| 3892 | postgres | pgAdmin 4 - DB:practica2     |     ::1     | 2026-11-08 09:58:52.052 |  idle  | SELECT DISTINCT att.attname as...        |
| 5344 | postgres | pgAdmin 4 - CONN:3262065     |     ::1     | 2026-11-08 09:58:49.868 |  idle  | SELECT...                                |
| 16324 | postgres | pgAdmin 4 - DB:postgres      |     ::1     | 2026-11-08 09:58:49.678 |  idle  | SELECT db.oid as did, db...              |

---

## 3. Respuestas a Preguntas de Análisis

1. ¿Qué columna permite relacionar una sesión con sus líneas de bitácora?
   La columna pid (Process ID), que aparece entre corchetes [%p] en las líneas de la bitácora.
2. Después de que termina el reporte, ¿qué fuente conserva evidencia de que existió?
   La bitácora del servidor. pg_stat_activity elimina la fila en cuanto la sesión o proceso finaliza.
3. ¿Por qué client_addr aparece vacío en la sesión del administrador?
   Porque la sesión del administrador se abrió mediante una conexión por socket Unix local, no a través de una pila de red TCP/IP.
