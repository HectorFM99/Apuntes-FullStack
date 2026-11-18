# 03 — REST Y DISEÑO DE APIs

> **REST** (*REpresentational State Transfer*) no es una librería ni un programa:
> es un **conjunto de reglas** para diseñar APIs sobre HTTP de forma ordenada y
> predecible. Una API que las sigue se llama **API REST** o **RESTful**.

---

## 3.1 Los principios de REST (versión sencilla)

| Principio | Qué significa en la práctica |
|-----------|------------------------------|
| **Recursos** | Todo gira alrededor de **cosas** (libros, anuncios, ofertas, usuarios), identificadas por una **URL** |
| **Métodos HTTP como verbos** | La **acción** la dice el método (`GET`, `POST`...), **no** la URL |
| **Representaciones** | El recurso viaja en un formato (casi siempre **JSON**) |
| **Sin estado (*stateless*)** | Cada petición lleva **todo** lo necesario. El servidor no "recuerda" la anterior |
| **Cliente-servidor** | Frontend y backend separados; se comunican solo por HTTP |
| **Interfaz uniforme** | Todas las rutas siguen el mismo patrón → si conoces una, adivinas las demás |
| **Cacheable** | Las respuestas pueden decir si se pueden guardar en caché (`Cache-Control`) |

> 📌 Si alguien ve `GET /api/books/1` y **adivina** que `DELETE /api/books/1`
> borra ese libro sin leer la documentación, tu API es **buena REST**.

---

## 3.2 Nombrar recursos (rutas)

### Reglas de oro

```
✅ /api/books                 ← colección (plural)
✅ /api/books/1               ← un elemento de la colección
✅ /api/listings/:id/offers   ← sub-recurso: las ofertas DE un anuncio
✅ /api/library/favorites     ← en minúscula, palabras con guion si hace falta

❌ /api/getBooks              ← el verbo sobra: ya lo dice GET
❌ /api/book/1                ← mejor plural siempre
❌ /api/deleteBook?id=1       ← verbo en la URL + id en la query
❌ /api/Books/1               ← evita mayúsculas
❌ /api/books/1/delete        ← la acción es el método DELETE
```

| Regla | ❌ | ✅ |
|-------|----|----|
| **Sustantivos**, no verbos | `POST /api/createListing` | `POST /api/listings` |
| **Plural** | `/api/offer/5` | `/api/offers/5` |
| El **id** en la **ruta** | `/api/books?id=1` | `/api/books/1` |
| **Filtros** en la **query** | `/api/books/country/chi` | `/api/books?country=chi` |
| **Relaciones** anidadas (1 nivel) | `/api/offers?listing=7` también vale | `/api/listings/7/offers` |
| Prefijo común | `/books` | `/api/books` (o `/api/v1/books` si versionas) |

### Ejemplo REAL — las rutas de `api-rastro`

Montadas en `api-rastro/api-rastro/src/index.ts`:

```ts
app.use("/api/listings", listingsRouter);
app.use("/api/offers", offersRouter);
```

Y el resultado (de `src/routes/listings.ts` y `src/routes/offers.ts`):

| Método | Ruta | Qué hace |
|--------|------|----------|
| `GET` | `/api/listings` | Lista anuncios (filtros `?category=`, `?city=`, `?maxPrice=`) |
| `GET` | `/api/listings/stats` | Estadísticas por categoría |
| `GET` | `/api/listings/:id` | Un anuncio (y suma una visita) |
| `GET` | `/api/listings/:id/offers` | Las ofertas **de** ese anuncio |
| `POST` | `/api/listings` | Publica un anuncio |
| `PATCH` | `/api/listings/:id` | Cambia el precio |
| `DELETE` | `/api/listings/:id` | Borra el anuncio |
| `POST` | `/api/offers` | Crea una oferta |
| `PATCH` | `/api/offers/:id` | Acepta una oferta (`{ "status": "ACCEPTED" }`) |
| `GET` | `/api/health` | ¿Está viva la API y la base de datos? |

> 📌 Fíjate en `PATCH /api/offers/:id` con `{ "status": "ACCEPTED" }`: en vez de
> inventar `POST /api/offers/:id/accept`, se **modifica el estado del recurso**.
> Eso es pensar en REST.

### Ejemplo REAL — las rutas de `ProyectoReproductor`

Sacadas de `ProyectoReproductor/dist/server/index.js`:

| Método | Ruta | Qué hace |
|--------|------|----------|
| `POST` | `/api/auth/register` | Crear cuenta |
| `POST` | `/api/auth/login` | Iniciar sesión |
| `POST` | `/api/auth/logout` | Cerrar sesión |
| `GET` | `/api/auth/me` | ¿Quién soy? (usuario de la sesión) |
| `GET` | `/api/tracks/search?q=` | Buscar canciones |
| `GET` | `/api/tracks/featured` | Canciones destacadas |
| `GET` | `/api/library/:kind` | `favorites` o `history` |
| `POST` | `/api/library/favorites/:trackId` | Guardar en favoritos |
| `DELETE` | `/api/library/favorites/:trackId` | Quitar de favoritos |
| `POST` | `/api/library/history` | Añadir al historial |

