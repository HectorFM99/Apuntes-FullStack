# 01 — 🌐 FUNDAMENTOS WEB Y HTTP

> Antes de React, Node o NestJS, todo se apoya en lo mismo: un **cliente** pide algo
> y un **servidor** responde usando el protocolo **HTTP**. Si dominas esto, el resto
> encaja solo.

---

## 1.1 Cliente, servidor y protocolo

| Concepto | Qué es | Ejemplo tuyo |
|----------|--------|--------------|
| **Cliente** | Quien hace la petición (navegador, app, Postman, `fetch`) | El navegador abriendo `ProyectoReproductor` |
| **Servidor** | Programa que escucha en un puerto y responde | Tu `app.listen(3000)` de Express |
| **HTTP** | Las "reglas" del mensaje: método, ruta, cabeceras, cuerpo | `GET /api/books` |
| **API** | Conjunto de rutas que un servidor expone para que otros programas las usen | `api-rastro`, `api-books-nest` |

---

## 1.2 ¿Qué pasa cuando escribes una URL y pulsas Enter? ⭐

La pregunta clásica. Respuesta por pasos:

1. **Parseo de la URL**: `https://www.ejemplo.com/productos?id=3`
   → protocolo `https`, dominio `www.ejemplo.com`, ruta `/productos`, query `id=3`.
2. **DNS**: el navegador pregunta "¿qué IP tiene `www.ejemplo.com`?" (mira primero en
   caché del navegador, del sistema, del router… y si no, al servidor DNS).
3. **Conexión TCP** con esa IP (puerto 443 para HTTPS, 80 para HTTP).
4. **TLS (handshake)**: si es HTTPS, se negocia el cifrado y se verifica el certificado.
5. **Petición HTTP**: `GET /productos?id=3` con sus cabeceras (cookies, `Accept`…).
6. **El servidor procesa**: rutas → lógica → base de datos → genera respuesta.
7. **Respuesta HTTP**: status code (`200`), cabeceras y cuerpo (HTML, JSON…).
8. **Renderizado**: el navegador parsea el HTML → construye el **DOM**, descarga CSS
   (→ **CSSOM**), JS e imágenes; calcula el layout y **pinta** la página.

```
Tú ──URL──▶ DNS ──IP──▶ TCP + TLS ──▶ GET /productos ──▶ Servidor ──▶ BD
                                                    ◀── 200 + HTML/JSON ◀──
                                       Navegador: DOM + CSSOM → layout → paint
```

> 📌 No hace falta que te sepas todos los detalles de redes. Con los 8 pasos en orden y
> bien explicados ya das una respuesta muy buena para un junior.

---

## 1.3 Anatomía de una petición y una respuesta

```http
POST /api/books HTTP/1.1            ← método + ruta + versión
Host: localhost:3000                ← cabeceras
Content-Type: application/json
Authorization: Bearer eyJhbGci...

{ "title": "Cien años de soledad" } ← cuerpo (body)
```

```http
HTTP/1.1 201 Created                ← status code
Content-Type: application/json

{ "id": 7, "title": "Cien años de soledad" }
```

| Parte | Para qué sirve |
|-------|----------------|
| **Método** | Qué acción quieres hacer |
| **Ruta + params** | Sobre qué recurso (`/books/7`) |
| **Query string** | Filtros/opciones (`?author=gabriel&maxPages=150`) |
| **Cabeceras** | Metadatos: tipo de contenido, auth, cookies, caché |
| **Body** | Los datos (normalmente JSON) |

---

## 1.4 Métodos HTTP

| Método | Uso | ¿Idempotente? | ¿Body? |
|--------|-----|---------------|--------|
| `GET` | Leer | ✅ Sí | ❌ No |
| `POST` | Crear | ❌ No | ✅ Sí |
| `PUT` | **Reemplazar** entero | ✅ Sí | ✅ Sí |
| `PATCH` | Modificar **parcialmente** | ⚠️ Normalmente no se garantiza | ✅ Sí |
| `DELETE` | Borrar | ✅ Sí | Normalmente no |

**Idempotente** = hacerlo 1 vez o 10 veces deja el servidor **igual**.
Borrar el libro 7 diez veces → sigue borrado. Crear (POST) diez veces → 10 libros.

```js
// ❌ PUT con solo un campo: técnicamente "reemplaza" el recurso y perderías el resto
PUT /api/books/7   { "pages": 300 }

// ✅ Para cambiar solo un campo, PATCH
PATCH /api/books/7 { "pages": 300 }
```

> 📌 En `api-books-nest` usaste `@Patch(':id')` precisamente para actualizar solo los
> campos que llegan en el DTO.

---

## 1.5 Status codes (códigos de estado)

| Familia | Significado | Los que debes saber |
|---------|-------------|---------------------|
| **1xx** | Informativo | (casi nunca preguntan) |
| **2xx** | ✅ Éxito | `200 OK`, `201 Created`, `204 No Content` |
| **3xx** | ↪️ Redirección | `301 Moved Permanently`, `302 Found`, `304 Not Modified` |
| **4xx** | 🙋 Error **del cliente** | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`, `429 Too Many Requests` |
| **5xx** | 💥 Error **del servidor** | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable` |

### 401 vs 403 (pregunta trampa)

| Código | Significa | Ejemplo |
|--------|-----------|---------|
| `401` | **No sé quién eres** (no autenticado) | No has hecho login |
| `403` | **Sé quién eres, pero no puedes** (no autorizado) | Usuario normal intentando borrar como admin |

