# 06 — MONGODB DESDE NODE: EXPRESS + TYPESCRIPT

> Para usar MongoDB desde tu API se usa el **driver oficial** (`npm install mongodb`).
> Los métodos son **los mismos que en `mongosh`** (`find`, `insertOne`, `updateOne`...),
> con dos diferencias: son **asíncronos** (`await`) y `find` devuelve un **cursor**
> que hay que convertir con `.toArray()`.

---

## 6.1 `mongosh` vs driver de Node

| `mongosh` | Driver de Node (TypeScript) |
|---|---|
| `db.products.find({ category: "muebles" })` | `await db.collection("products").find({ category: "muebles" }).toArray()` |
| `db.products.findOne({ _id: ObjectId("...") })` | `await db.collection("products").findOne({ _id: new ObjectId(id) })` |
| `db.products.insertOne({...})` | `await db.collection("products").insertOne({...})` |
| `ObjectId("...")` | `new ObjectId("...")` (importado de `"mongodb"`) |

### Instalación (de tus `package.json`)

```json
  "dependencies": {
    "express": "^5.3.0",
    "mongodb": "^7.8.0"
  }
```

> 📌 El paquete `mongodb` ya trae sus tipos de TypeScript: no hace falta `@types/mongodb`.

Y el script `dev` carga el `.env` con la opción de Node `--env-file`:

```json
"dev": "tsx watch --env-file=.env src/index.ts",
```

---

## 6.2 La conexión: `MongoClient` en `db.ts`

### Ejemplo REAL copiado (`mongo-api-libros/.../02b-express-mongodb/src/db.ts`)

```ts
import { Db, MongoClient } from "mongodb";

const uri = process.env.DATABASE_URL || process.env.MONGODB_URI;
if (!uri) {
  throw new Error("Falta DATABASE_URL o MONGODB_URI en el archivo .env");
}

const client = new MongoClient(uri);

let db: Db;

export async function connectDb() {
  await client.connect();
  db = client.db(process.env.DB_NAME ?? "bootcamp_api");

  // Reto 2: índice único para que no se repita el nombre de una categoría.
  // Si ya existe, no hace nada.
  await db.collection("categories").createIndex({ name: 1 }, { unique: true });

  console.log("Conectado a MongoDB");
}

export function getDb() {
  return db;
}
```

**Las piezas:**

```
new MongoClient(uri)       → crea el cliente (todavía NO conecta)
await client.connect()     → conecta (una sola vez al arrancar)
client.db("bootcamp_api")  → elige la base de datos (como "use" en mongosh)
getDb()                    → para usar la BD desde las rutas
```

> 📌 **Se conecta UNA vez** al arrancar y se reutiliza. El cliente mantiene un
> *pool* de conexiones. ❌ No abras una conexión nueva en cada petición.

> 📌 Si falta la URI, el `throw` hace que la app **falle al arrancar** con un mensaje
> claro, en vez de fallar más tarde con un error raro.

### Conectar ANTES de escuchar (`src/index.ts`)

```ts
async function start() {
  await connectDb();

  app.listen(PORT, () => {
    console.log(`Servidor escuchando en http://localhost:${PORT}`);
  });
}

start();
```

> 📌 Si `app.listen` fuera antes que `connectDb`, podrían llegar peticiones cuando
> `db` todavía es `undefined`.

### Variante (`api-rastro/api-rastro/src/db.ts`): timeout corto + helpers por colección

```ts
const client = new MongoClient(uri, { serverSelectionTimeoutMS: 5000 });

let db: Db;

/**
 * Conecta a la base de datos MongoDB y asigna la instancia de Db.
 */
export async function connectDb() {
  await client.connect();
  db = client.db(process.env.DB_NAME ?? "rastro");
  console.log(`Conectado a MongoDB (base de datos ${db.databaseName})`);
}

export function getDb() {
  return db;
}

export function usersCollection() {
  return getDb().collection<User>("users");
}

export function listingsCollection() {
  return getDb().collection<Listing>("listings");
}

