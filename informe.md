# Informe - LAB-01: Bitácoras y Auditoría del Dato en PostgreSQL

## 1. Despliegue y Situación Inicial

Para esta práctica se utilizó un servidor de PostgreSQL 16 ejecutado dentro de un contenedor Docker en un entorno local.

### Diagnóstico del Motor de Base de Datos

Desde la consola SQL se ejecutaron consultas iniciales para identificar la ubicación de la configuración y los archivos del sistema:

* Versión: PostgreSQL 18
* **Directorio de datos:**`C:\Program Files\PostgreSQL\18\data`
* **Archivo de configuración:**`C:\Program Files\PostgreSQL\18\data\postgresql.conf`
* **Archivo de control de accesos:**`C:\Program Files\PostgreSQL\18\data\pg_hba.conf`

### Estado inicial del servidor

En su instalación por defecto, el servidor no guardaba registros suficientes:

* El colector de bitácoras estaba desactivado, por lo que el motor no generaba archivos de log propios.
* El registro de conexiones y desconexiones estaba apagado.
* El parámetro de sentencias registraba el valor none, omitiendo cualquier consulta SQL.
* El prefijo de línea solo mostraba la fecha y el identificador de proceso, sin incluir usuarios ni direcciones IP.

Por esta razón no fue posible responder a los requerimientos sobre la modificación del RFC de la clienta María García. El servidor no estaba guardando las sentencias de actualización, no registraba el origen de la conexión y tampoco conservaba los valores anteriores del registro.

## 2. Configuración Aplicada

Se aplicaron los siguientes ajustes mediante el comando ALTER SYSTEM para modificar la configuración del servidor:

| Parámetro         | Valor                                  | Tipo de Aplicación | Función                                                                              |
| ------------------ | -------------------------------------- | ------------------- | ------------------------------------------------------------------------------------- |
| logging_collector  | on                                     | Reinicio            | Activa la captura de eventos en archivos de bitácora propios.                        |
| log_connections    | on                                     | Recarga             | Registra los intentos de conexión exitosos y fallidos.                               |
| log_disconnections | on                                     | Recarga             | Registra el cierre de las sesiones y su duración.                                    |
| log_line_prefix    | %m [%p] user=%u db=%d app=%a client=%h | Recarga             | Agrega fecha, PID, usuario, base de datos, aplicación e IP al inicio de cada línea. |
| log_statement      | mod                                    | Recarga             | Registra cambios de estructura y modificaciones de datos.                             |

## 3. Eventos Observados en la Bitácora

Al consultar la vista dba.bitacora se identificaron los siguientes eventos:

1. Acceso fallido por contraseña incorrecta del usuario app_ventas desde la dirección local.
2. Cambio de privilegios mediante la asignación de permisos de actualización al rol app_ventas.
3. Cambio de estructura por la adición de una columna nueva en la tabla clientes.
4. Modificación de datos correspondiente al cambio de teléfono ejecutado por la aplicación de ventas.
5. Error de permisos generado por un intento no autorizado de borrado por parte del usuario soporte_ana.
6. Cambio de configuración registrado por el proceso principal al cambiar el parámetro log_statement a ddl.

## 4. Auditoría del Dato con Tabla y Trigger

La bitácora del servidor muestra la sentencia ejecutada, pero no conserva el valor previo del dato. Para resolver esto se construyó la tabla auditoria.cambios_clientes y una función con disparador.

* La función se definió con SECURITY DEFINER para ejecutarse con los privilegios del propietario y permitir que las aplicaciones registren sus cambios sin tener acceso directo a la tabla de auditoría.
* Se estableció la ruta de búsqueda fija en pg_catalog y public para evitar la manipulación del esquema.
* Se utilizó la función session_user en lugar de current_user para guardar el nombre del usuario autenticado que originó la sesión.

Cuando la aplicación modificó el RFC de María García, la auditoría registró la operación en formato JSONB. La consulta de diferencias confirmó que el usuario app_ventas cambió la columna rfc del valor anterior GAMA780912XYZ al nuevo valor GAMA780912ZZZ.

## 5. Limitaciones del Mecanismo

