# 07 — EJEMPLOS COMPLETOS DE PROYECTOS

> Tu proyecto `mongo-atlas-render/express-mongo-atlas-render/` entero, copiado tal
> cual, con notas sobre **qué papel juega cada archivo cuando la base de datos es
> Atlas**. Contraseñas de desarrollo sustituidas por `TU_PASSWORD`; el `.env` real
> no se copia (su forma está en el archivo `02`, 2.2).
>
> 📌 Para la explicación línea a línea del CRUD con el driver, ver
> `../MONGODB/06-mongodb-con-node-express-typescript.md`.

---

## Recorrido de una petición en producción

```
POST https://<api>.onrender.com/api/categories  {"name":"cocina"}
   │
   ├─ index.ts        logger → CORS → express.json() → router
   ├─ categories.ts   validateName() → insertOne()
   ├─ db.ts           getDb() → MongoClient (pool) ──TLS──▶ Atlas cluster0 (PRIMARY)
   │                                                     └─ replica a 2 SECONDARY (w=majority)
   ├─ ¿nombre repetido? → índice único name_1 → E11000 → 409
   └─ 201 {"_id": "...", "name": "cocina"}
```

## Qué mirar en cada archivo

| Archivo | Papel con Atlas |
|---------|-----------------|
| `.env.example` | URI de **Docker**; en Render va la de Atlas |
| `compose.yaml` | Mongo local (standalone); Atlas lo sustituye en producción |
| `package.json` | `dev` carga `.env`; `build`/`start` para Render; `seed` usa el `mongosh` del contenedor |
| `src/db.ts` | Único `MongoClient`, `DB_NAME` por defecto, índice único al arrancar |
| `src/index.ts` | Conecta a Atlas **antes** de escuchar; `PORT` de Render; errores genéricos |
| `src/routes/products.ts` | `typeof === "string"` antes de usar `req.query` (evita inyección de operadores) |
| `src/routes/categories.ts` | `MongoServerError` 11000 → 409, que funciona porque el índice existe en Atlas |
| `mongo/seed.js` | Reutilizable contra Atlas (⚠️ hace `deleteMany({})`) |

### `.env.example`

```
MONGODB_URI=mongodb://admin:TU_PASSWORD@localhost:27017/?authSource=admin
DB_NAME=bootcamp_api

```

### `compose.yaml`

> En Atlas no hay `compose.yaml`: el cluster lo gestiona MongoDB. Este archivo solo existe para desarrollo.

```yaml
services:
  mongodb:
    image: mongo:8
    container_name: mongodb-practica
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: TU_PASSWORD
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:

```

### `package.json`

```json
{
  "name": "02b-express-mongodb",
  "version": "1.0.0",
  "scripts": {
    "dev": "tsx watch --env-file=.env src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "typecheck": "tsc --noEmit",
    "lint": "eslint .",
    "seed": "docker cp mongo/seed.js mongodb-practica:/tmp/seed.js && docker exec mongodb-practica mongosh --quiet -u admin -p TU_PASSWORD --authenticationDatabase admin /tmp/seed.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^5.3.0",
    "mongodb": "^7.8.0"
  },
  "devDependencies": {
    "@eslint/js": "^10.0.1",
    "@types/express": "^5.0.6",
    "@types/node": "^26.6.5",
    "eslint": "^10.14.0",
    "globals": "^17.14.0",
    "tsx": "^4.23.15",
    "typescript": "~6.0",
    "typescript-eslint": "^8.72.1"
  },
  "private": true
}

```

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "rootDir": "src",
    "outDir": "dist"
  },
  "include": ["src/**/*.ts"]
}

```

### `src/db.ts`

```ts
import { Db, MongoClient } from "mongodb";

