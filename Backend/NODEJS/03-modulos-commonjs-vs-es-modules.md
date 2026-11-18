# 03 — MÓDULOS: COMMONJS VS. ES MODULES

> Un **módulo** es simplemente **un archivo** que comparte (exporta) cosas para que
> otros archivos las usen (importen). Así repartes el código en piezas pequeñas en
> vez de tener un archivo gigante.
> En Node existen **dos sistemas** de módulos, y vas a ver los dos en tus proyectos.

---

## 3.1 Los dos sistemas de un vistazo

| | **CommonJS** (CJS) | **ES Modules** (ESM) |
|---|--------------------|----------------------|
| Importar | `const x = require("x")` | `import x from "x"` |
| Exportar | `module.exports = ...` | `export` / `export default` |
| Origen | El sistema "clásico" de Node (2009) | El estándar oficial de JavaScript (ES6, 2015) |
| Se activa con | `"type": "commonjs"` (o sin `type`) / extensión `.cjs` | `"type": "module"` / extensión `.mjs` |
| `await` fuera de funciones | ❌ No | ✅ Sí (*top-level await*) |
| `__dirname`, `__filename` | ✅ Sí | ❌ No (se usa `import.meta`) |
| Navegador | ❌ No lo entiende | ✅ Sí (`<script type="module">`) |

> 📌 **Hoy se recomienda ES Modules** para proyectos nuevos, pero CommonJS sigue
> estando en muchísimos proyectos y tutoriales, así que tienes que leer los dos.

---

## 3.2 El mismo programa en los dos sistemas

Tienes un ejemplo perfecto en tu carpeta `node-practica/prueba/`: el **mismo**
programa escrito de las dos formas.

### Ejemplo REAL copiado (`node-practica/prueba/index.js`) → **CommonJS**

```js
const dayjs = require("dayjs");

console.log("Hoy es:", dayjs().format("DD/MM/YYYY HH:mm:ss"));
```

### Ejemplo REAL copiado (`node-practica/prueba/index.ts`) → **sintaxis ESM (TypeScript)**

```ts
import dayjs from "dayjs";

console.log("Hoy es:", dayjs().format());
```

**Diferencias:**

```
const dayjs = require("dayjs");     ← CommonJS: una función normal que devuelve el módulo
import dayjs from "dayjs";          ← ESM: una palabra reservada del lenguaje
```

> 📌 Ojo con un detalle: ese proyecto tiene `"type": "commonjs"`. En TypeScript
> escribes `import`, pero al compilar con `tsc` (`npm run build`) se puede generar
> JavaScript con `require`. **TypeScript traduce la sintaxis** según su configuración.
> Lo que decide el sistema que usa **Node** al ejecutar es el `"type"` del `package.json`
> y la extensión del archivo.

---

## 3.3 CommonJS en detalle: `require` y `module.exports`

```js
// 📄 matematicas.js  (ejemplo propio)
function suma(a, b) {
  return a + b;
}

function resta(a, b) {
  return a - b;
}

const PI = 3.1416;

// Exportar VARIAS cosas: un objeto
module.exports = { suma, resta, PI };
```

```js
// 📄 index.js
const matematicas = require("./matematicas");    // ← ruta relativa, sin .js
console.log(matematicas.suma(2, 3));             // 5

// O con destructuring (lo más habitual)
const { suma, PI } = require("./matematicas");
console.log(suma(1, PI));
```

### Exportar UNA sola cosa

```js
// 📄 saludar.js
module.exports = function saludar(nombre) {
  return `Hola, ${nombre}`;
};

// 📄 index.js
const saludar = require("./saludar");
saludar("Hector");   // "Hola, Hector"
```

### ❌ Error típico: `exports = ...`

```js
// ❌ NO funciona: reasignas la variable local, no lo que se exporta
exports = { suma };

// ✅ Funciona: añades propiedades
exports.suma = suma;

// ✅ Funciona: reasignas module.exports
module.exports = { suma };
```

---

## 3.4 ES Modules en detalle: `import` y `export`

