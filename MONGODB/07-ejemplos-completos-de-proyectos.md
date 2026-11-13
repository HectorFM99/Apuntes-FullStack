# 07 — EJEMPLOS COMPLETOS COPIADOS DE TUS PROYECTOS

> 📁 **Los archivos de los proyectos NO se han modificado.** Todo lo de aquí abajo son
> **copias** de su contenido original, comentadas para estudiar.
> 🔒 No se copia ningún `.env` (solo `compose.yaml`, que usa la contraseña de clase).

---

## EJEMPLO 1 — MongoDB en Docker

**Origen:** `practica-MongoDB/compose.yaml` *(copiado íntegro)*

```yaml
services:
  mongodb:
    image: mongo:8
    container_name: mongodb-practica
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin_dev_123
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    restart: unless-stopped

volumes:
  mongodb_data:
```

**Qué enseña:**
- Imagen oficial `mongo:8` en el puerto por defecto **27017**.
- Usuario `admin` creado la primera vez gracias a `MONGO_INITDB_ROOT_*`.
- Volumen `mongodb_data` → los datos sobreviven a `docker compose down` (pero **no** a `down -v`).

```bash
docker compose up -d
docker exec -it mongodb-practica mongosh -u admin -p admin_dev_123 --authenticationDatabase admin
```

---

## EJEMPLO 2 — Script de datos de prueba (`seed`)

**Origen:** `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/mongo/seed.js` *(copiado íntegro)*

```js
// Datos de prueba para 02b. Se ejecuta con mongosh (ver README).
// Borra lo que hubiera y vuelve a empezar: se puede lanzar las veces que quieras.
/* global db, print */ // variables que pone mongosh (para que ESLint no se queje)

const database = db.getSiblingDB("bootcamp_api");

database.products.deleteMany({});
database.products.insertMany([
  { name: "Teclado", price: 29.99, category: "perifericos" },
  { name: "Ratón", price: 15, category: "perifericos" },
  { name: "Silla", price: 49.5, category: "muebles" }
]);

// Reto 2: categorías con nombre único
database.categories.deleteMany({});
database.categories.createIndex({ name: 1 }, { unique: true });
database.categories.insertMany([{ name: "perifericos" }, { name: "muebles" }]);

print(`products: ${database.products.countDocuments()} · categories: ${database.categories.countDocuments()}`);
```

**Qué enseña:**
- `db.getSiblingDB("bootcamp_api")` = el `use bootcamp_api` de los scripts.
- `deleteMany({})` + `insertMany([...])` → script **repetible**.
- `createIndex(..., { unique: true })` antes de insertar → nombres de categoría únicos.
- Se lanza con `npm run seed` (copia el archivo al contenedor y lo ejecuta con `mongosh`).

---

## EJEMPLO 3 — Consultas del Rastro en `mongosh` (archivo entero)

**Origen:** `mongo-rastro/mongo-rastro/rastro.js` *(copiado íntegro)*

