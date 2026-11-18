# 📋 CHEATSHEET — NODE.JS

> Chuleta rápida de comandos y código. Teoría en los documentos 01–08 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal

### Node

| Comando | Qué hace |
|---------|----------|
| `node -v` / `npm -v` | Versión de Node / npm |
| `node` | Abre el REPL (`.exit` o Ctrl+C dos veces para salir) |
| `node hola.js` | Ejecuta un archivo |
| `node saludar.js Hector` | Pasa argumentos → `process.argv` |
| `node --watch src/index.js` | Reinicia al guardar (nativo, sin nodemon) |
| `node --env-file=.env src/index.js` | Carga el `.env` en `process.env` (Node 20.6+) |

### npm

| Comando | Qué hace |
|---------|----------|
| `npm init -y` | Crea `package.json` con valores por defecto |
| `npm install` | Instala todo lo de `package.json` → `node_modules/` |
| `npm ci` | Instalación limpia y exacta desde `package-lock.json` (CI / producción) |
| `npm install express` | Dependencia de producción (`dependencies`) |
| `npm install -D typescript tsx @types/node` | Dependencia de desarrollo (`devDependencies`) |
| `npm i -g @nestjs/cli` | Instalación global (comando disponible en todo el sistema) |
| `npm uninstall dayjs` | Desinstala |
| `npm run dev` | Ejecuta el script `dev` |
| `npm start` / `npm test` | Scripts especiales: **sin** `run` |
| `npm outdated` | Lista dependencias desactualizadas |

### npx (ejecutar binarios de `node_modules`)

| Comando | Qué hace |
|---------|----------|
| `npx tsc --init` | Crea `tsconfig.json` |
| `npx tsc --noEmit` | Comprueba tipos sin generar JS |
| `npx tsx src/index.ts` | Ejecuta TS directamente una vez |
| `npx tsx watch src/index.ts` | Ejecuta TS y reinicia al guardar |
| `npx eslint . --fix` | Lint + corrige lo automático |

---

## 2. `package.json`

```jsonc
{
  "name": "api-rastro",
  "type": "module",                       // ES Modules (import/export). Sin esto → CommonJS
  "scripts": {
    "dev": "tsx watch --env-file=.env src/index.ts",   // desarrollo
    "build": "tsc",                                     // TS → dist/
    "start": "node dist/index.js",                      // producción
    "typecheck": "tsc --noEmit",
    "lint": "eslint ."
  },
  "dependencies": { "express": "^5.3.0" },               // van a producción
  "devDependencies": { "typescript": "...", "tsx": "..." }  // solo desarrollo
}
```

| Versión | Acepta |
|---------|--------|
| `^1.11.2` | `1.x.x` ≥ 1.11.2 (minor y patch) |
| `~1.11.2` | `1.11.x` (solo patch) |
| `1.11.2` | Exactamente esa |

> 📌 `node_modules/` y `.env` → `.gitignore`. `package-lock.json` → **sí** se sube.

---

## 3. Módulos: CommonJS vs ES Modules

```js
// CommonJS ("type": "commonjs" o sin campo)
const dayjs = require('dayjs');
module.exports = { sumar };              // exportar
const { sumar } = require('./utils');    // importar (sin extensión)
```

```js
// ES Modules ("type": "module")
import express from 'express';           // default
import { readFile } from 'node:fs/promises';   // nombrado + prefijo node:
export function sumar(a, b) { return a + b; }
export default app;
import { sumar } from './utils.js';      // ⚠️ en ESM puro hay que poner .js
```

| CommonJS | ES Modules |
|----------|-----------|
| `__dirname` | `import.meta.dirname` (Node 20.11+) |
| `__filename` | `import.meta.filename` |
| `require()` síncrono | `import` estático / `await import()` dinámico |
| Sin *top-level await* | `await` en el nivel superior ✅ |
| `.cjs` fuerza CJS | `.mjs` fuerza ESM |

---

## 4. Módulos nativos

### `fs` (archivos)

```js
import { readFile, writeFile, mkdir, readdir, rm } from 'node:fs/promises';
import { existsSync } from 'node:fs';

const texto = await readFile('datos.json', 'utf8');      // leer
const datos = JSON.parse(texto);
await writeFile('datos.json', JSON.stringify(datos, null, 2));   // escribir (sobrescribe)
await mkdir('uploads', { recursive: true });              // crear carpeta (y padres)
const archivos = await readdir('public');                 // listar
await rm('tmp', { recursive: true, force: true });        // borrar carpeta
if (existsSync('.env')) process.loadEnvFile();            // comprobar si existe
```

### `path`

```js
import path from 'node:path';

path.join('public', 'img', 'logo.png');    // 'public/img/logo.png' (separador correcto en cada SO)
path.resolve('dist');                       // ruta absoluta desde process.cwd()
path.join(import.meta.dirname, 'public');   // relativa al ARCHIVO, no a la terminal
path.extname('foto.png');                   // '.png'
path.basename('/a/b/foto.png');             // 'foto.png'
```

