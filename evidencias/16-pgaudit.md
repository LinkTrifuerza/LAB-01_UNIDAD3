# Evidencia 16 — Auditoría de Objetos sobre Columnas Sensibles con pgaudit

---

## 1. Comandos de Instalación y Configuración

SELECT name, default_version FROM pg_available_extensions WHERE name = 'pgaudit';

ALTER SYSTEM SET shared_preload_libraries = 'pgaudit';
-- Tras reiniciar el servidor/contenedor:

CREATE EXTENSION pgaudit;

CREATE ROLE auditor_datos NOLOGIN;
GRANT SELECT (rfc, telefono), UPDATE (rfc, telefono) ON clientes TO auditor_datos;

ALTER SYSTEM SET pgaudit.log = 'role, ddl';
ALTER SYSTEM SET pgaudit.role = 'auditor_datos';
ALTER SYSTEM SET pgaudit.log_relation = on;
SELECT pg_reload_conf();

---

## 2. Consultas Ejecutadas y Registros Generados

Consultas ejecutadas como app_ventas:

1. SELECT nombre, ciudad FROM clientes WHERE id = 2
2. SELECT nombre, rfc FROM clientes WHERE id = 3
3. UPDATE clientes SET rfc = 'LOAA850315XXX' WHERE id = 1
4. UPDATE clientes SET ciudad = 'Navolato' WHERE id = 2

Líneas capturadas en la bitácora:

* Para la consulta 1: No generó registro de auditoría.
* Para la consulta 2: LOG: AUDIT: OBJECT, 1, 1, READ, SELECT, TABLE, public.clientes, "SELECT nombre, rfc FROM clientes WHERE id = 3", <not logged></not>
* Para la consulta 3: LOG: AUDIT: OBJECT, 3, 1, WRITE, UPDATE, TABLE, public.clientes, UPDATE clientes SET rfc = 'LOAA850315XXX' WHERE id = 1, <not logged></not>
* Para la consulta 4: No generó registro de auditoría.

---

## 3. Respuestas a Preguntas de Análisis

1. ¿Por qué no apareció SELECT nombre, ciudad? ¿Por qué sí apareció SELECT * ...?
   No apareció SELECT nombre, ciudad porque el rol auditor_datos no tenía concedidos privilegios de auditoría sobre esas columnas. SELECT * sí aparece porque al incluir todas las columnas de la tabla termina leyendo rfc y telefono.
2. ¿Por qué no apareció UPDATE ciudad?
   Porque el rol auditor_datos únicamente audita modificaciones sobre las columnas rfc y telefono. El cambio en la columna ciudad queda fuera de sus privilegios.
3. ¿Qué diferencia de volumen esperarías frente a log_statement = 'all'?
   El volumen es significativamente menor. Mientras que log_statement = 'all' registra absolutamente todas las consultas de todas las tablas, pgaudit en modo objeto solo registra lecturas y escrituras en las columnas sensibles seleccionadas.
4. El README oficial advierte que pgaudit no puede auditar de forma confiable a los superusuarios. ¿Qué implica para la separación de funciones?
   Implica que las acciones de los administradores con privilegios totales no se pueden auditar de manera infalible únicamente con esta extensión, reforzando la necesidad de contar con custodia de logs en un servidor externo.
