# 07 — HERRAMIENTAS: `tsx`, `nodemon`, TypeScript (`tsc`) Y ESLINT

> Con `node archivo.js` basta para ejecutar JavaScript, pero en un proyecto real
> quieres más: que el servidor **se reinicie solo** al guardar, escribir en
> **TypeScript**, **compilar** para producción y que alguien **revise tu código**
> automáticamente. Para eso están estas herramientas, todas instaladas como
> **`devDependencies`**.

---

## 7.1 El problema: reiniciar a mano

```bash
node src/index.js        # arrancas el servidor
# ... cambias el código ...
# Ctrl + C
node src/index.js        # vuelves a arrancar
# ... cambias el código ...
# Ctrl + C  😩
```

Node carga el código **una vez** al arrancar. Si lo cambias, el proceso que está
corriendo **no se entera**. Las herramientas *watch* lo resuelven: **vigilan** tus
archivos y **reinician** el proceso al guardar.

---

## 7.2 `node --watch` (sin instalar nada, ejemplo propio)

Desde Node 18.11 / 20 viene incluido:

```bash
node --watch src/index.js
```

```json
"scripts": {
  "dev": "node --watch --env-file=.env src/index.js"
}
```

> 📌 Suficiente para proyectos en **JavaScript**. Para TypeScript, mira `tsx`.

---

## 7.3 `nodemon` (el clásico)

**`nodemon`** fue durante años **la** herramienta para esto. No aparece en tus
proyectos (usan `tsx`), pero la verás en muchísimos tutoriales:

```bash
npm install -D nodemon
```

```json
"scripts": {
  "dev": "nodemon src/index.js"
}
```

```bash
npm run dev
# [nodemon] watching path(s): *.*
# [nodemon] starting `node src/index.js`
# Servidor escuchando en http://localhost:3000
# ... guardas un archivo ...
# [nodemon] restarting due to changes...
```

> 📌 Escribe `rs` + Enter en la terminal para forzar un reinicio con nodemon.

---

## 7.4 `tsx` — ejecutar TypeScript directamente (el que usas tú)

Node, por sí solo, ejecuta **JavaScript**. Para ejecutar un `.ts` habría que
**compilarlo** primero con `tsc`. **`tsx`** se salta ese paso: compila **en memoria**
al vuelo y ejecuta.

```bash
npm install -D tsx typescript
```

