# 04 — MÓDULOS NATIVOS: `fs`, `path`, `http` (y `crypto`)

> Node trae de serie un montón de **módulos nativos** (*core modules*). No hay que
> instalarlos con npm: solo importarlos. Los más importantes para empezar son:
> **`fs`** (archivos), **`path`** (rutas), **`http`** (servidores) y **`crypto`**
> (seguridad).

---

## 4.1 Lista de módulos nativos útiles

| Módulo | Para qué sirve | ¿Aparece en tus proyectos? |
|--------|----------------|-----------------------------|
| `node:fs` | Leer, escribir, crear carpetas, copiar archivos | ✅ `ProyectoReproductor` |
| `node:path` | Unir y manipular rutas de forma segura | ✅ `ProyectoReproductor` |
| `node:crypto` | Números aleatorios seguros, hash de contraseñas | ✅ `ProyectoReproductor` |
| `node:http` | Crear un servidor web (lo usa Express por dentro) | Indirectamente (Express) |
| `node:url` | Trabajar con URLs | — |
| `node:os` | Info del sistema (CPU, memoria, usuario) | — |
| `node:events` | Emitir y escuchar eventos propios | — |
| `process` | Global (no se importa) → tema 05 | ✅ Todos |

---

## 4.2 `fs` — File System (sistema de archivos)

`fs` tiene **tres "sabores"** de cada función:

```js
import fs from "node:fs";
import { readFile } from "node:fs/promises";

// 1️⃣ SÍNCRONO (bloquea)
const texto1 = fs.readFileSync("notas.txt", "utf-8");

// 2️⃣ CALLBACK (no bloquea, estilo antiguo)
fs.readFile("notas.txt", "utf-8", (err, texto2) => {
  if (err) return console.error(err);
  console.log(texto2);
});

// 3️⃣ PROMESAS (no bloquea, estilo moderno) ← el recomendado
const texto3 = await readFile("notas.txt", "utf-8");
```

| Sabor | Nombre | ¿Bloquea? | Cuándo usarlo |
|-------|--------|-----------|---------------|
| Síncrono | `readFileSync` | ✅ Sí | Scripts, o una sola vez al arrancar |
| Callback | `readFile(ruta, cb)` | ❌ No | Código antiguo |
| Promesas | `fs/promises` → `await readFile` | ❌ No | **Código nuevo** |

> ⚠️ Si no pones `"utf-8"`, `readFile` devuelve un **`Buffer`** (bytes), no un texto.

### Operaciones más comunes (ejemplo propio)

```js
import { readFile, writeFile, appendFile, mkdir, readdir, rm, access } from "node:fs/promises";

await writeFile("saludo.txt", "Hola\n");               // crea o SOBRESCRIBE
await appendFile("saludo.txt", "Adiós\n");             // añade al final
const contenido = await readFile("saludo.txt", "utf-8");
await mkdir("logs/2026", { recursive: true });         // crea carpetas (y padres)
const archivos = await readdir(".");                   // lista una carpeta
await rm("saludo.txt");                                // borra
```

### Leer y escribir JSON (ejemplo propio — patrón MUY habitual)

```js
import { readFile, writeFile } from "node:fs/promises";

// Leer
const texto = await readFile("data/books.json", "utf-8");
const books = JSON.parse(texto);          // texto → objeto JS

// Modificar
books.push({ id: 99, title: "Nuevo libro" });

// Guardar (null, 2 = con sangría bonita)
await writeFile("data/books.json", JSON.stringify(books, null, 2));
```

### Ejemplo REAL copiado (`ProyectoReproductor/server/index.ts`)

```ts
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
```

**Paso a paso:**

```
1. dbUrl        → "file:./data/nightjar.sqlite"  (de .env, o el valor por defecto)
2. relativePath → "./data/nightjar.sqlite"       (quita "file:" con una regex)
3. dbPath       → "C:\...\ProyectoReproductor\data\nightjar.sqlite"
4. mkdirSync(join(dbPath, '..'), { recursive: true })
                → crea la carpeta "data" si no existe ('..' = carpeta padre del archivo)
5. readFileSync(dbPath)
                → intenta leer la base de datos guardada
   catch        → si no existe (primera vez), crea una base de datos vacía
```

Y para **guardar** en disco cada vez que hay un cambio:

```ts
function persist() { writeFileSync(dbPath, Buffer.from(db.export())); }
function run(sql: string, params: unknown[] = []) { db.run(sql, params as never[]); persist(); }
```

> 📌 Aquí se ve un patrón muy útil: `try { leer } catch { crear vacío }`. Si el
> archivo no existe, `readFileSync` **lanza un error** (`ENOENT`), y en vez de
> romper el servidor, se empieza de cero.

