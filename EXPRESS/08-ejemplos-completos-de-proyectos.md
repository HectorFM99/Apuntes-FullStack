# 08 — EJEMPLOS COMPLETOS COPIADOS DE TUS PROYECTOS

> 📁 **Los archivos de los proyectos NO se han modificado.** Todo lo de aquí abajo son
> **copias** de su contenido original, con explicaciones para estudiar.
> 🔒 No se ha copiado ningún `.env`.

---

## EJEMPLO 1 — `index.ts` de una API completa (la plantilla)

**Origen:** `API_Godines/api-express/api-books-claude/src/index.ts` *(copiado íntegro)*

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import booksRouter from "./routes/books-routes";

const app = express();
const PORT = 3000;

// 1. Registrar cada petición en la terminal
app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// 2. CORS: permitir peticiones desde nuestra página web
app.use((req, res, next) => {
  res.setHeader("Access-Control-Allow-Origin", "http://127.0.0.1:5500");
  res.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, PATCH, DELETE");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type");

  if (req.method === "OPTIONS") {
    res.sendStatus(204);
    return;
  }

  next();
});

// 3. Leer bodies JSON
app.use(express.json());

// 4. Rutas
app.get("/", (_req, res) => {
  res.send("Mi API con Libros express");
});

app.use("/api/books", booksRouter);

// 5. Ninguna ruta ha respondido: 404
app.use((req, res) => {
  res.status(404).json({ error: `No existe la ruta ${req.method} ${req.url}` });
});

// 6. Ha ocurrido un error: respuesta JSON sin detalles internos
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  if (err instanceof SyntaxError) {
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }

  console.error(err);
  res.status(500).json({ error: "Error interno del servidor" });
});

app.listen(PORT, () => {
  console.log(`Servidor escuchando en http://localhost:${PORT}`);
});
```

**Qué enseña:**
- Los **6 pasos numerados** en el orden correcto: logger → CORS → `express.json()` → rutas → 404 → errores.
- CORS hecho a mano, respondiendo `204` al preflight `OPTIONS`.
- Un router montado con prefijo: `app.use("/api/books", booksRouter)`.
- Manejador de errores de **4 parámetros** que distingue JSON mal formado (400) del resto (500).

> 📌 Compáralo con `mongo-api-libros/.../src/index.ts`: es **casi idéntico**. Solo cambia que
> monta dos routers (`/api/products` y `/api/categories`), tiene la ruta `/api/hello` y arranca
> con `async function start() { await connectDb(); app.listen(...) }` porque necesita MongoDB.

---

## EJEMPLO 2 — Router CRUD completo con filtros y validación

**Origen:** `API_Godines/api-express/api-books-claude/src/routes/books-routes.ts` *(copiado íntegro)*

```ts
import { Router } from "express";
import type { Response } from "express";
import type { Book, BookData, BooksStorage } from "../services/books-storage";
import { BooksStorageInMemory } from "../services/books-storage-in-memory";

// Parámetros de la URL; `unknown` obliga a comprobar el valor antes de usarlo
type Query = Record<string, unknown>;

// Las rutas no tocan los datos directamente: siempre a través del contrato BooksStorage.
// Para cambiar de almacenamiento (p. ej. MongoDB), solo hay que cambiar esta línea.
const storage: BooksStorage = new BooksStorageInMemory();

// Campos de texto: obligatorios al crear o sustituir un libro, opcionales al modificarlo (PATCH)
const TEXT_FIELDS = ["author", "country", "imageLink", "language", "link", "title"] as const;

// Todos los campos que se pueden editar (el id nunca)
const EDITABLE_FIELDS = [...TEXT_FIELDS, "pages", "year"] as const;

const router = Router();

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

/**
 * GET /api/books/:id
 *
 * Devuelve el libro con ese id.
 * 400 si el id no es un número entero, 404 si no existe.
 *
 * Ejemplo: /api/books/1
 */
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

/**
 * POST /api/books
 *
 * Crea un libro. El id lo asigna el almacenamiento (ver BooksStorageInMemory.create).
 * 400 con un error por cada campo no válido.
 *
 * Body: { author, country, imageLink, language, link, title, pages, year }
 */
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

