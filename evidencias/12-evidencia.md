# Evidencia 12 — Altas y Bajas de Registros

---

## 1. Operaciones Realizadas

INSERT INTO clientes (nombre, correo, rfc, edad, ciudad, codigo_postal)
VALUES ('Pedro Ruiz', 'pedro.ruiz@example.test', 'RUPE950101AAA', 30, 'Guasave', '81000');

DELETE FROM clientes WHERE nombre = 'Pedro Ruiz';

---

## 2. Registros Obtenidos en la Tabla de Auditoría



| **id_evento** | **momento**              | **usuario_sesion** | **operacion** | **valores_antes**                                                                                                                                       | **valores_despues**                                                                                                                                     |
| ------------------- | ------------------------------ | ------------------------ | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **2**         | `2026-11-08 11:09:54.960903` | `postgres`             | **INSERT**    | *NULL*                                                                                                                                                      | `{"id": 6, "rfc": "RUPE950101AAA", "edad": 30, "ciudad": "Guasave", "correo": "pedro.ruiz@example.test", "nombre": "Pedro Ruiz", "codigo_postal": "81000"}` |
| **3**         | `2026-11-08 11:09:54.961115` | `postgres`             | **DELETE**    | `{"id": 6, "rfc": "RUPE950101AAA", "edad": 30, "ciudad": "Guasave", "correo": "pedro.ruiz@example.test", "nombre": "Pedro Ruiz", "codigo_postal": "81000"}` | *NULL*                                                                                                                                                      |

---

## 3. Respuestas a Preguntas de Análisis

1. El cliente Pedro Ruiz ya no existe en clientes. ¿Dónde queda evidencia de que existió y de sus datos?
   En la tabla auditoria.cambios_clientes, dentro de los registros de tipo INSERT y DELETE guardados en formato JSONB.
2. ¿Por qué ip_cliente está vacía en los eventos 2 y 3?
   Porque las sentencias fueron ejecutadas por el administrador a través de una sesión de socket local sin dirección IP asociada.
3. La política de retención de la Unidad II pedía eliminar datos cuando ya no fueran necesarios. ¿Qué conflicto aparece? ¿Quién debe resolverlo?
   Se genera un conflicto entre la eliminación de datos personales por privacidad y la obligación legal de conservar evidencias en la auditoría. Este dilema debe ser resuelto por el área legal y de gobierno de datos.
