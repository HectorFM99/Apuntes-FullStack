# 📋 CHEATSHEET — APIs Y HTTP

> Chuleta rápida de comandos y código. Teoría en los documentos 01–08 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal (curl)

| Opción | Qué hace |
|--------|----------|
| `-i` | Muestra status + cabeceras de la respuesta |
| `-X METODO` | Cambia el método (`POST`, `PATCH`, `DELETE`…) |
| `-H "Clave: valor"` | Añade una cabecera |
| `-d '{...}'` | Envía un body (sin `-X` pasa a ser POST) |
| `-d "@archivo.json"` | Envía el body desde un archivo |
| `-v` | Modo verbose: todo lo que se envía y recibe |
| `-c cookies.txt` / `-b cookies.txt` | Guardar / enviar cookies |

```bash
curl -i http://localhost:3000/api/health                                   # ¿está viva? → 200 {"status":"ok"}
curl -i http://localhost:3000/api/products                                 # GET lista → 200 + array
curl -i "http://localhost:3000/api/products?category=muebles&search=TEC"   # query (comillas por el &)
curl -i http://localhost:3000/api/products/<ID>                            # GET uno → 200 / 400 / 404
curl -i -X POST http://localhost:3000/api/products \
  -H "Content-Type: application/json" -d '{"name":"Monitor","price":149.99}'   # crear → 201
curl -i -X PATCH http://localhost:3000/api/products/<ID> \
  -H "Content-Type: application/json" -d '{"price":34.5}'                       # modificar → 200
curl -i -X DELETE http://localhost:3000/api/products/<ID>                       # borrar → 204
```

```bash
# Sesión con cookie
curl -i -c cookies.txt -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" -d '{"email":"yo@mail.com","password":"..."}'
curl -i -b cookies.txt http://localhost:3000/api/auth/me                   # privada con cookie
curl -i http://localhost:3000/api/library/favorites                        # sin cookie → 401

# JWT
curl -i http://localhost:3000/api/perfil -H "Authorization: Bearer <TOKEN>"
```

```powershell
# ⚠️ PowerShell 5.1: "curl" es alias de Invoke-WebRequest → usa curl.exe
curl.exe -i http://localhost:3000/api/books
curl.exe -i -X POST http://localhost:3000/api/books -H "Content-Type: application/json" -d "@libro.json"
```

---

## 2. Métodos HTTP ↔ CRUD

| Método | CRUD | Ruta típica | Body | Respuesta OK | Idempotente |
|--------|------|-------------|:----:|--------------|:-----------:|
| `GET` | Read | `/api/books`, `/api/books/:id` | ❌ | 200 | ✅ |
| `POST` | Create | `/api/books` | ✅ | **201** + recurso | ❌ |
| `PUT` | Update (entero) | `/api/books/:id` | ✅ todo | 200 | ✅ |
| `PATCH` | Update (parcial) | `/api/books/:id` | ✅ algunos campos | 200 | ⚠️ normalmente |
| `DELETE` | Delete | `/api/books/:id` | ❌ | **204** sin body (o 200 + borrado) | ✅ |

---

## 3. Códigos de estado

| Código | Nombre | Cuándo |
|--------|--------|--------|
| **200** | OK | GET / PUT / PATCH correctos |
| **201** | Created | POST que crea |
| **204** | No Content | DELETE sin body |
| 301 / 304 | Moved / Not Modified | Redirección / caché |
| **400** | Bad Request | Body inválido, id con formato incorrecto, JSON roto |
| **401** | Unauthorized | No autenticado (no sé quién eres) |
| **403** | Forbidden | Autenticado pero sin permiso |
| **404** | Not Found | El recurso o la ruta no existe |
| **409** | Conflict | Duplicado (`unique`, error Mongo `11000`) |
| 422 | Unprocessable | Validación semántica (alternativa a 400) |
| **500** | Internal Server Error | Fallo del servidor (no dar detalles al cliente) |
| 503 | Service Unavailable | Servidor/BD no disponible |

> 400 = "me lo has pedido mal" · 404 = "no existe" · 409 = "choca con lo que ya hay".

---

## 4. Diseño REST

```
✅ GET    /api/books              ← colección en plural
✅ GET    /api/books/1            ← un elemento (id en la RUTA)
✅ GET    /api/books?country=chi&limit=5   ← filtros en la QUERY
✅ GET    /api/books/1/reviews    ← recurso anidado
❌ /api/getBooks   ❌ /api/books/1/delete   ❌ /api/books?id=1
```

| Dato | Dónde | En Express |
|------|-------|-----------|
| Identificar UN recurso | Ruta `/api/books/:id` | `req.params.id` |
| Filtrar, ordenar, paginar | Query `?country=chi&limit=5` | `req.query.country` |
| Datos a crear/modificar | Body JSON | `req.body` |

```json
// Formato de error consistente
{ "error": "El id debe ser un número entero" }
```

> 📌 Rutas fijas **antes** que `/:id` (`/api/books/stats` antes de `/api/books/:id`).

---

## 5. `fetch`: leer (GET)

```js
// Plantilla mínima con async/await
async function getJSON(url) {
  const response = await fetch(url);                         // 1. pedir
  if (!response.ok) throw new Error(`Error HTTP: ${response.status}`);  // 2. fetch NO falla con 404/500
  return response.json();                                    // 3. body → objeto JS
}
```