```js
// use rastro

// Ejercicio B2 
db.anuncios.insertOne({
  titulo: "Bicicleta de montaña Orbea",
  descripcion: "Revisada en taller.",
  precio: 300,
  categoria: "Deporte",
  estado: "DISPONIBLE",
  ciudad: "Castellón",
  fechaPublicacion: new Date("2026-10-27"),
  visitas: 0,
  vendedor: {
    id: ObjectId("660000000000000000000004"),
    nombre: "Hugo Sanz"
  },
  fotos: [{ url: "https://img.rastro.es/bici-1.jpg", orden: 1 }],
  atributos: { talla: "L", tipo: "montaña", pulgadasRueda: 29 }
})


db.anuncios.insertOne({
  titulo: "Google Pixel 7",
  descripcion: "Libre, con caja.",
  precio: 260,
  categoria: "Móviles",
  estado: "DISPONIBLE",
  ciudad: "Valencia",
  fechaPublicacion: new Date("2026-10-28"),
  visitas: 0,
  vendedor: {
    id: ObjectId("660000000000000000000005"),
    nombre: "Sara López"
  },
  fotos: [
    { url: "https://img.rastro.es/pixel-1.jpg", orden: 1 },
    { url: "https://img.rastro.es/pixel-2.jpg", orden: 2 }
  ],
  atributos: { marca: "Google", modelo: "Pixel 7", almacenamientoGB: 128, bateriaPct: 92 }
})

// Ejercicio B3 — Índices

db.usuarios.createIndex({ email: 1 }, { unique: true })


db.valoraciones.createIndex(
  { "autor.id": 1, anuncioId: 1 },
  { unique: true }
)


db.anuncios.createIndex({ categoria: 1, precio: 1 })


db.usuarios.getIndexes()
db.valoraciones.getIndexes()
db.anuncios.getIndexes()


db.anuncios
  .find({ categoria: "Coches" })
  .sort({ precio: 1 })
  .explain("executionStats")

// Ejercicio B4

db.usuarios.insertOne({
  nombre: "Ana Copia",
  email: "ana@rastro.es",
  ciudad: "Valencia",
  fechaAlta: new Date(),
  favoritos: [],
  valoracionMedia: 0,
  numValoraciones: 0
})

// B4.2
db.valoraciones.insertOne({
  anuncioId: ObjectId("670000000000000000000004"),
  autor: { id: ObjectId("660000000000000000000004"), nombre: "Hugo Sanz" },
  valoradoId: ObjectId("660000000000000000000003"),
  estrellas: 1,
  comentario: "Cambio de opinión",
  fecha: new Date()
})

// B4.3
db.anuncios.insertOne({
  titulo: "Prueba fantasma",
  precio: 10,
  categoria: "Libros",
  estado: "DISPONIBLE",
  ciudad: "Valencia",
  fechaPublicacion: new Date(),
  visitas: 0,
  vendedor: { id: ObjectId("660000000000000000000099"), nombre: "Nadie" },
  fotos: [],
  atributos: {}
})

// B4.4
db.anuncios.insertOne({
  titulo: "Prueba lámpara",
  precio: "75",
  categoria: "Muebles",
  estado: "DISPONIBLE",
  ciudad: "Valencia",
  fechaPublicacion: new Date(),
  visitas: 0,
  vendedor: { id: ObjectId("660000000000000000000001"), nombre: "Ana García" },
  fotos: [],
  atributos: { material: "metal" }
})


db.anuncios.find(
  { precio: { $lt: 100 } },
  { titulo: 1, precio: 1, _id: 0 }
)

// B4.5
db.anuncios.updateOne(
  { _id: ObjectId("670000000000000000000005") },
  { precio: 80 }
)

// B4.6
db.anuncios.findOne({ _id: "670000000000000000000003" })


db.anuncios.deleteMany({ titulo: /^Prueba/ })

// Ejercicio B5
db.anuncios.find(
  { estado: "DISPONIBLE", ciudad: "Valencia" },
  { titulo: 1, precio: 1, _id: 0 }
).sort({ precio: 1 })

// B5.2
db.anuncios.find({ categoria: "Coches", "atributos.km": { $lt: 100000 } })

// B5.3
db.anuncios.find({ categoria: "Móviles", "atributos.almacenamientoGB": { $gte: 128 } })

// B5.4
db.anuncios.find({ precio: { $gte: 50, $lte: 400 } })

// B5.5
db.anuncios.find({ titulo: /nike/i })

// B5.6
db.anuncios.find({ "vendedor.id": ObjectId("660000000000000000000002") })

// B5.7
db.anuncios.find({ fotos: { $size: 0 } })

// B5.8
db.ofertas.find({ "anuncio.id": ObjectId("670000000000000000000001"), estado: "PENDIENTE" })

// B5.9
db.usuarios.find(
  { favoritos: ObjectId("670000000000000000000008") },
  { nombre: 1, _id: 0 }
)

// B5.10
db.anuncios.countDocuments({ estado: "VENDIDO" })

// B5.11
db.anuncios.find({}, { titulo: 1, visitas: 1, _id: 0 }).sort({ visitas: -1 }).limit(3)

// Ejercicio B6

// B6.1
db.anuncios.aggregate([
  { $group: { _id: "$categoria", cantidad: { $sum: 1 } } },
  { $sort: { cantidad: -1 } }
])

// B6.2
db.anuncios.aggregate([
  { $group: { _id: "$categoria", precioMedio: { $avg: "$precio" } } },
  { $sort: { _id: 1 } }
])

// B6.3
db.anuncios.aggregate([
  { $group: { _id: "$ciudad", cantidad: { $sum: 1 } } },
  { $sort: { cantidad: -1 } }
])

// B6.4
db.anuncios.aggregate([
  { $match: { estado: "DISPONIBLE" } },
  {
    $group: {
      _id: "$vendedor.nombre",
      valorEnVenta: { $sum: "$precio" }
    }
  },
  { $sort: { valorEnVenta: -1 } }
])

// B6.5
db.anuncios.aggregate([
  { $group: { _id: "$categoria", visitasTotales: { $sum: "$visitas" } } },
  { $sort: { visitasTotales: -1 } },
  { $limit: 1 }
])

// Ejercicio B7

// B7.1
db.anuncios.updateOne(
  { _id: ObjectId("670000000000000000000003") },
  { $push: { fotos: { url: "https://img.rastro.es/a3-4.jpg", orden: 4 } } }
)

// B7.2
db.anuncios.updateOne(
  { _id: ObjectId("670000000000000000000005") },
  { $inc: { visitas: 1 } }
)

// B7.3
db.usuarios.updateOne(
  { _id: ObjectId("660000000000000000000005") },
  { $pull: { favoritos: ObjectId("670000000000000000000003") } }
)
db.usuarios.updateOne(
  { _id: ObjectId("660000000000000000000005") },
  { $addToSet: { favoritos: ObjectId("670000000000000000000002") } }
)


db.usuarios.updateOne(
  { _id: ObjectId("660000000000000000000005") },
  { $addToSet: { favoritos: ObjectId("670000000000000000000002") } }
)

// B7.4
db.ofertas.updateOne(
  { _id: ObjectId("680000000000000000000003") },
  { $set: { estado: "ACEPTADA" } }
)
db.anuncios.updateOne(
  { _id: ObjectId("670000000000000000000001") },
  { $set: { estado: "RESERVADO" } }
)

// B7.5
db.anuncios.updateMany(
  { categoria: "Coches" },
  { $mul: { precio: 0.9 } }
)

// B7.6
db.usuarios.updateOne(
  { _id: ObjectId("660000000000000000000002") },
  { $set: { nombre: "Luis M. Martín" } }
)


db.anuncios.updateMany(
  { "vendedor.id": ObjectId("660000000000000000000002") },
  { $set: { "vendedor.nombre": "Luis M. Martín" } }
)


db.ofertas.updateMany(
  { "comprador.id": ObjectId("660000000000000000000002") },
  { $set: { "comprador.nombre": "Luis M. Martín" } }
)


db.valoraciones.updateMany(
  { "autor.id": ObjectId("660000000000000000000002") },
  { $set: { "autor.nombre": "Luis M. Martín" } }
)
```

