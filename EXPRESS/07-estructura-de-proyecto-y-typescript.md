# 07 — ESTRUCTURA DE PROYECTO Y EXPRESS CON TYPESCRIPT

> Express no te impone carpetas. Pero tus proyectos siguen una estructura muy parecida,
> y conocerla te permite empezar cualquier API nueva en 5 minutos. Además, con
> **TypeScript** los errores de tipos (un `price` que es string, un campo mal escrito...)
> aparecen **antes** de ejecutar.

---

## 7.1 Estructura de carpetas de tus proyectos

### `API_Godines/api-express/api-books-claude/` (arquitectura con interfaz)

```
api-books-claude/
├── docs/
│   └── prd-books.md                     ← qué tiene que hacer la API (requisitos)
├── src/
│   ├── index.ts                         ← arranque: middleware, routers, 404, errores, listen
│   ├── data/
│   │   └── books.json                   ← datos iniciales
│   ├── routes/
│   │   └── books-routes.ts              ← rutas HTTP de /api/books
│   └── services/
│       ├── books-storage.ts             ← interfaz (contrato) + tipos Book/BookData
│       └── books-storage-in-memory.ts   ← implementación en memoria
├── package.json
├── tsconfig.json
└── .gitignore
```

### `api-rastro/api-rastro/` (Express + MongoDB)

```
api-rastro/
├── .github/workflows/ci.yml   ← GitHub Actions: lint + typecheck en cada push
├── src/
│   ├── index.ts               ← arranque
│   ├── db.ts                  ← conexión a MongoDB + colecciones tipadas
│   ├── types.ts               ← tipos de los documentos (User, Listing, Offer)
│   └── routes/
│       ├── listings.ts        ← /api/listings
│       └── offers.ts          ← /api/offers
├── .env                       ← 🔒 credenciales reales (NO en Git)
├── .env.example               ← plantilla de variables
├── eslint.config.mjs
├── package.json
└── tsconfig.json
```

### `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/`

```
02b-express-mongodb/
├── compose.yaml               ← levanta MongoDB con Docker
├── mongo/seed.js              ← datos de ejemplo para la BD
├── src/
│   ├── index.ts
│   ├── db.ts
│   ├── routes/
│   │   ├── products.ts
│   │   └── categories.ts
│   └── soluciones/            ← variantes de ejercicios (no montadas por defecto)
│       ├── ejercicio-07-patch-updateOne.ts
│       └── reto-1-products-id.ts
├── .env / .env.example
└── package.json, tsconfig.json, eslint.config.mjs
```

### `ProyectoReproductor/` (fullstack en un solo servidor)

```
ProyectoReproductor/
├── server/index.ts     ← TODO el backend (Express + SQLite + auth)
├── public/             ← frontend que sirve express.static
├── src/                ← código del cliente
├── data/               ← archivo SQLite
└── package.json, tsconfig.json
```

| Carpeta / archivo | Responsabilidad |
|-------------------|-----------------|
| `src/index.ts` | **Solo** montar la app: middleware, routers, 404, errores, `listen` |
| `src/routes/` | Un archivo por recurso con su `Router` |
| `src/services/` | Lógica y acceso a datos detrás de una interfaz |
| `src/db.ts` | Conexión a la base de datos |
| `src/types.ts` | Tipos compartidos |
| `docs/` | Requisitos (PRD), decisiones |

> 📌 Si `index.ts` empieza a tener lógica de negocio, es señal de que algo debería ir a
> `routes/` o `services/`.

---

## 7.2 `package.json`: scripts

### Ejemplo REAL copiado (`api-rastro/api-rastro/package.json`, fragmento)

```json
  "scripts": {
    "dev": "tsx watch --env-file=.env src/index.ts",
    "lint": "eslint .",
    "typecheck": "tsc --noEmit"
  },
  "type": "commonjs",
  "devDependencies": {
    "@eslint/js": "^10.0.1",
    "@types/express": "^5.0.6",
    "@types/node": "^26.6.6",
    "eslint": "^10.14.0",
    "globals": "^17.14.0",
    "tsx": "^4.23.15",
    "typescript": "~6.0",
    "typescript-eslint": "^8.72.1"
  },
  "dependencies": {
    "express": "^5.3.0",
    "mongodb": "^7.8.0"
  }
```

| Script | Comando | Para qué |
|--------|---------|----------|
| `npm run dev` | `tsx watch --env-file=.env src/index.ts` | Desarrollo: ejecuta TS, recarga al guardar, carga `.env` |
| `npm run lint` | `eslint .` | Revisa estilo y errores comunes |
| `npm run typecheck` | `tsc --noEmit` | Revisa tipos sin generar JS |

### Con build para producción (`ProyectoReproductor/package.json`)

