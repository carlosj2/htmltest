# Herramienta Profesional de Validación CORS (GitHub Pages)

Esta página permite ejecutar solicitudes HTTP desde un **origen cruzado** (GitHub Pages) hacia un dominio objetivo, para validar:
- CORS (Access-Control-Allow-Origin)
- Preflight (OPTIONS)
- Envío de credenciales (cookies) y su impacto
- Diferentes tipos de cuerpo (JSON, form-urlencoded, FormData, texto)
- Diagnóstico de redirecciones y comportamiento del navegador
- Ejecución de una suite de pruebas reproducible
- Exportación de evidencia (JSON) y perfiles de configuración

> **Nota:** CORS es una política del navegador. La confirmación final de CORS/Preflight se obtiene en **DevTools → Network**.

---

## 1) Flujo recomendado de validación (paso a paso)

### Paso A — Elegir el endpoint correcto
1. Evite probar la “home” HTML del sitio. Use endpoints de API (idealmente JSON).
2. Pruebe primero un endpoint **público**:
   - Ej.: `https://TU-DOMINIO/wp-json/wp/v2/posts?per_page=1`
3. Luego pruebe un endpoint **sensitivo** (requiere sesión):
   - Ej.: “/me”, “/profile”, “/account” o endpoints propios autenticados.

### Paso B — Ejecutar una prueba simple (sin preflight)
Configurar:
- **Method:** GET (o POST si el endpoint lo requiere)
- **Credentials:** omit
- **Headers:** vacío
- **BodyType:** none

Interpretación:
- Si el navegador permite leer la respuesta, podrás ver status/body.
- Si bloquea, normalmente verás error “Failed to fetch” y el detalle exacto aparecerá en **Console**.

### Paso C — Forzar y validar preflight (OPTIONS)
Para obligar preflight:
- Agrega un header no simple (por ejemplo: `X-Test: 1`) o usa “Forzar preflight”.

Validar en DevTools:
1. Abre **DevTools → Network**
2. Debes ver una solicitud **OPTIONS** antes del request real.
3. En la respuesta de OPTIONS el servidor debe incluir:
   - `Access-Control-Allow-Origin`
   - `Access-Control-Allow-Methods`
   - `Access-Control-Allow-Headers`
   - (si aplica) `Access-Control-Allow-Credentials: true`

### Paso D — Prueba con credenciales (cookies)
Si tu autenticación es por cookies/sesión:
- Coloca **Credentials: include**
- Ejecuta la prueba
- En **Network → Request Headers**, verifica si el navegador envía `Cookie:`.

> Si NO se envían cookies, revisa `SameSite` de la cookie (Lax/Strict suelen impedir envío cross-site). En muchos casos se requiere `SameSite=None; Secure`.

---

## 2) Campos de la herramienta (qué significa cada uno)

### URL objetivo
Endpoint a consultar. Debe incluir `https://...`.

### Method
- GET/POST/PUT/PATCH/DELETE/OPTIONS/HEAD
- Si el endpoint es WordPress custom y exige POST, usar POST.

### Mode
- `cors`: modo normal para validar CORS.
- `no-cors`: el navegador limita lectura (útil solo para casos específicos).
- `same-origin`: solo funciona si es el mismo origen (no aplica para CORS típico).

### Credentials (cookies)
- `omit`: no envía cookies (pruebas públicas).
- `same-origin`: envía cookies solo si es mismo origen.
- `include`: envía cookies si el navegador lo permite (caso de mayor impacto).

### Redirect
- `follow`: sigue redirecciones.
- `manual`: diagnóstico (puede resultar `opaqueredirect`).
- `error`: falla si hay redirecciones.

### Headers
Uno por línea:
Nombre: valor

Ejemplos:
- `X-Test: 1`
- `Authorization: Bearer ...`
- `X-WP-Nonce: ...`

### Forzar preflight (atajo)
Agrega un header adicional si no existe ya, para disparar preflight.

### Timeout
Abortará el fetch si el tiempo se excede.

### BodyType y Body
- **none:** sin body
- **JSON:** envía `application/json`
- **x-www-form-urlencoded:** envía `application/x-www-form-urlencoded`
  - Formato en textarea: `clave=valor` por línea
- **multipart/FormData:** construye FormData con `clave=valor` por línea (sin archivos)
- **text/plain:** texto libre

---

## 3) Interpretación de resultados

### Lectura permitida
Si ves status + body en pantalla, el navegador **pudo leer** la respuesta.

### CORS bloqueado (típico)
Si ves `TypeError: Failed to fetch`, revisa **Console**:
- “No ‘Access-Control-Allow-Origin’ header…”
- “Response to preflight request doesn’t pass access control check…”

### Tipos de respuesta (resp.type)
- `basic`: respuesta normal
- `cors`: respuesta CORS (legible)
- `opaque`: típico de no-cors (no legible)
- `opaqueredirect`: redirección en modo manual

> Las cabeceras `Access-Control-*` deben revisarse de forma fiable en **Network → Response Headers**.

---

## 4) Suite de pruebas (reproducibilidad)
El botón **Ejecutar suite** corre una batería estándar para detectar:
- diferencias entre request simple vs preflight
- cambios por tipo de cuerpo
- impacto de credenciales

Al terminar, exporta el resultado con “Exportar último resultado (JSON)”.

---

## 5) Perfiles (guardar/cargar)
Permite guardar configuraciones repetibles (URL, headers, body, etc.).
- Guardar: asigna nombre y “Guardar”
- Cargar: selecciona y “Cargar”
- Exportar: “Exportar perfiles” (JSON)

---

## 6) Privacidad
- **Persistencia OFF**: no guarda perfiles en el navegador.
- **Body oculto**: evita exponer respuestas con datos sensibles en pantalla.
- **Limpiar**: reinicia formulario y registros.

---

## 7) Ejemplo específico (WordPress custom POST)
Si tu endpoint requiere POST y form-urlencoded:
- Method: POST
- BodyType: x-www-form-urlencoded
- Body:
q=test
- Credentials: omit (primero) / include (si necesitas cookies)

Validación final: en Network confirma:
- `Origin: https://TU-USUARIO.github.io`
- `Access-Control-Allow-Origin: https://TU-USUARIO.github.io`
- `Access-Control-Allow-Credentials: true` (solo si se requiere)

---

## 8) Buenas prácticas de seguridad CORS (resumen)
- Permitir solo orígenes estrictamente necesarios (exact match).
- Si hay `Allow-Credentials: true`, no usar `*`.
- Manejar correctamente OPTIONS (preflight).
- Evitar permitir orígenes que no controles plenamente.