```bash
npx tsx src/index.ts           # ejecutar una vez
npx tsx watch src/index.ts     # ejecutar y REINICIAR al guardar (como nodemon)
```

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/package.json`)

```json
"scripts": {
  "dev": "tsx watch src/index.ts",
  "typecheck": "tsc --noEmit"
},
```

### Ejemplo REAL copiado (`api-rastro/api-rastro/package.json`)

```json
"scripts": {
  "dev": "tsx watch --env-file=.env src/index.ts",
  "lint": "eslint .",
  "typecheck": "tsc --noEmit"
},
```

### Ejemplo REAL copiado (`ProyectoReproductor/package.json`)

```json
"scripts": {"dev":"tsx watch server/index.ts", "build":"tsc && node scripts/copy-client.mjs", "check":"tsc --noEmit",  "start":"node dist/server/index.js"},
```

| Parte | Qué significa |
|-------|---------------|
| `tsx` | Ejecuta TypeScript sin compilar a disco |
| `watch` | Reinicia al guardar cualquier archivo importado |
| `--env-file=.env` | Flag de Node: carga las variables del `.env` (tema 05) |
| `src/index.ts` | El **punto de entrada** de la app |

> ⚠️ **Muy importante:** `tsx` **NO comprueba los tipos**. Solo quita los tipos y
> ejecuta. Si tienes un error de tipos, el servidor puede arrancar igual. Por eso
> todos tus proyectos tienen además un script `typecheck` / `check` con `tsc --noEmit`.

---

## 7.5 `tsc` — el compilador de TypeScript

`tsc` (*TypeScript Compiler*) hace dos cosas:

1. **Comprobar tipos** → te avisa de errores.
2. **Compilar** `.ts` → `.js` en una carpeta (normalmente `dist/`).

| Comando | Comprueba tipos | Genera `.js` | Uso |
|---------|-----------------|--------------|-----|
| `tsc` | ✅ | ✅ (en `outDir`) | `npm run build` |
| `tsc --noEmit` | ✅ | ❌ | `npm run typecheck` / `check` |
| `tsx watch ...` | ❌ | ❌ (en memoria) | `npm run dev` |

### Ejemplo REAL: compilar y ejecutar el resultado (`node-practica/prueba/package.json`)

```json
"scripts": {
  "build": "tsc",
  "start": "node dist/index.js"
},
```

```bash
npm run build        # src TS → dist/index.js
npm start            # node dist/index.js  ← Node ejecuta el JS compilado
```

### `tsconfig.json` — la configuración de `tsc`

**Ejemplo REAL copiado** (`api-rastro/api-rastro/tsconfig.json`):

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
| `target: "ES2022"` | Versión de JS que se genera (Node moderno la entiende entera) |
| `module` / `moduleResolution: "NodeNext"` | Sigue las reglas de módulos **de Node**: mira el `"type"` del `package.json` para decidir CommonJS o ESM (tema 03) |
| `strict: true` | Comprobaciones estrictas (null, any implícito...) → **siempre activado** |
| `esModuleInterop: true` | Permite `import express from "express"` con paquetes CommonJS |
| `skipLibCheck: true` | No revisa los tipos de `node_modules` (más rápido) |
| `outDir: "dist"` | Dónde deja los `.js` compilados |
| `include` | Qué archivos compila |

> 📌 `API_Godines/.../tsconfig.json` añade `"resolveJsonModule": true` para poder
> hacer `import booksData from "../data/books.json"`.

### `@types/...`: tipos para librerías de JavaScript

**Ejemplo REAL** (`ProyectoReproductor/package.json`, `devDependencies`):

```json
"@types/cookie-parser":"^1.4.8", "@types/express":"^5.0.0", "@types/node":"^22.10.5", "@types/sql.js":"^1.4.9"
```

| Paquete | Le dice a TypeScript... |
|---------|-------------------------|
| `@types/node` | Qué es `process`, `Buffer`, `fs`, `path`... |
| `@types/express` | Qué tienen `req`, `res`, `app`... |
| `@types/cookie-parser`, `@types/sql.js` | Los tipos de esas librerías |

> 📌 Sin `@types/node`, TypeScript diría `Cannot find name 'process'`.

---

## 7.6 Flujo completo: desarrollo vs. producción

```
          DESARROLLO                                PRODUCCIÓN
  ┌───────────────────────────┐          ┌───────────────────────────┐
  │ npm run dev               │          │ npm run build  (tsc)      │
  │  └─ tsx watch src/index.ts│          │  └─ src/*.ts → dist/*.js  │
  │     (rápido, se reinicia, │          │                           │
  │      sin comprobar tipos) │          │ npm start                 │
  │                           │          │  └─ node dist/index.js    │
  │ npm run typecheck         │          │     (JS puro, sin tsx ni  │
  │  └─ tsc --noEmit          │          │      TypeScript)          │
  └───────────────────────────┘          └───────────────────────────┘
```

**Ejemplo REAL — `ProyectoReproductor` tiene los dos lados:**

```json
"dev":   "tsx watch server/index.ts",
"build": "tsc && node scripts/copy-client.mjs",
"check": "tsc --noEmit",
"start": "node dist/server/index.js"
```

> 📌 Por eso `dist/` va en el `.gitignore`: **se genera** con `npm run build`.

---

## 7.7 ESLint — el "corrector" del código

**ESLint** lee tu código **sin ejecutarlo** y te avisa de errores y malas prácticas:
variables sin usar, `==` en vez de `===`, `var`, `any`...

```bash
npm install -D eslint @eslint/js typescript-eslint globals
```

### Ejemplo REAL copiado (`api-rastro/api-rastro/eslint.config.mjs`)

```js
import js from "@eslint/js";
import tseslint from "typescript-eslint";
import globals from "globals";

export default tseslint.config(
  // Carpetas que no se revisan
  { ignores: ["dist/", "coverage/", "node_modules/"] },

  // Reglas recomendadas de ESLint y de TypeScript
  js.configs.recommended,
  tseslint.configs.recommended,

  // El código se ejecuta en Node.js
  {
    languageOptions: {
      globals: globals.node,
    },
  },

  // Nuestras reglas
  {
    rules: {
      eqeqeq: "error", // === en vez de ==
      "no-var": "error", // let o const en vez de var
      "prefer-const": "error", // const si la variable no cambia
      "no-empty": "error", // nada de bloques vacíos
      "@typescript-eslint/no-explicit-any": "error", // nada de any
      "@typescript-eslint/no-unused-vars": [
        "error",
        { argsIgnorePattern: "^_" }, // _req, _next: parámetros que no usamos a propósito
      ],
    },
  },
);
```

**Desglose:**

| Bloque | Para qué |
|--------|----------|
| `ignores` | No revisar carpetas generadas |
| `js.configs.recommended` | Reglas básicas de JavaScript |
| `tseslint.configs.recommended` | Reglas para TypeScript |
| `globals: globals.node` | Le dice a ESLint que existen `process`, `Buffer`... (si no, diría "no definido") |
| `rules` | Tus reglas extra |

> 📌 Es un archivo `.mjs` → ES Module, por eso usa `import` y `export default` aunque
> el proyecto sea `"type": "commonjs"` (tema 03).

### La regla `argsIgnorePattern: "^_"` en acción

**Ejemplo REAL** (`api-rastro/api-rastro/src/index.ts`):

```ts
app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
```

> 📌 `_res`, `_req`, `_next` empiezan por `_` → "no los uso, pero tienen que estar
> ahí por la posición". ESLint no se queja gracias a esa regla.

### Ejecutarlo

```bash
npm run lint            # "lint": "eslint ."
npx eslint . --fix      # corrige automáticamente lo que pueda
```

```
src/routes/listings.ts
  12:7  error  'total' is never reassigned. Use 'const' instead  prefer-const
  30:9  error  Expected '===' and instead saw '=='               eqeqeq
```

### ❌ / ✅ con las reglas de tu configuración

```ts
// ❌ eqeqeq
if (req.query.maxPrice == "") {}
// ✅
if (req.query.maxPrice === "") {}

// ❌ no-var / prefer-const
var PORT = 3000;
let app = express();          // nunca se reasigna
// ✅
const PORT = 3000;
const app = express();

// ❌ no-explicit-any
function guardar(data: any) {}
// ✅
function guardar(data: unknown) {}

// ❌ no-empty
try { algo(); } catch (e) {}
```

---

## 7.8 Comprobaciones automáticas en GitHub

`api-rastro` y `mongo-api-libros` tienen un archivo `.github/workflows/ci.yml`: ahí
se configura **GitHub Actions** para lanzar tus scripts de npm automáticamente en
cada *push* o *pull request* a `main`.

### Ejemplo REAL copiado (fragmento de `api-rastro/api-rastro/.github/workflows/ci.yml`)

```yaml
  # Check 2: revisa el código TypeScript con ESLint (tu script "lint")
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Descargar el código
        uses: actions/checkout@v7
      - name: Preparar Node.js
        uses: actions/setup-node@v7
        with:
          node-version: 24
          cache: npm
      - name: Instalar dependencias
        run: npm ci
      - name: Lint
        run: npm run lint
      # Verificación de tipos con TypeScript
      - name: Comprobar los tipos
        run: npm run typecheck
```

**Lo que hace el servidor de GitHub, en orden:**

```bash
# descarga tu código, instala Node 24 y...
npm ci               # instalación limpia y exacta (usa package-lock.json)
npm run lint         # ESLint
npm run typecheck    # tsc --noEmit
```

> 📌 Si alguno falla, GitHub marca el commit con ❌. Así nadie sube código roto
> sin darse cuenta.

---

## 7.9 Tabla resumen de herramientas

| Herramienta | Qué hace | Script típico | ¿En tus proyectos? |
|-------------|----------|---------------|--------------------|
| `node` | Ejecuta JavaScript | `"start": "node dist/index.js"` | ✅ |
| `node --watch` | Reinicia al guardar (JS) | `"dev": "node --watch src/index.js"` | — |
| `nodemon` | Reinicia al guardar (clásico) | `"dev": "nodemon src/index.js"` | — |
| `tsx` | Ejecuta TS sin compilar (+ `watch`) | `"dev": "tsx watch src/index.ts"` | ✅ Todos los de TS |
| `tsc` | Comprueba tipos y compila | `"build": "tsc"`, `"typecheck": "tsc --noEmit"` | ✅ |
| `eslint` | Revisa calidad del código | `"lint": "eslint ."` | ✅ `api-rastro`, `mongo-api-libros` |

---

## ✅ Resumen rápido

1. Node no se entera de tus cambios: usa un modo *watch* (`tsx watch`, `node --watch`, `nodemon`).
2. **`tsx`** ejecuta TypeScript directamente, pero **no comprueba tipos**.
3. **`tsc --noEmit`** comprueba tipos; **`tsc`** compila a `dist/`.
4. Desarrollo: `npm run dev`. Producción: `npm run build` + `npm start`.
5. `tsconfig.json` con `module: "NodeNext"` y `strict: true`.
6. `@types/node` da tipos a `process`, `fs`, etc.
7. **ESLint** revisa el código; `globals.node` para que conozca los globales de Node; `_param` para parámetros sin usar.

👉 **Siguiente:** `08-ejemplos-completos-de-proyectos.md`