/**
 * PUT /api/books/:id
 *
 * Sustituye por completo el libro con ese id (hay que enviar todos los campos).
 * El id de la URL manda: si el body trae un id, se ignora.
 * 400 si el id no es un número entero o algún campo no es válido, 404 si no existe.
 *
 * Body: { author, country, imageLink, language, link, title, pages, year }
 */
router.put("/:id", async (req, res) => {
  const id = parseId(req.params.id);

  if (id === undefined) {
    res.status(400).json({ error: "El id debe ser un número entero" });
    return;
  }

  // Se comprueba antes de validar el body: un id inexistente da 404 aunque el body sea inválido
  if (!(await storage.findById(id))) {
    sendNotFound(res, id);
    return;
  }

  const body: Query = req.body ?? {};
  const errors = validateBook(body);

  if (Object.keys(errors).length > 0) {
    res.status(400).json({ errors });
    return;
  }

  const updatedBook = await storage.replace(id, toBookData(body));

  // Por si se borró entre la comprobación y la escritura (posible con una base de datos)
  if (!updatedBook) {
    sendNotFound(res, id);
    return;
  }

  res.json(updatedBook);
});

/**
 * PATCH /api/books/:id
 *
 * Modifica solo los campos que se envían; el resto del libro no cambia.
 * Cada campo enviado se valida con las mismas reglas que en POST y PUT.
 * El id de la URL manda: si el body trae un id, se ignora. Los campos desconocidos también se ignoran.
 * 400 si el id no es un número entero, no se envía ningún campo editable o alguno no es válido.
 * 404 si no existe.
 *
 * Body (todos opcionales, al menos uno): { author, country, imageLink, language, link, title, pages, year }
 *
 * Ejemplo: PATCH /api/books/1  { "pages": 210 }
 */
router.patch("/:id", async (req, res) => {
  const id = parseId(req.params.id);

  if (id === undefined) {
    res.status(400).json({ error: "El id debe ser un número entero" });
    return;
  }

  // Se comprueba antes de validar el body: un id inexistente da 404 aunque el body sea inválido
  if (!(await storage.findById(id))) {
    sendNotFound(res, id);
    return;
  }

  const body: Query = req.body ?? {};

  if (!EDITABLE_FIELDS.some((field) => body[field] !== undefined)) {
    res.status(400).json({
      error: `Hay que enviar al menos un campo: ${EDITABLE_FIELDS.join(", ")}`,
    });
    return;
  }

  const errors = validateBook(body, { partial: true });

  if (Object.keys(errors).length > 0) {
    res.status(400).json({ errors });
    return;
  }

  const patchedBook = await storage.update(id, toPartialBookData(body));

  if (!patchedBook) {
    sendNotFound(res, id);
    return;
  }

  res.json(patchedBook);
});

/**
 * DELETE /api/books/:id
 *
 * Borra el libro con ese id y lo devuelve en la respuesta (200).
 * 400 si el id no es un número entero, 404 si no existe.
 *
 * Ejemplo: DELETE /api/books/1
 */
router.delete("/:id", async (req, res) => {
  const id = parseId(req.params.id);

  if (id === undefined) {
    res.status(400).json({ error: "El id debe ser un número entero" });
    return;
  }

  const deletedBook = await storage.remove(id);

  if (!deletedBook) {
    sendNotFound(res, id);
    return;
  }

  res.json(deletedBook);
});

// Respuesta 404 común a todas las rutas con :id
function sendNotFound(res: Response, id: number): void {
  res.status(404).json({ error: `No existe ningún libro con id ${id}` });
}

/**
 * Devuelve un error por cada campo no válido (vacío si todo está bien).
 *
 * @param body - Body de la petición, sin validar.
 * @param options.partial - Si es true (PATCH), los campos ausentes no se validan;
 *   si es false (POST y PUT), todos son obligatorios.
 * @returns Un objeto campo -> mensaje de error.
 */