### `http` (servidor sin Express)

```js
import http from 'node:http';

const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/api/hello') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ message: 'Hola' }));
    return;
  }
  res.writeHead(404).end('No encontrado');
});
server.listen(3000, () => console.log('http://localhost:3000'));
```

### `crypto`

```js
import { randomUUID, randomBytes, createHash } from 'node:crypto';

randomUUID();                                        // id único
randomBytes(32).toString('hex');                     // token aleatorio (sesiones)
createHash('sha256').update('texto').digest('hex');  // hash (⚠️ NO para contraseñas: usa bcrypt)
```

---

## 5. `process` y variables de entorno

```js
process.env.PORT                    // variable de entorno (siempre string o undefined)
process.argv                        // ['node', 'ruta/script.js', 'arg1', ...]
process.argv.slice(2)               // solo tus argumentos
process.cwd()                       // carpeta desde donde se lanzó el comando
process.exit(1)                     // terminar con error
process.loadEnvFile()               // carga .env (Node 20.12+, sin dotenv)
```

```js
// Formas de cargar el .env
// 1) Script:   "dev": "tsx watch --env-file=.env src/index.ts"
// 2) Código:   process.loadEnvFile();
// 3) dotenv:   import 'dotenv/config';   (npm install dotenv)
```

```js
// Valores por defecto y validación (patrón de tus APIs)
const PORT = Number(process.env.PORT ?? 3000);          // ?? solo si es null/undefined
const DB_NAME = process.env.DB_NAME || 'bootcamp_api';  // || también si es ''

const uri = process.env.MONGODB_URI;
if (!uri) throw new Error('Falta MONGODB_URI en el archivo .env');   // fallar rápido
```

```bash
# .env.example  ← se sube a Git, sin valores reales
PORT=3000
MONGODB_URI=mongodb://admin:TU_PASSWORD@localhost:27017/?authSource=admin
DB_NAME=bootcamp_api
```

---

## 6. Asincronía

```js
// Callback (estilo antiguo)
fs.readFile('a.txt', 'utf8', (err, data) => { if (err) return console.error(err); console.log(data); });

// Promesa con .then
fetch(url).then(res => res.json()).then(data => console.log(data)).catch(console.error);

// async / await  ✅
async function cargar() {
  try {
    const res = await fetch(url);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);   // fetch no lanza en 404/500
    return await res.json();
  } catch (err) {
    console.error(err);
  }
}
```

| Método | Qué hace |
|--------|----------|
| `Promise.all([p1, p2])` | En paralelo; falla si **una** falla |
| `Promise.allSettled([...])` | En paralelo; devuelve todas (ok o error) |
| `Promise.race([...])` | La primera que termine |
| `Promise.any([...])` | La primera que **salga bien** |
| `new Promise((resolve, reject) => ...)` | Crear una promesa |
| `await new Promise(r => setTimeout(r, 1000))` | Esperar 1 s |

```js
// Arranque típico de una API
async function start() {
  await connectDb();                       // primero la BD
  app.listen(PORT, () => console.log(`http://localhost:${PORT}`));
}
start();
```

### Orden del event loop

```
código síncrono → process.nextTick → microtareas (promesas) → timers (setTimeout) → I/O → setImmediate
```

---

## 7. TypeScript y herramientas

```bash
npm install -D typescript tsx @types/node
npx tsc --init
```

```jsonc
// tsconfig.json mínimo para una API
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "dist",             // JS compilado
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

| Herramienta | Para | Comando |
|-------------|------|---------|
| `tsx` | Ejecutar TS en desarrollo | `tsx watch src/index.ts` |
| `nodemon` | Reiniciar al guardar (JS) | `nodemon src/index.js` |
| `tsc` | Compilar TS → JS | `tsc` / `tsc --noEmit` |
| ESLint | Errores y estilo | `eslint .` |

### Flujo dev → build → start

```
npm run dev     → tsx watch src/index.ts       (TS directo, reinicia al guardar)
npm run build   → tsc                           (src/*.ts → dist/*.js)
npm start       → node dist/index.js            (producción: JS compilado)
```

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `Cannot use import statement outside a module` | Añade `"type": "module"` o usa `require` |
| `require is not defined in ES module scope` | Estás en ESM → usa `import` |
| `__dirname is not defined` | ESM → `import.meta.dirname` |
| `Cannot find module './utils'` en ESM | Pon la extensión: `./utils.js` |
| `EADDRINUSE: address already in use :::3000` | Ya hay un servidor en ese puerto: ciérralo o cambia `PORT` |
| `process.env.X` es `undefined` | No cargaste el `.env` (`--env-file`, `loadEnvFile`, `dotenv`) o el nombre no coincide |
| `tsc: command not found` | Usa `npx tsc` o el script de `package.json` |
| `MODULE_NOT_FOUND` tras clonar | Falta `npm install` |
