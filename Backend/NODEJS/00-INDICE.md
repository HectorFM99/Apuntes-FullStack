# 🟢 CARPETA NODEJS — ÍNDICE DE ESTUDIO

Teoría de **Node.js** explicada con ejemplos reales **copiados y pegados de tus
proyectos** del Bootcamp.

> ⚠️ Los proyectos **NO** se han modificado. Solo se han copiado fragmentos de su código.
> 🔒 Nunca se copia el contenido de un archivo `.env` (tiene secretos). Como mucho se
> muestra el `.env.example`, que es la plantilla sin valores reales.

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a aprender |
|---|---------|--------------------|
| 1 | `01-que-es-node-y-como-ejecutar.md` | Qué es Node, V8, *runtime*, hilo único, event loop, I/O no bloqueante, `node archivo.js` |
| 2 | `02-npm-package-json-y-dependencias.md` | `npm init`, `package.json`, `scripts`, `dependencies` vs `devDependencies`, `node_modules`, `package-lock.json`, versiones `^` y `~` |
| 3 | `03-modulos-commonjs-vs-es-modules.md` | `require`/`module.exports` vs `import`/`export`, el campo `"type"`, `.mjs`/`.cjs`, prefijo `node:` |
| 4 | `04-modulos-nativos-fs-path-http.md` | `fs` (leer/escribir archivos), `path` (rutas), `http` (servidor sin Express), `crypto` |
| 5 | `05-process-y-variables-de-entorno.md` | `process.env`, `process.cwd()`, `process.argv`, `.env`, `dotenv`, `--env-file`, `.env.example` |
| 6 | `06-asincronia-callbacks-promesas-async-await.md` | Callbacks → Promesas → `async`/`await`, `Promise.all`, errores con `try/catch` |
| 7 | `07-herramientas-tsx-nodemon-typescript-eslint.md` | `tsx watch`, `nodemon`, `tsc`, `tsconfig.json`, ESLint, flujo dev/build/start |
| 8 | `08-ejemplos-completos-de-proyectos.md` | Archivos enteros copiados de tus proyectos |
| 📋 | `99-CHEATSHEET.md` | Chuleta de comandos y código para consultar rápido |

---

## 🗂️ Proyectos de los que se han extraído los ejemplos

| Proyecto | Archivo(s) | Temas que cubre |
|----------|-----------|-----------------|
| `node-practica/` | `hola.js` | Tu primer script en Node, `node hola.js` |
| `node-practica/prueba/` | `index.js`, `index.ts`, `package.json` | `require` vs `import`, dependencia externa (`dayjs`), `scripts`, `"type": "commonjs"` |
| `Async-hector/Advanced-JS-Asynchronous-code/` | `index.js` | `async`/`await`, `fetch`, `.then()`, `Promise.all` |
| `Async-Hector-2/advanced-js-bikram-async-await/` | `index.js` | `async`/`await` encadenados, varias peticiones seguidas |
| `ProyectoReproductor/` | `server/index.ts`, `package.json`, `scripts/copy-client.mjs`, `.env.example` | ES Modules, `dotenv`, `process.env`, `fs`, `path`, `crypto`, *top-level await*, `tsx watch` |
| `api-rastro/api-rastro/` | `package.json`, `src/db.ts`, `src/index.ts`, `eslint.config.mjs` | `--env-file`, validar variables de entorno, `async function start()`, ESLint |
| `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/` | `package.json`, `src/db.ts`, `src/index.ts` | Scripts `dev`/`lint`/`typecheck`/`seed`, `??` y `\|\|` para valores por defecto |
| `API_Godines/api-express/api-books-claude/` | `package.json`, `src/index.ts`, `src/services/*.ts` | Proyecto mínimo con `tsx`, métodos que devuelven `Promise` |

---

## 🚀 Orden de estudio recomendado

1. `01` → entender **qué es Node** y por qué es "asíncrono" (sin esto, lo demás no tiene sentido).
2. `02` → **npm y `package.json`**: lo primero que ves al abrir cualquier proyecto.
3. `03` → **módulos**: cómo se reparten el código los archivos (`require` / `import`).
4. `04` → los **módulos que trae Node** de serie (`fs`, `path`, `http`).
5. `05` → **variables de entorno**: configuración y secretos fuera del código.
6. `06` → **asincronía**: callbacks, promesas y `async`/`await` (el tema más importante).
7. `07` → **herramientas** del día a día (`tsx`, `tsc`, ESLint).
8. `08` → código real completo de tus proyectos.

> 📌 Express tiene su **propia carpeta** (`Apuntes/EXPRESS/`). Aquí solo aparece de pasada
> para que veas dónde encaja Node.

---

## 💡 Diferencia clave: JavaScript en el navegador vs. en Node

```js
// 🌐 NAVEGADOR: hay página, hay DOM
document.querySelector("h1").textContent = "Hola";
window.alert("Hola");
```

```js
// 🟢 NODE: no hay página ni DOM, pero hay disco, red y sistema operativo
const fs = require("fs");
fs.writeFileSync("saludo.txt", "Hola desde Node");
console.log(process.env.PORT);
```

| | Navegador | Node.js |
|---|-----------|---------|
| Dónde se ejecuta | Dentro de Chrome, Firefox... | En tu terminal / un servidor |
| Objeto global | `window` | `globalThis` (y `global`) |
| DOM (`document`) | ✅ Sí | ❌ No existe |
| Leer archivos del disco | ❌ No | ✅ `fs` |
| Crear un servidor | ❌ No | ✅ `http`, Express... |
| `fetch` | ✅ Sí | ✅ Sí (desde Node 18) |
| Para qué lo usas en el bootcamp | Frontend | **Backend / APIs** |
