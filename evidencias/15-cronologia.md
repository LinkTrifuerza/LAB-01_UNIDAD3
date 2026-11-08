# Evidencia 15 — Reconstrucción Cronológica

---

## 1. Exportación e Integridad de la Bitácora

* El archivo bitacora-lab01.log fue exportado mediante el comando psql reemplazando la clave en texto plano por la etiqueta `<REDACTED>`.
* Se generó el archivo de firma criptográfica bitacora-lab01.sha256 utilizando el algoritmo SHA-256.

---

## 2. Matriz Cronológica Integrada

| # | Fecha y Hora | Usuario     | IP / App       | Tipo Evento    | Descripción de lo Ocurrido                    |
| - | ------------ | ----------- | -------------- | -------------- | ---------------------------------------------- |
| 1 | 22:49:50     | postgres    | local          | Configuración | Recarga del parámetro log_statement a ddl.    |
| 2 | 22:50:00     | app_ventas  | 127.0.0.1      | Acceso         | Intento fallido por contraseña incorrecta.    |
| 3 | 22:50:00     | app_ventas  | app-ventas-web | Acceso         | Conexión exitosa desde la aplicación.        |
| 4 | 22:50:09     | postgres    | local          | Privilegio     | Asignación de permiso UPDATE a app_ventas.    |
| 5 | 22:50:09     | postgres    | local          | Estructura     | Adición de la columna segmento en clientes.   |
| 6 | 22:50:19     | postgres    | local          | Configuración | Cambio de parámetro log_statement a mod.      |
| 7 | 22:50:20     | soporte_ana | 127.0.0.1      | Error          | Intento rechazar de borrado en tabla clientes. |
| 8 | 22:50:52     | app_ventas  | app-ventas-web | Datos          | Modificación de RFC del cliente id=3.         |

---

## 3. Respuestas a las Preguntas de Dirección

1. ¿Quién cambió el RFC la primera vez (Actividad 3)? ¿Y en las Actividades 7, 12 y 14?

   * En la Actividad 3: No hay evidencia. El colector estaba apagado.
   * En la Actividad 7, 12 y 14: Fue realizado por el usuario app_ventas desde 127.0.0.1.
2. ¿Quién cambió el teléfono?
   El usuario app_ventas desde la aplicación app-ventas-web.
3. ¿Hubo intentos de acceso fallidos? ¿De qué usuarios y desde dónde?
   Sí. Se registraron dos intentos fallidos desde 127.0.0.1 (uno de app_ventas y otro de usuario_inexistente).
4. ¿Qué valor tenía el RFC antes de cada cambio?
   Antes de la modificación en la Actividad 12, el valor original era GAMA780912XYZ.