function validateBook(body: Query, { partial = false } = {}): Record<string, string> {
  const errors: Record<string, string> = {};

  // En PATCH solo se valida lo que llega
  const shouldCheck = (field: string) => !partial || body[field] !== undefined;

  for (const field of TEXT_FIELDS) {
    if (!shouldCheck(field)) continue;

    const value = body[field];
    if (typeof value !== "string" || value.trim() === "") {
      errors[field] = "Debe ser un texto no vacío";
    }
  }

  // isSafeInteger rechaza valores enormes como 1e300 (isInteger los aceptaría)
  if (shouldCheck("pages") && (!Number.isSafeInteger(body.pages) || (body.pages as number) <= 0)) {
    errors.pages = "Debe ser un número entero mayor que 0";
  }

  // Los años negativos están permitidos (a. C.), los futuros no
  if (
    shouldCheck("year") &&
    (!Number.isSafeInteger(body.year) || (body.year as number) > new Date().getFullYear())
  ) {
    errors.year = "Debe ser un número entero no posterior al año actual";
  }

  return errors;
}

// Construye los datos del libro (sin id) a partir de un body ya validado
function toBookData(body: Query): BookData {
  return {
    // Se guarda recortado, igual que se valida
    author: (body.author as string).trim(),
    country: (body.country as string).trim(),
    imageLink: (body.imageLink as string).trim(),
    language: (body.language as string).trim(),
    link: (body.link as string).trim(),
    title: (body.title as string).trim(),
    pages: body.pages as number,
    year: body.year as number,
  };
}

/**
 * Construye solo los campos que llegan en un body de PATCH ya validado.
 * Los textos se guardan recortados, igual que en {@link toBookData}.
 *
 * @param body - Body ya validado con `validateBook(body, { partial: true })`.
 * @returns Los campos enviados, listos para mezclarlos con el libro existente.
 */
function toPartialBookData(body: Query): Partial<BookData> {
  const data: Partial<BookData> = {};

  for (const field of TEXT_FIELDS) {
    if (body[field] !== undefined) data[field] = (body[field] as string).trim();
  }

  if (body.pages !== undefined) data.pages = body.pages as number;
  if (body.year !== undefined) data.year = body.year as number;

  return data;
}

function matchesAllFilters(book: Book, query: Query): boolean {
  return (
    matchesText(book.author, query.author) &&
    matchesText(book.country, query.country) &&
    matchesText(book.language, query.language) &&
    matchesText(book.title, query.title) &&
    matchesText(book.link, query.link) &&
    matchesText(book.imageLink, query.imageLink) &&
    matchesNumber(book.pages, query, "pages") &&
    matchesNumber(book.year, query, "year")
  );
}

// "China" coincide con "chi", "CHI", "ina"...
function matchesText(bookValue: string, search: unknown): boolean {
  if (typeof search !== "string") return true;

  return bookValue.toLowerCase().includes(search.toLowerCase());
}

// Comprueba field, field_gt, field_gte, field_lt y field_lte
function matchesNumber(bookValue: number, query: Query, field: string): boolean {
  const equal = parseNumber(query[field]);
  const greaterThan = parseNumber(query[`${field}_gt`]);
  const greaterOrEqual = parseNumber(query[`${field}_gte`]);
  const lessThan = parseNumber(query[`${field}_lt`]);
  const lessOrEqual = parseNumber(query[`${field}_lte`]);

  if (equal !== undefined && bookValue !== equal) return false;
  if (greaterThan !== undefined && bookValue <= greaterThan) return false;
  if (greaterOrEqual !== undefined && bookValue < greaterOrEqual) return false;
  if (lessThan !== undefined && bookValue >= lessThan) return false;
  if (lessOrEqual !== undefined && bookValue > lessOrEqual) return false;

  return true;
}

function applyLimit(books: Book[], limitParam: unknown): Book[] {
  const limit = parseNumber(limitParam);
  if (limit === undefined || limit < 0) return books;

  return books.slice(0, limit);
}

// Devuelve undefined si el parámetro no viene, está en blanco o no es un número finito
// (Number(" ") sería 0 y Number("Infinity") sería Infinity)
function parseNumber(value: unknown): number | undefined {
  if (typeof value !== "string" || value.trim() === "") return undefined;

  const number = Number(value);
  return Number.isFinite(number) ? number : undefined;
}

// Solo acepta dígitos: rechaza "0x1", "1.0", "1e2", " 1"...
function parseId(raw: string): number | undefined {
  return /^\d+$/.test(raw) ? Number(raw) : undefined;
}