### Ejemplo REAL: copiar archivos (`ProyectoReproductor/scripts/copy-client.mjs`)

```js
import { copyFileSync, mkdirSync } from 'node:fs';
mkdirSync('public', { recursive: true });
copyFileSync('dist/src/login.js', 'public/login.js');
copyFileSync('dist/src/app.js', 'public/app.js');
```

> 📌 Es un **script de utilidad** que se lanza desde `npm run build`. Como solo se
> ejecuta una vez y termina, usar las versiones `Sync` está perfecto.

### Errores típicos de `fs`

| Código | Significa |
|--------|-----------|
| `ENOENT` | *No such file or directory* → la ruta no existe |
| `EACCES` / `EPERM` | Sin permisos |
| `EEXIST` | Ya existe (por ejemplo, `mkdir` sin `recursive`) |
| `EISDIR` | Intentas leer una carpeta como si fuera un archivo |

---

## 4.3 `path` — rutas seguras

En Windows las rutas usan `\` y en Mac/Linux `/`. Si las escribes "a mano",
tu código puede fallar en otro sistema. `path` lo resuelve.

```js
// ❌ Concatenar a mano
const ruta = carpeta + "/" + "data" + "/" + "books.json";

// ✅ Con path.join: pone el separador correcto del sistema
import { join } from "node:path";
const ruta2 = join(carpeta, "data", "books.json");
```

| Función | Ejemplo | Resultado |
|---------|---------|-----------|
| `join(...)` | `join("src", "data", "a.json")` | `src/data/a.json` |
| `join(x, "..")` | `join("data/db.sqlite", "..")` | `data` (carpeta padre) |
| `resolve(...)` | `resolve("data")` | Ruta **absoluta**: `C:\...\proyecto\data` |
| `basename(p)` | `basename("/a/b/foto.png")` | `foto.png` |
| `dirname(p)` | `dirname("/a/b/foto.png")` | `/a/b` |
| `extname(p)` | `extname("foto.png")` | `.png` |

### Ejemplo REAL copiado (`ProyectoReproductor/server/index.ts`)

```ts
const dbPath = join(process.cwd(), relativePath);
mkdirSync(join(dbPath, '..'), { recursive: true });
```

Y en la última ruta del servidor, para servir el `index.html` del frontend:

```ts
app.get('*',(_req,res)=>res.sendFile(join(process.cwd(),'public','index.html')));
```

> 📌 `join(process.cwd(), 'public', 'index.html')` → ruta **absoluta** al archivo,
> que es lo que necesita `res.sendFile`.

---

## 4.4 `http` — un servidor web SIN Express

Antes de Express, hay que entender qué hace Node por sí solo. Un servidor HTTP es un
programa que **escucha** en un puerto y, por cada **petición** (*request*),
devuelve una **respuesta** (*response*).

> ⚠️ Tus proyectos usan Express, no `http` directamente. Este ejemplo es **propio**
> para que veas lo que Express te ahorra.

```js
// 📄 servidor.js  (ejemplo propio, CommonJS)
const http = require("node:http");

const PORT = 3000;

const server = http.createServer((req, res) => {
  console.log(`${req.method} ${req.url}`);

  if (req.method === "GET" && req.url === "/") {
    res.writeHead(200, { "Content-Type": "text/plain; charset=utf-8" });
    res.end("Mi primer servidor con Node");
    return;
  }

  if (req.method === "GET" && req.url === "/api/hello") {
    res.writeHead(200, { "Content-Type": "application/json" });
    res.end(JSON.stringify({ message: "Hola, persona" }));
    return;
  }

  res.writeHead(404, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ error: `No existe la ruta ${req.method} ${req.url}` }));
});