```js
// 📄 matematicas.js  (ejemplo propio, ESM)
export function suma(a, b) {      // export "con nombre" (named)
  return a + b;
}

export const PI = 3.1416;

export default function resta(a, b) {   // export POR DEFECTO (solo uno por archivo)
  return a - b;
}
```

```js
// 📄 index.js
import resta, { suma, PI } from "./matematicas.js";   // ⚠️ en ESM puro, la extensión .js es OBLIGATORIA
//     ▲        ▲
//  default   con nombre (entre llaves, MISMO nombre)

import * as mates from "./matematicas.js";   // todo junto en un objeto
mates.suma(1, 2);
```

| Tipo | Exportar | Importar |
|------|----------|----------|
| **Con nombre** | `export function suma() {}` | `import { suma } from "./x.js"` |
| **Por defecto** | `export default router` | `import cualquierNombre from "./x.js"` |
| **Renombrar** | — | `import { suma as sumar } from "./x.js"` |
| **Solo tipos** (TS) | `export interface Book {}` | `import type { Book } from "./x"` |

### Ejemplo REAL: exportar funciones con nombre (`api-rastro/api-rastro/src/db.ts`)

```ts
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
```

### Ejemplo REAL: importarlas (`api-rastro/api-rastro/src/index.ts`)

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import { connectDb, getDb } from "./db";
import listingsRouter from "./routes/listings";
import offersRouter from "./routes/offers";
```

**Desglose:**

```
import express from "express";                 → default de un paquete de node_modules
import type { Request, ... } from "express";   → solo TIPOS (TypeScript, desaparecen al compilar)
import { connectDb, getDb } from "./db";       → exports con nombre de un archivo tuyo
import listingsRouter from "./routes/listings";→ export default de un archivo tuyo
```

> 📌 En TypeScript ejecutado con `tsx` no hace falta poner `.js` en las rutas
> (`"./db"`). En **JavaScript ESM puro** con `node`, sí: `"./db.js"`.

---

## 3.5 Activar ES Modules: `"type": "module"` y `.mjs`

### Opción 1: en el `package.json` (todo el proyecto)

**Ejemplo REAL** (`ProyectoReproductor/package.json`):

```json
{
  "name": "nightjar-music",
  "private": true,
  "type": "module",
  ...
}
```

Con `"type": "module"`, **todos** los `.js` del proyecto son ESM.

### Opción 2: la extensión del archivo

| Extensión | Siempre es... (da igual el `"type"`) |
|-----------|--------------------------------------|
| `.mjs` | ES Module |
| `.cjs` | CommonJS |
| `.js` | Lo que diga `"type"` (por defecto, CommonJS) |

**Ejemplo REAL** (`ProyectoReproductor/scripts/copy-client.mjs`):

```js
import { copyFileSync, mkdirSync } from 'node:fs';
mkdirSync('public', { recursive: true });
copyFileSync('dist/src/login.js', 'public/login.js');
copyFileSync('dist/src/app.js', 'public/app.js');
```

> 📌 Usa `import` aunque esté en una carpeta suelta porque **termina en `.mjs`**.
> Lo mismo pasa con `eslint.config.mjs` en `api-rastro` y `mongo-api-libros`.

### ❌ Mezclar sistemas

```js
// En un archivo CommonJS (.js con "type": "commonjs")
import dayjs from "dayjs";
// ❌ SyntaxError: Cannot use import statement outside a module

// En un archivo ESM (.mjs o "type": "module")
const dayjs = require("dayjs");
// ❌ ReferenceError: require is not defined in ES module scope
```

---

## 3.6 *Top-level await* (solo ESM)

En ES Modules puedes usar `await` **fuera** de una función `async`.

**Ejemplo REAL** (`ProyectoReproductor/server/index.ts`, con `"type": "module"`):

```ts
const app = express();
const SQL = await initSqlJs();      // ← await en el nivel superior del archivo
let db: SqlDatabase;
```

En CommonJS esto no se puede, así que se envuelve en una función:

**Ejemplo REAL** (`api-rastro/api-rastro/src/index.ts`, con `"type": "commonjs"`):

```ts
async function start() {
  await connectDb();

  app.listen(PORT, () => {
    console.log(`Servidor escuchando en http://localhost:${PORT}`);
  });
}