export default router;
```

**Qué enseña:**

| Ruta | Códigos | Conceptos |
|------|---------|-----------|
| `GET /` | 200 | `req.query`, filtros de texto y numéricos (`_gt`, `_lte`...), `limit` |
| `GET /:id` | 200, 400, 404 | `parseId` con regex |
| `POST /` | 201, 400 | `validateBook` devuelve **todos** los errores; `toBookData` limpia el body |
| `PUT /:id` | 200, 400, 404 | Sustituye entero; comprueba el 404 **antes** de validar el body |
| `PATCH /:id` | 200, 400, 404 | `partial: true`; "al menos un campo"; `toPartialBookData` |
| `DELETE /:id` | 200, 400, 404 | Devuelve el libro borrado |

- Las rutas **solo** hablan con `storage` (la interfaz `BooksStorage`): nunca tocan el array directamente.
- Funciones auxiliares **debajo** de las rutas (`sendNotFound`, `validateBook`, `parseNumber`...):
  las rutas se leen como un resumen y el detalle queda abajo.
- Comentarios JSDoc encima de cada ruta documentando URL, body y códigos.

---

## EJEMPLO 3 — La capa de almacenamiento (implementación en memoria)

**Origen:** `API_Godines/api-express/api-books-claude/src/services/books-storage-in-memory.ts` *(copiado íntegro)*

```ts
import booksData from "../data/books.json";
import type { Book, BookData, BooksStorage } from "./books-storage";

/**
 * Almacenamiento de libros en memoria.
 *
 * Arranca con los libros de `data/books.json`. Los cambios se pierden al
 * reiniciar el servidor.
 */
export class BooksStorageInMemory implements BooksStorage {
  private books: Book[] = [...booksData];

  async findAll(): Promise<Book[]> {
    // Copia: quien la reciba no puede modificar el almacenamiento desde fuera
    return [...this.books];
  }

  async findById(id: number): Promise<Book | undefined> {
    return this.books.find((book) => book.id === id);
  }

  /**
   * El id es el mayor id existente + 1 (no `books.length + 1`): así no se
   * repiten ids si hay huecos.
   * Ojo: si se borra el libro con el mayor id, ese id se vuelve a usar.
   */
  async create(data: BookData): Promise<Book> {
    const nextId = Math.max(0, ...this.books.map((book) => book.id)) + 1;
    const newBook: Book = { id: nextId, ...data };

    this.books.push(newBook);
    return newBook;
  }

  async replace(id: number, data: BookData): Promise<Book | undefined> {
    const index = this.findIndex(id);
    if (index === -1) return undefined;

    const replacedBook: Book = { id, ...data };

    this.books[index] = replacedBook;
    return replacedBook;
  }

  async update(id: number, data: Partial<BookData>): Promise<Book | undefined> {
    const index = this.findIndex(id);
    if (index === -1) return undefined;

    // El id va al final para que nunca se pueda sobrescribir
    const updatedBook: Book = { ...this.books[index], ...data, id };

    this.books[index] = updatedBook;
    return updatedBook;
  }

  async remove(id: number): Promise<Book | undefined> {
    const index = this.findIndex(id);
    if (index === -1) return undefined;

    const [removedBook] = this.books.splice(index, 1);
    return removedBook;
  }

  private findIndex(id: number): number {
    return this.books.findIndex((book) => book.id === id);
  }
}
```

> 📌 La interfaz `BooksStorage` que implementa esta clase está copiada entera en
> `06-arquitectura-por-capas-y-base-de-datos.md` (sección 6.3).

**Qué enseña:**
- `implements BooksStorage` → la clase cumple el contrato.
- `async` en todos los métodos aunque la memoria sea síncrona → mismo contrato que una BD.
- Devolver **copias** (`[...this.books]`) para que nadie modifique el almacenamiento desde fuera.
- `undefined` cuando no existe → la ruta decide el 404.

---

## EJEMPLO 4 — Conexión a MongoDB y arranque

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

**Origen:** `api-rastro/api-rastro/src/index.ts` *(copiado íntegro)*

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import { connectDb, getDb } from "./db";
import listingsRouter from "./routes/listings";
import offersRouter from "./routes/offers";

const app = express();
const PORT = 3000;

app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

app.use(express.json());

app.get("/api/health", async (_req, res) => {
  await getDb().command({ ping: 1 });
  res.json({ status: "ok", db: getDb().databaseName });
});

app.use("/api/listings", listingsRouter);
app.use("/api/offers", offersRouter);

app.use((req, res) => {
  res.status(404).json({ error: `No existe la ruta ${req.method} ${req.url}` });
});

app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  if (err instanceof SyntaxError) {
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }

  console.error(err);
  res.status(500).json({ error: "Error interno del servidor" });
});

async function start() {
  await connectDb();

  app.listen(PORT, () => {
    console.log(`Servidor escuchando en http://localhost:${PORT}`);
  });
}

