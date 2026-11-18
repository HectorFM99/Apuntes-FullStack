# 📋 CHEATSHEET — EXPRESS

> Chuleta rápida de comandos y código. Teoría en los documentos 01–08 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal

| Comando | Qué hace |
|---------|----------|
| `npm init -y` | Crea `package.json` |
| `npm install express` | Instala Express |
| `npm install -D typescript tsx @types/express @types/node` | TypeScript + tipos |
| `npx tsc --init` | Crea `tsconfig.json` |
| `npm run dev` | `tsx watch --env-file=.env src/index.ts` |
| `npm run typecheck` | `tsc --noEmit` |
| `npm run build` / `npm start` | Compilar a `dist/` / ejecutar `node dist/index.js` |
| `curl http://localhost:3000/api/books` | Probar un GET |
| `curl -X POST localhost:3000/api/books -H "Content-Type: application/json" -d '{"title":"X"}'` | Probar un POST |

---

## 2. Servidor mínimo

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";

const app = express();
const PORT = process.env.PORT ?? 3000;     // Render pone PORT; en local, 3000

app.use(express.json());                    // lee bodies JSON → req.body

app.get("/", (_req, res) => {
  res.send("Mi API con Libros express");
});

app.listen(PORT, () => {
  console.log(`Servidor escuchando en http://localhost:${PORT}`);
});
```

---

## 3. Rutas

```ts
app.get("/api/books", handler);           // leer lista
app.get("/api/books/:id", handler);       // leer uno → req.params.id
app.post("/api/books", handler);          // crear
app.put("/api/books/:id", handler);       // reemplazar
app.patch("/api/books/:id", handler);     // modificar parcialmente
app.delete("/api/books/:id", handler);    // borrar
```

### Router en archivo aparte

```ts
// src/routes/products.ts
import { Router } from "express";
const router = Router();

router.get("/", async (req, res) => { /* ... */ });       // GET /api/products
router.get("/:id", async (req, res) => { /* ... */ });    // GET /api/products/:id
export default router;

// src/index.ts
import productsRouter from "./routes/products";
app.use("/api/products", productsRouter);                 // prefijo
```

> ⚠️ **Orden**: `/count` antes de `/:id`, si no `"count"` se toma como id.

---

## 4. Request y Response

| `req.` | Ejemplo | Contiene |
|--------|---------|----------|
| `params` | `/books/:id` → `req.params.id` | Partes de la ruta (string) |
| `query` | `?author=x&maxPages=150` → `req.query.author` | Filtros (string o array) |
| `body` | `req.body.title` | Datos enviados (necesita `express.json()`) |
| `headers` | `req.headers.authorization` | Cabeceras |
| `method` / `url` | `"GET"` / `"/api/books"` | Método y ruta |
| `cookies` | `req.cookies.sid` | Cookies (con `cookie-parser`) |

| `res.` | Qué hace |
|--------|----------|
| `res.json(obj)` | Responde JSON (200 por defecto) |
| `res.status(201).json(obj)` | Código + JSON |
| `res.status(204).send()` | Sin cuerpo (DELETE) |
| `res.sendStatus(204)` | Solo código |
| `res.send("texto")` | Texto/HTML |
| `res.setHeader("X", "v")` | Cabecera |
| `res.cookie("sid", token, { httpOnly: true })` | Cookie |
| `res.redirect("/login")` | Redirección |

```ts
// Query: siempre llega como texto
const category = typeof req.query.category === "string" ? req.query.category : undefined;
const max = req.query.maxPages ? Number(req.query.maxPages) : undefined;
```

### Códigos más usados

| Código | Cuándo |
|--------|--------|
| 200 | OK (GET, PATCH) |
| 201 | Creado (POST) |
| 204 | Borrado, sin cuerpo |
| 400 | Datos o id inválidos |
| 401 / 403 | No autenticado / sin permiso |
| 404 | No existe |
| 409 | Conflicto (duplicado, error Mongo `11000`) |
| 500 | Error del servidor |

---

## 5. CRUD completo (patrón de `express-mongo-atlas-render`)

```ts
router.get("/:id", async (req, res) => {
  if (!ObjectId.isValid(req.params.id)) {                       // 400: id mal formado
    res.status(400).json({ error: "El ID no tiene un formato válido" });
    return;
  }
  const product = await productsCollection().findOne({ _id: new ObjectId(req.params.id) });
  if (!product) {                                               // 404
    res.status(404).json({ error: "Producto no encontrado" });
    return;
  }
  res.json(product);                                            // 200
});

router.post("/", async (req, res) => {
  const error = validateProduct(req.body ?? {}, false);
  if (error) { res.status(400).json({ error }); return; }      // 400
  const result = await productsCollection().insertOne(newProduct);
  res.status(201).json({ _id: result.insertedId, ...newProduct });   // 201
});