start();
```

> 📌 Los dos hacen lo mismo: **esperar** a que la base de datos esté lista **antes**
> de empezar a escuchar peticiones.

---

## 3.7 El prefijo `node:` para módulos nativos

Los módulos que trae Node (`fs`, `path`, `crypto`, `http`...) se pueden importar con
el prefijo `node:`.

**Ejemplo REAL** (`ProyectoReproductor/server/index.ts`):

```ts
import { randomBytes, scryptSync, timingSafeEqual } from 'node:crypto';
import { join } from 'node:path';
import { mkdirSync, readFileSync, writeFileSync } from 'node:fs';
```

```js
import fs from "fs";          // ✅ funciona
import fs from "node:fs";     // ✅ funciona y es MÁS CLARO: "esto es de Node, no de npm"
```

> 📌 Recomendado: usa siempre `node:`. Así nadie confunde un módulo nativo con un
> paquete de npm que se llame igual.

---

## 3.8 Importar solo por sus efectos

A veces importas un módulo **no para usar algo suyo**, sino porque al cargarse
**hace algo**.

**Ejemplo REAL** (`ProyectoReproductor/server/index.ts`, primera línea):

```ts
import 'dotenv/config';
```

> 📌 No guarda nada en una variable: simplemente, al cargarse, lee el archivo `.env`
> y rellena `process.env`. Por eso tiene que ser **la primera línea** (tema 05).

---

## 3.9 `__dirname` en CommonJS vs. ESM

```js
// CommonJS
console.log(__dirname);    // C:\...\mi-proyecto\src
console.log(__filename);   // C:\...\mi-proyecto\src\index.js
```

```js
// ESM: __dirname NO existe
console.log(import.meta.dirname);    // Node 20.11+
console.log(import.meta.filename);

// Forma antigua (compatible con versiones viejas)
import { fileURLToPath } from "node:url";
import { dirname } from "node:path";
const __dirname = dirname(fileURLToPath(import.meta.url));
```

> 📌 Tu `ProyectoReproductor` usa `process.cwd()` en su lugar (la carpeta desde la que
> ejecutas `npm run dev`), lo que evita el problema:
> ```ts
> const dbPath = join(process.cwd(), relativePath);
> ```

---

## 3.10 Importar JSON

**Ejemplo REAL** (`API_Godines/api-express/api-books-claude/src/services/books-storage-in-memory.ts`):

```ts
import booksData from "../data/books.json";
import type { Book, BookData, BooksStorage } from "./books-storage";
```

Funciona porque su `tsconfig.json` activa:

```json
"resolveJsonModule": true,
```

```js
// En CommonJS puro también se puede:
const books = require("./data/books.json");
```

---

## 3.11 Errores comunes con módulos

| Error | Causa | Solución |
|-------|-------|----------|
| `Cannot use import statement outside a module` | `import` en un archivo CommonJS | `"type": "module"` o extensión `.mjs` |
| `require is not defined in ES module scope` | `require` en ESM | Cámbialo por `import` |
| `Cannot find module './db'` (ESM puro) | Falta la extensión | `"./db.js"` |
| `X is not a function` al importar | Confundir default y con nombre | `import { x }` vs `import x` |
| `does not provide an export named 'x'` | Importas con nombre algo que no se exporta así | Revisa el `export` del otro archivo |
| `__dirname is not defined` | Estás en ESM | `import.meta.dirname` o `process.cwd()` |

---

## ✅ Resumen rápido

1. Un **módulo** = un archivo que exporta e importa cosas.
2. **CommonJS**: `require()` / `module.exports`. **ESM**: `import` / `export`.
3. Lo decide `"type"` en `package.json` (`"commonjs"` o `"module"`) o la extensión (`.cjs` / `.mjs`).
4. `export default` → se importa sin llaves; `export` con nombre → con `{ llaves }`.
5. Rutas propias con `./`; paquetes y nativos sin `./`. Nativos mejor con `node:`.
6. *Top-level await* solo en ESM; en CommonJS usa `async function start() {}`.
7. `import 'dotenv/config'` = importar por efectos (va el primero).

👉 **Siguiente:** `04-modulos-nativos-fs-path-http.md`