**Qué enseña (por ejercicio):**

| Ejercicio | Tema | Apunte |
|---|---|---|
| B2 | `insertOne` con subdocumentos, arrays y `atributos` libres | `02` |
| B3 | Índices simples, compuestos y únicos; `getIndexes`; `explain` | `04` |
| B4 | Lo que Mongo **no** valida: E11000, referencias fantasma, `"75"`, update sin `$set`, `_id` como texto | `02`, `05` |
| B5 | Filtros: igualdad, `$lt`, `$gte`/`$lte`, regex, notación punto, `$size`, arrays, `countDocuments`, `sort`/`limit` | `03` |
| B6 | `aggregate`: `$match`, `$group` (`$sum`, `$avg`), `$sort`, `$limit` | `04` |
| B7 | `$push`, `$inc`, `$pull`, `$addToSet`, `$set`, `$mul`, `updateMany` sobre copias desnormalizadas | `02`, `05` |

---

## EJEMPLO 4 — Conexión a MongoDB (API de productos)

**Origen:** `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/src/db.ts` *(copiado íntegro)*

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

**Qué enseña:**
- Lee la URI del `.env` y falla al arrancar si no está.
- Un único `MongoClient` para toda la app; `connectDb()` se llama en `start()` antes de `app.listen`.
- Crea el índice único al arrancar (idempotente).