start();
```

**Qué enseña:**
- La URL de MongoDB sale de `process.env` (cargado con `tsx --env-file=.env`). Si falta, la app **no arranca**.
- Un único `MongoClient`; `connectDb()` se espera **antes** de `app.listen`.
- Colecciones tipadas: `collection<Listing>("listings")`.
- `/api/health` comprueba que la BD responde (`ping`).
- Esta API no tiene middleware de CORS: no la consume ninguna web desde otro origen (de momento).

**Plantilla de variables** (`api-rastro/api-rastro/.env.example`):

```
MONGODB_URI=mongodb://usuario:contraseña@localhost:27017/?authSource=admin
DB_NAME=rastro
```

---

## EJEMPLO 5 — Router con reglas de negocio (400 / 404 / 409)

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
- Destructuring del body con `req.body ?? {}`.
- Validación en orden: **formato** de ids → **tipo** de `amount` → **existencia** en BD → **reglas**
  (ofertar por tu propio anuncio → 400; anuncio no disponible → 409).
- Campos que pone el servidor: `status: "PENDING"`, `createdAt: new Date()`.
- Datos **copiados** dentro de la oferta (`listing.title`, `buyer.name`): patrón típico de MongoDB
  para no tener que hacer "joins" al leer.
- En el PATCH, `findOneAndUpdate({ _id: offerId, status: "PENDING" }, ...)`: el filtro incluye
  el estado para que, si dos peticiones aceptan a la vez, **solo una** lo consiga.

---

## EJEMPLO 6 — Router con `try/catch` y errores de la BD

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
- Un CRUD completo con MongoDB en un solo archivo (GET lista, GET por id, POST, PATCH, DELETE).
- `validateName` devuelve `string | null` (mensaje de error o nada).
- La **unicidad** la garantiza un índice único de MongoDB; el error `11000` se traduce a **409**.
- En el `catch` se trata solo el error esperado y `throw err` para el resto → Express 5 lo lleva
  al manejador de errores → **500**.
- `res.status(204).send()` al borrar.

---

## EJEMPLO 7 — Servidor fullstack en un archivo (Express 4 + SQLite + sesiones)

**Origen:** `ProyectoReproductor/server/index.ts` *(copiado íntegro; el original está escrito en líneas muy largas)*

```ts
import 'dotenv/config';
import express from 'express';
import cookieParser from 'cookie-parser';
import initSqlJs, { type Database as SqlDatabase } from 'sql.js';
import { randomBytes, scryptSync, timingSafeEqual } from 'node:crypto';
import { join } from 'node:path';
import { mkdirSync, readFileSync, writeFileSync } from 'node:fs';

