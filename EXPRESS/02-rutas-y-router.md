# 02 — RUTAS Y ROUTER

> Una **ruta** es la combinación de un **método HTTP** + una **URL** + la **función** que
> se ejecuta cuando llega esa petición. El conjunto de rutas es "el menú" de tu API.

---

## 2.1 Anatomía de una ruta

```ts
app.get("/api/books/:id", (req, res) => { ... });
//  ▲          ▲                    ▲
//  │          │                    └─ handler: función que responde
//  │          └─ path (ruta). ":id" es un parámetro variable
//  └─ método HTTP
```

| Método | Para qué (CRUD) | Ejemplo en tus proyectos |
|--------|-----------------|--------------------------|
| `app.get` | **Leer** (Read) | `GET /api/books` → lista de libros |
| `app.post` | **Crear** (Create) | `POST /api/listings` → publica un anuncio |
| `app.put` | **Sustituir** entero (Update) | `PUT /api/books/:id` → reemplaza el libro completo |
| `app.patch` | **Modificar** parte (Update) | `PATCH /api/offers/:id` → acepta una oferta |
| `app.delete` | **Borrar** (Delete) | `DELETE /api/products/:id` |

### PUT vs PATCH — explicado en tu PRD (`API_Godines/api-express/api-books-claude/docs/prd-books.md`)

```md
4. Editar y borrar un libro:
   - `PUT /api/books/:id`: sustituye el libro entero (todos los campos obligatorios).
   - `PATCH /api/books/:id`: modifica solo los campos enviados (al menos uno).
   - `DELETE /api/books/:id`: borra el libro y lo devuelve (200).
```

```
Libro guardado:   { id: 1, title: "A", pages: 100, year: 1958 }

PUT   /api/books/1  { title: "B", pages: 200, year: 1960 }  → { id: 1, title: "B", pages: 200, year: 1960 }
PATCH /api/books/1  { pages: 210 }                          → { id: 1, title: "A", pages: 210, year: 1958 }
```

---

## 2.2 Rutas directamente en `app`

Para APIs pequeñas puedes colgar todas las rutas de `app`.

### Ejemplo REAL copiado (`ProyectoReproductor/server/index.ts`, formateado para leerlo mejor)

```ts
app.post('/api/auth/register', (req, res) => { ... });
app.post('/api/auth/login',    (req, res) => { ... });
app.post('/api/auth/logout',   (req, res) => { ... });
app.get('/api/auth/me',        (req, res) => { ... });

app.get('/api/tracks/search',   session, async (req, res) => { ... });
app.get('/api/tracks/featured', session, async (req, res) => { ... });

app.get('/api/library/:kind',                 session, (req, res) => { ... });
app.post('/api/library/favorites/:trackId',   session, (req, res) => { ... });
app.delete('/api/library/favorites/:trackId', session, (req, res) => { ... });
app.post('/api/library/history',              session, (req, res) => { ... });

app.get('*', (_req, res) => res.sendFile(join(process.cwd(), 'public', 'index.html')));
```

> 📌 Fíjate en `session` entre la URL y el handler: es un **middleware** que se ejecuta
> antes de la ruta (comprueba que hay sesión). Lo verás en `04`.
> 📌 `app.get('*', ...)` al final: cualquier otra URL devuelve el `index.html` del
> frontend (típico de una SPA). En **Express 5** se escribiría `app.get('/*splat', ...)`.

**Problema:** cuando la API crece, un solo archivo con 30 rutas es difícil de leer.
La solución es el `Router`.

---

## 2.3 `Router` — un "mini-app" para agrupar rutas

`Router()` crea un objeto con los **mismos métodos** que `app` (`get`, `post`, `use`...),
pero que no escucha en ningún puerto: se **monta** dentro de `app` bajo un prefijo.

```ts
// routes/books-routes.ts
import { Router } from "express";

const router = Router();

router.get("/", (req, res) => { ... });       // GET  /api/books
router.get("/:id", (req, res) => { ... });    // GET  /api/books/:id
router.post("/", (req, res) => { ... });      // POST /api/books

export default router;
```

```ts
// index.ts
import booksRouter from "./routes/books-routes";

app.use("/api/books", booksRouter);   // ← prefijo + router
```

```
app.use("/api/books", booksRouter)
                │
                └─ dentro del router, "/" significa "/api/books"
                   y "/:id" significa "/api/books/:id"
```

> 📌 Dentro del router **no repites el prefijo**. `router.get("/api/books/:id")` sería
> `/api/books/api/books/:id` ❌.