const uri = process.env.MONGODB_URI;
if (!uri) {
  throw new Error("Falta MONGODB_URI en el archivo .env");
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

### `src/index.ts`

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import productsRouter from "./routes/products";
import categoriesRouter from "./routes/categories";
import { connectDb } from "./db";

const app = express();
// Render nos dice el puerto en la variable PORT. En tu ordenador no existe: usamos 3000.
const PORT = process.env.PORT ?? 3000;

// 1. Registrar cada petición en la terminal
app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// 2. CORS: permitir peticiones desde nuestra página web
app.use((req, res, next) => {
  res.setHeader("Access-Control-Allow-Origin", "http://127.0.0.1:5500");
  res.setHeader("Access-Control-Allow-Methods", "GET, POST, PATCH, DELETE");
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
  res.send("Mi primera API con Express");
});

app.get("/api/hello", (req, res) => {
  const name = req.query.name;

  res.json({ message: `Hola, ${name ?? "persona"}` });
});

app.use("/api/products", productsRouter);
app.use("/api/categories", categoriesRouter); // Reto 2

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

async function start() {
  await connectDb();

  app.listen(PORT, () => {
    console.log(`Servidor escuchando en http://localhost:${PORT}`);
  });
}

start();

```

### `src/routes/products.ts`

```ts
import { Router } from "express";
import { ObjectId } from "mongodb";
import { getDb } from "../db";

const router = Router();

// Ejercicio 7.1: categoría en cada producto
export type Product = {
  name: string;
  price: number;
  category: string;
};

export function productsCollection() {
  return getDb().collection<Product>("products");
}

// Devuelve un mensaje de error, o null si los datos son válidos.
// partial = true: los campos pueden faltar (para PATCH).
export function validateProduct(
  body: { name?: unknown; price?: unknown; category?: unknown },
  partial: boolean
): string | null {
  const { name, price, category } = body;

  if (name !== undefined || !partial) {
    if (typeof name !== "string" || name.trim() === "") {
      return "El nombre debe ser un texto no vacío";
    }
  }

  if (price !== undefined || !partial) {
    if (typeof price !== "number" || !Number.isFinite(price) || price < 0) {
      return "El precio debe ser un número igual o mayor que cero";
    }
  }

  // Ejercicio 7.1: la categoría es opcional, pero si llega tiene que ser un texto
  if (category !== undefined) {
    if (typeof category !== "string" || category.trim() === "") {
      return "La categoría debe ser un texto no vacío";
    }
  }

  return null;
}

// Ejercicio 7: GET /api/products?category=muebles&search=sil
router.get("/", async (req, res) => {
  const filter: { category?: string; name?: { $regex: string; $options: string } } = {};

  // Ejercicio 7.2: filtrar por categoría
  if (typeof req.query.category === "string") {
    filter.category = req.query.category;
  }

  // Ejercicio 7.3: buscar por nombre sin distinguir mayúsculas
  if (typeof req.query.search === "string") {
    filter.name = { $regex: req.query.search, $options: "i" };
  }

  const products = await productsCollection().find(filter).toArray();
  res.json(products);
});

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
    // Ejercicio 7.1: si no llega categoría, "sin-categoria"
    category: body.category?.trim() ?? "sin-categoria"
  };

  const result = await productsCollection().insertOne(newProduct);

  res.status(201).json({ _id: result.insertedId, ...newProduct });
});

router.patch("/:id", async (req, res) => {
  if (!ObjectId.isValid(req.params.id)) {
    res.status(400).json({ error: "El ID no tiene un formato válido" });
    return;
  }

  const body = req.body ?? {};

  if (body.name === undefined && body.price === undefined && body.category === undefined) {
    res.status(400).json({ error: "Debes enviar al menos un campo para modificar" });
    return;
  }

  const error = validateProduct(body, true);

  if (error) {
    res.status(400).json({ error });
    return;
  }

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
});

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

export default router;

```

### `src/routes/categories.ts`

```ts
// Reto 2: el CRUD de categorías del apunte 02 (reto 2), ahora en MongoDB.
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

### `mongo/seed.js`

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

### `README.md` (sección de despliegue)

```markdown
## Desplegar en Render

Paso a paso en los apuntes `06_despliegue_render.md`. Resumen:

| Campo de Render | Valor |
|---|---|
| Root Directory | `02b-express-mongodb` (si el proyecto está en una subcarpeta) |
| Build Command | `npm install && npm run build` |
| Start Command | `npm start` |
| Variables | `MONGODB_URI` (la de MongoDB Atlas) y `DB_NAME=bootcamp_api` |

`npm run build` compila TypeScript a `dist/` y `npm start` ejecuta `node dist/index.js`. El puerto sale de `process.env.PORT` (Render lo pone solo; en tu ordenador es 3000).
```

---

## Cambios que harían el proyecto más "Atlas-ready" (NO están en tu proyecto)

```ts
// src/db.ts
const client = new MongoClient(uri, {
  appName: "api-products",        // identificar la app en las métricas de Atlas
  serverSelectionTimeoutMS: 5000, // fallar en 5 s, no en 30
});
```

```
# .env — quitar lo que el código no usa y añadir DB_NAME explícito
MONGODB_URI="mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net/?retryWrites=true&w=majority&appName=api-products"
DB_NAME=bootcamp_api
```

```ts
// src/index.ts — CORS configurable para el frontend desplegado
const ALLOWED_ORIGIN = process.env.CORS_ORIGIN ?? "http://127.0.0.1:5500";
res.setHeader("Access-Control-Allow-Origin", ALLOWED_ORIGIN);
```
