# Evidencia 06 — Cambios de Privilegios y Estructura

---

## 1. Sentencias Ejecutadas

REVOKE UPDATE ON clientes FROM soporte_ana;
GRANT UPDATE ON clientes TO app_ventas;
ALTER TABLE clientes ADD COLUMN segmento VARCHAR(20);
CREATE ROLE rol_temporal LOGIN PASSWORD 'Temporal_Practica_2026';
DROP ROLE rol_temporal;

---

## 2. Fragmentos Registrados en la Bitácora

* LOG: statement: REVOKE UPDATE ON clientes FROM soporte_ana
* LOG: statement: GRANT UPDATE ON clientes TO app_ventas
* LOG: statement: ALTER TABLE clientes ADD COLUMN segmento VARCHAR(20)
* LOG: statement: CREATE ROLE rol_temporal LOGIN PASSWORD 'Temporal_Practica_2026'
* LOG: statement: DROP ROLE rol_temporal

---

## 3. Respuestas a Preguntas de Análisis

1. ¿Qué tipo de evento es cada una de las tres primeras sentencias (privilegio o estructura)?
   REVOKE y GRANT son eventos de tipo Privilegio. ALTER TABLE es un evento de tipo Estructura.
2. ¿Por qué GRANT y REVOKE quedan registrados con log_statement = 'ddl'?
   Porque en PostgreSQL el nivel ddl incluye tanto las sentencias de definición de lenguaje (CREATE, ALTER, DROP) como las sentencias de control de derechos y permisos (GRANT, REVOKE).
3. Si la bitácora contiene contraseñas, ¿quién debería poder leerla? Relaciónalo con la gestión de secretos de la Unidad II.
   Solo personal autorizado de administración de bases de datos y seguridad. Escribir contraseñas directamente en sentencias SQL expone los secretos en texto plano en la bitácora; por ello se debe usar el metacomando \password o hashes previamente calculados.
