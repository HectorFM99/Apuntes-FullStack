# 08 — EJEMPLOS COMPLETOS DE TUS PROYECTOS

> Archivos **enteros** copiados tal cual de tus proyectos del Bootcamp, para que veas
> cómo encajan todas las piezas de los temas anteriores en código real.
> Cada bloque indica de qué tema son los conceptos que aparecen.

> ⚠️ No se ha modificado ningún proyecto. No se incluye ningún archivo `.env` (solo
> `.env.example`).

---

## 8.1 Tu primer script — `node-practica/hola.js`

**Archivo:** `node-practica/hola.js`

Temas: **01** (ejecutar con `node hola.js`).

```js
console.log("Hola desde Node.js");
console.log("Estamos ejecutando JavaScript fuera del navegador");
```

---

## 8.2 Usar un paquete de npm (CommonJS) — `node-practica/prueba/index.js`

**Archivo:** `node-practica/prueba/index.js`

Temas: **02** (dependencia `dayjs`), **03** (`require`).

```js
const dayjs = require("dayjs");

console.log("Hoy es:", dayjs().format("DD/MM/YYYY HH:mm:ss"));
```

---

## 8.3 El mismo programa en TypeScript (sintaxis ESM) — `node-practica/prueba/index.ts`

**Archivo:** `node-practica/prueba/index.ts`

Temas: **03** (`import` vs `require`), **07** (se compila con `tsc`).

```ts
import dayjs from "dayjs";

console.log("Hoy es:", dayjs().format());
```

---

## 8.4 `package.json` mínimo — `node-practica/prueba/package.json`

**Archivo:** `node-practica/prueba/package.json`

Temas: **02** (`dependencies` vs `devDependencies`, `scripts`), **03** (`"type": "commonjs"`).

```json
{
  "name": "prueba",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
 
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs",
  "dependencies": {
    "dayjs": "^1.11.23"
  },
  "devDependencies": {
    "typescript": "^7.0.2"
  }
}
```

> 📌 Flujo: `npm install` → `npm run build` (genera `dist/index.js` con `tsc`) → `npm start`.

---

## 8.5 Asincronía con `async`/`await` y `Promise.all` — `Async-hector`

**Archivo:** `Async-hector/Advanced-JS-Asynchronous-code/index.js`

Temas: **06** (`async`/`await`, dos `await` por `fetch`, `.then`, `Promise.all`, callbacks de eventos).

```js
//RESUELVE TUS EJERCICIOS AQUI

// 1.
async function getAllBreeds() {
    const res = await fetch('https://dog.ceo/api/breeds/list/all');
    const data = await res.json();
    return Object.keys(data.message);
}

// 2.
async function getRandomDog() {
    const res = await fetch('https://dog.ceo/api/breeds/image/random');
    const data = await res.json();
    return data.message;
}

// 3.
async function getAllImagesByBreed(breed = 'komondor') {
    const res = await fetch(`https://dog.ceo/api/breed/${breed}/images`);
    const data = await res.json();
    return data.message;
}

// 4.
async function getAllImagesByBreed2(breed) {
    const res = await fetch(`https://dog.ceo/api/breed/${breed}/images`);
    const data = await res.json();
    return data.message;
}

// 5.
async function getGitHubUserProfile(username) {
    const res = await fetch(`https://api.github.com/users/${username}`);
    const profile = await res.json();
    return profile;
}

// 6.
async function printGithubUserProfile(username) {
    const profile = await getGitHubUserProfile(username);
    const user = { img: profile.avatar_url, name: profile.name };

    const section = document.createElement('section');
    section.innerHTML = `
        <img src="${user.img}" alt="${user.name}">
        <h1>${user.name}</h1>
    `;
    document.body.appendChild(section);

    return user;
}

// 7.
async function getAndPrintGitHubUserProfile(username) {
    const profile = await getGitHubUserProfile(username);

    const card = `<section>
    <img src="${profile.avatar_url}" alt="${profile.name}">
    <h1>${profile.name}</h1>
    <p>Public repos: ${profile.public_repos}</p>
</section>`;

    document.body.insertAdjacentHTML('beforeend', card);

    return card;
}