export function offersCollection() {
  return getDb().collection<Offer>("offers");
}
```

> 📌 `serverSelectionTimeoutMS: 5000`: si Mongo está parado, falla a los **5 s**
> (por defecto son 30 s). Tu pregunta 1 de `api-rastro/respuestas.md`: *"Con MongoDB
> parado, `GET /api/health` tarda unos 5 segundos (...) y devuelve **`500`**"*.

### Health check (`api-rastro/api-rastro/src/index.ts`)

```ts
app.get("/api/health", async (_req, res) => {
  await getDb().command({ ping: 1 });
  res.json({ status: "ok", db: getDb().databaseName });
});
```

---

## 6.3 Colecciones tipadas con TypeScript

`collection<Tipo>("nombre")` le dice a TypeScript **qué forma tienen** los documentos:

### Ejemplo REAL copiado (`mongo-api-libros/.../src/routes/products.ts`)

```ts
// Ejercicio 5: categoría en cada producto
export type Product = {
  name: string;
  price: number;
  category: string;
};

export function productsCollection() {
  return getDb().collection<Product>("products");
}
```

> 📌 El tipo **no lleva `_id`**: el driver lo añade solo. Lo que te devuelve `find`
> es `WithId<Product>` = `Product & { _id: ObjectId }`.

### Tipos con `ObjectId` y subdocumentos (`api-rastro/api-rastro/src/types.ts`)

```ts
import type { ObjectId } from "mongodb";

export type ListingStatus = "AVAILABLE" | "RESERVED" | "SOLD";
export type OfferStatus = "PENDING" | "ACCEPTED" | "REJECTED";

export type Listing = {
  title: string;
  description: string;
  price: number;
  category: string;
  status: ListingStatus;
  city: string;
  publishedAt: Date;
  views: number;
  seller: { id: ObjectId; name: string };
  photos: { url: string; order: number }[];
  attributes: Record<string, unknown>;
};
```

> 📌 `status: "AVAILABLE" | "RESERVED" | "SOLD"` → TypeScript no te deja escribir
> `status: "VENDIDO"` por error. `attributes: Record<string, unknown>` → objeto libre
> (el esquema flexible de Mongo).

---

## 6.4 Leer: `find` con filtros desde `req.query`

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

**La idea:** se empieza con un filtro vacío `{}` (= todos) y se le van **añadiendo
campos** solo si llegan en la URL.

```
GET /api/products                          → filter = {}
GET /api/products?category=muebles         → filter = { category: "muebles" }
GET /api/products?category=muebles&search=sil → filter = { category: "muebles", name: { $regex: "sil", $options: "i" } }
```

### Con `Filter<T>`, proyección y orden (`api-rastro/.../src/routes/listings.ts`)

```ts
const LIST_FIELDS = { title: 1, price: 1, category: 1, city: 1, status: 1 };

/**
 * Obtiene el listado de anuncios con filtros opcionales.
 */
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

> 📌 `Filter<Listing>` (importado de `"mongodb"`) es el tipo de un filtro válido para
> `Listing`: TypeScript te avisa si pones un campo que no existe.

> 📌 En el driver, la proyección se pone con `.project({...})` encadenado, o como
> opción: `findOne(filtro, { projection: { _id: 1 } })`.

---

## 6.5 Ids: de texto (`req.params.id`) a `ObjectId`

El id llega en la URL **como texto**. Hay que:

1. Comprobar que tiene formato válido → si no, **400**.
2. Convertirlo con `new ObjectId(texto)`.
3. Si no se encuentra → **404**.

### Ejemplo REAL copiado (`mongo-api-libros/.../src/routes/products.ts`)

```ts
router.get("/:id", async (req, res) => {
  if (!ObjectId.isValid(req.params.id)) {
    res.status(400).json({ error: "El ID no tiene un formato válido" });
    return;
  }

  const product = await productsCollection().findOne({ _id: new ObjectId(req.params.id) });

  if (!product) {
    res.status(404).json({ error: "Producto no encontrado" });
    return;
  }

  res.json(product);
});
```