---

## 2.4 Separar rutas en archivos

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/index.ts`)

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import { connectDb, getDb } from "./db";
import listingsRouter from "./routes/listings";
import offersRouter from "./routes/offers";

const app = express();
const PORT = 3000;

// ...

app.use("/api/listings", listingsRouter);
app.use("/api/offers", offersRouter);
```

### Y el router (`api-rastro/api-rastro/src/routes/offers.ts`, solo la estructura)

```ts
import { Router } from "express";
import { ObjectId } from "mongodb";
import { listingsCollection, offersCollection, usersCollection } from "../db";
import type { Offer } from "../types";

const router = Router();

/**
 * Crea una nueva oferta para un anuncio.
 */
router.post("/", async (req, res) => { ... });       // POST  /api/offers

/**
 * Acepta una oferta pendiente y reserva el anuncio.
 */
router.patch("/:id", async (req, res) => { ... });   // PATCH /api/offers/:id

export default router;
```

**Mapa de rutas completo de `api-rastro`:**

| Método | URL | Archivo |
|--------|-----|---------|
| GET | `/api/health` | `index.ts` |
| GET | `/api/listings` | `routes/listings.ts` |
| GET | `/api/listings/stats` | `routes/listings.ts` |
| GET | `/api/listings/:id` | `routes/listings.ts` |
| GET | `/api/listings/:id/offers` | `routes/listings.ts` |
| POST | `/api/listings` | `routes/listings.ts` |
| PATCH | `/api/listings/:id` | `routes/listings.ts` |
| DELETE | `/api/listings/:id` | `routes/listings.ts` |
| POST | `/api/offers` | `routes/offers.ts` |
| PATCH | `/api/offers/:id` | `routes/offers.ts` |

> 📌 Regla práctica: **un archivo de rutas por recurso** (`books`, `listings`, `offers`,
> `products`, `categories`). El nombre del recurso va en **plural** en la URL.

---

## 2.5 Parámetros de ruta (`:id`) y rutas anidadas

Todo lo que empieza por `:` en la ruta es un **parámetro**: Express lo guarda en
`req.params`.

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
router.get("/:id/offers", async (req, res) => {
  const { id } = req.params;

  if (!ObjectId.isValid(id)) {
    res.status(400).json({ error: "El id no tiene un formato válido" });
    return;
  }

  const listingId = new ObjectId(id);

  const listing = await listingsCollection().findOne({ _id: listingId }, { projection: { _id: 1 } });
  if (!listing) {
    res.status(404).json({ error: "No existe ningún anuncio con ese id" });
    return;
  }

  const offers = await offersCollection().find({ "listing.id": listingId }).sort({ createdAt: -1 }).toArray();

  res.json(offers);
});
```

```
GET /api/listings/665f1c.../offers
                  └──┬──┘
               req.params.id = "665f1c..."
```

> 📌 `/:id/offers` = "las ofertas **de** este anuncio". Es la forma REST de expresar
> relaciones: recurso padre → id → recurso hijo.

### Varios parámetros (`ProyectoReproductor/server/index.ts`)

```ts
app.get('/api/library/:kind', session, (req, res) => {
  const kind = String(req.params.kind);
  if (!['favorites', 'history'].includes(kind)) return res.status(404).json({ error: 'Biblioteca no encontrada.' });
  // ...
});
```

Aquí `:kind` puede ser `favorites` o `history`; cualquier otra cosa → 404.

---

## 2.6 ⚠️ El ORDEN de las rutas importa

Express prueba las rutas **de arriba abajo** y usa la **primera que coincide**.
`/:id` coincide con **cualquier cosa**, incluida la palabra `stats`.

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
/**
 * Obtiene estadísticas de anuncios agrupadas por categoría.
 */
router.get("/stats", async (_req, res) => {     // ← PRIMERO la ruta fija
  const stats = await listingsCollection()
    .aggregate([ /* ... */ ])
    .toArray();

  res.json(stats);
});

router.get("/:id", async (req, res) => {         // ← DESPUÉS la ruta con parámetro
  const { id } = req.params;
  // ...
});
```

```ts
// ❌ Al revés: GET /api/listings/stats entra en "/:id" con id = "stats"
router.get("/:id", ...);
router.get("/stats", ...);   // nunca se llega aquí → 400 "El id no tiene un formato válido"

// ✅ Rutas fijas antes que rutas con parámetros
router.get("/stats", ...);
router.get("/:id", ...);
```

