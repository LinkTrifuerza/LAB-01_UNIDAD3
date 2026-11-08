# Evidencia 11 — El Incidente Auditado

---

## 1. Prueba de Acceso Negado a la Aplicación

Comando ejecutado por app_ventas:
psql "postgresql://app_ventas:Ventas_Practica_2026@127.0.0.1/comercial" -c "SELECT * FROM auditoria.cambios_clientes"

Resultado:
ERROR: permission denied for schema auditoria

---

## 2. Consulta de Diferencias Realizada por el Administrador

Comando ejecutado por postgres:
SELECT a.id_evento, a.momento, a.usuario_sesion, a.ip_cliente,
       d.key AS campo,
       a.valores_antes ->> d.key AS antes,
       d.value #>> '{}' AS despues
FROM auditoria.cambios_clientes a
CROSS JOIN LATERAL jsonb_each(a.valores_despues) d
WHERE a.operacion = 'UPDATE'
  AND a.valores_antes -> d.key IS DISTINCT FROM d.value
ORDER BY a.id_evento;

Resultado obtenido:


| id_evento | momento                 | usuario_sesion | ip_cliente | campo    | antes           | despues               |
| :-------: | :---------------------- | :------------- | :--------: | :------- | :-------------- | :-------------------- |
|     1     | 2026-11-08 10:37:37.495 | postgres       |  ::1/128  | correo   | carlos@mail.com | carlos.nuevo@mail.com |
|     1     | 2026-11-08 10:37:37.495 | postgres       |  ::1/128  | telefono | 6671112233      | 6679990011            |

---

## 3. Sentencia de Restauración del Dato

UPDATE clientes SET rfc = 'GAMA780912XYZ' WHERE id = 3;

---

## 4. Respuestas a Preguntas de Análisis

1. Compara tu hipótesis de la Actividad 11 con este resultado.
   Si la función no hubiese sido SECURITY DEFINER, el intento de UPDATE de app_ventas habría fallado por no tener permisos de inserción sobre auditoria.cambios_clientes.
2. ¿Qué columna conserva más valor para una investigación: aplicacion o ip_cliente? ¿Cuál puede falsificar el cliente con más facilidad?
   La columna ip_cliente conserva más valor técnico. application_name puede ser modificado libremente por el cliente en la cadena de conexión.