```js
// Con .then
fetch("https://dog.ceo/api/breeds/image/random")
  .then((res) => res.json())
  .then((data) => console.log(data.message))
  .catch((err) => console.error(err.message));
```

```js
// Leer el mensaje de error de TU API ({ error: "..." })
if (!response.ok) {
  const body = await response.json().catch(() => ({}));
  throw new Error(body.error || `Error HTTP: ${response.status}`);
}
```

```js
// URL dinámica y query segura
fetch(`http://localhost:3000/api/books/${id}`);
fetch(`/api/tracks/search?q=${encodeURIComponent(q)}`);
const params = new URLSearchParams({ country: "chi", limit: "5" });
fetch(`/api/books?${params}`);                               // ?country=chi&limit=5
```

| `Response` | Qué es |
|------------|--------|
| `response.ok` | `true` si status 200–299 |
| `response.status` | Código (`404`) |
| `response.json()` / `.text()` | Leer el body (promesa) |
| `response.headers.get("content-type")` | Leer una cabecera |

---

## 6. `fetch`: enviar (POST / PUT / PATCH / DELETE)

```js
const response = await fetch(url, {
  method: "POST",                                    // "PUT" | "PATCH" | "DELETE"
  headers: { "Content-Type": "application/json" },   // "te mando JSON"
  body: JSON.stringify(datos),                       // objeto → texto
});
```

```js
// DELETE: sin body ni Content-Type; 204 = no hay JSON que leer
const res = await fetch(`/api/books/${id}`, { method: "DELETE" });
if (res.status === 204) console.log("Borrado");
```

```js
// Formulario → JSON
form.addEventListener("submit", async (event) => {
  event.preventDefault();                            // no recargar
  const data = new FormData(event.currentTarget);
  await request("/api/auth/login", {
    method: "POST",
    body: JSON.stringify({ email: data.get("email"), password: data.get("password") }),
  });
});
```

```js
// Cliente reutilizable (patrón de tu reproductor)
async function request(url, options) {
  const response = await fetch(url, {
    headers: { "Content-Type": "application/json", ...(options?.headers || {}) },
    ...options,
  });
  if (response.status === 401) throw new Error("Sesión expirada");
  if (!response.ok) {
    const body = await response.json().catch(() => ({}));
    throw new Error(body.error || "No se pudo completar la solicitud.");
  }
  return response.status === 204 ? undefined : response.json();
}
```

---

## 7. Varias peticiones

```js
// En paralelo
const [resUser, resPost] = await Promise.all([
  fetch(".../users/1"),
  fetch(".../posts/1"),
]);

// Encadenadas (la 2ª necesita datos de la 1ª)
const character = await (await fetch(url)).json();
const episode = await (await fetch(character.episode[0])).json();
```

```js
// Estados de carga en la UI
setStatus("Buscando...");                 // cargando
try { const data = await request(url); setStatus(`${data.length} resultados`); }
catch (e) { setStatus(e.message, true); } // error
```

---

## 8. Autenticación y CORS

```js
// JWT: enviarlo en cada petición
fetch("/api/perfil", { headers: { Authorization: `Bearer ${token}` } });

// Cookie de sesión entre orígenes distintos
fetch("http://localhost:3000/api/auth/me", { credentials: "include" });
```

```js
// Servidor Express: permitir el frontend
import cors from "cors";
app.use(cors({ origin: "http://localhost:5500" }));                     // un origen
app.use(cors({ origin: "http://localhost:5500", credentials: true }));  // + cookies
```

| | Cookie de sesión | JWT `Authorization` |
|---|---|---|
| Estado | En el servidor | En el token (firmado) |
| Se envía | Sola (`credentials: "include"`) | A mano en la cabecera |
| Revocar | Fácil | Difícil |

| Cabecera | Para qué |
|----------|----------|
| `Content-Type: application/json` | Formato del body |
| `Authorization: Bearer <token>` | Credenciales |
| `Access-Control-Allow-Origin` | Origen permitido (CORS, la pone el servidor) |
| `Set-Cookie` / `Cookie` | Sesión |

---

## 9. `/api/health`

```ts
app.get("/api/health", async (_req, res) => {
  await getDb().command({ ping: 1 });                 // comprueba la BD
  res.json({ status: "ok", db: getDb().databaseName });
});
```

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `blocked by CORS policy` (con curl funciona) | Permitir el origen en el servidor (`cors({ origin })`) |
| `data.title` es `undefined` tras un 404 | Comprobar `response.ok` antes de `.json()` |
| `req.body` llega `undefined` / `{}` | Falta `Content-Type: application/json` o `JSON.stringify` (y `express.json()` en el server) |
| `Unexpected end of JSON input` | Llamar `.json()` sobre un 204 sin body |
| `Failed to fetch` | API caída, URL/puerto mal, `http` vs `https` |
| curl en PowerShell da error de parámetros | Usar `curl.exe` |
| Query con `&` cortada en la terminal | Poner la URL entre comillas |
| 401 en rutas privadas desde el navegador | Falta `credentials: "include"` o el `Bearer` |
