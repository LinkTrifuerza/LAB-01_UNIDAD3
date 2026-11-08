# LAB-01 — Bitácoras y auditoría del dato en PostgreSQL

---

## 1. Descripción del Despliegue del Servidor

Para esta práctica el servidor se desplegó en un contenedor Docker utilizando la imagen oficial de PostgreSQL 16 sobre el sistema operativo Debian.

- Motor de Base de Datos: PostgreSQL 18
- Tipo de Despliegue: Servicio Local / Instalación Nativa (Windows)
- Servidor / Host: 127.0.0.1 (localhost) en el puerto 5432
- Base de Datos: comercial
- Autenticación en pg_hba.conf: Configurado con el método scram-sha-256 para las conexiones locales y de red TCP.

## 2. Estructura del Repositorio de Entregables

La carpeta principal del laboratorio contiene la siguiente organización de archivos:

LAB-01/
├── README.md
├── sql/
│   ├── 01-colector-y-vista.sql
│   ├── 02-configuracion-bitacora.sql
│   ├── 03-privilegios-estructura.sql
│   ├── 04-esquema-auditoria.sql
│   ├── 05-funcion-disparador.sql
│   └── 06-campos-modificados.sql
└── evidencias/
    ├── 01-configuracion-inicial.md
    ├── 02-colector.md
    ├── 03-incidente-sin-bitacora.md
    ├── 04-configuracion-bitacora.md
    ├── 05-accesos.md
    ├── 06-privilegios-estructura.md
    ├── 07-modificaciones.md
    ├── 08-errores.md
    ├── 09-pg-stat-activity.md
    ├── 10-diseno-auditoria.md
    ├── 11-incidente-auditado.md
    ├── 12-altas-bajas.md
    ├── 13-manipulacion.md
    ├── 14-limites.md
    ├── 15-cronologia.md
    ├── 16-pgaudit.md
    ├── bitacora-lab01.log
    ├── bitacora-lab01.sha256
    └── informe.md

---

## 3. Pasos para Reproducir el Entorno

### Inicializar el servidor PostgreSQL

Se inicia el contenedor Docker con el puerto expuesto y la base de datos comercial creada:

docker run --name dba-u3-pg -e POSTGRES_PASSWORD=Practica_U3_2026 -e POSTGRES_DB=comercial -p 5432:5432 -d postgres:16

### Ajustar las reglas de autenticación

Se modifican las reglas en pg_hba.conf para reemplazar trust por scram-sha-256 y forzar el uso de contraseña:

docker exec dba-u3-pg bash -c "sed -i -E 's/^(host +all +all +(127.0.0.1\/32|::1\/128) +)trust/\1scram-sha-256/' /var/lib/postgresql/data/pg_hba.conf"

En la sesión SQL del servidor se recarga la configuración:

SELECT pg_reload_conf();

### Orden de ejecución de scripts SQL

1. 01-colector-y-vista.sql: Activa el colector de bitácora y crea la vista dba.bitacora. Requiere un reinicio del contenedor con docker restart dba-u3-pg.
2. 02-configuracion-bitacora.sql: Ajusta los parámetros de registro de conexiones, desconexiones, prefijo de línea y sentencias DDL.
3. 03-privilegios-estructura.sql: Asigna permisos a los roles, agrega columnas a la tabla clientes y demuestra la captura de sentencias.
4. 04-esquema-auditoria.sql: Crea el esquema auditoria y la tabla cambios_clientes.
5. 05-funcion-disparador.sql: Crea la función con SECURITY DEFINER y el disparador trg_auditoria_clientes.
6. 06-campos-modificados.sql: Ejecuta la consulta JSONB para obtener las diferencias de valores antes y después de cada cambio.

---

## 4. Exportación Segura e Integridad de la Bitácora

### Exportación y limpieza de datos sensibles

Se extrae el archivo de bitácora reemplazando cualquier contraseña visible en texto claro por una etiqueta de edición:

psql -U postgres -d comercial -At -c "SELECT replace(pg_read_file(pg_current_logfile()), 'Temporal_Practica_2026', '<REDACTED></redacted>')" > evidencias/bitacora-lab01.log

### Generar el hash de verificación

Se calcula el resumen criptográfico SHA-256 para respaldar la integridad del archivo exportado:

sha256sum evidencias/bitacora-lab01.log > evidencias/bitacora-lab01.sha256
