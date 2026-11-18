# 07 — MÓDULOS (IMPORT/EXPORT) Y TYPESCRIPT CON NODE, EXPRESS Y NEST

> En un proyecto real el código se reparte en **muchos archivos**. Cada archivo es un
> **módulo**: decide qué **exporta** (comparte) y qué **importa** de otros. En
> TypeScript además se exportan e importan **tipos**.

---

## 7.1 `export` e `import` con nombre

```ts
// archivo: matematicas.ts
export function sumar(a: number, b: number): number { return a + b; }
export const PI = 3.1416;
export type Operacion = (a: number, b: number) => number;
```

```ts
// archivo: index.ts
import { sumar, PI } from "./matematicas";
import type { Operacion } from "./matematicas";
```

### Ejemplo REAL — un archivo que solo exporta tipos (`api-rastro/api-rastro/src/types.ts`)

```ts
import type { ObjectId } from "mongodb";

export type ListingStatus = "AVAILABLE" | "RESERVED" | "SOLD";
export type OfferStatus = "PENDING" | "ACCEPTED" | "REJECTED";

export type User = {
  name: string;
  // ...
};

export type Listing = { /* ... */ };
export type Offer = { /* ... */ };
```

### Ejemplo REAL — exportar funciones (`api-rastro/api-rastro/src/db.ts`)

```ts
import { Db, MongoClient } from "mongodb";
import type { Listing, Offer, User } from "./types";

// ...

export async function connectDb() { /* ... */ }
export function getDb() { return db; }
export function usersCollection() { return getDb().collection<User>("users"); }
export function listingsCollection() { return getDb().collection<Listing>("listings"); }
export function offersCollection() { return getDb().collection<Offer>("offers"); }
```

**Y quien lo usa (`api-rastro/api-rastro/src/routes/listings.ts`):**

```ts
import { listingsCollection, offersCollection, usersCollection } from "../db";
import type { Listing } from "../types";
```

> 📌 `"./db"` = mismo directorio; `"../db"` = carpeta de arriba. **Sin extensión** en
> proyectos CommonJS como este. Lo que **no** se exporta (`client`, `uri`) queda
> **privado** dentro de `db.ts`.

---

## 7.2 `export default`

Cada archivo puede tener **una** exportación por defecto. Se importa **sin llaves** y
con el nombre que tú quieras.

### Ejemplo REAL (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
const router = Router();

// ... rutas ...

export default router;
```

**Se importa así (`api-rastro/api-rastro/src/index.ts`):**

```ts
import listingsRouter from "./routes/listings";
import offersRouter from "./routes/offers";
```

| | Con nombre | Por defecto |
|---|-----------|-------------|
| Exportar | `export function f() {}` | `export default router;` |
| Importar | `import { f } from "./x"` | `import loQueSea from "./x"` |
| Cuántas por archivo | Las que quieras | Una |
| El nombre al importar | Debe coincidir (o `as`) | Libre |

```ts
// Renombrar al importar algo con nombre
import { listingsCollection as anuncios } from "./db";
```

---

## 7.3 `import type` — importar SOLO tipos

```ts
import type { Listing } from "../types";
```

Le dice a TS: "esto es solo un tipo; **bórralo** del JS final". Como los tipos no
existen en ejecución, no tiene sentido cargar el archivo solo por ellos.

### Ejemplo REAL — mezclar valores y tipos (`api-rastro/api-rastro/src/index.ts`)

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
```

### Ejemplo REAL — `type` en línea (`ProyectoReproductor/server/index.ts`)

```ts
import initSqlJs, { type Database as SqlDatabase } from 'sql.js';
```

```
initSqlJs                        → valor (función), se importa de verdad
type Database as SqlDatabase     → solo el tipo, renombrado, desaparece al compilar
```

> 📌 Con `verbatimModuleSyntax: true` (lo tienes en `practica-typescript/tsconfig.json`)
> **es obligatorio** usar `import type` para los tipos. Si no, error.

---

## 7.4 CommonJS vs ES Modules (lo que pasa por debajo)

Node tiene **dos** sistemas de módulos. Escribas lo que escribas en TS (`import`), lo
que sale al compilar depende de la configuración:

| | CommonJS | ES Modules (ESM) |
|---|----------|------------------|
| Sintaxis en JS | `require()` / `module.exports` | `import` / `export` |
| `package.json` | `"type": "commonjs"` (o nada) | `"type": "module"` |
| `await` fuera de funciones | ❌ | ✅ (top-level await) |
| Proyectos tuyos | `practica-typescript`, `api-rastro`, `node-practica` | `ProyectoReproductor` |

