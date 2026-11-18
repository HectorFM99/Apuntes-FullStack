# 03 — REQUEST Y RESPONSE: LEER DATOS Y RESPONDER

> El cliente te puede mandar información por **tres sitios**: la **URL** (`req.params`),
> la **query string** (`req.query`) y el **cuerpo** (`req.body`). Tú contestas con `res`,
> eligiendo un **código de estado** y un **contenido** (normalmente JSON).

---

## 3.1 Los tres sitios por donde llegan datos

```
PATCH http://localhost:3000/api/books/7?notify=true
      ─────────────────────── ─┬─ ─────┬─────
                               │       └── req.query  → { notify: "true" }
                               └────────── req.params → { id: "7" }

Content-Type: application/json
{ "pages": 210 }               ─────────── req.body   → { pages: 210 }
```

| Dónde | Objeto | Se usa para | Tipo de los valores |
|-------|--------|-------------|---------------------|
| Parte de la ruta (`/:id`) | `req.params` | **Identificar** un recurso concreto | Siempre `string` |
| Después de `?` | `req.query` | **Filtrar, buscar, ordenar, paginar** | `string` (o array si se repite) |
| Cuerpo de la petición | `req.body` | **Enviar datos** para crear/modificar (POST, PUT, PATCH) | Lo que venga en el JSON |

> ⚠️ `req.params` y `req.query` son **SIEMPRE texto**. `/api/books/7` → `req.params.id`
> es `"7"`, no `7`. Hay que convertir con `Number(...)` y comprobar.

---

## 3.2 `req.params` — el id de la URL

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/src/routes/books-routes.ts`)

```ts
router.get("/:id", async (req, res) => {
  const id = parseId(req.params.id);

  if (id === undefined) {
    res.status(400).json({ error: "El id debe ser un número entero" });
    return;
  }

  const book = await storage.findById(id);

  if (!book) {
    sendNotFound(res, id);
    return;
  }

  res.json(book);
});

// ...

// Solo acepta dígitos: rechaza "0x1", "1.0", "1e2", " 1"...
function parseId(raw: string): number | undefined {
  return /^\d+$/.test(raw) ? Number(raw) : undefined;
}
```

**¿Por qué no basta con `Number(req.params.id)`?**

```ts
Number("7")     // 7      ✅
Number("abc")   // NaN    → hay que detectarlo
Number("1e2")   // 100    😱 parece un id válido
Number("0x1")   // 1      😱
Number(" 1")    // 1      😱
```

> 📌 La regex `/^\d+$/` significa "de principio (`^`) a fin (`$`), solo dígitos (`\d+`)".
> Es la forma más estricta de aceptar solo enteros positivos.

### Con MongoDB el id es un `ObjectId` (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
router.get("/:id", async (req, res) => {
  const { id } = req.params;                       // destructuring

  if (!ObjectId.isValid(id)) {
    res.status(400).json({ error: "El id no tiene un formato válido" });
    return;
  }

  const listing = await listingsCollection().findOneAndUpdate(
    { _id: new ObjectId(id) },
    { $inc: { views: 1 } },
    { returnDocument: "after" },
  );

  if (!listing) {
    res.status(404).json({ error: "No existe ningún anuncio con ese id" });
    return;
  }

  res.json(listing);
});
```

---

## 3.3 `req.query` — filtros y búsquedas

Todo lo que va detrás de `?` en la URL, separado por `&`.

```
/api/products?category=muebles&search=sil
req.query = { category: "muebles", search: "sil" }
```

### Ejemplo REAL copiado (`mongo-api-libros/.../src/routes/products.ts`)

```ts
// Ejercicios 5 y 6: GET /api/products?category=muebles&search=sil
router.get("/", async (req, res) => {
  const filter: { category?: string; name?: { $regex: string; $options: string } } = {};

  // Ejercicio 5: filtrar por categoría
  if (typeof req.query.category === "string") {
    filter.category = req.query.category;
  }

  // Ejercicio 6: buscar por nombre sin distinguir mayúsculas
  if (typeof req.query.search === "string") {
    filter.name = { $regex: req.query.search, $options: "i" };
  }

  const products = await productsCollection().find(filter).toArray();
  res.json(products);
});
```

