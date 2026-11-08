# Evidencia 03 — Incidente sin Bitácora

---

## 1. Comando Ejecutado por Red

psql "postgresql://app_ventas:Ventas_Practica_2026@127.0.0.1/comercial?application_name=app-ventas-web" -c "UPDATE clientes SET rfc = 'GAMA780912XXX' WHERE id = 3"

## 2. Búsqueda de Evidencia en la Bitácora

SELECT linea FROM dba.bitacora
WHERE linea ILIKE '%update%' OR linea LIKE '%app_ventas%';

Resultado: 0 filas devueltas.

---

## 3. Respuestas a Preguntas de Análisis

1. ¿Puedes demostrar quién modificó el RFC?
   No. Al no estar activado el registro de sentencias ni el de conexiones, el servidor no conservó ningún rastro de la operación.
2. ¿Puedes demostrar que el cambio no lo hizo soporte_ana?
   Tampoco. La falta de registros impide descartar a cualquier usuario que tuviese algún nivel de acceso.
3. Los privilegios de la Unidad II, ¿impidieron el cambio? ¿Debían impedirlo?
   No lo impidieron porque app_ventas tenía asignado el permiso legítimo de UPDATE sobre la tabla clientes. El control de acceso decide quién puede actuar, pero no registra lo que la persona hace.
4. ¿Qué tendría que haber estado configurado antes del incidente?
   El colector de bitácora activo, el registro de conexiones encendido, un prefijo de línea que incluyera el usuario e IP, y un nivel de registro de sentencias o una tabla de auditoría con disparador.