// 8.
window.addEventListener('DOMContentLoaded', () => {
    const searchInput = document.createElement('input');
    searchInput.type = 'text';
    searchInput.placeholder = 'GitHub username';

    const searchButton = document.createElement('button');
    searchButton.textContent = 'Buscar';

    searchButton.addEventListener('click', () => {
        const username = searchInput.value.trim();
        if (username) getAndPrintGitHubUserProfile(username);
    });

    document.body.appendChild(searchInput);
    document.body.appendChild(searchButton);
});

// 9.
async function fetchGithubUsers(userNames) {
    const promises = userNames.map(name =>
        fetch(`https://api.github.com/users/${name}`).then(res => res.json())
    );
    const users = await Promise.all(promises);

    users.forEach(user => {
        console.log('Repo url:', user.html_url);
        console.log('Name:', user.name);
    });

    return users;
}
```

> ⚠️ Los ejercicios 6, 7 y 8 usan `document` y `window` → este archivo se ejecuta en el **navegador** (con `index.html`), no con `node`. Las funciones 1–5 y 9 sí funcionarían igual en Node 18+, porque `fetch` existe en los dos.

---

## 8.6 `await` encadenados — `Async-Hector-2`

**Archivo:** `Async-Hector-2/advanced-js-bikram-async-await/index.js`

Temas: **06** (funciones `async` que llaman a otras `async`, peticiones que dependen de la anterior).

```js
//DESARROLLA AQUI TUS SOLUCIONES

//1
async function getRandomPokemon() {
    const id = Math.floor(Math.random() * 1025) + 1;
    const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`);
    const data = await response.json();
    return data;
}

//2
async function getImageAndName(pokemon) {

    let response = await fetch(`https://pokeapi.co/api/v2/pokemon/${pokemon}`)
    let data = await response.json();
    let name = data.name;
    let img = data.sprites.front_default;
    return { name, img }

}

//3
async function printImageAndName(pokemon) {
    let name = pokemon;
    if (!name) {
        const randomPokemon = await getRandomPokemon();
        name = randomPokemon.name;
    }

    const data = await getImageAndName(name);

    return `<section>
    <img src="${data.img}" alt="${data.name}">
    <h1>${data.name}</h1>
    </section>`;
}


//4
async function getRandomDogImage() {
    const response = await fetch('https://dog.ceo/api/breeds/image/random');
    const data = await response.json();
    return data.message;
}

//5
async function getRandomPokemonImage() {
    const pokemon = await getRandomPokemon();
    const img = pokemon.sprites.front_default ||
        pokemon.sprites.other['official-artwork'].front_default;
    return img;
}

//6
async function printPugVsPikachu() {
    const dogResponse = await fetch('https://dog.ceo/api/breed/pug/images/random');
    const dogData = await dogResponse.json();
    const pugImg = dogData.message;

    const pikachu = await getImageAndName('pikachu');

    const battle = `<section>
    <h2>Pug VS Pikachu</h2>
    <div>
        <img src="${pugImg}" alt="Pug">
        <h1>Pug</h1>
    </div>
    <div>
        <img src="${pikachu.img}" alt="${pikachu.name}">
        <h1>${pikachu.name}</h1>
    </div>
</section>`;

    document.body.insertAdjacentHTML('beforeend', battle);
    return battle;
}

//7
async function getRandomCharacter() {
    const id = Math.floor(Math.random() * 826) + 1;
    const response = await fetch(`https://rickandmortyapi.com/api/character/${id}`);
    const data = await response.json();
    return data;
}

//8
async function getRandomCharacterInfo() {
    const character = await getRandomCharacter();

    const img = character.image;
    const name = character.name;
    const episodes = character.episode;


    const episodeResponse = await fetch(episodes[0]);
    const episodeData = await episodeResponse.json();

    return {
        img,
        name,
        episodes,
        firstEpisode: episodeData.name,
        dateEpisode: episodeData.air_date
    };
}

