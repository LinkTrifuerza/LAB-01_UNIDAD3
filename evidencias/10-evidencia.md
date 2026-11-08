# Evidencia 10 — Diseño de la Tabla de Auditoría

---

## 1. Matriz de Diseño de Campos

| Campo            | Tipo        | Pregunta que responde      | Permite Nulos | Justificación                                    |
| ---------------- | ----------- | -------------------------- | ------------- | ------------------------------------------------- |
| id_evento        | BIGSERIAL   | Identificador único       | NO            | Clave primaria autoincremental.                   |
| momento          | TIMESTAMPTZ | ¿Cuándo?                 | NO            | Guarda fecha y hora exacta con zona horaria.      |
| usuario_sesion   | TEXT        | ¿Quién lo hizo?          | NO            | Rol real autenticado (session_user).              |
| usuario_efectivo | TEXT        | Contexto de ejecución     | NO            | Rol con cuyos permisos se ejecuta (current_user). |
| aplicacion       | TEXT        | ¿Desde qué app?          | SÍ           | Captura application_name de la sesión.           |
| ip_cliente       | INET        | ¿Desde qué equipo?       | SÍ           | IP de la conexión (nulo en sockets locales).     |
| operacion        | TEXT        | ¿Qué acción?            | NO            | Tipo de operación (INSERT, UPDATE, DELETE).      |
| id_cliente       | BIGINT      | ¿Qué registro?           | SÍ           | Identificador de la fila modificada.              |
| valores_antes    | JSONB       | ¿Qué valor tenía antes? | SÍ           | Estado previo en formato JSON. Nulo en INSERT.    |
| valores_despues  | JSONB       | ¿Qué valor tiene ahora?  | SÍ           | Estado nuevo en formato JSON. Nulo en DELETE.     |

---

## 2. Respuestas a Preguntas de Análisis

1. ¿Por qué momento usa TIMESTAMPTZ y no TIMESTAMP?
   Porque TIMESTAMPTZ almacena el momento preciso en formato UTC sin ambigüedad, permitiendo convertir la fecha a cualquier zona horaria local de análisis.
2. ¿Por qué se guarda el registro completo en JSONB y no solo la columna que cambió?
   Guardar la fila completa en JSONB ofrece flexibilidad ante cambios futuros en la estructura de la tabla sin necesidad de alterar la tabla de auditoría, y conserva el contexto completo del registro.
3. La tabla de auditoría también contendrá RFC. ¿Qué implica eso para su clasificación y su control de acceso?
   Implica que la tabla de auditoría se convierte en un depósito de datos sensibles. Debe ubicarse en un esquema restringido y sin acceso de lectura para los roles de la aplicación.