1. Lecturas de datos: Las consultas de selección no ejecutan disparadores ni quedan registradas con el nivel mod de la bitácora.
2. Volumen de información: Una sola sentencia de actualización masiva genera una línea en la bitácora pero crea una fila individual por cada registro en la tabla de auditoría, lo que incrementa el uso de almacenamiento.
3. Control del administrador: Un usuario con privilegios totales puede desactivar el disparador, realizar modificaciones directas y volver a activarlo sin dejar rastro en la tabla de auditoría.

## 6. Riesgos y Protección de la Bitácora

* Secretos expuestos: Las sentencias de creación de roles con contraseña en texto plano quedan escritas en la bitácora. Por ello, el archivo exportado requiere la redacción previa de cualquier clave sensible.
* Control de integridad: Se generó un resumen SHA-256 del archivo de bitácora exportado para validar que la evidencia no haya sido modificada o alterada posteriormente.

## 7. Cronología de Eventos y Respuestas a Dirección

### Cronología Resumida

| # | Fecha y Hora | Usuario     | IP        | Evento         | Descripción                                    |
| - | ------------ | ----------- | --------- | -------------- | ----------------------------------------------- |
| 1 | 22:49:50     | sistema     | local     | Configuración | Recarga de parámetros de bitácora.            |
| 2 | 22:50:00     | app_ventas  | 127.0.0.1 | Acceso         | Intento fallido por contraseña incorrecta.     |
| 3 | 22:50:00     | app_ventas  | 127.0.0.1 | Acceso         | Conexión exitosa desde app-ventas-web.         |
| 4 | 22:50:09     | postgres    | local     | Privilegio     | Concesión de permiso UPDATE a app_ventas.      |
| 5 | 22:50:09     | postgres    | local     | Estructura     | Adición de columna segmento en tabla clientes. |
| 6 | 22:50:19     | postgres    | local     | Configuración | Cambio de parámetro log_statement a mod.       |
| 7 | 22:50:20     | soporte_ana | 127.0.0.1 | Error          | Intento rechazado de eliminación de registro.  |
| 8 | 22:50:52     | app_ventas  | 127.0.0.1 | Datos          | Actualización de RFC del cliente 3.            |

### Respuestas a Dirección

1. ¿Quién cambió el RFC la primera vez? No hay evidencia, ya que el servidor operaba con la configuración por defecto y el colector apagado.
2. ¿Quién cambió el RFC la segunda vez? El usuario app_ventas a través de la aplicación app-ventas-web desde la dirección 127.0.0.1.
3. ¿Quién cambió el teléfono? El usuario app_ventas, según consta en la bitácora del servidor.
4. ¿Hubo intentos de acceso fallidos? Sí, dos intentos fallidos registrados desde la dirección 127.0.0.1.
5. ¿Qué valor tenía el RFC antes del cambio? El valor original registrado en la auditoría era GAMA780912XYZ.

## 8. Preguntas de Reflexión

1. ¿Por qué la auditoría no puede configurarse después de un incidente para investigarlo?
   La auditoría actúa durante y después de la operación, pero no de manera retroactiva. Los eventos y valores que no fueron capturados al momento de ocurrir no se pueden reconstruir.
2. ¿Por qué conviene que la bitácora la administre el propio servidor y no el mecanismo de despliegue?
   Permite que el motor gestione y rote sus archivos de forma estandarizada e independiente de la plataforma, facilitando su consulta directa mediante sentencias SQL.
3. ¿Por qué el prefijo de línea es tan importante como el nivel de log_statement?
   Porque log_statement registra la sentencia ejecutada, mientras que el prefijo agrega la identidad del usuario, la aplicación, la base de datos, la IP y la marca de tiempo exacta.
4. ¿Qué diferencia práctica hay entre registrar la sentencia y registrar el cambio en el dato?
   Registrar la sentencia guarda el texto de la consulta en la bitácora del servidor, mientras que registrar el cambio guarda el valor anterior y el valor nuevo del registro en la tabla de auditoría.
5. ¿Qué evidencia ofrece pg_stat_activity que no ofrece la bitácora, y viceversa?
   pg_stat_activity muestra el estado actual y las sesiones activas en el instante presente, mientras que la bitácora conserva el historial cronológico de eventos ya concluidos.
6. Si el administrador puede cambiar log_statement a none y desactivar el disparador, ¿quién audita al administrador?
   Dentro del motor no hay un control que restrinja al superusuario. Para auditar al administrador se requiere enviar la bitácora en tiempo real a un almacenamiento externo custodiado por un equipo independiente.