---

## EJEMPLO 5 — CRUD completo con índice único (`categories`)

**Origen:** `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/src/routes/categories.ts` *(copiado íntegro)*

```ts
// Reto 2: el CRUD de categorías del apunte 02 (ejercicio 13), ahora en MongoDB.
// El nombre no se repite gracias a un índice único (ver connectDb en src/db.ts
// y mongo/seed.js). Si se repite, el driver lanza un error con code 11000 → 409.
import { Router } from "express";
import { MongoServerError, ObjectId } from "mongodb";
import { getDb } from "../db";

const router = Router();

type Category = {
  name: string;
};

function categoriesCollection() {
  return getDb().collection<Category>("categories");
}

// ¿Es el error de "clave duplicada" del índice único?
function isDuplicateKey(err: unknown): boolean {
  return err instanceof MongoServerError && err.code === 11000;
}

// Devuelve un mensaje de error, o null si el nombre es válido
function validateName(name: unknown): string | null {
  if (typeof name !== "string" || name.trim() === "") {
    return "El nombre debe ser un texto no vacío";
  }
  return null;
}

router.get("/", async (_req, res) => {
  const categories = await categoriesCollection().find().toArray();
  res.json(categories);
});

router.get("/:id", async (req, res) => {
  if (!ObjectId.isValid(req.params.id)) {
    res.status(400).json({ error: "El ID no tiene un formato válido" });
    return;
  }

  const category = await categoriesCollection().findOne({ _id: new ObjectId(req.params.id) });

  if (!category) {
    res.status(404).json({ error: "Categoría no encontrada" });
    return;
  }

  res.json(category);
});

router.post("/", async (req, res) => {
  const { name } = req.body ?? {};
  const error = validateName(name);

  if (error) {
    res.status(400).json({ error });
    return;
  }

  const newCategory: Category = { name: name.trim() };

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
});

router.patch("/:id", async (req, res) => {
  if (!ObjectId.isValid(req.params.id)) {
    res.status(400).json({ error: "El ID no tiene un formato válido" });
    return;
  }

  const { name } = req.body ?? {};

  if (name === undefined) {
    res.status(400).json({ error: "Debes enviar al menos un campo para modificar" });
    return;
  }

  const error = validateName(name);

  if (error) {
    res.status(400).json({ error });
    return;
  }

  try {
    const updated = await categoriesCollection().findOneAndUpdate(
      { _id: new ObjectId(req.params.id) },
      { $set: { name: name.trim() } },
      { returnDocument: "after" }
    );

    if (!updated) {
      res.status(404).json({ error: "Categoría no encontrada" });
      return;
    }

    res.json(updated);
  } catch (err) {
    if (isDuplicateKey(err)) {
      res.status(409).json({ error: "Ya existe una categoría con ese nombre" });
      return;
    }
    throw err;
  }
});

router.delete("/:id", async (req, res) => {
  if (!ObjectId.isValid(req.params.id)) {
    res.status(400).json({ error: "El ID no tiene un formato válido" });
    return;
  }

  const result = await categoriesCollection().deleteOne({ _id: new ObjectId(req.params.id) });

  if (result.deletedCount === 0) {
    res.status(404).json({ error: "Categoría no encontrada" });
    return;
  }

  res.status(204).send();
});

export default router;
```

