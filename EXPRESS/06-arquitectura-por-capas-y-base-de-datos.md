# 06 — ARQUITECTURA POR CAPAS Y CONEXIÓN A BASE DE DATOS

> Cuando todo está en la ruta (leer la petición, validar, consultar la BD, responder), el
> código funciona pero es difícil de cambiar. **Separar en capas** significa que cada
> archivo tiene **una sola responsabilidad**, y que puedes cambiar una capa (por ejemplo,
> la base de datos) **sin tocar** las demás.

---

## 6.1 Las capas

```
┌─────────────────────────────────────────────┐
│  index.ts        → monta la app (middleware,│
│                    routers, 404, errores)   │
├─────────────────────────────────────────────┤
│  routes/         → HTTP: leer req, validar, │
│                    elegir código, res.json  │
├─────────────────────────────────────────────┤
│  services/       → lógica / contrato de     │
│                    datos (interfaz)         │
├─────────────────────────────────────────────┤
│  storage / db    → cómo se guardan los datos│
│                    (memoria, MongoDB, SQL)  │
└─────────────────────────────────────────────┘
```

| Capa | Sabe de HTTP (`req`, `res`) | Sabe de la BD |
|------|-----------------------------|---------------|
| Rutas | ✅ | ❌ (idealmente) |
| Servicios / almacenamiento | ❌ | ✅ |

> 📌 Regla: **la capa de datos nunca recibe `req` ni `res`**. Recibe datos normales
> (`id: number`, `data: BookData`) y devuelve datos normales.

---

## 6.2 Tu objetivo, escrito en el PRD

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/docs/prd-books.md`)

```md
## Hecho cuando...
- Todos los endpoints responden con los códigos HTTP correctos (200, 201, 400, 404).
- Se puede cambiar de memoria a MongoDB sin tocar las rutas.
```

"Cambiar de memoria a MongoDB **sin tocar las rutas**" es exactamente lo que consigue
la arquitectura por capas. Veamos cómo.

---

## 6.3 Capa 1 — El contrato (interfaz)

Una **interfaz** de TypeScript dice **qué** operaciones existen, pero no **cómo** se hacen.

### Ejemplo REAL copiado (`api-books-claude/src/services/books-storage.ts`)

```ts
/**
 * Contrato del almacenamiento de libros.
 *
 * Las rutas solo conocen esta interfaz, no cómo se guardan los datos.
 * Para cambiar de almacenamiento (por ejemplo, de memoria a MongoDB) basta con
 * escribir otra clase que implemente `BooksStorage`; las rutas no cambian.
 *
 * Los métodos devuelven `Promise` aunque la versión en memoria no lo necesite:
 * así el contrato ya sirve para almacenamientos asíncronos como una base de datos.
 */

/** Un libro tal y como se guarda y se devuelve en la API. */
export interface Book {
  id: number;
  author: string;
  country: string;
  imageLink: string;
  language: string;
  link: string;
  pages: number;
  title: string;
  year: number;
}

/** Los datos de un libro sin el id (lo asigna el almacenamiento). */
export type BookData = Omit<Book, "id">;

export interface BooksStorage {
  /** Devuelve todos los libros. */
  findAll(): Promise<Book[]>;

  /**
   * Busca un libro por su id.
   * @returns El libro, o `undefined` si no existe.
   */
  findById(id: number): Promise<Book | undefined>;

  /**
   * Guarda un libro nuevo y le asigna un id.
   * @param data - Datos ya validados.
   * @returns El libro creado, con su id.
   */
  create(data: BookData): Promise<Book>;

  /**
   * Sustituye por completo los datos de un libro (PUT). El id no cambia.
   * @returns El libro sustituido, o `undefined` si no existe.
   */
  replace(id: number, data: BookData): Promise<Book | undefined>;