> 📌 ¿Por qué `typeof req.query.category === "string"`? Porque si la URL es
> `?category=a&category=b`, Express devuelve un **array** `["a", "b"]`. Comprobando que es
> string te proteges de valores raros.

### Query numérica con validación (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
router.get("/", async (req, res) => {
  // Construcción de filtros de búsqueda (categoría, ciudad, precio máximo)
  const filter: Filter<Listing> = {};

  if (typeof req.query.category === "string") {
    filter.category = req.query.category;
  }

  if (typeof req.query.city === "string") {
    filter.city = req.query.city;
  }

  if (req.query.maxPrice !== undefined) {
    const maxPrice = typeof req.query.maxPrice === "string" ? Number(req.query.maxPrice) : NaN;
    if (req.query.maxPrice === "" || Number.isNaN(maxPrice)) {
      res.status(400).json({ error: "maxPrice tiene que ser un número" });
      return;
    }
    filter.price = { $lte: maxPrice };
  }

  const listings = await listingsCollection()
    .find(filter)
    .project(LIST_FIELDS)
    .sort({ publishedAt: -1 })
    .toArray();

  res.json(listings);
});
```

```
GET /api/listings?city=Madrid&maxPrice=50     → 200, anuncios de Madrid de 50€ o menos
GET /api/listings?maxPrice=barato             → 400 { error: "maxPrice tiene que ser un número" }
```

### Filtros avanzados en memoria (`api-books-claude/src/routes/books-routes.ts`)

```ts
/**
 * GET /api/books
 *
 * Texto (busca dentro del texto):  ?author=  ?country=  ?language=  ?title=  ?link=  ?imageLink=
 * Números (pages, year):           ?pages=  ?pages_gt=  ?pages_gte=  ?pages_lt=  ?pages_lte=
 *                                  ?year=1958  ?year_gt=  ?year_gte=  ?year_lt=  ?year_lte=
 * Límite de resultados:            ?limit=10
 *
 * Ejemplo: /api/books?country=chi&pages_gte=200&limit=5
 */
router.get("/", async (req, res) => {
  const query: Query = req.query;
  const books = await storage.findAll();

  const filteredBooks = books.filter((book) => matchesAllFilters(book, query));

  res.json(applyLimit(filteredBooks, query.limit));
});
```

Y la función que convierte texto a número con cuidado:

```ts
// Devuelve undefined si el parámetro no viene, está en blanco o no es un número finito
// (Number(" ") sería 0 y Number("Infinity") sería Infinity)
function parseNumber(value: unknown): number | undefined {
  if (typeof value !== "string" || value.trim() === "") return undefined;

  const number = Number(value);
  return Number.isFinite(number) ? number : undefined;
}
```

> 📌 Convención muy usada: `_gt` (mayor que), `_gte` (mayor o igual), `_lt` (menor que),
> `_lte` (menor o igual). Coincide con los operadores de MongoDB `$gt`, `$gte`...

---

## 3.4 `req.body` y `express.json()`

El body llega como **texto**. Para convertirlo en objeto JS necesitas el middleware
`express.json()`:

```ts
app.use(express.json());   // ← SIN esto, req.body es undefined
```

### Ejemplo REAL copiado (`api-books-claude/src/index.ts`)

```ts
// 3. Leer bodies JSON
app.use(express.json());

// 4. Rutas
app.get("/", (_req, res) => {
  res.send("Mi API con Libros express");
});

app.use("/api/books", booksRouter);
```

> ⚠️ `express.json()` tiene que ir **antes** de las rutas. Si lo pones después, cuando la
> ruta se ejecuta el body todavía no se ha leído.

### Con límite de tamaño (`ProyectoReproductor/server/index.ts`)

```ts
app.use(express.json({ limit: '32kb' }));
```

Rechaza bodies de más de 32 KB (por defecto son 100 KB). Protege tu servidor de
peticiones gigantes.

### Leer el body en una ruta (`api-rastro/api-rastro/src/routes/offers.ts`)

```ts
router.post("/", async (req, res) => {
  const { listingId, buyerId, amount } = req.body ?? {};

  if (typeof listingId !== "string" || !ObjectId.isValid(listingId)) {
    res.status(400).json({ error: "listingId no tiene un formato válido" });
    return;
  }
  // ...
});
```

```ts
// ❌ Si la petición no trae body, req.body es undefined y esto PETA:
const { listingId } = req.body;          // TypeError: Cannot destructure 'undefined'

