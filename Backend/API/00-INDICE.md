# 🌐 CARPETA API — ÍNDICE DE ESTUDIO

Teoría de **APIs REST, HTTP y consumo de APIs** explicada con ejemplos reales
**copiados y pegados de tus proyectos** del Bootcamp.

> ⚠️ Los proyectos **NO** se han modificado. Solo se han copiado fragmentos de su código.
> Los archivos `.env` **no** se han copiado (solo los `.env.example`).
>
> 📌 Aquí se ve la API desde el lado **HTTP / REST / cliente**. El código del
> servidor Express en detalle (middlewares, `Router`, `req`/`res`...) está en la
> carpeta `EXPRESS`; las rutas del servidor aparecen aquí solo como ejemplos.

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a aprender |
|---|---------|--------------------|
| 1 | `01-que-es-una-api-y-http.md` | Qué es una API, cliente-servidor, URL, petición y respuesta HTTP, headers, body, JSON |
| 2 | `02-metodos-http-y-codigos-de-estado.md` | `GET`/`POST`/`PUT`/`PATCH`/`DELETE`, CRUD, PUT vs PATCH, idempotencia, códigos 2xx/3xx/4xx/5xx |
| 3 | `03-rest-y-diseno-de-apis.md` | Principios REST, nombrar rutas, params vs query vs body, formato de errores, diseñar y documentar (PRD) |
| 4 | `04-consumir-apis-con-fetch.md` | `fetch` con `.then` y `async/await`, `response.ok`, `try/catch`, `Promise.all`, peticiones encadenadas |
| 5 | `05-enviar-datos-post-put-patch-delete.md` | `method`, `headers`, `body`, `JSON.stringify`, formularios, 204, un cliente reutilizable |
| 6 | `06-autenticacion-cors-y-seguridad.md` | Sesiones con cookie, JWT y `Authorization`, API keys, `.env`, contraseñas, **CORS** |
| 7 | `07-probar-apis-curl-postman-thunder.md` | DevTools Network, `curl`, Postman, Thunder Client, `/api/health`, diagnosticar errores |
| 8 | `08-ejemplos-completos-de-proyectos.md` | Archivos enteros copiados de tus proyectos |
| 📋 | `99-CHEATSHEET.md` | Chuleta de comandos y código para consultar rápido |

---

## 🗂️ Proyectos de los que se han extraído los ejemplos

| Proyecto | Archivo(s) | Temas que cubre |
|----------|-----------|-----------------|
| `Prueba-Api/` | `prueba.js`, `02-obtener-publicacion-id2.js`, `03-url-que-no-existe.js`, `prueba2.js`, `prueba3.js`, `README.md` | `fetch`, `response.ok`, 404, **POST**, 201, `Promise.all` (API JSONPlaceholder) |
| `Async-hector/Advanced-JS-Asynchronous-code/` | `index.js`, `readme.md` | `async/await`, Dog API, GitHub API, DOM, `Promise.all` con `map` |
| `Async-Hector-2/advanced-js-bikram-async-await/` | `index.js`, `readme.md` | PokéAPI, Rick and Morty, JSON anidado, peticiones encadenadas |
| `ProyectoReproductor/` | `public/app.js`, `public/login.js` | Función `request` reutilizable, POST/DELETE, 401, 204, sesión, formularios |
| `ProyectoReproductor/` | `dist/server/index.js`, `replica/backend.md`, `.env.example` | Rutas de auth, **cookies de sesión**, hash de contraseñas, llamar a otra API (Jamendo), secretos |
| `API_Godines/api-express/api-books-claude/` | `docs/prd-books.md`, `src/routes/books-routes.ts`, `src/data/books.json` | Diseño (PRD), CRUD completo, PUT vs PATCH, filtros por query, 400/404 |
| `api-rastro/api-rastro/` | `src/routes/listings.ts`, `src/routes/offers.ts`, `src/index.ts`, `respuestas.md`, `.env.example` | Rutas REST, sub-recursos, 400 vs 404 vs 409, 204, `/api/health`, errores en JSON |
| `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/` | `README.md` | Batería de pruebas con **curl** |

> 📌 La carpeta `autenticacion/` existe pero está **vacía**: los ejemplos de
> autenticación salen de `ProyectoReproductor`. JWT y CORS se explican con ejemplos
> propios (marcados como tales) porque no aparecen en tus proyectos.

---

## 🚀 Orden de estudio recomendado

1. `01` → qué viaja entre cliente y servidor (sin esto nada más tiene sentido).
2. `02` → métodos y códigos de estado: el "vocabulario" de HTTP.
3. `03` → REST: cómo se **diseña** una API ordenada.
4. `04` → **leer** datos de una API con `fetch` (lo que más vas a hacer).
5. `05` → **mandar** datos: POST, PUT, PATCH y DELETE.
6. `06` → quién eres (sesiones/tokens) y por qué el navegador a veces bloquea (CORS).
7. `07` → probar APIs sin frontend y diagnosticar errores.
8. `08` → código real completo de tus proyectos.

---

## 💡 La idea clave en 10 líneas

```js
// CLIENTE (navegador)                                   // SERVIDOR (Express)
const res = await fetch("/api/books/1", {                // router.patch("/:id", (req, res) => {
  method: "PATCH",                                       //   req.params.id   → "1"
  headers: { "Content-Type": "application/json" },       //   req.body        → { pages: 210 }
  body: JSON.stringify({ pages: 210 }),                  //   ...
});                                                      //   res.status(200).json(libro);
if (!res.ok) throw new Error(`Error ${res.status}`);     // });
const libro = await res.json();
```

**URL** (qué recurso) + **método** (qué acción) + **headers** (cómo) + **body**
(datos) → **código de estado** (cómo ha ido) + **JSON** (resultado).