  /**
   * Modifica solo los campos recibidos de un libro (PATCH). El id no cambia.
   * @returns El libro modificado, o `undefined` si no existe.
   */
  update(id: number, data: Partial<BookData>): Promise<Book | undefined>;

  /**
   * Borra un libro.
   * @returns El libro borrado, o `undefined` si no existía.
   */
  remove(id: number): Promise<Book | undefined>;
}
```

**Cosas importantes:**

| Detalle | Por qué |
|---------|---------|
| `Omit<Book, "id">` | Al crear, el cliente **no** manda el id: lo pone el almacenamiento |
| `Partial<BookData>` | En PATCH todos los campos son **opcionales** |
| `Book \| undefined` | "Si no existe, devuelvo `undefined`" → la ruta decide el **404** |
| Todo devuelve `Promise` | Aunque la memoria es instantánea, una BD real es asíncrona. Así el contrato sirve para las dos |

> 📌 Fíjate: el almacenamiento **no** devuelve códigos HTTP ni lanza "404". Solo dice
> "no existe" (`undefined`). Traducirlo a HTTP es trabajo de la ruta.

---

## 6.4 Capa 2 — Una implementación (en memoria)

### Ejemplo REAL copiado (`api-books-claude/src/services/books-storage-in-memory.ts`)

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

**Trucos que merece la pena entender:**

```ts
// ¿Por qué no books.length + 1?
// books con ids [1, 2, 5]  → length + 1 = 4  ✅ por suerte
// books con ids [1, 2, 3], borras el 2 → [1, 3] → length + 1 = 3 ❌ ¡ya existe!
Math.max(0, ...[1, 3]) + 1   // 4 ✅
```

```ts
// PATCH: el orden del spread importa
{ ...libroViejo, ...cambios, id }
//   1º base      2º pisa     3º el id SIEMPRE el correcto
```

> 📌 `implements BooksStorage` obliga a la clase a tener **todos** los métodos de la
> interfaz con los tipos exactos. Si te falta uno, TypeScript da error.

---

## 6.5 Capa 3 — Las rutas usan el contrato, no la implementación

### Ejemplo REAL copiado (`api-books-claude/src/routes/books-routes.ts`)

```ts
import { Router } from "express";
import type { Response } from "express";
import type { Book, BookData, BooksStorage } from "../services/books-storage";
import { BooksStorageInMemory } from "../services/books-storage-in-memory";

// Las rutas no tocan los datos directamente: siempre a través del contrato BooksStorage.
// Para cambiar de almacenamiento (p. ej. MongoDB), solo hay que cambiar esta línea.
const storage: BooksStorage = new BooksStorageInMemory();
```

Y en cada ruta solo se llama al contrato:

```ts
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
```

```
            Rutas (books-routes.ts)
                     │
                     │ solo conocen
                     ▼
          interface BooksStorage   ◄── el "enchufe"
               ▲             ▲
   implements  │             │  implements
               │             │
BooksStorageInMemory    BooksStorageMongo (futuro)
```

### Cómo sería la versión MongoDB (ejemplo propio, NO está en el proyecto)

```ts
// services/books-storage-mongo.ts  — ejemplo propio para ver la idea
import type { Collection } from "mongodb";
import type { Book, BookData, BooksStorage } from "./books-storage";

export class BooksStorageMongo implements BooksStorage {
  constructor(private collection: Collection<Book>) {}

  async findAll() {
    return this.collection.find({}, { projection: { _id: 0 } }).toArray();
  }

  async findById(id: number) {
    return (await this.collection.findOne({ id }, { projection: { _id: 0 } })) ?? undefined;
  }

  // ... create, replace, update, remove
}
```

Y en las rutas solo cambiaría **una línea**:

```ts
const storage: BooksStorage = new BooksStorageMongo(getDb().collection<Book>("books"));
```

---

## 6.6 Arquitectura "routes + db" (sin capa de servicios)

Tus proyectos con MongoDB usan una versión más sencilla: un `db.ts` que gestiona la
conexión y expone las colecciones, y las rutas las usan directamente.

```
index.ts ──► routes/listings.ts ──► db.ts (listingsCollection()) ──► MongoDB
         └─► routes/offers.ts   ──┘