**Qué enseña:**
- Las 5 rutas CRUD: `find().toArray()`, `findOne`, `insertOne`, `findOneAndUpdate`, `deleteOne`.
- El patrón **400 (id inválido) → 404 (no existe)** en todas las rutas con `:id`.
- `try/catch` + `err.code === 11000` → **409**; cualquier otro error se relanza (`throw err`).

> 📌 `src/routes/products.ts` sigue exactamente el mismo esquema (con filtros
> `category` y `search` con `$regex`); está explicado por partes en el `06`.

---

## EJEMPLO 6 — Tipos y conexión de la API del Rastro

**Origen:** `api-rastro/api-rastro/src/types.ts` *(copiado íntegro)*

```ts
import type { ObjectId } from "mongodb";

export type ListingStatus = "AVAILABLE" | "RESERVED" | "SOLD";
export type OfferStatus = "PENDING" | "ACCEPTED" | "REJECTED";

export type User = {
  name: string;
  email: string;
  city: string;
  joinedAt: Date;
  favorites: ObjectId[];
  ratingAverage: number;
  ratingCount: number;
};

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

export type Offer = {
  amount: number;
  status: OfferStatus;
  createdAt: Date;
  listing: { id: ObjectId; title: string; price: number };
  buyer: { id: ObjectId; name: string };
};
```

**Origen:** `api-rastro/api-rastro/src/db.ts` *(copiado íntegro)*

```ts
import { Db, MongoClient } from "mongodb";
import type { Listing, Offer, User } from "./types";

const uri = process.env.MONGODB_URI;
if (!uri) {
  throw new Error("Falta MONGODB_URI en el archivo .env");
}

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

**Qué enseña:**
- El modelo del `05` pasado a TypeScript: `seller: { id: ObjectId; name: string }` (referencia + copia del nombre), `photos` embebidas, `attributes` libres.
- Una función por colección (`listingsCollection()`...) ya tipada → las rutas no repiten `getDb().collection<...>(...)`.
- `serverSelectionTimeoutMS: 5000` para no esperar 30 s si Mongo está caído.

---

## EJEMPLO 7 — Ofertas: referencias, copias históricas y estados

**Origen:** `api-rastro/api-rastro/src/routes/offers.ts` *(copiado íntegro)*

```ts
import { Router } from "express";
import { ObjectId } from "mongodb";
import { listingsCollection, offersCollection, usersCollection } from "../db";
import type { Offer } from "../types";

const router = Router();

/**
 * Crea una nueva oferta para un anuncio.
 */
router.post("/", async (req, res) => {
  const { listingId, buyerId, amount } = req.body ?? {};

  if (typeof listingId !== "string" || !ObjectId.isValid(listingId)) {
    res.status(400).json({ error: "listingId no tiene un formato válido" });
    return;
  }
  if (typeof buyerId !== "string" || !ObjectId.isValid(buyerId)) {
    res.status(400).json({ error: "buyerId no tiene un formato válido" });
    return;
  }

  if (typeof amount !== "number" || !Number.isFinite(amount) || amount <= 0) {
    res.status(400).json({ error: "amount tiene que ser un número mayor que 0" });
    return;
  }

  const listing = await listingsCollection().findOne({ _id: new ObjectId(listingId) });
  if (!listing) {
    res.status(400).json({ error: "No existe ningún anuncio con ese listingId" });
    return;
  }

  const buyer = await usersCollection().findOne({ _id: new ObjectId(buyerId) });
  if (!buyer) {
    res.status(400).json({ error: "No existe ningún usuario con ese buyerId" });
    return;
  }

  // Validación: el comprador no puede ser el vendedor del anuncio
  if (listing.seller.id.equals(buyer._id)) {
    res.status(400).json({ error: "No puedes ofertar por tu propio anuncio" });
    return;
  }

  if (listing.status !== "AVAILABLE") {
    res.status(409).json({ error: `No se puede ofertar: el anuncio está ${listing.status}` });
    return;
  }

  const newOffer: Offer = {
    amount,
    status: "PENDING",
    createdAt: new Date(),
    listing: { id: listing._id, title: listing.title, price: listing.price },
    buyer: { id: buyer._id, name: buyer.name },
  };

  const result = await offersCollection().insertOne(newOffer);

  res.status(201).json({ _id: result.insertedId, ...newOffer });
});