> 📌 Las rutas de `auth` (`/login`, `/logout`) **sí** usan verbos. Es una excepción
> aceptada en casi todas las APIs: login/logout son **acciones**, no recursos.
> Si quisieras ser 100% REST: `POST /api/sessions` (login) y
> `DELETE /api/sessions` (logout).

---

## 3.3 Parámetros de ruta vs parámetros de query vs body

| Dónde | Sintaxis | Para qué | En Express se lee con |
|-------|----------|----------|-----------------------|
| **Ruta** (*path param*) | `/api/books/:id` → `/api/books/7` | **Identificar** UN recurso | `req.params.id` |
| **Query** | `/api/books?country=chi&limit=5` | **Filtrar, ordenar, paginar** | `req.query.country` |
| **Body** | JSON en el cuerpo | **Datos** para crear/modificar | `req.body` |

### Ejemplo REAL copiado — query params (`api-rastro/.../listings.ts`)

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
```

Desde el cliente:

```js
const res = await fetch("http://localhost:3000/api/listings?city=Madrid&maxPrice=400");
```

### Ejemplo REAL copiado — path param + body (`ProyectoReproductor/public/app.js`)

```js
await request(`/api/library/favorites/${track.id}`, { method: 'POST', body: JSON.stringify({ track }) });
```

- `track.id` va en la **ruta** → identifica **qué** canción.
- `{ track }` va en el **body** → los **datos** de la canción para guardarla.

---

## 3.4 El orden de las rutas importa: fijas antes que `/:id`

Tu reto 3 de `api-rastro/api-rastro/respuestas.md`:

> Escrita **después** de `router.get("/:id", ...)`, `GET /api/listings/stats`
> devuelve **`400`** (`El id no tiene un formato válido`). Express prueba las rutas
> en el orden en que se registran, y `/:id` encaja con cualquier texto, así que toma
> `"stats"` como id [...]. Se arregla registrando `/stats` **antes** de `/:id`: las
> rutas fijas tienen que ir antes que las que tienen parámetros.

```ts
// ❌
router.get("/:id", ...);     // "stats" entra aquí como si fuera un id
router.get("/stats", ...);   // nunca se alcanza

// ✅ (así está en tu listings.ts)
router.get("/stats", ...);
router.get("/:id", ...);
```

---

## 3.5 Qué decide el cliente y qué decide el servidor

Un buen diseño **no se fía** de todo lo que manda el cliente. Pregunta 3 de tu
`respuestas.md`:

> Porque esos campos los decide el servidor, no el cliente: un anuncio nuevo siempre
> empieza `AVAILABLE` y con 0 visitas. Si la API los aceptara, cualquiera podría
> inflar sus visitas (`views: 9999`) para salir como anuncio popular, o publicarlo
> directamente como `SOLD`/`RESERVED`, falseando los datos que ven los demás usuarios.

### Ejemplo REAL copiado (`api-rastro/.../listings.ts`, `POST /`)

```ts
const newListing: Listing = {
  title: body.title,
  description: body.description,
  price: body.price,
  category: body.category,
  status: "AVAILABLE",          // ← lo pone el servidor
  city: body.city,
  publishedAt: new Date(),      // ← lo pone el servidor
  views: 0,                     // ← lo pone el servidor
  seller: { id: seller._id, name: seller.name },
  photos: [],
  attributes: body.attributes ?? {},
};

const result = await listingsCollection().insertOne(newListing);

res.status(201).json({ _id: result.insertedId, ...newListing });
```

| Campo | Lo decide | Por qué |
|-------|-----------|---------|
| `id` / `_id` | Servidor / BD | Para que sea único |
| Fechas de creación | Servidor | El reloj del cliente puede mentir |
| `status` inicial, contadores | Servidor | Evitar trampas |
| Datos del formulario (título, precio...) | Cliente | Es lo que el usuario escribe |

> 📌 Lo mismo en tu API de libros: *"El id de la URL manda: si el body trae un id,
> se ignora."* (comentario de `PUT /api/books/:id`).

---

## 3.6 Formato de las respuestas: sé consistente

Elige **una** forma para los errores y úsala en **todas** las rutas.

```json
{ "error": "No existe ningún libro con id 9999" }
```

Y si hay varios errores de validación, tu API de libros devuelve un objeto
`campo → mensaje` (copiado de `books-routes.ts`):

```ts
const errors = validateBook(body);