### Ejemplo REAL — salida CommonJS (`practica-typescript/index.js`, generado)

```js
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
```

### Ejemplo REAL — `await` en el nivel superior (`ProyectoReproductor/server/index.ts`)

```ts
const app = express();
const SQL = await initSqlJs();
```

> 📌 Eso solo es posible porque `ProyectoReproductor/package.json` tiene
> `"type": "module"` y el `tsconfig` usa `"module": "NodeNext"`. En un proyecto
> CommonJS hay que meterlo en una función `async` (como `start()` en `api-rastro`).

### `node-practica/prueba` — comparación TS vs JS

**`index.ts`** (real):

```ts
import dayjs from "dayjs";

console.log("Hoy es:", dayjs().format());
```

**`index.js`** (real, escrito a mano en CommonJS, no es salida de `tsc`):

```js
const dayjs = require("dayjs");

console.log("Hoy es:", dayjs().format("DD/MM/YYYY HH:mm:ss"));
```

> 📌 Misma idea, dos sintaxis. Fíjate en que el `package.json` de ese proyecto tiene
> `"start": "node dist/index.js"` pero **no hay `tsconfig.json`** ni carpeta `dist/`:
> para que `npm run build && npm start` funcione faltaría un `tsconfig.json` con
> `"outDir": "dist"`.

---

## 7.5 Tipos de librerías: `@types/...`

Muchas librerías están escritas en JS. Para que TS sepa qué funciones tienen, hacen
falta sus **archivos de declaración** (`.d.ts`, ver `01`):

| Caso | Qué hacer | Ejemplo |
|------|-----------|---------|
| La librería **trae** sus tipos | Nada | `dayjs`, `mongodb`, `@nestjs/common` |
| Los tipos están aparte | `npm i -D @types/libreria` | `@types/express`, `@types/node` |
| No existen tipos | Declararlos tú (`declare module "x";`) | Raro |

### Ejemplo REAL (`ProyectoReproductor/package.json`, devDependencies)

```json
"devDependencies": {"@types/cookie-parser":"^1.4.8", "@types/express":"^5.0.0", "@types/node":"^22.10.5", "@types/sql.js":"^1.4.9", "tsx":"^4.19.2", "typescript":"^5.7.2"}
```

> 📌 Por cada librería JS (`express`, `cookie-parser`, `sql.js`) hay su `@types/...`.
> `@types/node` aporta `process`, `Buffer`, `node:crypto`, `node:fs`...
> Siempre en **devDependencies**: solo hacen falta para programar.

---

## 7.6 Variables de entorno: `process.env` es `string | undefined`

### Ejemplo REAL (`api-rastro/api-rastro/src/db.ts`)

```ts
const uri = process.env.MONGODB_URI;
if (!uri) {
  throw new Error("Falta MONGODB_URI en el archivo .env");
}

const client = new MongoClient(uri, { serverSelectionTimeoutMS: 5000 });
```

```
process.env.MONGODB_URI   → string | undefined
if (!uri) throw ...       → narrowing (ver 06)
new MongoClient(uri)      → aquí uri ya es string ✅
```

### Ejemplo REAL (`ProyectoReproductor/server/index.ts`)

```ts
const dbUrl = process.env.DATABASE_URL ?? 'file:./data/nightjar.sqlite';
// ...
const port=Number(process.env.PORT||3000);
```

> 📌 Todas las variables de entorno son **texto**. `PORT` llega como `"3000"`:
> por eso `Number(...)`.

**Plantilla de variables (`api-rastro/api-rastro/.env.example`)** — el `.env` real
**no** se copia aquí porque contiene secretos:

```
MONGODB_URI=mongodb://usuario:contraseña@localhost:27017/?authSource=admin
DB_NAME=rastro
```

---

## 7.7 Express con TypeScript

### Lo básico: los handlers se tipan solos

```ts
import { Router } from "express";

const router = Router();

router.get("/:id", async (req, res) => {
  // req: Request, res: Response → inferidos por Express
  const { id } = req.params;     // string
  res.json({ id });
});
```

> 📌 Cuando el callback va **directamente** dentro de `router.get(...)` o `app.use(...)`,
> **no hace falta** anotar `req` y `res`. Solo hay que hacerlo cuando defines la
> función **aparte**.

### Ejemplo REAL — handler definido aparte (`ProyectoReproductor/server/index.ts`)