```ts
// ❌ Sin comprobar: new ObjectId("abc") LANZA un error → 500
const product = await productsCollection().findOne({ _id: new ObjectId(req.params.id) });

// ❌ Sin convertir: compara texto con ObjectId → siempre null → 404 aunque exista
const product = await productsCollection().findOne({ _id: req.params.id });

// ✅ isValid → 400 · new ObjectId → buscar · null → 404
```

De la tabla de pruebas del `README.md`:

```bash
curl -i http://localhost:3000/api/products/abc                      # 400: formato incorrecto
curl -i http://localhost:3000/api/products/66f000000000000000000000 # 404: no existe
```

| Situación | Código |
|---|---|
| Id con formato raro (`abc`) | **400** Bad Request |
| Id válido pero no existe | **404** Not Found |
| Existe | **200** OK |

---

## 6.6 Crear: `insertOne` + `insertedId`

### Ejemplo REAL copiado (`mongo-api-libros/.../src/routes/products.ts`)

```ts
router.post("/", async (req, res) => {
  const body = req.body ?? {};
  const error = validateProduct(body, false);

  if (error) {
    res.status(400).json({ error });
    return;
  }

  const newProduct: Product = {
    name: body.name.trim(),
    price: body.price,
    // Ejercicio 5: si no llega categoría, "sin-categoria"
    category: body.category?.trim() ?? "sin-categoria"
  };

  const result = await productsCollection().insertOne(newProduct);

  res.status(201).json({ _id: result.insertedId, ...newProduct });
});
```

> 📌 `insertOne` devuelve `{ acknowledged, insertedId }`, **no** el documento. Por
> eso se responde `{ _id: result.insertedId, ...newProduct }` con **201 Created**.

> 📌 **Nunca** metas `req.body` directamente en `insertOne`: alguien podría mandar
> campos extra. Construye el objeto (`newProduct`) solo con lo que has validado.

### Validar antes de guardar

Mongo acepta cualquier cosa (ver `02`, apartado 2.7), así que se valida en la API:

```ts
  if (price !== undefined || !partial) {
    if (typeof price !== "number" || !Number.isFinite(price) || price < 0) {
      return "El precio debe ser un número igual o mayor que cero";
    }
  }
```

Y los campos que decide el servidor se ponen **en el servidor**
(`api-rastro/.../src/routes/listings.ts`):

```ts
  const newListing: Listing = {
    title: body.title,
    description: body.description,
    price: body.price,
    category: body.category,
    status: "AVAILABLE",
    city: body.city,
    publishedAt: new Date(),
    views: 0,
    seller: { id: seller._id, name: seller.name },
    photos: [],
    attributes: body.attributes ?? {},
  };
```

> 📌 `status`, `publishedAt` y `views` **no** salen del body. Tu pregunta 3 de
> `api-rastro/respuestas.md`: *"cualquiera podría inflar sus visitas (`views: 9999`)"*.

---

## 6.7 Actualizar: `findOneAndUpdate` vs `updateOne`

### Opción 1 — `findOneAndUpdate` (actualiza y devuelve)

**Ejemplo REAL copiado (`mongo-api-libros/.../src/routes/products.ts`):**

```ts
  // Solo los campos que han llegado
  const changes: Partial<Product> = {};
  if (body.name !== undefined) changes.name = body.name.trim();
  if (body.price !== undefined) changes.price = body.price;
  if (body.category !== undefined) changes.category = body.category.trim();

  const updated = await productsCollection().findOneAndUpdate(
    { _id: new ObjectId(req.params.id) },
    { $set: changes },
    { returnDocument: "after" }
  );

  if (!updated) {
    res.status(404).json({ error: "Producto no encontrado" });
    return;
  }

  res.json(updated);
```

> 📌 `Partial<Product>` = todos los campos opcionales → perfecto para un **PATCH**
> (solo cambias lo que llega). `{ returnDocument: "after" }` = devuelve el documento
> **ya cambiado**. Si no existe, devuelve `null` → 404.

### Opción 2 — `updateOne` + `matchedCount` (ejercicio 7)

**Ejemplo REAL copiado (`mongo-api-libros/.../src/soluciones/ejercicio-07-patch-updateOne.ts`):**