> 📌 En `ProyectoReproductor` devuelves `401` si no hay sesión, `409` si el correo ya
> existe al registrarse, `201` al crear y `204` al hacer logout. ¡Es un buen ejemplo!

---

## 1.6 Cookies, sesiones, `localStorage` y `sessionStorage`

| | Dónde vive | ¿Se envía solo al servidor? | Duración | Tamaño |
|---|-----------|-----------------------------|----------|--------|
| **Cookie** | Navegador | ✅ Sí, en cada petición a ese dominio | La que diga `Expires`/`Max-Age` | ~4 KB |
| **`localStorage`** | Navegador | ❌ No | Hasta que se borre | ~5 MB |
| **`sessionStorage`** | Navegador (por pestaña) | ❌ No | Hasta cerrar la pestaña | ~5 MB |
| **Sesión** | **Servidor** (memoria/BD) | Se identifica con un id guardado en una cookie | La que defina el servidor | — |

### Flags de cookie importantes

| Flag | Para qué |
|------|----------|
| `HttpOnly` | JS **no puede leerla** (`document.cookie`) → protege contra XSS |
| `Secure` | Solo se envía por **HTTPS** |
| `SameSite=Lax/Strict` | No se envía en peticiones de otros sitios → protege contra CSRF |
| `Max-Age` / `Expires` | Cuándo caduca |

```ts
// Ejemplo de ProyectoReproductor (server/index.ts), simplificado
res.cookie('nightjar_session', token, {
  httpOnly: true,
  sameSite: 'lax',
  secure: process.env.NODE_ENV === 'production',
  maxAge: 1209600000, // 14 días
});
```

> 📌 HTTP es **sin estado (stateless)**: cada petición es independiente. Las cookies y
> las sesiones/tokens son la forma de "recordar" quién eres entre peticiones.

---

## 1.7 CORS ⭐

**Same-Origin Policy**: por seguridad, el navegador no deja que el JS de una página lea
respuestas de **otro origen**.

**Origen** = protocolo + dominio + puerto.

| URL A | URL B | ¿Mismo origen? |
|-------|-------|----------------|
| `http://localhost:5173` | `http://localhost:3000` | ❌ (puerto distinto) |
| `http://midominio.com` | `https://midominio.com` | ❌ (protocolo distinto) |
| `https://midominio.com/a` | `https://midominio.com/b` | ✅ |

**CORS** (*Cross-Origin Resource Sharing*) es el mecanismo para que el **servidor**
diga "este otro origen sí puede leerme", con cabeceras como:

```http
Access-Control-Allow-Origin: http://localhost:5173
Access-Control-Allow-Methods: GET, POST, PATCH, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

```js
// Express
import cors from "cors";
app.use(cors({ origin: "http://localhost:5173" }));

// NestJS
app.enableCors({ origin: "http://localhost:5173" });
```

**Preflight**: para peticiones "no simples" (p. ej. `PATCH`, o `Content-Type: application/json`)
el navegador manda antes una petición `OPTIONS` para preguntar si está permitido.

```
❌ "CORS es un error del servidor" → No: lo aplica el NAVEGADOR. Postman o curl no tienen CORS.
✅ "El navegador bloquea leer la respuesta si el servidor no lo permite con sus cabeceras."
```

---

## 1.8 Otros conceptos que pueden salir

| Concepto | En una frase |
|----------|--------------|
| **HTTPS** | HTTP + cifrado TLS: nadie en medio puede leer ni modificar los datos |
| **DNS** | La "agenda de contactos" de Internet: dominio → IP |
| **REST** | Estilo para diseñar APIs con recursos y métodos HTTP (ver `05`) |
| **JSON** | Formato de texto para intercambiar datos (`JSON.stringify` / `JSON.parse`) |
| **Caché** | Guardar respuestas para no pedirlas otra vez (`Cache-Control`, `ETag` → `304`) |
| **SSR vs CSR** | El HTML se genera en el servidor vs. lo genera JS en el navegador |
| **WebSocket** | Conexión bidireccional abierta (chats, tiempo real), a diferencia de HTTP petición-respuesta |

---

## 🎯 Cómo responderlo en la entrevista

**¿Qué pasa cuando escribes una URL?**
> "El navegador resuelve el dominio con DNS para obtener la IP, abre una conexión TCP
> y, si es HTTPS, negocia el cifrado TLS. Luego envía una petición HTTP GET; el
> servidor la procesa (rutas, lógica, base de datos) y devuelve una respuesta con un
> código de estado y el contenido. Finalmente el navegador construye el DOM y el CSSOM,
> ejecuta el JS y pinta la página."

**¿Diferencia entre PUT y PATCH?**
> "PUT reemplaza el recurso completo; PATCH modifica solo los campos que envías. En mi
> API de libros con NestJS usé PATCH para actualizaciones parciales."

**¿401 vs 403?**
> "401 es que no estás autenticado, el servidor no sabe quién eres. 403 es que sí sabe
> quién eres pero no tienes permiso para esa acción."

**¿Qué es CORS?**
> "Es un mecanismo del navegador. Por la política de mismo origen, una página no puede
> leer respuestas de otro origen salvo que el servidor lo permita con cabeceras
> `Access-Control-Allow-Origin`. Se configura en el backend, por ejemplo con el
> middleware `cors` en Express o `enableCors` en Nest."

**¿Cookies vs localStorage?**
> "Las cookies se envían automáticamente al servidor en cada petición y pueden ser
> `HttpOnly`, así que son mejores para guardar la sesión. `localStorage` solo vive en el
> navegador, no se envía solo y cualquier JS de la página puede leerlo, así que no es
> buen sitio para tokens sensibles."