```ts
function userFor(req: express.Request) { const token = req.cookies?.nightjar_session; return token ? one<{id:string;email:string}>('SELECT users.id, users.email FROM sessions JOIN users ON users.id=sessions.user_id WHERE sessions.token=? AND sessions.expires_at>?',[token,Date.now()]) : undefined; }
function session(req: express.Request, res: express.Response, next: express.NextFunction) { const user=userFor(req); if(!user)return res.status(401).json({error:'Inicia sesión para continuar.'}); res.locals.user=user; next(); }
```

Formateado:

```ts
function session(req: express.Request, res: express.Response, next: express.NextFunction) {
  const user = userFor(req);
  if (!user) return res.status(401).json({ error: 'Inicia sesión para continuar.' });
  res.locals.user = user;   // pasa el usuario a la siguiente función
  next();
}
```

> 📌 `express.Request` / `express.Response` / `express.NextFunction` son los tipos de
> Express. Otra forma de importarlos es `import type { Request, Response, NextFunction } from "express"`
> (como en `api-rastro`).

### Ejemplo REAL — middleware de errores (`api-rastro/api-rastro/src/index.ts`)

```ts
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  if (err instanceof SyntaxError) {
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }

  console.error(err);
  res.status(500).json({ error: "Error interno del servidor" });
});
```

**Detalles de TypeScript:**

```
4 parámetros           → Express lo reconoce como middleware de ERRORES
err: unknown           → puede llegar cualquier cosa (ver 06)
_req, _next            → el "_" indica "no lo uso" (evita avisos de parámetros sin usar)
res.status(...); return;  → se responde y se sale (no hagas "return res.status(...)")
```

### `req.body`, `req.query`, `req.params`

| Propiedad | Tipo en Express | Cómo tratarlo |
|-----------|-----------------|---------------|
| `req.params.id` | `string` | Validar formato (`ObjectId.isValid`, `Number(...)`) |
| `req.query.x` | `string \| string[] \| ... \| undefined` | `typeof x === "string"` |
| `req.body` | `any` ⚠️ | Validar **cada campo** antes de usarlo |

### Ejemplo REAL — validar el body y construir un objeto tipado (`api-rastro/.../routes/listings.ts`)

```ts
router.post("/", async (req, res) => {
  const body = req.body ?? {};

  for (const field of ["title", "description", "category", "city"] as const) {
    if (!isNonEmptyString(body[field])) {
      res.status(400).json({ error: `${field} es obligatorio y tiene que ser un texto no vacío` });
      return;
    }
  }

  if (!isValidPrice(body.price)) {
    res.status(400).json({ error: "price es obligatorio y tiene que ser un número mayor o igual que 0" });
    return;
  }

  // ... más validaciones ...

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

  const result = await listingsCollection().insertOne(newListing);

  res.status(201).json({ _id: result.insertedId, ...newListing });
});
```

> 📌 El flujo profesional: **body `any`** → **validación** (narrowing + type predicates)
> → **objeto tipado** (`const newListing: Listing`). Desde ahí, TS garantiza que lo que
> guardas en MongoDB tiene la forma correcta. Si olvidas `views`, error.

### Ejemplo REAL — filtros tipados de MongoDB (`api-rastro/.../routes/listings.ts`)

```ts
import type { Filter } from "mongodb";
// ...
const filter: Filter<Listing> = {};

if (typeof req.query.category === "string") {
    filter.category = req.query.category;
}
// ...
filter.price = { $lte: maxPrice };
```

> 📌 `Filter<Listing>` es un genérico de la librería `mongodb`: solo permite filtrar
> por campos que **existen** en `Listing` (`filter.categoria = ...` daría error).

---

## 7.8 NestJS: TypeScript "de serie"

NestJS está construido **sobre** TypeScript: clases, decoradores, interfaces e
inyección de dependencias por tipo. (La teoría completa está en la carpeta `NESTJS`.)

### Decoradores `@Algo(...)`

Un **decorador** es una función que se "pega" a una clase, método, propiedad o
parámetro para añadirle comportamiento. Requiere en el `tsconfig.json`:

```json
"emitDecoratorMetadata": true,
"experimentalDecorators": true,
```

(fragmento real de `puta/api-books-nest/tsconfig.json`)

### Ejemplo REAL — DTO con validación (`puta/api-books-nest/src/books/dto/create-book.dto.ts`)