```

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/db.ts`)

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

**Paso a paso:**

1. Lee la URL de conexión de `process.env.MONGODB_URI`. Si falta → **error al arrancar**
   (mejor fallar pronto que a mitad de una petición).
2. Crea **un solo** `MongoClient` para toda la app (no uno por petición).
3. `connectDb()` conecta y guarda la `db` en una variable del módulo.
4. `xxxCollection()` devuelve la colección **tipada** (`collection<Listing>`) → TypeScript
   sabe qué campos tiene cada documento.
5. `serverSelectionTimeoutMS: 5000` → si MongoDB no responde en 5 s, error (en vez de
   esperar 30 s por defecto).

> 📌 ¿Por qué funciones (`listingsCollection()`) y no constantes? Porque `db` **no existe**
> hasta que se llama a `connectDb()`. Una función la busca en el momento de usarla.

### Uso desde una ruta (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
import { listingsCollection, offersCollection, usersCollection } from "../db";
// ...
const result = await listingsCollection().insertOne(newListing);

res.status(201).json({ _id: result.insertedId, ...newListing });
```

### Health check: ¿está viva la BD? (`api-rastro/api-rastro/src/index.ts`)

```ts
app.get("/api/health", async (_req, res) => {
  await getDb().command({ ping: 1 });
  res.json({ status: "ok", db: getDb().databaseName });
});
```

Si MongoDB está caído, `ping` lanza error → (Express 5) manejador de errores → 500.

---

## 6.7 Preparar la BD al arrancar (índices)

### Ejemplo REAL copiado (`mongo-api-libros/.../src/db.ts`)

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

> 📌 `createIndex` es **idempotente**: si el índice ya existe, no hace nada. Por eso se
> puede llamar en cada arranque sin problema.

---

## 6.8 Variables de entorno (`.env`)

La URL de la base de datos lleva **usuario y contraseña** → **nunca** va en el código ni
en Git. Va en un archivo `.env` que está en `.gitignore`.

### Ejemplo REAL copiado (`api-rastro/api-rastro/.env.example`)

```
MONGODB_URI=mongodb://usuario:contraseña@localhost:27017/?authSource=admin
DB_NAME=rastro
```

| Archivo | ¿Va a Git? | Contenido |
|---------|-----------|-----------|
| `.env` | ❌ NO (en `.gitignore`) | Los valores **reales** |
| `.env.example` | ✅ SÍ | Plantilla con valores **de ejemplo**, para que otros sepan qué variables hacen falta |

### ¿Cómo se carga el `.env`?

**Opción A — flag de Node/tsx** (`api-rastro/api-rastro/package.json`):

```json
"scripts": {
  "dev": "tsx watch --env-file=.env src/index.ts"
}
```

**Opción B — paquete `dotenv`** (`ProyectoReproductor/server/index.ts`):

```ts
import 'dotenv/config';     // primera línea del archivo: carga .env en process.env
```

```ts
// ❌ Contraseña en el código → acaba en GitHub
const client = new MongoClient("mongodb://admin:miPassword@localhost:27017");