const dbUrl = process.env.DATABASE_URL ?? 'file:./data/nightjar.sqlite';
const relativePath = dbUrl.replace(/^file:/, '');
const dbPath = join(process.cwd(), relativePath);
mkdirSync(join(dbPath, '..'), { recursive: true });
const app = express();
const SQL = await initSqlJs();
let db: SqlDatabase;
try { db = new SQL.Database(readFileSync(dbPath)); } catch { db = new SQL.Database(); }
db.run('PRAGMA foreign_keys = ON');
db.run(`CREATE TABLE IF NOT EXISTS users (id TEXT PRIMARY KEY, email TEXT UNIQUE NOT NULL, password_hash TEXT NOT NULL, created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP);
CREATE TABLE IF NOT EXISTS sessions (token TEXT PRIMARY KEY, user_id TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE, expires_at INTEGER NOT NULL);
CREATE TABLE IF NOT EXISTS favorites (user_id TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE, track_id TEXT NOT NULL, track_json TEXT NOT NULL, created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP, PRIMARY KEY(user_id, track_id));
CREATE TABLE IF NOT EXISTS history (id INTEGER PRIMARY KEY AUTOINCREMENT, user_id TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE, track_id TEXT NOT NULL, track_json TEXT NOT NULL, played_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP);`);
function persist() { writeFileSync(dbPath, Buffer.from(db.export())); }
function run(sql: string, params: unknown[] = []) { db.run(sql, params as never[]); persist(); }
function one<T>(sql: string, params: unknown[] = []): T | undefined { const statement = db.prepare(sql); statement.bind(params as never[]); const result = statement.step() ? statement.getAsObject() as T : undefined; statement.free(); return result; }
function many<T>(sql: string, params: unknown[] = []): T[] { const statement = db.prepare(sql); statement.bind(params as never[]); const results: T[] = []; while (statement.step()) results.push(statement.getAsObject() as T); statement.free(); return results; }
app.use(express.json({ limit: '32kb' })); app.use(cookieParser()); app.use((req,res,next)=>{ if(req.path.startsWith('/api/')) res.setHeader('Cache-Control','no-store'); next(); }); app.use(express.static('public'));
function userFor(req: express.Request) { const token = req.cookies?.nightjar_session; return token ? one<{id:string;email:string}>('SELECT users.id, users.email FROM sessions JOIN users ON users.id=sessions.user_id WHERE sessions.token=? AND sessions.expires_at>?',[token,Date.now()]) : undefined; }
function session(req: express.Request, res: express.Response, next: express.NextFunction) { const user=userFor(req); if(!user)return res.status(401).json({error:'Inicia sesión para continuar.'}); res.locals.user=user; next(); }
function createSession(userId:string,res:express.Response) { const token=randomBytes(32).toString('hex'); run('INSERT INTO sessions(token,user_id,expires_at) VALUES(?,?,?)',[token,userId,Date.now()+1209600000]); res.cookie('nightjar_session',token,{httpOnly:true,sameSite:'lax',secure:process.env.NODE_ENV==='production',maxAge:1209600000}); }
function hashPassword(password:string,salt=randomBytes(16).toString('hex')) { return `${salt}:${scryptSync(password,salt,64).toString('hex')}`; }
function passwordMatches(password:string,stored:string) { const [salt,hash]=stored.split(':'); const actual=scryptSync(password,salt,64),expected=Buffer.from(hash,'hex'); return actual.length===expected.length&&timingSafeEqual(actual,expected); }
app.post('/api/auth/register',(req,res)=>{const email=String(req.body?.email??'').trim().toLowerCase(),password=String(req.body?.password??'');if(!/^\S+@\S+\.\S+$/.test(email)||password.length<10||password.length>128)return res.status(400).json({error:'Introduce un correo válido y una contraseña de 10 a 128 caracteres.'});const id=randomBytes(16).toString('hex');try{run('INSERT INTO users(id,email,password_hash) VALUES(?,?,?)',[id,email,hashPassword(password)]);}catch{return res.status(409).json({error:'Ya existe una cuenta con ese correo.'});}createSession(id,res);res.status(201).json({id,email});});
app.post('/api/auth/login',(req,res)=>{const email=String(req.body?.email??'').trim().toLowerCase(),password=String(req.body?.password??''),user=one<{id:string;email:string;password_hash:string}>('SELECT id,email,password_hash FROM users WHERE email=?',[email]);if(!user||!passwordMatches(password,user.password_hash))return res.status(401).json({error:'Correo o contraseña incorrectos.'});createSession(user.id,res);res.json({id:user.id,email:user.email});});
app.post('/api/auth/logout',(req,res)=>{const token=req.cookies?.nightjar_session;if(token)run('DELETE FROM sessions WHERE token=?',[token]);res.clearCookie('nightjar_session');res.status(204).end();});
app.get('/api/auth/me',(req,res)=>{const user=userFor(req);user?res.json(user):res.status(401).json({error:'Sesión no iniciada.'});});
app.get('/api/tracks/search',session,async(req,res)=>{const q=String(req.query.q??'').trim();if(!q||q.length>100)return res.status(400).json({error:'Escribe una búsqueda de hasta 100 caracteres.'});if(!process.env.JAMENDO_CLIENT_ID)return res.status(503).json({error:'Falta configurar el acceso al catálogo de Jamendo.'});try{const url=new URL('https://api.jamendo.com/v3.0/tracks/');url.search=new URLSearchParams({client_id:process.env.JAMENDO_CLIENT_ID,format:'json',limit:'30',search:q,audioformat:'mp32',include:'musicinfo'}).toString();const response=await fetch(url,{signal:AbortSignal.timeout(10000)});if(!response.ok)throw new Error();const payload=await response.json() as {headers?:{status?:string;error_message?:string};results?:Array<Record<string,unknown>>};if(payload.headers?.status==='failed')return res.status(502).json({error:payload.headers.error_message==='Jamendo Api Invalid Client Id Error: Your credential is not authorized.'?'Jamendo no autorizó este Client ID. Revisa la credencial en tu portal de desarrollador.':'Jamendo no pudo completar la búsqueda.'});res.json((payload.results??[]).map(t=>({id:String(t.id),name:t.name,artist:t.artist_name,album:t.album_name,image:t.album_image,audio:t.audio,duration:t.duration})));}catch{res.status(502).json({error:'No se pudo consultar Jamendo. Comprueba la conexión e inténtalo de nuevo.'});}});
app.get('/api/tracks/featured',session,async(req,res)=>{if(!process.env.JAMENDO_CLIENT_ID)return res.status(503).json({error:'Falta configurar el acceso al catálogo de Jamendo.'});try{const url=new URL('https://api.jamendo.com/v3.0/tracks/');url.search=new URLSearchParams({client_id:process.env.JAMENDO_CLIENT_ID,format:'json',limit:'15',order:'popularity_total',audioformat:'mp32',include:'musicinfo'}).toString();const response=await fetch(url,{signal:AbortSignal.timeout(10000)});if(!response.ok)throw new Error();const payload=await response.json() as {headers?:{status?:string;error_message?:string};results?:Array<Record<string,unknown>>};if(payload.headers?.status==='failed')return res.status(502).json({error:'Jamendo no pudo completar la solicitud de destacados.'});res.json((payload.results??[]).map(t=>({id:String(t.id),name:t.name,artist:t.artist_name,album:t.album_name,image:t.album_image,audio:t.audio,duration:t.duration})));}catch{res.status(502).json({error:'No se pudo consultar Jamendo. Comprueba la conexión e inténtalo de nuevo.'});}});
app.get('/api/library/:kind',session,(req,res)=>{const kind=String(req.params.kind);if(!['favorites','history'].includes(kind))return res.status(404).json({error:'Biblioteca no encontrada.'});const orderColumn=kind==='history'?'played_at':'created_at';const rows=many<{track_json:string}>(`SELECT track_json FROM ${kind} WHERE user_id=? ORDER BY ${orderColumn} DESC LIMIT 100`,[res.locals.user.id]);res.json(rows.map(row=>JSON.parse(row.track_json)));});
app.post('/api/library/favorites/:trackId',session,(req,res)=>{const track=req.body?.track;if(!track||String(track.id)!==String(req.params.trackId)||!track.audio)return res.status(400).json({error:'La canción no tiene datos válidos.'});run('INSERT OR REPLACE INTO favorites(user_id,track_id,track_json) VALUES(?,?,?)',[res.locals.user.id,String(req.params.trackId),JSON.stringify(track)]);res.status(201).json({saved:true});});
app.delete('/api/library/favorites/:trackId',session,(req,res)=>{run('DELETE FROM favorites WHERE user_id=? AND track_id=?',[res.locals.user.id,String(req.params.trackId)]);res.status(204).end();});
app.post('/api/library/history',session,(req,res)=>{const track=req.body?.track;if(!track||!track.id||!track.audio)return res.status(400).json({error:'La canción no tiene datos válidos.'});run('INSERT INTO history(user_id,track_id,track_json) VALUES(?,?,?)',[res.locals.user.id,String(track.id),JSON.stringify(track)]);res.status(201).json({saved:true});});
app.get('*',(_req,res)=>res.sendFile(join(process.cwd(),'public','index.html')));
const port=Number(process.env.PORT||3000);app.listen(port,()=>console.log(`Nightjar disponible en http://localhost:${port}`));
```

### Las rutas de autenticación, formateadas para leerlas

*(mismo código que arriba, solo con saltos de línea y comentarios añadidos)*

```ts
// REGISTRO
app.post('/api/auth/register', (req, res) => {
  const email = String(req.body?.email ?? '').trim().toLowerCase(),
        password = String(req.body?.password ?? '');

  // Validación → 400
  if (!/^\S+@\S+\.\S+$/.test(email) || password.length < 10 || password.length > 128)
    return res.status(400).json({ error: 'Introduce un correo válido y una contraseña de 10 a 128 caracteres.' });

  const id = randomBytes(16).toString('hex');
  try {
    run('INSERT INTO users(id,email,password_hash) VALUES(?,?,?)', [id, email, hashPassword(password)]);
  } catch {
    // email es UNIQUE en la tabla → si ya existe, el INSERT falla → 409
    return res.status(409).json({ error: 'Ya existe una cuenta con ese correo.' });
  }
  createSession(id, res);              // crea la sesión y la cookie
  res.status(201).json({ id, email });
});