//9
async function printCharacterInfo() {
    const info = await getRandomCharacterInfo();

    const character = `<section>
    <img src="${info.img}" alt="${info.name}">
    <h1>${info.name}</h1>
    <p>Episodios en los que aparece: ${info.episodes.length}</p>
    <p>Primer episodio: ${info.firstEpisode} - ${info.dateEpisode}</p>
    </section>`;

    document.body.insertAdjacentHTML('beforeend', character);
    return character;
}
```

---

## 8.7 `package.json` de una API con TypeScript — `api-rastro`

**Archivo:** `api-rastro/api-rastro/package.json`

Temas: **02** (`scripts`, versiones `^`/`~`), **05** (`--env-file`), **07** (`tsx`, `tsc`, `eslint`).

```json
{
  "name": "api-rastro",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "tsx watch --env-file=.env src/index.ts",
    "lint": "eslint .",
    "typecheck": "tsc --noEmit"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/HectorFM99/api-rastro.git"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs",
  "bugs": {
    "url": "https://github.com/HectorFM99/api-rastro/issues"
  },
  "homepage": "https://github.com/HectorFM99/api-rastro#readme",
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
}
```

---

## 8.8 `.env.example` — `api-rastro`

**Archivo:** `api-rastro/api-rastro/.env.example`

Temas: **05** (plantilla de variables de entorno; el `.env` real **no** se copia).

```bash
MONGODB_URI=mongodb://usuario:contraseña@localhost:27017/?authSource=admin
DB_NAME=rastro
```

---

## 8.9 Conexión a la base de datos — `api-rastro/api-rastro/src/db.ts`

**Archivo:** `api-rastro/api-rastro/src/db.ts`

Temas: **03** (`export` con nombre, `import type`), **05** (`process.env`, validar variables, `??`), **06** (`async function connectDb`).

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

---

## 8.10 Punto de entrada del servidor — `api-rastro/api-rastro/src/index.ts`

**Archivo:** `api-rastro/api-rastro/src/index.ts`

Temas: **03** (imports), **06** (`async function start()`, `await connectDb()` antes de `listen`), **07** (parámetros `_req`, `_next` por ESLint). Express en detalle: `Apuntes/EXPRESS/`.

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import { connectDb, getDb } from "./db";
import listingsRouter from "./routes/listings";
import offersRouter from "./routes/offers";

const app = express();
const PORT = 3000;

app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

app.use(express.json());

app.get("/api/health", async (_req, res) => {
  await getDb().command({ ping: 1 });
  res.json({ status: "ok", db: getDb().databaseName });
});

app.use("/api/listings", listingsRouter);
app.use("/api/offers", offersRouter);

app.use((req, res) => {
  res.status(404).json({ error: `No existe la ruta ${req.method} ${req.url}` });
});

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

---

## 8.11 Conexión a MongoDB con dos variables posibles — `mongo-api-libros`

**Archivo:** `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/src/db.ts`

Archivo completo: `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/src/db.ts`. Temas: **05** (`||` entre dos variables, `??` para el nombre de la BD), **06** (`await` en serie).

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

---

## 8.12 API mínima sin base de datos — `API_Godines`

**Archivo:** `API_Godines/api-express/api-books-claude/src/index.ts`

Temas: **03** (imports), **01** (`app.listen` mantiene vivo el proceso). Compárala con 8.10: aquí no hay `start()` porque no hay que esperar a ninguna base de datos.

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import booksRouter from "./routes/books-routes";

const app = express();
const PORT = 3000;

// 1. Registrar cada petición en la terminal
app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// 2. CORS: permitir peticiones desde nuestra página web
app.use((req, res, next) => {
  res.setHeader("Access-Control-Allow-Origin", "http://127.0.0.1:5500");
  res.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, PATCH, DELETE");
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
  res.send("Mi API con Libros express");
});

app.use("/api/books", booksRouter);

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

app.listen(PORT, () => {
  console.log(`Servidor escuchando en http://localhost:${PORT}`);
});
```

---

## 8.13 Almacenamiento con métodos `async` — `API_Godines`

**Archivo:** `API_Godines/api-express/api-books-claude/src/services/books-storage-in-memory.ts`

Temas: **03** (importar JSON, `import type`), **06** (métodos `async` que devuelven `Promise` aunque no esperen nada).

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

---

## 8.14 `package.json` con ES Modules — `ProyectoReproductor`

**Archivo:** `ProyectoReproductor/package.json`

Temas: **02** (dependencias), **03** (`"type": "module"`), **07** (flujo `dev` / `build` / `start`).

```json
{
  "name": "nightjar-music",
  "private": true,
  "type": "module",
  "scripts": {"dev":"tsx watch server/index.ts", "build":"tsc && node scripts/copy-client.mjs", "check":"tsc --noEmit",  "start":"node dist/server/index.js"},
  "dependencies": {"cookie-parser":"^1.4.7", "dotenv":"^16.4.7", "express":"^4.21.2", "sql.js":"^1.13.0"},
  "devDependencies": {"@types/cookie-parser":"^1.4.8", "@types/express":"^5.0.0", "@types/node":"^22.10.5", "@types/sql.js":"^1.4.9", "tsx":"^4.19.2", "typescript":"^5.7.2"}
}
```

---

## 8.15 `.env.example` — `ProyectoReproductor`

**Archivo:** `ProyectoReproductor/.env.example`

Temas: **05**.

```bash
JAMENDO_CLIENT_ID=
SESSION_SECRET=
PORT=3000
DATABASE_URL=file:./data/nightjar.sqlite
```

---

## 8.16 Script de utilidad con `fs` — `ProyectoReproductor/scripts/copy-client.mjs`

**Archivo:** `ProyectoReproductor/scripts/copy-client.mjs`

Temas: **03** (extensión `.mjs`, prefijo `node:`), **04** (`fs` síncrono en un script).

```js
import { copyFileSync, mkdirSync } from 'node:fs';
mkdirSync('public', { recursive: true });
copyFileSync('dist/src/login.js', 'public/login.js');
copyFileSync('dist/src/app.js', 'public/app.js');
```

---

## 8.17 Servidor completo — `ProyectoReproductor/server/index.ts`

**Archivo:** `ProyectoReproductor/server/index.ts`

Temas: **03** (`import 'dotenv/config'`, `node:`, *top-level await*), **04** (`fs`, `path`, `crypto`, `Buffer`), **05** (`process.env`, `process.cwd()`, `NODE_ENV`, `??` y `||`), **06** (rutas `async`, `fetch` con `try/catch`, `res.ok`, `AbortSignal.timeout`). El original tiene muchas instrucciones en una sola línea; está copiado **tal cual**.

```ts
import 'dotenv/config';
import express from 'express';
import cookieParser from 'cookie-parser';
import initSqlJs, { type Database as SqlDatabase } from 'sql.js';
import { randomBytes, scryptSync, timingSafeEqual } from 'node:crypto';
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
db.run('PRAGMA foreign_keys = ON');
db.run(`CREATE TABLE IF NOT EXISTS users (id TEXT PRIMARY KEY, email TEXT UNIQUE NOT NULL, password_hash TEXT NOT NULL, created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP);
CREATE TABLE IF NOT EXISTS sessions (token TEXT PRIMARY KEY, user_id TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE, expires_at INTEGER NOT NULL);
CREATE TABLE IF NOT EXISTS favorites (user_id TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE, track_id TEXT NOT NULL, track_json TEXT NOT NULL, created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP, PRIMARY KEY(user_id, track_id));
CREATE TABLE IF NOT EXISTS history (id INTEGER PRIMARY KEY AUTOINCREMENT, user_id TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE, track_id TEXT NOT NULL, track_json TEXT NOT NULL, played_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP);`);
function persist() { writeFileSync(dbPath, Buffer.from(db.export())); }
function run(sql: string, params: unknown[] = []) { db.run(sql, params as never[]); persist(); }
function one<T>(sql: string, params: unknown[] = []): T | undefined { const statement = db.prepare(sql); statement.bind(params as never[]); const result = statement.step() ? statement.getAsObject() as T : undefined; statement.free(); return result; }
function many<T>(sql: string, params: unknown[] = []): T[] { const statement = db.prepare(sql); statement.bind(params as never[]); const results: T[] = []; while (statement.step()) results.push(statement.getAsObject() as T); statement.free(); return results; }
app.use(express.json({ limit: '32kb' })); app.use(cookieParser()); app.use((req,res,next)=>{ if(req.path.startsWith('/api/')) res.setHeader('Cache-Control','no-store'); next(); }); app.use(express.static('public'));
function userFor(req: express.Request) { const token = req.cookies?.nightjar_session; return token ? one<{id:string;email:string}>('SELECT users.id, users.email FROM sessions JOIN users ON users.id=sessions.user_id WHERE sessions.token=? AND sessions.expires_at>?',[token,Date.now()]) : undefined; }
function session(req: express.Request, res: express.Response, next: express.NextFunction) { const user=userFor(req); if(!user)return res.status(401).json({error:'Inicia sesión para continuar.'}); res.locals.user=user; next(); }
function createSession(userId:string,res:express.Response) { const token=randomBytes(32).toString('hex'); run('INSERT INTO sessions(token,user_id,expires_at) VALUES(?,?,?)',[token,userId,Date.now()+1209600000]); res.cookie('nightjar_session',token,{httpOnly:true,sameSite:'lax',secure:process.env.NODE_ENV==='production',maxAge:1209600000}); }
function hashPassword(password:string,salt=randomBytes(16).toString('hex')) { return `${salt}:${scryptSync(password,salt,64).toString('hex')}`; }
function passwordMatches(password:string,stored:string) { const [salt,hash]=stored.split(':'); const actual=scryptSync(password,salt,64),expected=Buffer.from(hash,'hex'); return actual.length===expected.length&&timingSafeEqual(actual,expected); }
app.post('/api/auth/register',(req,res)=>{const email=String(req.body?.email??'').trim().toLowerCase(),password=String(req.body?.password??'');if(!/^\S+@\S+\.\S+$/.test(email)||password.length<10||password.length>128)return res.status(400).json({error:'Introduce un correo válido y una contraseña de 10 a 128 caracteres.'});const id=randomBytes(16).toString('hex');try{run('INSERT INTO users(id,email,password_hash) VALUES(?,?,?)',[id,email,hashPassword(password)]);}catch{return res.status(409).json({error:'Ya existe una cuenta con ese correo.'});}createSession(id,res);res.status(201).json({id,email});});
app.post('/api/auth/login',(req,res)=>{const email=String(req.body?.email??'').trim().toLowerCase(),password=String(req.body?.password??''),user=one<{id:string;email:string;password_hash:string}>('SELECT id,email,password_hash FROM users WHERE email=?',[email]);if(!user||!passwordMatches(password,user.password_hash))return res.status(401).json({error:'Correo o contraseña incorrectos.'});createSession(user.id,res);res.json({id:user.id,email:user.email});});
app.post('/api/auth/logout',(req,res)=>{const token=req.cookies?.nightjar_session;if(token)run('DELETE FROM sessions WHERE token=?',[token]);res.clearCookie('nightjar_session');res.status(204).end();});
app.get('/api/auth/me',(req,res)=>{const user=userFor(req);user?res.json(user):res.status(401).json({error:'Sesión no iniciada.'});});
app.get('/api/tracks/search',session,async(req,res)=>{const q=String(req.query.q??'').trim();if(!q||q.length>100)return res.status(400).json({error:'Escribe una búsqueda de hasta 100 caracteres.'});if(!process.env.JAMENDO_CLIENT_ID)return res.status(503).json({error:'Falta configurar el acceso al catálogo de Jamendo.'});try{const url=new URL('https://api.jamendo.com/v3.0/tracks/');url.search=new URLSearchParams({client_id:process.env.JAMENDO_CLIENT_ID,format:'json',limit:'30',search:q,audioformat:'mp32',include:'musicinfo'}).toString();const response=await fetch(url,{signal:AbortSignal.timeout(10000)});if(!response.ok)throw new Error();const payload=await response.json() as {headers?:{status?:string;error_message?:string};results?:Array<Record<string,unknown>>};if(payload.headers?.status==='failed')return res.status(502).json({error:payload.headers.error_message==='Jamendo Api Invalid Client Id Error: Your credential is not authorized.'?'Jamendo no autorizó este Client ID. Revisa la credencial en tu portal de desarrollador.':'Jamendo no pudo completar la búsqueda.'});res.json((payload.results??[]).map(t=>({id:String(t.id),name:t.name,artist:t.artist_name,album:t.album_name,image:t.album_image,audio:t.audio,duration:t.duration})));}catch{res.status(502).json({error:'No se pudo consultar Jamendo. Comprueba la conexión e inténtalo de nuevo.'});}});
app.get('/api/tracks/featured',session,async(req,res)=>{if(!process.env.JAMENDO_CLIENT_ID)return res.status(503).json({error:'Falta configurar el acceso al catálogo de Jamendo.'});try{const url=new URL('https://api.jamendo.com/v3.0/tracks/');url.search=new URLSearchParams({client_id:process.env.JAMENDO_CLIENT_ID,format:'json',limit:'15',order:'popularity_total',audioformat:'mp32',include:'musicinfo'}).toString();const response=await fetch(url,{signal:AbortSignal.timeout(10000)});if(!response.ok)throw new Error();const payload=await response.json() as {headers?:{status?:string;error_message?:string};results?:Array<Record<string,unknown>>};if(payload.headers?.status==='failed')return res.status(502).json({error:'Jamendo no pudo completar la solicitud de destacados.'});res.json((payload.results??[]).map(t=>({id:String(t.id),name:t.name,artist:t.artist_name,album:t.album_name,image:t.album_image,audio:t.audio,duration:t.duration})));}catch{res.status(502).json({error:'No se pudo consultar Jamendo. Comprueba la conexión e inténtalo de nuevo.'});}});
app.get('/api/library/:kind',session,(req,res)=>{const kind=String(req.params.kind);if(!['favorites','history'].includes(kind))return res.status(404).json({error:'Biblioteca no encontrada.'});const orderColumn=kind==='history'?'played_at':'created_at';const rows=many<{track_json:string}>(`SELECT track_json FROM ${kind} WHERE user_id=? ORDER BY ${orderColumn} DESC LIMIT 100`,[res.locals.user.id]);res.json(rows.map(row=>JSON.parse(row.track_json)));});
app.post('/api/library/favorites/:trackId',session,(req,res)=>{const track=req.body?.track;if(!track||String(track.id)!==String(req.params.trackId)||!track.audio)return res.status(400).json({error:'La canción no tiene datos válidos.'});run('INSERT OR REPLACE INTO favorites(user_id,track_id,track_json) VALUES(?,?,?)',[res.locals.user.id,String(req.params.trackId),JSON.stringify(track)]);res.status(201).json({saved:true});});
app.delete('/api/library/favorites/:trackId',session,(req,res)=>{run('DELETE FROM favorites WHERE user_id=? AND track_id=?',[res.locals.user.id,String(req.params.trackId)]);res.status(204).end();});
app.post('/api/library/history',session,(req,res)=>{const track=req.body?.track;if(!track||!track.id||!track.audio)return res.status(400).json({error:'La canción no tiene datos válidos.'});run('INSERT INTO history(user_id,track_id,track_json) VALUES(?,?,?)',[res.locals.user.id,String(track.id),JSON.stringify(track)]);res.status(201).json({saved:true});});
app.get('*',(_req,res)=>res.sendFile(join(process.cwd(),'public','index.html')));
const port=Number(process.env.PORT||3000);app.listen(port,()=>console.log(`Nightjar disponible en http://localhost:${port}`));
```

> 📌 **Cómo leerlo:** (1) carga `.env`; (2) calcula la ruta de la base de datos con `path` y crea la carpeta con `fs`; (3) espera (`await`) a que cargue SQLite; (4) define funciones de ayuda (`persist`, `run`, `one`, `many`); (5) contraseñas y sesiones con `crypto`; (6) rutas de la API; (7) `app.listen` con el puerto de `process.env.PORT || 3000`.

---

## ✅ Qué buscar en cada archivo al repasar

| Si quieres repasar... | Mira |
|-----------------------|------|
| Ejecutar un script | 8.1 |
| `require` vs `import` | 8.2, 8.3, 8.16 |
| `package.json` y `scripts` | 8.4, 8.7, 8.14 |
| `async`/`await`, `Promise.all` | 8.5, 8.6, 8.10 |
| Variables de entorno | 8.8, 8.9, 8.11, 8.15, 8.17 |
| `fs`, `path`, `crypto` | 8.16, 8.17 |
| Arrancar un servidor correctamente | 8.10, 8.12, 8.17 |

👉 **Vuelve al índice:** `00-INDICE.md`