if (Object.keys(errors).length > 0) {
  res.status(400).json({ errors });
  return;
}
```

Respuesta de ejemplo para un `POST /api/books` con `{ "title": "", "pages": -3 }`:

```json
{
  "errors": {
    "author": "Debe ser un texto no vacío",
    "country": "Debe ser un texto no vacío",
    "imageLink": "Debe ser un texto no vacío",
    "language": "Debe ser un texto no vacío",
    "link": "Debe ser un texto no vacío",
    "title": "Debe ser un texto no vacío",
    "pages": "Debe ser un número entero mayor que 0",
    "year": "Debe ser un número entero no posterior al año actual"
  }
}
```

> 📌 Esa consistencia es la que permite al frontend tener **una sola** función que
> lee los errores. Tu reproductor lo hace: `throw new Error(body.error || 'No se
> pudo completar la solicitud.')` funciona con todas las rutas porque **todas**
> devuelven `{ error: "..." }`.

### Ruta que no existe → también JSON

Copiado de `api-rastro/api-rastro/src/index.ts`:

```ts
app.use((req, res) => {
  res.status(404).json({ error: `No existe la ruta ${req.method} ${req.url}` });
});
```

Sin esto, Express devuelve una página HTML (`Cannot GET /api/xxx`) y el
`response.json()` del cliente fallaría.

---

## 3.7 Datos "copiados" en una relación (foto del momento)

Pregunta 4 de tu `respuestas.md`:

> La copia `listing.price: 380` de la oferta es una **foto del momento** en que Sara
> ofertó [...] El precio actual siempre se puede consultar en el propio anuncio; si
> el `PATCH` cambiara la copia, perderíamos el contexto de la oferta.

```ts
listing: { id: listing._id, title: listing.title, price: listing.price },
buyer: { id: buyer._id, name: buyer.name },
```

→ Al diseñar el JSON de una respuesta piensa: ¿este dato debe **seguir al
original** o **congelarse**?

---

## 3.8 Diseñar una API ANTES de programarla

Pasos que seguiste en `API_Godines/api-express/api-books-claude/`:

1. **Objetivo y usuarios** (PRD).
2. **Funcionalidades** → se convierten en **endpoints**.
3. **Fuera de alcance** (lo que NO harás todavía).
4. **"Hecho cuando..."** → criterios para saber que has terminado.

### Ejemplo REAL copiado (`docs/prd-books.md`, completo)

```md
# PRD: API de libros

## Objetivo
Una API para consultar y gestionar el catálogo de libros de una librería.

## Usuarios
- Clientes: buscan libros.
- Empleados: añaden, editan y borran libros.

## Funcionalidades
1. Listar libros, con filtros por autor, género y precio máximo.
2. Ver un libro por su id.
3. Crear un libro (título, autor y precio obligatorios).
4. Editar y borrar un libro:
   - `PUT /api/books/:id`: sustituye el libro entero (todos los campos obligatorios).
   - `PATCH /api/books/:id`: modifica solo los campos enviados (al menos uno).
   - `DELETE /api/books/:id`: borra el libro y lo devuelve (200).

## Fuera de alcance (por ahora)
- Login y usuarios.
- Pagos.

## Hecho cuando...
- Todos los endpoints responden con los códigos HTTP correctos (200, 201, 400, 404).
- Se puede cambiar de memoria a MongoDB sin tocar las rutas.
```

### Plantilla para documentar CADA endpoint

Tus comentarios JSDoc en `books-routes.ts` ya son documentación. Ejemplo:

```ts
/**
 * POST /api/books
 *
 * Crea un libro. El id lo asigna el almacenamiento (ver BooksStorageInMemory.create).
 * 400 con un error por cada campo no válido.
 *
 * Body: { author, country, imageLink, language, link, title, pages, year }
 */
```

Una ficha completa de endpoint debería tener:

```md
### POST /api/books
Crea un libro.

**Body (JSON):**
| Campo | Tipo | Obligatorio | Regla |
|-------|------|-------------|-------|
| title | string | sí | no vacío |
| pages | number | sí | entero > 0 |
| year  | number | sí | entero, no futuro |

**Respuestas:**
- 201 → el libro creado (con `id`)
- 400 → `{ "errors": { "campo": "mensaje" } }`

**Ejemplo:**
curl -X POST http://localhost:3000/api/books -H "Content-Type: application/json" -d '{ ... }'
```

> 📌 En empresas se escribe en formato **OpenAPI** (antes "Swagger"): un archivo
> YAML/JSON que describe todos los endpoints y genera una web de documentación
> interactiva. La idea es la misma que tu ficha: ruta, método, body, respuestas.

---

## 3.9 Checklist de una buena API REST

- [ ] Rutas con **sustantivos en plural**, sin verbos.
- [ ] El **método** dice la acción.
- [ ] **Ids** en la ruta, **filtros** en la query, **datos** en el body.
- [ ] Rutas fijas (`/stats`) **antes** que `/:id`.
- [ ] Códigos correctos: `201` al crear, `204` sin body, `400`/`404`/`409` bien elegidos.
- [ ] **Siempre JSON**, también en errores y en rutas inexistentes.
- [ ] Formato de error **consistente**.
- [ ] El servidor decide ids, fechas, estados iniciales.
- [ ] Nunca devolver detalles internos en un `500`.
- [ ] Documentado (PRD + ficha por endpoint).
