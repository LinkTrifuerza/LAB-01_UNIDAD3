# Evidencia 05 — Accesos Exitosos y Fallidos

---

## 1. Eventos de Conexión Ejecutados

1. Intento con contraseña incorrecta (app_ventas).
2. Intento con usuario inexistente (usuario_inexistente).
3. Conexión exitosa y consulta de conteo (app_ventas).

---

## 2. Registros Obtenidos en la Bitácora

* Intento 1: user=app_ventas db=comercial client=127.0.0.1 FATAL: password authentication failed for user "app_ventas"
* Intento 2: user=usuario_inexistente db=comercial client=127.0.0.1 FATAL: password authentication failed for user "usuario_inexistente" (DETAIL: Role "usuario_inexistente" does not exist)
* Intento 3: user=app_ventas db=comercial app=app-ventas-web client=127.0.0.1 LOG: connection authorized... seguido por LOG: disconnection...

---

## 3. Respuestas a Preguntas de Análisis

1. La bitácora registra el usuario app_ventas en el intento fallido. ¿Eso prueba que la aplicación de ventas intentó entrar? ¿Qué prueba realmente?
   No prueba que haya sido la aplicación legítima. Solo prueba que alguien desde la IP 127.0.0.1 intentó autenticarse declarando llamarse app_ventas.
2. ¿Qué información adicional registra PostgreSQL cuando el rol no existe? ¿Por qué aparece en la bitácora y no en el mensaje que recibe el cliente?
   Registra en el campo DETAIL que el rol no existe en el sistema. Aparece solo en la bitácora por seguridad, para evitar darle información a un atacante en la red sobre cuáles usuarios existen verdaderamente.
3. ¿Por qué app=[unknown] en el intento fallido, pero app=app-ventas-web en la desconexión?
   Porque durante la fase de autenticación la conexión aún no ha validado los parámetros de la sesión (como application_name). Una vez autorizada la sesión, se asigna el nombre de la aplicación.
4. ¿Qué patrón de varias líneas te haría sospechar de un ataque de fuerza bruta? Escribe la consulta que contaría los intentos fallidos por usuario.
   Un alto número de líneas con la etiqueta FATAL y la severidad de falla de contraseña desde una misma IP en pocos segundos.

Consulta para contar intentos fallidos:
SELECT linea FROM dba.bitacora WHERE linea LIKE '%FATAL:  password authentication failed%' ORDER BY num;