// LOGIN
app.post('/api/auth/login', (req, res) => {
  const email = String(req.body?.email ?? '').trim().toLowerCase(),
        password = String(req.body?.password ?? ''),
        user = one<{ id: string; email: string; password_hash: string }>(
          'SELECT id,email,password_hash FROM users WHERE email=?', [email]);

  // Mismo mensaje si no existe el usuario o si la contraseña es mala (no das pistas)
  if (!user || !passwordMatches(password, user.password_hash))
    return res.status(401).json({ error: 'Correo o contraseña incorrectos.' });

  createSession(user.id, res);
  res.json({ id: user.id, email: user.email });
});

// LOGOUT
app.post('/api/auth/logout', (req, res) => {
  const token = req.cookies?.nightjar_session;
  if (token) run('DELETE FROM sessions WHERE token=?', [token]);
  res.clearCookie('nightjar_session');
  res.status(204).end();
});
```

**Qué enseña:**
- `import 'dotenv/config'` para cargar el `.env`; puerto desde `process.env.PORT`.
- Middlewares de paquetes: `express.json({ limit })`, `cookieParser()`, `express.static('public')`.
- **Middleware de ruta** `session`, que protege rutas y deja el usuario en `res.locals.user`.
- **Contraseñas** nunca en texto plano: `scryptSync` con *salt* y comparación con `timingSafeEqual`.
- Cookie de sesión `httpOnly` (el JavaScript del navegador no puede leerla) y `sameSite: 'lax'`.
- La API hace de **intermediaria** con otra API (Jamendo): el `client_id` se queda en el servidor
  y nunca llega al navegador. Error de esa API → **502**; falta configuración → **503**.
- `app.get('*', ...)` al final sirve el `index.html` (SPA). En Express 5 sería `'/*splat'`.

> 📌 Funciona, pero todo en un archivo y en líneas larguísimas es difícil de mantener.
> Buen ejercicio: **refactorizarlo** (en una copia) a `db.ts`, `middleware/session.ts`,
> `routes/auth.ts`, `routes/tracks.ts` y `routes/library.ts`, como en tus otros proyectos.

---

## 🧪 Ejercicio final: comprueba que lo entiendes

Con `api-books-claude` arrancado (`npm run dev`), predice el código y la respuesta **antes**
de lanzar cada petición:

| # | Petición | Tu predicción |
|---|----------|---------------|
| 1 | `GET /api/books/abc` | ? |
| 2 | `GET /api/books/99999` | ? |
| 3 | `POST /api/books` con body `{}` | ? |
| 4 | `PATCH /api/books/1` con body `{}` | ? |
| 5 | `PATCH /api/books/1` con body `{ "pages": 210 }` | ? |
| 6 | `POST /api/books` con un JSON roto: `{ "title": ` | ? |
| 7 | `GET /api/nada` | ? |
| 8 | `DELETE /api/books/1` dos veces seguidas | ? |

<details>
<summary>Soluciones</summary>

1. **400** `{ "error": "El id debe ser un número entero" }`
2. **404** `{ "error": "No existe ningún libro con id 99999" }`
3. **400** `{ "errors": { ...un error por cada campo... } }`
4. **400** `{ "error": "Hay que enviar al menos un campo: author, country, imageLink, language, link, title, pages, year" }`
5. **200** con el libro 1 y `pages: 210`
6. **400** `{ "error": "El body no es un JSON válido" }` (lo da el manejador de errores)
7. **404** `{ "error": "No existe la ruta GET /api/nada" }` (lo da el 404 final)
8. Primera: **200** con el libro borrado. Segunda: **404**.

</details>

👉 **Vuelve al índice:** `00-INDICE.md`