// ✅ Desde el entorno
const uri = process.env.MONGODB_URI;
if (!uri) throw new Error("Falta MONGODB_URI en el archivo .env");
const client = new MongoClient(uri);
```

---

## 6.9 Otra base de datos: SQLite en el reproductor

Express no está atado a MongoDB. `ProyectoReproductor` usa **SQLite** con `sql.js`.

### Ejemplo REAL copiado (`ProyectoReproductor/server/index.ts`)

```ts
const dbUrl = process.env.DATABASE_URL ?? 'file:./data/nightjar.sqlite';
const relativePath = dbUrl.replace(/^file:/, '');
const dbPath = join(process.cwd(), relativePath);
mkdirSync(join(dbPath, '..'), { recursive: true });
const app = express();
const SQL = await initSqlJs();
let db: SqlDatabase;
try { db = new SQL.Database(readFileSync(dbPath)); } catch { db = new SQL.Database(); }
db.run('PRAGMA foreign_keys = ON');
```

Y tres **funciones de acceso a datos** (una mini capa de datos):

```ts
function persist() { writeFileSync(dbPath, Buffer.from(db.export())); }
function run(sql: string, params: unknown[] = []) { db.run(sql, params as never[]); persist(); }
function one<T>(sql: string, params: unknown[] = []): T | undefined { const statement = db.prepare(sql); statement.bind(params as never[]); const result = statement.step() ? statement.getAsObject() as T : undefined; statement.free(); return result; }
function many<T>(sql: string, params: unknown[] = []): T[] { const statement = db.prepare(sql); statement.bind(params as never[]); const results: T[] = []; while (statement.step()) results.push(statement.getAsObject() as T); statement.free(); return results; }
```

| Función | Para qué | Devuelve |
|---------|----------|----------|
| `run(sql, params)` | INSERT / UPDATE / DELETE (y guarda el archivo) | nada |
| `one<T>(sql, params)` | SELECT de **una** fila | `T \| undefined` |
| `many<T>(sql, params)` | SELECT de **varias** filas | `T[]` |

> 🔒 Fíjate en los `?` de las consultas: `'SELECT ... WHERE email=?', [email]`. Son
> **parámetros**: la librería escapa el valor y así se evita la **inyección SQL**.

```ts
// ❌ Inyección SQL: si email = "' OR 1=1 --" devuelve todos los usuarios
one(`SELECT * FROM users WHERE email='${email}'`);

// ✅ Consulta parametrizada
one('SELECT id,email,password_hash FROM users WHERE email=?', [email]);
```

> 📌 En el reproductor todo está en un solo archivo `index.ts`. Funciona, pero si crece,
> lo natural sería moverlo a `db.ts` + `routes/auth.ts` + `routes/library.ts`, como en
> tus otros proyectos.

---

## 6.10 Comparativa de tus cuatro proyectos

| Proyecto | Datos | Capas | ¿Puede cambiar de BD sin tocar rutas? |
|----------|-------|-------|---------------------------------------|
| `api-books-claude` | Memoria (JSON) | `index` → `routes` → **interfaz** → implementación | ✅ Sí |
| `api-rastro` | MongoDB | `index` → `routes` → `db.ts` | ❌ Las rutas usan MongoDB directamente |
| `02b-express-mongodb` | MongoDB | `index` → `routes` → `db.ts` | ❌ Igual |
| `ProyectoReproductor` | SQLite (`sql.js`) | Todo en `index.ts` con helpers `run/one/many` | ❌ |

> 📌 Las tres formas funcionan. La de **interfaz** es la más flexible y la que verás en
> frameworks como **NestJS** (servicios + repositorios + inyección de dependencias).

---

## ✅ Resumen rápido

1. Rutas = HTTP. Almacenamiento = datos. **No mezclar** `req/res` con la BD.
2. Una **interfaz** (`BooksStorage`) define el contrato; las clases lo implementan.
3. Las rutas solo conocen la interfaz → cambiar de BD = cambiar **una línea**.
4. El almacenamiento devuelve `undefined` si no existe; la ruta lo convierte en **404**.
5. `db.ts`: un único cliente, `connectDb()` al arrancar, funciones `xxxCollection()` tipadas.
6. Credenciales en `.env` (fuera de Git) + `.env.example` como plantilla.
7. En SQL, consultas **parametrizadas** (`?`) siempre.

👉 **Siguiente:** `07-estructura-de-proyecto-y-typescript.md`