server.listen(PORT, () => {
  console.log(`Servidor escuchando en http://localhost:${PORT}`);
});
```

```bash
node servidor.js
# Abre http://localhost:3000 y http://localhost:3000/api/hello en el navegador
```

**Desglose:**

```
http.createServer((req, res) => {...})  → la función se ejecuta en CADA petición (es un callback)
req.method / req.url                    → qué te piden: "GET", "/api/hello"
res.writeHead(200, {...})               → código de estado + cabeceras
res.end("...")                          → envía el cuerpo y CIERRA la respuesta
server.listen(3000, cb)                 → empieza a escuchar; cb se ejecuta cuando está listo
```

### Leer el body de un POST con `http` puro (ejemplo propio)

```js
if (req.method === "POST" && req.url === "/api/books") {
  let body = "";
  req.on("data", (trozo) => { body += trozo; });   // el body llega por TROZOS
  req.on("end", () => {
    try {
      const book = JSON.parse(body);
      res.writeHead(201, { "Content-Type": "application/json" });
      res.end(JSON.stringify(book));
    } catch {
      res.writeHead(400, { "Content-Type": "application/json" });
      res.end(JSON.stringify({ error: "El body no es un JSON válido" }));
    }
  });
  return;
}
```

### Lo mismo con Express (`mongo-api-libros/02b-express-mongodb/02b-express-mongodb/src/index.ts`)

```ts
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
```

| Con `http` puro | Con Express |
|-----------------|-------------|
| `if (req.method === "GET" && req.url === "/")` | `app.get("/", ...)` |
| `res.writeHead(200, {...}); res.end(JSON.stringify(x))` | `res.json(x)` |
| Leer el body a trozos + `JSON.parse` | `app.use(express.json())` y `req.body` |
| Parsear `?name=Hector` a mano | `req.query.name` |
| `server.listen(PORT, cb)` | `app.listen(PORT, cb)` (igual) |

> 📌 Por eso todo el mundo usa Express (o similares): **es Node + `http` con
> atajos**. Los detalles de Express están en `Apuntes/EXPRESS/`.

---

## 4.5 `crypto` — aleatoriedad y contraseñas

### Ejemplo REAL copiado (`ProyectoReproductor/server/index.ts`)

```ts
import { randomBytes, scryptSync, timingSafeEqual } from 'node:crypto';

function hashPassword(password:string,salt=randomBytes(16).toString('hex')) { return `${salt}:${scryptSync(password,salt,64).toString('hex')}`; }
function passwordMatches(password:string,stored:string) { const [salt,hash]=stored.split(':'); const actual=scryptSync(password,salt,64),expected=Buffer.from(hash,'hex'); return actual.length===expected.length&&timingSafeEqual(actual,expected); }
```

**Ordenado y explicado:**

```ts
function hashPassword(password: string, salt = randomBytes(16).toString('hex')) {
  // randomBytes(16) → 16 bytes aleatorios SEGUROS → "a3f9...": la "sal"
  // scryptSync     → convierte la contraseña en un hash difícil de revertir
  return `${salt}:${scryptSync(password, salt, 64).toString('hex')}`;
}

function passwordMatches(password: string, stored: string) {
  const [salt, hash] = stored.split(':');              // separa sal y hash guardados
  const actual = scryptSync(password, salt, 64);       // hashea lo que escribe el usuario
  const expected = Buffer.from(hash, 'hex');
  return actual.length === expected.length && timingSafeEqual(actual, expected);
}
```

> 📌 **Nunca** se guardan contraseñas en texto plano. Se guarda `sal:hash`, y para
> comprobar se repite el hash con la misma sal.
> `timingSafeEqual` compara sin dar pistas por el tiempo que tarda (más seguro que `===`).

También para generar **tokens de sesión**:

```ts
const token=randomBytes(32).toString('hex');
```

```js
// ❌ Math.random() NO es seguro para tokens o contraseñas
const token = Math.random().toString(36);

// ✅ crypto sí
import { randomBytes, randomUUID } from "node:crypto";
const token2 = randomBytes(32).toString("hex");
const id = randomUUID();      // "3b241101-e2bb-4255-8caf-4136c566a962"
```

---

## 4.6 `os` y `events` (ejemplos propios, por cultura general)

```js
import os from "node:os";
console.log(os.platform());    // "win32", "linux", "darwin"
console.log(os.cpus().length); // núcleos de la CPU
console.log(os.homedir());     // C:\Users\TuUsuario
```

```js
import { EventEmitter } from "node:events";

const tienda = new EventEmitter();

tienda.on("venta", (producto) => {          // escuchar
  console.log(`Vendido: ${producto}`);
});

tienda.emit("venta", "Guitarra");           // emitir → "Vendido: Guitarra"
```

> 📌 Muchos objetos de Node son `EventEmitter`: el `server` de `http`, el `req`
> (`req.on("data", ...)`), los *streams*... Es el mismo patrón que
> `addEventListener` del navegador.

---

## ✅ Resumen rápido

1. Los módulos nativos no se instalan: `import ... from "node:fs"`.
2. `fs` tiene versión `Sync`, callback y `fs/promises` → usa **promesas** en código nuevo.
3. Pon siempre `"utf-8"` al leer texto; `JSON.parse` / `JSON.stringify` para JSON.
4. `path.join` para construir rutas sin pelearte con `/` y `\`.
5. `http.createServer` crea un servidor a mano; **Express** hace lo mismo más cómodo.
6. `crypto` para tokens y contraseñas; nunca `Math.random()` para eso.

👉 **Siguiente:** `05-process-y-variables-de-entorno.md`