router.delete("/:id", async (req, res) => {
  const result = await productsCollection().deleteOne({ _id: new ObjectId(req.params.id) });
  if (result.deletedCount === 0) { res.status(404).json({ error: "Producto no encontrado" }); return; }
  res.status(204).send();                                       // 204
});
```

> 📌 `return` después de responder: evita *Cannot set headers after they are sent*.

---

## 6. Validación

```ts
// Devuelve un mensaje de error o null. partial = true para PATCH
export function validateProduct(body: { name?: unknown; price?: unknown }, partial: boolean): string | null {
  const { name, price } = body;
  if (name !== undefined || !partial) {
    if (typeof name !== "string" || name.trim() === "") return "El nombre debe ser un texto no vacío";
  }
  if (price !== undefined || !partial) {
    if (typeof price !== "number" || !Number.isFinite(price) || price < 0)
      return "El precio debe ser un número igual o mayor que cero";
  }
  return null;
}
```

```ts
// Id numérico (SQL / memoria)
const id = Number(req.params.id);
if (!Number.isInteger(id) || id <= 0) { res.status(400).json({ error: "Id inválido" }); return; }
```

---

## 7. Middleware

```ts
// Firma: (req, res, next) => ...   next() pasa al siguiente
app.use((req, _res, next) => {                 // logger
  console.log(`${req.method} ${req.url}`);
  next();
});
```

### CORS a mano

```ts
app.use((req, res, next) => {
  res.setHeader("Access-Control-Allow-Origin", process.env.CORS_ORIGIN ?? "http://localhost:5173");
  res.setHeader("Access-Control-Allow-Methods", "GET, POST, PATCH, DELETE");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type");
  if (req.method === "OPTIONS") { res.sendStatus(204); return; }   // preflight
  next();
});
// Alternativa: npm i cors → app.use(cors({ origin: process.env.CORS_ORIGIN }))
```

### Middlewares útiles

| Código | Qué hace |
|--------|----------|
| `express.json({ limit: '32kb' })` | Parsea JSON con límite de tamaño |
| `express.static('public')` | Sirve archivos estáticos (front + API en el mismo servidor) |
| `cookieParser()` | `req.cookies` (paquete `cookie-parser`) |
| `cors()` | CORS configurable (paquete `cors`) |

### Orden recomendado en `index.ts`

```
1. logger → 2. CORS → 3. express.json() → 4. rutas → 5. 404 → 6. manejador de errores
```

### 404 y manejador de errores (van AL FINAL)

```ts
app.use((req, res) => {                                       // ninguna ruta respondió
  res.status(404).json({ error: `No existe la ruta ${req.method} ${req.url}` });
});

app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {   // 4 parámetros
  if (err instanceof SyntaxError) {
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }
  console.error(err);                                         // detalle al log
  res.status(500).json({ error: "Error interno del servidor" });   // genérico al cliente
});
```

> 📌 Express 5 captura solo los errores de handlers `async` (los pasa al manejador).
> En Express 4 había que hacer `try/catch` + `next(err)`.

---

## 8. Arquitectura por capas

```
src/
├── index.ts                 ← app, middlewares, montar routers, listen
├── db.ts                    ← conexión (connectDb, getDb)
├── types.ts                 ← tipos de dominio
├── routes/books-routes.ts   ← HTTP: leer req, validar, responder
└── services/
    ├── books-storage.ts            ← interfaz (contrato)
    └── books-storage-in-memory.ts  ← implementación
```

```ts
// Contrato: las rutas no saben si es memoria, Mongo o SQL
export interface BooksStorage {
  findAll(): Promise<Book[]>;
  findById(id: number): Promise<Book | undefined>;   // undefined si no existe → 404
  // ...create, update, remove
}
```

| Capa | Responsabilidad |
|------|-----------------|
| Rutas | HTTP: params, body, status codes |
| Servicios | Lógica de negocio |
| Almacenamiento | Hablar con la BD |

---

## 9. Conexión a MongoDB (`db.ts`)

```ts
import { Db, MongoClient } from "mongodb";

const uri = process.env.MONGODB_URI;
if (!uri) throw new Error("Falta MONGODB_URI en el archivo .env");
const client = new MongoClient(uri);
let db: Db;

export async function connectDb() {
  await client.connect();
  db = client.db(process.env.DB_NAME ?? "bootcamp_api");
}
export function getDb() { return db; }
```

---

## 10. Tipos de Express en TypeScript

```ts
import type { Request, Response, NextFunction, RequestHandler } from "express";

// Tipar params, body y query
router.get("/:id", (req: Request<{ id: string }>, res: Response) => { /* req.params.id */ });
router.post("/", (req: Request<{}, {}, NewBook>, res) => { /* req.body: NewBook */ });
```

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `req.body` es `undefined` | Falta `app.use(express.json())` (antes de las rutas) o el cliente no envía `Content-Type: application/json` |
| `Cannot set headers after they are sent` | Respondiste dos veces: pon `return` tras cada `res...` |
| La petición se queda colgada | Un middleware no llama a `next()` ni responde |
| `/books/count` devuelve "id inválido" | Ruta `/:id` declarada antes que `/count` |
| `blocked by CORS policy` | Permite el origen del front (`Access-Control-Allow-Origin` / `cors`) |
| El 404 salta siempre | El middleware 404 está antes de las rutas: muévelo al final |
| El manejador de errores no se ejecuta | Le faltan los **4** parámetros `(err, req, res, next)` |
| `EADDRINUSE :::3000` | Otro servidor usa el puerto |