> 📌 Lo mismo pasa con los routers en `app`: el **404 final** y el **manejador de errores**
> van siempre **al final** del archivo, después de todas las rutas.

---

## 2.7 Montar varios routers en el mismo prefijo

`app.use(prefijo, routerA, routerB)` prueba primero `routerA`; si ninguna de sus rutas
coincide, pasa a `routerB`. Tu proyecto lo usa para "sobrescribir" una ruta.

### Ejemplo REAL copiado (`mongo-api-libros/.../src/soluciones/ejercicio-07-patch-updateOne.ts`)

```ts
// Ejercicio 7: el PATCH de productos con updateOne en lugar de findOneAndUpdate.
//
// No está montado en la API. Para probarlo, en src/index.ts cambia
//   app.use("/api/products", productsRouter);
// por
//   app.use("/api/products", patchWithUpdateOne, productsRouter);
// e importa: import patchWithUpdateOne from "./soluciones/ejercicio-07-patch-updateOne";
// (al ir delante, su PATCH responde antes que el del router de productos)
import { Router } from "express";
import { ObjectId } from "mongodb";
import { productsCollection, validateProduct } from "../routes/products";
import type { Product } from "../routes/products";

const router = Router();

router.patch("/:id", async (req, res) => {
  // ...
});

export default router;
```

```
PATCH /api/products/123
   │
   ├─ patchWithUpdateOne  → tiene router.patch("/:id") → ✅ responde aquí
   └─ productsRouter      → no llega

GET /api/products
   │
   ├─ patchWithUpdateOne  → no tiene GET "/" → sigue
   └─ productsRouter      → router.get("/") → ✅ responde aquí
```

---

## 2.8 Cambiar un router entero sin tocar el resto

Como cada router es un módulo con `export default`, puedes **cambiar de implementación**
solo cambiando el `import`.

### Ejemplo REAL copiado (`mongo-api-libros/.../src/soluciones/reto-1-products-id.ts`)

```ts
// Reto 1: devolver "id" (texto) en lugar de "_id", sin cambiar lo que se guarda en MongoDB.
//
// Es el router de productos completo usando toApi() en todas las respuestas.
// Para usarlo, en src/index.ts cambia el import de productsRouter por:
//   import productsRouter from "./soluciones/reto-1-products-id";
```

> 📌 `index.ts` solo sabe que hay "un router de productos"; no le importa cómo está hecho
> por dentro. Esa es la ventaja de separar en archivos.

---

## 2.9 Encadenar rutas con `router.route()` (no está en tus proyectos)

Cuando varias rutas comparten la misma URL, Express permite agruparlas. Tus proyectos no
lo usan, pero lo verás en documentación:

```ts
// Ejemplo propio
router.route("/:id")
  .get((req, res) => { /* leer */ })
  .put((req, res) => { /* sustituir */ })
  .patch((req, res) => { /* modificar */ })
  .delete((req, res) => { /* borrar */ });
```

Es **exactamente lo mismo** que escribir `router.get("/:id")`, `router.put("/:id")`...
por separado (que es lo que hacen tus proyectos).

---

## 2.10 Errores comunes con rutas

| Error | Causa | Solución |
|-------|-------|----------|
| `Cannot GET /api/books/` y no lo entiendes | Método equivocado (hiciste POST a una ruta GET, o al revés) | Revisa método + URL |
| La ruta `/stats` devuelve "id no válido" | `/:id` está **antes** | Rutas fijas primero |
| Rutas duplicadas `/api/books/api/books` | Repetiste el prefijo dentro del router | En el router usa `/` y `/:id` |
| El router no responde | Olvidaste `export default router` o `app.use(...)` | Revisa ambos lados |
| Todas las rutas dan 404 | El `app.use` del 404 está **antes** de las rutas | El 404 va al final |

---

## ✅ Resumen rápido

1. Ruta = **método** + **URL** + **handler**.
2. GET leer, POST crear, PUT sustituir, PATCH modificar, DELETE borrar.
3. `Router()` agrupa rutas; se monta con `app.use("/prefijo", router)`.
4. Un archivo de rutas por recurso; `export default router`.
5. `:id` → `req.params.id`.
6. Las rutas se evalúan **en orden**: fijas (`/stats`) antes que variables (`/:id`).
7. `app.use(prefijo, routerA, routerB)` → el primero que responda gana.

👉 **Siguiente:** `03-request-y-response.md`