/**
 * Acepta una oferta pendiente y reserva el anuncio.
 */
router.patch("/:id", async (req, res) => {
  const { id } = req.params;

  if (!ObjectId.isValid(id)) {
    res.status(400).json({ error: "El id no tiene un formato válido" });
    return;
  }

  if (req.body?.status !== "ACCEPTED") {
    res.status(400).json({ error: 'status tiene que ser "ACCEPTED"' });
    return;
  }

  const offerId = new ObjectId(id);

  const offer = await offersCollection().findOne({ _id: offerId });
  if (!offer) {
    res.status(404).json({ error: "No existe ninguna oferta con ese id" });
    return;
  }

  if (offer.status !== "PENDING") {
    res.status(409).json({ error: `No se puede aceptar: la oferta está ${offer.status}` });
    return;
  }

  const listing = await listingsCollection().findOne({ _id: offer.listing.id });
  if (!listing || listing.status !== "AVAILABLE") {
    res.status(409).json({ error: "No se puede aceptar: el anuncio ya no está disponible" });
    return;
  }

  const acceptedOffer = await offersCollection().findOneAndUpdate(
    { _id: offerId, status: "PENDING" },
    { $set: { status: "ACCEPTED" } },
    { returnDocument: "after" },
  );
  if (!acceptedOffer) {
    res.status(409).json({ error: "No se puede aceptar: la oferta ya no está PENDING" });
    return;
  }

  await listingsCollection().updateOne({ _id: offer.listing.id }, { $set: { status: "RESERVED" } });

  res.json(acceptedOffer);
});

export default router;
```

**Qué enseña:**
- **Integridad a mano:** comprueba que `listingId` y `buyerId` existen antes de insertar (Mongo no tiene claves ajenas).
- **`ObjectId.equals()`** para comparar ids (`===` no sirve).
- **Copia histórica:** `listing: { id, title, price }` guarda el precio del momento.
- **409** cuando la petición es correcta pero choca con el **estado** (`status !== "AVAILABLE"`).
- `findOneAndUpdate({ _id, status: "PENDING" }, ...)` → solo acepta si sigue pendiente.
- Dos escrituras seguidas (oferta + anuncio) **no son atómicas juntas** → harían falta transacciones (reto 1 de `respuestas.md`).

---

## 🧭 Mapa: qué archivo mirar para cada concepto

| Concepto | Archivo |
|---|---|
| Docker + Mongo | `practica-MongoDB/compose.yaml` |
| Script de datos | `02b-express-mongodb/mongo/seed.js` |
| Consultas en `mongosh` | `mongo-rastro/rastro.js` |
| Teoría de modelado | `mongo-rastro/respuestas.md` |
| Conexión con el driver | `02b-express-mongodb/src/db.ts`, `api-rastro/src/db.ts` |
| CRUD en Express | `02b-express-mongodb/src/routes/products.ts`, `categories.ts` |
| `updateOne` vs `findOneAndUpdate` | `02b-express-mongodb/src/soluciones/ejercicio-07-patch-updateOne.ts` |
| `_id` → `id` | `02b-express-mongodb/src/soluciones/reto-1-products-id.ts` |
| `aggregate` en la API, `$inc` de visitas | `api-rastro/src/routes/listings.ts` |
| Referencias y estados | `api-rastro/src/routes/offers.ts` |

👉 **Vuelve al índice:** `00-INDICE.md`