```ts
import { Transform } from 'class-transformer';
import { IsInt, IsString, Max, MaxLength, Min, MinLength } from 'class-validator';

// Lo que tiene que enviar el cliente para crear un libro
export class CreateBookDto {
  // Si llega un texto, le quita los espacios del principio y del final
  @Transform(({ value }: { value: unknown }) =>
    typeof value === 'string' ? value.trim() : value,
  )
  @IsString() // tiene que ser texto
  @MinLength(1) // no puede estar vacío
  @MaxLength(200) // como mucho 200 caracteres
  title: string;

  @IsString()
  @MinLength(1)
  @MaxLength(100)
  author: string;

  @IsInt() // número entero
  @Min(1, { message: 'El libro tiene que tener al menos 1 página' }) // al menos 1 página
  @Max(2000) // como mucho 2000 páginas
  pages: number;
}
```

### 🧠 ¿Por qué el DTO es una `class` y no una `interface`?

| | `interface` | `class` |
|---|-------------|---------|
| Existe en ejecución | ❌ Se borra al compilar | ✅ Sigue en el JS |
| Puede llevar decoradores | ❌ | ✅ |
| Sirve para validar el body en runtime | ❌ | ✅ (con `ValidationPipe`) |

> 📌 **Clave:** los tipos de TS **no validan nada en ejecución**. Si el cliente manda
> `pages: "muchas"`, `pages: number` no lo impide. Lo impide `@IsInt()`, que **sí** se
> ejecuta. En Express lo haces a mano (`isValidPrice`); en Nest, con decoradores.

Compara con `interface Book` (`book.ts`): esa sí puede ser interfaz porque solo describe
lo que **devuelves**, no lo que tienes que validar.

### Ejemplo REAL — tipos en un controlador (`puta/api-books-nest/src/books/books.controller.ts`)

```ts
@Get() // GET /api/books?author=gabriel&maxPages=150
findAll(
    @Query('author') author?: string,
    @Query('maxPages') maxPages?: string, // llega como texto: "150"
) {
    const max = maxPages ? Number(maxPages) : undefined; // lo pasamos a número
    return this.booksService.findAll(author, max);
}

@Get(':id') // GET /api/books/1
findOne(@Param('id', ParseIntPipe) id: number) {
    return this.booksService.findOne(id);
}
```

> 📌 Fíjate en `maxPages?: string`: el comentario lo dice claro, de la URL **todo
> llega como texto**. Anotarlo como `number` sería **mentir** a TS. Para `:id` se usa
> `ParseIntPipe`, que **sí** convierte (y responde 400 si no es un número), por eso ahí
> `id: number` es verdad.

### Inyección por tipo

```ts
constructor(private readonly booksService: BooksService) {}   // controller
constructor(private readonly pool: Pool) {}                   // service
```

> 📌 Nest lee el **tipo** del parámetro (`BooksService`, `Pool`) gracias a
> `emitDecoratorMetadata` y te pasa la instancia correcta. El tipo de TS se usa aquí
> como **instrucción**, no solo como comprobación.

---

## 7.9 Errores comunes en proyectos Node + TS

| Error | Causa | Solución |
|-------|-------|----------|
| `Cannot find module './db'` | Ruta mal escrita, o en ESM falta la extensión | Revisa la ruta; en `"type": "module"` + `NodeNext` se importa `"./db.js"` |
| `Could not find a declaration file for module 'express'` | Falta `@types/express` | `npm i -D @types/express` |
| `'X' is a type and must be imported using a type-only import` | `verbatimModuleSyntax` activo | `import type { X }` |
| `Top-level 'await' expressions are only allowed when...` | Proyecto CommonJS | Mete el código en `async function start()` |
| `Argument of type 'string \| undefined' is not assignable to parameter of type 'string'` | `process.env.X` sin comprobar | `if (!x) throw ...` o `?? "valor"` |
| La API acepta datos basura aunque está tipada | Los tipos no validan en runtime | Valida a mano o con `class-validator` |

---

## ✅ Resumen rápido

1. `export` / `import { x }` (con nombre) y `export default` / `import x` (uno por archivo).
2. `import type { T }` para tipos → desaparecen del JS.
3. CommonJS (`require`) vs ESM (`import`, `"type": "module"`, top-level await).
4. Librerías JS necesitan `@types/...` en devDependencies.
5. `process.env.X` es `string | undefined` → compruébalo.
6. Express: `req`/`res` se infieren; `req.body` es `any` → **valida** y construye objetos tipados.
7. Nest: DTOs son **clases** con decoradores porque los tipos no existen en runtime.

👉 **Siguiente:** `08-ejemplos-completos-de-proyectos.md`