// ✅ ?? {} → si es undefined/null, usa un objeto vacío
const { listingId } = req.body ?? {};    // listingId = undefined → la validación da 400
```

```ts
// También con encadenamiento opcional (api-rastro, PATCH /api/listings/:id)
const price = req.body?.price;
```

> 📌 Para que `express.json()` lea el body, el cliente **debe** mandar la cabecera
> `Content-Type: application/json`. En `fetch`:

```js
fetch("http://localhost:3000/api/books", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ title: "Mi libro", author: "Yo" }),
});
```

---

## 3.5 Otras cosas de `req`

| Propiedad | Qué es | Ejemplo en tus proyectos |
|-----------|--------|--------------------------|
| `req.method` | `"GET"`, `"POST"`... | Logger: `` console.log(`${req.method} ${req.url}`) `` |
| `req.url` | URL completa con query | Logger y mensaje 404 |
| `req.path` | URL **sin** query | Reproductor: `req.path.startsWith('/api/')` |
| `req.headers` | Cabeceras | (no usado directamente) |
| `req.cookies` | Cookies (necesita `cookie-parser`) | Reproductor: `req.cookies?.nightjar_session` |

---

## 3.6 Responder: los métodos de `res`

| Método | Qué hace | Ejemplo |
|--------|----------|---------|
| `res.json(obj)` | Envía JSON (y pone `Content-Type: application/json`) | `res.json(book)` |
| `res.status(código)` | Pone el código HTTP. **No envía**; se encadena | `res.status(201).json(newBook)` |
| `res.send(texto)` | Envía texto/HTML | `res.send("Mi API con Libros express")` |
| `res.sendStatus(código)` | Pone el código **y** envía su texto estándar | `res.sendStatus(204)` |
| `res.end()` | Termina sin cuerpo | `res.status(204).end()` |
| `res.setHeader(nombre, valor)` | Pone una cabecera | CORS |
| `res.cookie(nombre, valor, opciones)` | Crea una cookie | Reproductor, sesión |
| `res.clearCookie(nombre)` | Borra una cookie | Reproductor, logout |
| `res.sendFile(ruta)` | Envía un archivo | Reproductor, `index.html` |

```ts
// ❌ status solo NO responde: la petición se queda colgada
res.status(404);

// ✅ status + json
res.status(404).json({ error: "No encontrado" });

// ✅ json solo → código 200 por defecto
res.json(book);
```

### Ejemplo REAL: 201 al crear (`api-books-claude/src/routes/books-routes.ts`)

```ts
router.post("/", async (req, res) => {
  const body: Query = req.body ?? {};
  const errors = validateBook(body);

  if (Object.keys(errors).length > 0) {
    res.status(400).json({ errors });
    return;
  }

  const newBook = await storage.create(toBookData(body));

  res.status(201).json(newBook);
});
```

### Ejemplo REAL: 204 al borrar (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
router.delete("/:id", async (req, res) => {
  const { id } = req.params;

  if (!ObjectId.isValid(id)) {
    res.status(400).json({ error: "El id no tiene un formato válido" });
    return;
  }

  const result = await listingsCollection().deleteOne({ _id: new ObjectId(id) });

  if (result.deletedCount === 0) {
    res.status(404).json({ error: "No existe ningún anuncio con ese id" });
    return;
  }

  res.status(204).end();
});
```

> 📌 Dos estilos válidos para DELETE:
> - `api-rastro` y `02b` → **204 No Content** (borrado, sin cuerpo).
> - `api-books-claude` → **200** y devuelve el libro borrado (lo pide su PRD).

---

## 3.7 Códigos de estado HTTP que usas