```json
  "type": "module",
  "scripts": {"dev":"tsx watch server/index.ts", "build":"tsc && node scripts/copy-client.mjs", "check":"tsc --noEmit",  "start":"node dist/server/index.js"},
```

| Script | Para qué |
|--------|----------|
| `dev` | Desarrollo con `tsx` |
| `build` | `tsc` compila TS → JS en `dist/` |
| `start` | **Producción**: ejecuta el JS ya compilado con `node` (más rápido, sin `tsx`) |

> 📌 `"type": "module"` (ESM) permite usar `await` en el nivel superior del archivo, como
> hace el reproductor con `const SQL = await initSqlJs();`. Con `"type": "commonjs"` eso
> no funciona y hay que envolverlo en una función `async` (como el `start()` de `api-rastro`).

---

## 7.3 `tsconfig.json`

### Ejemplo REAL copiado (`api-rastro/api-rastro/tsconfig.json`)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "dist"
  },
  "include": ["src/**/*.ts"]
}
```

| Opción | Qué hace |
|--------|----------|
| `target: "ES2022"` | Versión de JS que genera (moderna: `?.`, `??`, clases...) |
| `module` / `moduleResolution: "NodeNext"` | Resuelve `import` igual que Node |
| `strict: true` | Todas las comprobaciones estrictas (**siempre** actívalo) |
| `esModuleInterop: true` | Permite `import express from "express"` (Express es CommonJS) |
| `skipLibCheck: true` | No revisa los tipos de `node_modules` (más rápido) |
| `outDir: "dist"` | Dónde deja el JS compilado |
| `include` | Qué archivos compila |

### Leer JSON con `import` (`api-books-claude/tsconfig.json`)

```json
    "resolveJsonModule": true,
```

Esto permite hacer, en `books-storage-in-memory.ts`:

```ts
import booksData from "../data/books.json";
```

y TypeScript **deduce el tipo** del JSON automáticamente.

---

## 7.4 Tipos de Express: `Request`, `Response`, `NextFunction`

En las rutas **no hace falta tiparlos**: TypeScript los deduce porque sabe que el
callback de `router.get` recibe un `Request` y un `Response`.

```ts
router.get("/", async (req, res) => {
  //                   ▲    ▲  TypeScript ya sabe que son Request y Response
});
```

Pero cuando escribes un middleware o una función **suelta**, sí hay que importarlos.

### Ejemplo REAL copiado (`api-books-claude/src/index.ts`)

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";

// ...

app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  // ...
});
```

### Ejemplo REAL copiado (`api-books-claude/src/routes/books-routes.ts`)

```ts
import type { Response } from "express";

// Respuesta 404 común a todas las rutas con :id
function sendNotFound(res: Response, id: number): void {
  res.status(404).json({ error: `No existe ningún libro con id ${id}` });
}
```

### Con el espacio de nombres (`ProyectoReproductor/server/index.ts`)

```ts
function session(req: express.Request, res: express.Response, next: express.NextFunction) { ... }
```

`express.Request` y `Request` (importado) son **lo mismo**; es cuestión de estilo.

> 📌 `import type { ... }` importa **solo tipos**: desaparece al compilar a JS. Úsalo
> siempre que lo que importas sea un tipo/interfaz.

---

## 7.5 `err: unknown` en el manejador de errores

```ts
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  if (err instanceof SyntaxError) {     // ← "estrechar" el tipo antes de usarlo
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }
  // ...
});
```

| Tipo | Qué te deja hacer |
|------|-------------------|
| `any` | Cualquier cosa sin comprobar (peligroso; tu ESLint lo **prohíbe**) |
| `unknown` | Nada hasta que compruebes qué es (`instanceof`, `typeof`) ✅ |

Tu `eslint.config.mjs` (`api-rastro/api-rastro/eslint.config.mjs`) lo deja claro:

```js
      "@typescript-eslint/no-explicit-any": "error", // nada de any
      "@typescript-eslint/no-unused-vars": [
        "error",
        { argsIgnorePattern: "^_" }, // _req, _next: parámetros que no usamos a propósito
      ],
```

> 📌 Y de ahí el `_req` y `_next`: ESLint ignora los parámetros que empiezan por `_`.

---

## 7.6 Tipar los datos que llegan: `unknown` + comprobaciones

`req.body` es `any` en los tipos de Express. Tus proyectos lo convierten a algo más
seguro.

### Ejemplo REAL copiado (`api-books-claude/src/routes/books-routes.ts`)

```ts
// Parámetros de la URL; `unknown` obliga a comprobar el valor antes de usarlo
type Query = Record<string, unknown>;

// ...
router.post("/", async (req, res) => {
  const body: Query = req.body ?? {};
  // body.title es unknown → hay que validar antes de usarlo como string
});
```