```ts
  const id = new ObjectId(req.params.id);

  // updateOne NO devuelve el documento: devuelve { matchedCount, modifiedCount, ... }
  const result = await productsCollection().updateOne({ _id: id }, { $set: changes });

  // matchedCount = cuántos documentos cumplían el filtro. 0 → no existe.
  if (result.matchedCount === 0) {
    res.status(404).json({ error: "Producto no encontrado" });
    return;
  }

  // Para devolver el producto modificado hace falta una segunda consulta.
  // findOneAndUpdate hace las dos cosas en una sola operación.
  const updated = await productsCollection().findOne({ _id: id });
  res.json(updated);
```

| | `findOneAndUpdate` | `updateOne` |
|---|---|---|
| Devuelve | El documento (o `null`) | `{ matchedCount, modifiedCount }` |
| ¿Existe? | `if (!updated)` | `if (result.matchedCount === 0)` |
| Consultas para responder el doc | 1 | 2 (`updateOne` + `findOne`) |

### `$inc` en la API: sumar una visita al ver el anuncio (`api-rastro`)

```ts
  const listing = await listingsCollection().findOneAndUpdate(
    { _id: new ObjectId(id) },
    { $inc: { views: 1 } },
    { returnDocument: "after" },
  );
```

### Filtro con condición de estado (evitar "carreras") — `api-rastro/.../offers.ts`

```ts
  const acceptedOffer = await offersCollection().findOneAndUpdate(
    { _id: offerId, status: "PENDING" },
    { $set: { status: "ACCEPTED" } },
    { returnDocument: "after" },
  );
  if (!acceptedOffer) {
    res.status(409).json({ error: "No se puede aceptar: la oferta ya no está PENDING" });
    return;
  }
```

> 📌 Poner `status: "PENDING"` **en el filtro** hace que, si dos peticiones intentan
> aceptar la misma oferta a la vez, solo una lo consiga (la otra recibe `null` → 409).

---

## 6.8 Borrar: `deleteOne` + `deletedCount`

### Ejemplo REAL copiado (`mongo-api-libros/.../src/routes/products.ts`)

```ts
router.delete("/:id", async (req, res) => {
  if (!ObjectId.isValid(req.params.id)) {
    res.status(400).json({ error: "El ID no tiene un formato válido" });
    return;
  }

  const result = await productsCollection().deleteOne({ _id: new ObjectId(req.params.id) });

  if (result.deletedCount === 0) {
    res.status(404).json({ error: "Producto no encontrado" });
    return;
  }

  res.status(204).send();
});
```

> 📌 **204 No Content** = borrado correcto, sin cuerpo.

---

## 6.9 Índice único → error 11000 → 409 Conflict

### Ejemplo REAL copiado (`mongo-api-libros/.../src/routes/categories.ts`)

```ts
import { MongoServerError, ObjectId } from "mongodb";

// ¿Es el error de "clave duplicada" del índice único?
function isDuplicateKey(err: unknown): boolean {
  return err instanceof MongoServerError && err.code === 11000;
}
```

```ts
  try {
    const result = await categoriesCollection().insertOne(newCategory);
    res.status(201).json({ _id: result.insertedId, ...newCategory });
  } catch (err) {
    if (isDuplicateKey(err)) {
      res.status(409).json({ error: "Ya existe una categoría con ese nombre" });
      return;
    }
    throw err; // cualquier otro error, al middleware de errores (500)
  }
```

```bash
curl -i -X POST http://localhost:3000/api/categories -H "Content-Type: application/json" -d '{"name":"cocina"}'   # 201
curl -i -X POST http://localhost:3000/api/categories -H "Content-Type: application/json" -d '{"name":"cocina"}'   # 409
```

> 📌 `throw err` en el `catch`: si **no** es un duplicado, no te lo tragues; deja que
> lo maneje el middleware de errores (500).

---

## 6.10 Devolver `id` en vez de `_id`

Muchas APIs prefieren `"id": "665f..."` (texto) en vez de `"_id"`.

### Ejemplo REAL copiado (`mongo-api-libros/.../src/soluciones/reto-1-products-id.ts`)