| Código | Nombre | Cuándo | Dónde aparece |
|--------|--------|--------|---------------|
| **200** | OK | Todo bien (por defecto con `res.json`) | Todos los GET, PUT, PATCH |
| **201** | Created | Se ha **creado** algo (POST) | `res.status(201).json(newBook)` |
| **204** | No Content | Bien, pero sin cuerpo (DELETE, OPTIONS, logout) | `res.status(204).end()` |
| **400** | Bad Request | Datos del cliente **inválidos** | `"El id debe ser un número entero"` |
| **401** | Unauthorized | No ha iniciado sesión / credenciales malas | Reproductor: `'Inicia sesión para continuar.'` |
| **404** | Not Found | El recurso o la ruta **no existe** | `"No existe ningún libro con id 7"` |
| **409** | Conflict | La petición choca con el **estado actual** (duplicado, ya vendido...) | `api-rastro`: `"No se puede ofertar: el anuncio está SOLD"` |
| **500** | Internal Server Error | Fallo **tuyo** (bug, BD caída) | Manejador de errores |
| **502** | Bad Gateway | Falla **otra API** de la que dependes | Reproductor: error de Jamendo |
| **503** | Service Unavailable | El servicio no está disponible/configurado | Reproductor: falta `JAMENDO_CLIENT_ID` |

```
2xx → ✅ éxito
4xx → 🙋 culpa del CLIENTE (ha mandado algo mal)
5xx → 💥 culpa del SERVIDOR
```

### 400 vs 409 — Ejemplo REAL (`api-rastro/api-rastro/src/routes/offers.ts`)

```ts
  // Validación: el comprador no puede ser el vendedor del anuncio
  if (listing.seller.id.equals(buyer._id)) {
    res.status(400).json({ error: "No puedes ofertar por tu propio anuncio" });
    return;
  }

  if (listing.status !== "AVAILABLE") {
    res.status(409).json({ error: `No se puede ofertar: el anuncio está ${listing.status}` });
    return;
  }
```

- **400** → la petición **nunca** podría ser válida (ofertar por lo tuyo).
- **409** → la petición sería válida, pero **ahora mismo** choca con el estado (el anuncio
  ya está reservado/vendido).

---

## 3.8 Formato de los errores: sé consistente

Tus APIs siempre devuelven los errores como JSON con una clave `error` (o `errors` si son
varios):

```json
{ "error": "El id no tiene un formato válido" }
```

```json
{
  "errors": {
    "title": "Debe ser un texto no vacío",
    "pages": "Debe ser un número entero mayor que 0"
  }
}
```

> 📌 El frontend puede hacer siempre `data.error` para mostrar el mensaje. Si cada ruta
> devolviera el error de una forma distinta, el frontend sería un caos.

### Helper para no repetir (`api-books-claude/src/routes/books-routes.ts`)

```ts
// Respuesta 404 común a todas las rutas con :id
function sendNotFound(res: Response, id: number): void {
  res.status(404).json({ error: `No existe ningún libro con id ${id}` });
}
```

---

## 3.9 Transformar la respuesta (no devolver los datos "en crudo")

A veces lo que guardas en la BD no es lo que quieres enseñar.

### Ejemplo REAL copiado (`mongo-api-libros/.../src/soluciones/reto-1-products-id.ts`)

```ts
// Convierte un documento de MongoDB en lo que devuelve la API
function toApi(document: WithId<Product>) {
  const { _id, ...rest } = document;
  return { id: _id.toHexString(), ...rest };
}

// ...
  const products = await productsCollection().find(filter).toArray();
  res.json(products.map(toApi));
```

```
MongoDB guarda:  { _id: ObjectId("665f..."), name: "Silla", price: 20 }
La API devuelve: { id: "665f...",            name: "Silla", price: 20 }
```

### Y en el reproductor, adaptando la respuesta de otra API (`ProyectoReproductor/server/index.ts`)

```ts
res.json((payload.results??[]).map(t=>({id:String(t.id),name:t.name,artist:t.artist_name,album:t.album_name,image:t.album_image,audio:t.audio,duration:t.duration})));
```

Coge los campos de Jamendo (`artist_name`, `album_image`...) y los devuelve con nombres
limpios (`artist`, `image`...).

---

## ✅ Resumen rápido

1. `req.params` → `/:id` (siempre string). `req.query` → `?a=1` (strings). `req.body` → JSON.
2. `app.use(express.json())` **antes** de las rutas, o `req.body` será `undefined`.
3. Usa `req.body ?? {}` para no petar si no llega body.
4. `res.status(código).json(datos)`; `res.json()` solo = 200.
5. 201 crear, 204 sin contenido, 400 datos malos, 404 no existe, 409 conflicto, 500 fallo tuyo.
6. Errores siempre con el mismo formato: `{ error: "..." }`.

👉 **Siguiente:** `04-middleware-cors-y-errores.md`
