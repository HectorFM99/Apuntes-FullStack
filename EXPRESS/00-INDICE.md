# 🚂 CARPETA EXPRESS — ÍNDICE DE ESTUDIO

Teoría de **Express** (el framework para crear APIs con Node.js) explicada con
ejemplos reales **copiados y pegados de tus proyectos** del Bootcamp.

> ⚠️ Los proyectos **NO** se han modificado. Solo se han copiado fragmentos de su código.
> 🔒 Los archivos `.env` **no** se han copiado (tienen contraseñas). Solo se muestran los `.env.example`.

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a aprender |
|---|---------|--------------------|
| 1 | `01-introduccion-y-primer-servidor.md` | Qué es Express, instalarlo, `express()`, `app.listen()`, ciclo petición → respuesta |
| 2 | `02-rutas-y-router.md` | `app.get/post/put/patch/delete`, rutas con `:id`, `Router`, separar rutas en archivos, orden de las rutas |
| 3 | `03-request-y-response.md` | `req.params`, `req.query`, `req.body`, `express.json()`, `res.json()`, `res.status()`, códigos HTTP |
| 4 | `04-middleware-cors-y-errores.md` | Middleware, `next()`, orden, logger, CORS, middleware de sesión, 404 y manejador de errores |
| 5 | `05-validacion-de-datos.md` | Validar ids, query y body; funciones `validateX`, type guards, errores 400/404/409 |
| 6 | `06-arquitectura-por-capas-y-base-de-datos.md` | Rutas → servicios → almacenamiento, interfaces, conectar a MongoDB y SQLite, variables de entorno |
| 7 | `07-estructura-de-proyecto-y-typescript.md` | Carpetas, `package.json`, `tsconfig.json`, scripts, tipos de Express en TypeScript |
| 8 | `08-ejemplos-completos-de-proyectos.md` | Archivos enteros copiados de tus proyectos, comentados |

---

## 🗂️ Proyectos de los que se han extraído los ejemplos

| Proyecto | Archivos | Temas que cubre |
|----------|----------|-----------------|
| `API_Godines/api-express/api-books-claude/` | `src/index.ts`, `src/routes/books-routes.ts`, `src/services/books-storage.ts`, `src/services/books-storage-in-memory.ts`, `docs/prd-books.md` | CRUD completo en memoria, filtros por query, validación, **arquitectura por capas** con interfaz |
| `api-rastro/api-rastro/` | `src/index.ts`, `src/db.ts`, `src/types.ts`, `src/routes/listings.ts`, `src/routes/offers.ts` | API tipo "Wallapop" con **MongoDB**, varios routers, `409 Conflict`, agregaciones |
| `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/` | `src/index.ts`, `src/db.ts`, `src/routes/products.ts`, `src/routes/categories.ts`, `src/soluciones/*` | Express + MongoDB, CORS a mano, índice único, `try/catch` + `throw err` |
| `ProyectoReproductor/server/` | `index.ts` | Express 4 + **SQLite** (`sql.js`), cookies, middleware de **sesión**, `express.static`, llamar a otra API con `fetch` |

---

## 🚀 Orden de estudio recomendado

1. `01` → montar un servidor que responda "hola".
2. `02` → definir rutas (el "menú" de tu API).
3. `03` → leer lo que llega (`req`) y contestar bien (`res` + códigos HTTP).
4. `04` → **middleware**: la idea más importante de Express.
5. `05` → validar los datos (nunca te fíes del cliente).
6. `06` → separar en capas y conectar una base de datos.
7. `07` → organizar el proyecto y tiparlo con TypeScript.
8. `08` → leer código real completo de principio a fin.

---

## 💡 La idea de Express en 10 líneas

```ts
import express from "express";

const app = express();          // 1. Crear la aplicación
app.use(express.json());        // 2. Middleware: leer bodies JSON

app.get("/api/hello", (req, res) => {        // 3. Ruta: método + URL + función
  res.json({ message: "Hola" });             // 4. Responder
});

app.listen(3000, () => console.log("Escuchando en http://localhost:3000")); // 5. Arrancar
```

```
Navegador / Postman / fetch
        │  GET /api/hello
        ▼
┌──────────────────────────────┐
│  Express                     │
│  middleware → middleware →   │
│  ruta que coincide → res.json│
└──────────────────────────────┘
        │  200 { "message": "Hola" }
        ▼
     Cliente
```