```ts
import type { WithId } from "mongodb";

// Convierte un documento de MongoDB en lo que devuelve la API
function toApi(document: WithId<Product>) {
  const { _id, ...rest } = document;
  return { id: _id.toHexString(), ...rest };
}
```

```ts
  const products = await productsCollection().find(filter).toArray();
  res.json(products.map(toApi));
```

```
{ _id: ObjectId("665f..."), name: "Teclado", ... }   ← en Mongo (no cambia)
{ id: "665f...", name: "Teclado", ... }              ← lo que ve el cliente
```

> 📌 `toHexString()` convierte el `ObjectId` en los 24 caracteres de texto. Al
> responder con `res.json`, un `ObjectId` también se convierte a texto solo.

---

## 6.11 Errores de Mongo y el middleware de errores

Express 5 pasa automáticamente al middleware de errores cualquier excepción de una
ruta `async` (por ejemplo, si Mongo está caído):

```ts
// 6. Ha ocurrido un error: respuesta JSON sin detalles internos
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  if (err instanceof SyntaxError) {
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }

  console.error(err);
  res.status(500).json({ error: "Error interno del servidor" });
});
```

*(copiado de `mongo-api-libros/.../02b-express-mongodb/src/index.ts`)*

### Resumen de códigos HTTP con Mongo

| Código | Cuándo |
|---|---|
| 200 | Lectura / actualización correcta |
| 201 | `insertOne` correcto |
| 204 | `deleteOne` correcto |
| 400 | Body inválido, id con formato incorrecto, referencia que no existe (`sellerId`) |
| 404 | `findOne` → `null`, `matchedCount`/`deletedCount` = 0 |
| 409 | Duplicado (11000) o conflicto con el estado (anuncio ya `SOLD`) |
| 500 | Mongo caído u otro error inesperado |

---

## 6.12 Mongoose (mención breve)

Tus proyectos usan el **driver nativo** (el `README.md` de 02b lo dice: *"con el
driver oficial (sin Mongoose)"*). **Mongoose** es una librería por encima del driver
que añade **esquemas** y validación. Ejemplo mío:

```ts
import mongoose from "mongoose";

await mongoose.connect(process.env.MONGODB_URI!);

const productSchema = new mongoose.Schema({
  name: { type: String, required: true, trim: true },
  price: { type: Number, required: true, min: 0 },
  category: { type: String, default: "sin-categoria" }
}, { timestamps: true });

const Product = mongoose.model("Product", productSchema);   // colección "products"

const nuevo = await Product.create({ name: "Monitor", price: 149.99 });  // valida solo
const baratos = await Product.find({ price: { $lt: 30 } });
const uno = await Product.findById(id);       // convierte el texto a ObjectId por ti
```

| Driver nativo (`mongodb`) | Mongoose |
|---|---|
| Ligero, mismos métodos que `mongosh` | Esquemas, validación, `default`, `populate` (≈ `$lookup`) |
| Validas tú a mano | Valida el esquema |
| `new ObjectId(id)` a mano | `findById(id)` |
| Lo que has usado en clase | Muy común en tutoriales y empresas |

---

## ✅ Resumen rápido

1. `npm i mongodb` → `new MongoClient(uri)` → `await client.connect()` **una vez** → `client.db(nombre)`.
2. Conecta **antes** de `app.listen`. URI y nombre de BD desde `.env`.
3. `collection<Tipo>("nombre")` para tener tipos; `Filter<T>`, `WithId<T>`, `Partial<T>`.
4. `find(...).toArray()`; `findOne` → documento o `null`.
5. Ids: `ObjectId.isValid` (400) → `new ObjectId(id)` → `null` (404).
6. `insertOne` → `insertedId` (201); `findOneAndUpdate` + `returnDocument: "after"`; `updateOne` → `matchedCount`; `deleteOne` → `deletedCount` (204).
7. Índice único → `MongoServerError` con `code === 11000` → 409.
8. Valida en la API: Mongo acepta cualquier tipo y no comprueba referencias.

👉 **Siguiente:** `07-ejemplos-completos-de-proyectos.md`