```ts
// ❌ any: TypeScript no te avisa de nada
const title = req.body.title;
title.trim();        // si title es un número → TypeError en ejecución

// ✅ unknown: TypeScript te OBLIGA a comprobar
const body: Record<string, unknown> = req.body ?? {};
if (typeof body.title === "string") {
  body.title.trim(); // ✅ seguro
}
```

---

## 7.7 Tipos de los documentos de la BD

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/types.ts`)

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

**Qué ganas:**

```ts
const newOffer: Offer = {
  amount,
  status: "PENDIENTE",   // ❌ TypeScript: no es "PENDING" | "ACCEPTED" | "REJECTED"
  // ...
};
```

- **Uniones de strings** (`"AVAILABLE" | "RESERVED" | "SOLD"`) → solo valores permitidos.
- `getDb().collection<Listing>("listings")` → `find`, `insertOne`... saben qué campos hay.
- El `_id` **no** está en el tipo: MongoDB lo añade. Al leer, el driver devuelve
  `WithId<Listing>` (el tipo + `_id`).

### Tipos utilitarios que usas (`api-books-claude`)

| Tipo | Significado | Uso |
|------|-------------|-----|
| `Omit<Book, "id">` | `Book` sin `id` | Datos para crear |
| `Partial<BookData>` | Todos los campos opcionales | PATCH |
| `Record<string, string>` | Objeto con claves y valores string | Objeto de errores |
| `WithId<Product>` | `Product` + `_id` (de MongoDB) | `toApi()` |
| `Filter<Listing>` | Filtro de consulta de MongoDB válido para `Listing` | `GET /api/listings` |

---

## 7.8 Tipar `res.locals` y `req` extendido (no está en tus proyectos)

En el reproductor `res.locals.user` es `any`. Para tiparlo hay varias opciones; la más
sencilla:

```ts
// Ejemplo propio
type SessionUser = { id: string; email: string };

app.get("/api/library/:kind", session, (req, res) => {
  const user = res.locals.user as SessionUser;
  // user.id ahora es string
});
```

O, más avanzado, ampliar los tipos de Express (*declaration merging*):

```ts
// Ejemplo propio: src/types/express.d.ts
declare global {
  namespace Express {
    interface Locals {
      user?: { id: string; email: string };
    }
  }
}
export {};
```

---

## 7.9 Plantilla para empezar una API nueva

Resumen de todo lo anterior, basado en tus proyectos (ejemplo propio montado con sus
piezas):

```bash
mkdir mi-api && cd mi-api
npm init -y
npm install express
npm install -D typescript tsx @types/express @types/node
npx tsc --init
mkdir -p src/routes src/services
```

```ts
// src/index.ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import itemsRouter from "./routes/items";

const app = express();
const PORT = Number(process.env.PORT || 3000);

app.use((req, _res, next) => { console.log(`${req.method} ${req.url}`); next(); });   // 1. logger
app.use(express.json());                                                              // 3. body JSON
app.use("/api/items", itemsRouter);                                                   // 4. rutas
app.use((req, res) => {                                                               // 5. 404
  res.status(404).json({ error: `No existe la ruta ${req.method} ${req.url}` });
});
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {        // 6. errores
  if (err instanceof SyntaxError) { res.status(400).json({ error: "El body no es un JSON válido" }); return; }
  console.error(err);
  res.status(500).json({ error: "Error interno del servidor" });
});

app.listen(PORT, () => console.log(`Servidor escuchando en http://localhost:${PORT}`));
```

```ts
// src/routes/items.ts
import { Router } from "express";

const router = Router();
const items: { id: number; name: string }[] = [];

router.get("/", (_req, res) => {
  res.json(items);
});

router.post("/", (req, res) => {
  const { name } = req.body ?? {};
  if (typeof name !== "string" || name.trim() === "") {
    res.status(400).json({ error: "name tiene que ser un texto no vacío" });
    return;
  }
  const item = { id: Math.max(0, ...items.map((i) => i.id)) + 1, name: name.trim() };
  items.push(item);
  res.status(201).json(item);
});

export default router;
```

```json
// package.json → scripts
"dev": "tsx watch src/index.ts",
"typecheck": "tsc --noEmit"
```

---

## ✅ Resumen rápido

1. `src/index.ts` monta la app; `routes/` un archivo por recurso; `services/` o `db.ts` para datos.
2. Scripts: `dev` (tsx watch), `typecheck` (tsc --noEmit), `lint`, y `build`/`start` para producción.
3. `tsconfig`: `strict: true`, `NodeNext`, `esModuleInterop`.
4. `import type { Request, Response, NextFunction } from "express"` para middlewares y helpers.
5. `unknown` en vez de `any`; compruebas con `typeof`/`instanceof`.
6. Tipos de documentos en `types.ts` + colecciones tipadas.
7. `Omit`, `Partial`, `Record`, `WithId`, `Filter` son tus mejores amigos.

👉 **Siguiente:** `08-ejemplos-completos-de-proyectos.md`
