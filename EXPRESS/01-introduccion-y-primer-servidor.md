# 01 — INTRODUCCIÓN A EXPRESS Y PRIMER SERVIDOR

> **Express** es una librería para **Node.js** que sirve para crear **servidores web y
> APIs** con muy poco código. Node.js ya sabe recibir peticiones HTTP, pero hacerlo "a mano"
> es incómodo; Express te da **rutas**, **middleware** y **métodos para responder** listos
> para usar.

---

## 1.1 ¿Qué es una API y dónde encaja Express?

Una **API REST** es un programa que vive en un servidor y responde a **peticiones HTTP**
con **datos** (normalmente JSON), en vez de con páginas HTML.

```
FRONTEND (navegador)                    BACKEND (Node + Express)            BASE DE DATOS
  fetch("/api/books")    ── GET ──►     app.get("/api/books", ...)   ──►    MongoDB / SQLite / memoria
                         ◄── JSON ──    res.json(books)              ◄──
```

| Pieza | Qué hace |
|-------|----------|
| **Node.js** | Ejecuta JavaScript/TypeScript **fuera del navegador** |
| **Express** | Organiza las peticiones: "si llega `GET /api/books`, ejecuta esta función" |
| **Base de datos** | Guarda los datos de forma permanente |

> 📌 Express es **minimalista**: no te obliga a ninguna estructura. Por eso es muy
> importante que tú organices bien el proyecto (lo verás en `06` y `07`).

---

## 1.2 Instalar Express

```bash
npm init -y                      # crea package.json
npm install express              # Express (dependencia de producción)
npm install -D typescript tsx @types/express @types/node   # para usar TypeScript
```

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/package.json`)

```json
{
  "name": "api-books",
  "version": "1.0.0",
  "main": "src/index.ts",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "express": "^5.3.0"
  },
  "devDependencies": {
    "@types/express": "^5.0.6",
    "@types/node": "^26.6.6",
    "tsx": "^4.23.15",
    "typescript": "^7.0.2"
  }
}
```

**Qué significa cada cosa:**

| Paquete | Para qué |
|---------|----------|
| `express` | El framework en sí |
| `@types/express` | Los **tipos** de Express para TypeScript (`Request`, `Response`...) |
| `@types/node` | Los tipos de Node (`process.env`, `fs`...) |
| `tsx` | Ejecuta archivos `.ts` directamente, sin compilar a mano |
| `typescript` | El compilador `tsc` (lo usas para `typecheck`) |

- `npm run dev` → `tsx watch src/index.ts` → arranca el servidor y lo **reinicia solo**
  cada vez que guardas un archivo.
- `npm run typecheck` → `tsc --noEmit` → revisa los tipos **sin generar** archivos `.js`.

> 📌 Tus proyectos `api-books-claude`, `api-rastro` y `02b-express-mongodb` usan
> **Express 5**. `ProyectoReproductor` usa **Express 4**. Casi todo es igual; las
> diferencias importantes están en la sección 1.7.

---

## 1.3 Crear la aplicación y arrancarla (`express()` + `listen`)

Todo servidor Express tiene **tres pasos mínimos**:

```ts
import express from "express";

const app = express();      // 1. Crear la app
const PORT = 3000;

app.get("/", (req, res) => {          // 2. Definir al menos una ruta
  res.send("Hola desde Express");
});

app.listen(PORT, () => {              // 3. Ponerla a escuchar en un puerto
  console.log(`Servidor escuchando en http://localhost:${PORT}`);
});
```

- `express()` devuelve un objeto `app`: **tu servidor**.
- `app.get(...)` registra una **ruta**.
- `app.listen(puerto, callback)` abre el puerto. El callback se ejecuta **cuando ya
  está escuchando** (solo sirve para avisar por consola).

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/src/index.ts`)

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import booksRouter from "./routes/books-routes";

const app = express();
const PORT = 3000;

// ... middlewares y rutas ...

app.get("/", (_req, res) => {
  res.send("Mi API con Libros express");
});

// ...

app.listen(PORT, () => {
  console.log(`Servidor escuchando en http://localhost:${PORT}`);
});
```

> 📌 `_req` lleva **guion bajo** porque la función no usa ese parámetro. Es una
> convención para decir "sé que existe, pero no lo necesito" (y así ESLint/TypeScript no
> se quejan de variable sin usar).

---

## 1.4 El puerto desde variable de entorno

Poner `3000` fijo está bien para practicar, pero en producción el puerto lo decide el
servidor donde despliegas. Por eso se lee de `process.env.PORT`.

### Ejemplo REAL copiado (`ProyectoReproductor/server/index.ts`)

```ts
const port=Number(process.env.PORT||3000);app.listen(port,()=>console.log(`Nightjar disponible en http://localhost:${port}`));
```

Lo mismo, más legible:

```ts
const port = Number(process.env.PORT || 3000);   // si no hay PORT, usa 3000

app.listen(port, () => {
  console.log(`Nightjar disponible en http://localhost:${port}`);
});
```

> 📌 `process.env.PORT` siempre es un **string** (o `undefined`). Por eso se envuelve en
> `Number(...)`.

---

## 1.5 Arrancar DESPUÉS de conectar a la base de datos

Si tu API usa una base de datos, **no** tiene sentido aceptar peticiones antes de estar
conectado. El patrón es una función `start()` asíncrona:

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/index.ts`)

```ts
async function start() {
  await connectDb();

  app.listen(PORT, () => {
    console.log(`Servidor escuchando en http://localhost:${PORT}`);
  });
}

start();
```

```
start()
  │
  ├─ await connectDb()   ← espera a MongoDB (si falla, lanza error y NO arranca)
  │
  └─ app.listen(...)     ← solo entonces abre el puerto
```

```ts
// ❌ Escucha antes de tener la BD: las primeras peticiones pueden fallar
app.listen(PORT);
connectDb();

// ✅ Primero conectar, luego escuchar
async function start() {
  await connectDb();
  app.listen(PORT, () => console.log("Listo"));
}
start();
```

---

## 1.6 El ciclo petición → respuesta

Cada vez que alguien hace una petición, Express crea **dos objetos** y se los pasa a tu
función:

| Objeto | Nombre | Qué contiene |
|--------|--------|--------------|
| `req` | **Request** (petición) | Lo que **envía el cliente**: método, URL, parámetros, query, body, cabeceras, cookies |
| `res` | **Response** (respuesta) | Las herramientas para **contestar**: `res.json()`, `res.status()`, `res.send()`... |

```ts
app.get("/api/hello", (req, res) => {
  //                   ▲    ▲
  //                   │    └─ con esto RESPONDES
  //                   └────── de aquí LEES
  const name = req.query.name;
  res.json({ message: `Hola, ${name ?? "persona"}` });
});
```

### Ejemplo REAL copiado (`mongo-api-libros/02b-express-mongodb/02b-express-mongodb/src/index.ts`)

```ts
app.get("/", (_req, res) => {
  res.send("Mi primera API con Express");
});

app.get("/api/hello", (req, res) => {
  const name = req.query.name;

  res.json({ message: `Hola, ${name ?? "persona"}` });
});
```

**Prueba:**

```
GET http://localhost:3000/api/hello            → { "message": "Hola, persona" }
GET http://localhost:3000/api/hello?name=Hector → { "message": "Hola, Hector" }
```

> ⚠️ **Regla de oro:** cada petición debe recibir **exactamente UNA respuesta**.
> - Si no respondes nunca → el cliente se queda "cargando" hasta que salta un timeout.
> - Si respondes dos veces → error `Cannot set headers after they are sent to the client`.

```ts
// ❌ Dos respuestas
app.get("/x", (req, res) => {
  if (!req.query.id) {
    res.status(400).json({ error: "Falta id" });   // responde...
  }
  res.json({ ok: true });                           // ...¡y vuelve a responder!
});

// ✅ return después de responder
app.get("/x", (req, res) => {
  if (!req.query.id) {
    res.status(400).json({ error: "Falta id" });
    return;                                         // sale de la función
  }
  res.json({ ok: true });
});
```

> 📌 En **todos** tus proyectos verás este patrón `res.status(...).json(...); return;`.
> Es la forma más clara de evitar el doble envío.

---

## 1.7 Express 4 vs Express 5 (lo que te afecta)

| Tema | Express 4 (`ProyectoReproductor`) | Express 5 (`api-books`, `api-rastro`, `02b`) |
|------|-----------------------------------|----------------------------------------------|
| Errores en rutas `async` | Si una promesa falla, **no** llega al manejador de errores (hay que usar `try/catch` + `next(err)`) | Si una función `async` lanza un error, Express lo **manda solo** al manejador de errores |
| Ruta "comodín" | `app.get('*', ...)` | `app.get('/*splat', ...)` (el `*` solo ya no vale) |
| `req.body` cuando la petición no trae body (con `express.json()` puesto) | `{}` | `undefined` → por eso usas `req.body ?? {}` |

```ts
// Express 5: esto es seguro. Si findAll() falla, va al middleware de errores → 500
router.get("/", async (req, res) => {
  const books = await storage.findAll();
  res.json(books);
});

// Express 4: tendrías que hacer esto
router.get("/", async (req, res, next) => {
  try {
    const books = await storage.findAll();
    res.json(books);
  } catch (err) {
    next(err);    // pasar el error a mano
  }
});
```

> 📌 Esto lo verás en detalle en `04-middleware-cors-y-errores.md`.

---

## 1.8 Probar la API

| Herramienta | Cómo |
|-------------|------|
| **Navegador** | Solo sirve para `GET`: escribe `http://localhost:3000/api/books` |
| **Postman / Thunder Client / Bruno** | Cualquier método, con body JSON |
| **curl** (terminal) | `curl -X POST http://localhost:3000/api/books -H "Content-Type: application/json" -d '{"title":"..."}'` |
| **fetch** (desde tu web) | `fetch("http://localhost:3000/api/books")` (necesita CORS, ver `04`) |

```bash
# Ejemplos con curl contra api-books-claude
curl http://localhost:3000/api/books?country=chi&limit=5
curl http://localhost:3000/api/books/1
curl -X PATCH http://localhost:3000/api/books/1 -H "Content-Type: application/json" -d '{"pages":210}'
curl -X DELETE http://localhost:3000/api/books/1
```

---

## 1.9 Errores comunes al empezar

| Error | Causa | Solución |
|-------|-------|----------|
| `EADDRINUSE: address already in use :::3000` | Ya hay otro servidor en el puerto 3000 | Cierra la otra terminal o cambia de puerto |
| `Cannot GET /api/libros` | La ruta no existe (o está mal escrita) | Revisa la URL y el método |
| La petición se queda "cargando" | Nunca llamas a `res.json/send/end` ni a `next()` | Responde siempre |
| `Cannot set headers after they are sent` | Respondiste dos veces | `return` después de responder |
| `req.body` es `undefined` | Falta `app.use(express.json())` o no mandas `Content-Type: application/json` | Ver `03` |

---

## ✅ Resumen rápido

1. Express = librería de Node para crear servidores/APIs fácilmente.
2. `const app = express();` crea el servidor.
3. `app.get("/ruta", (req, res) => { ... })` define una ruta.
4. `app.listen(PORT, callback)` lo arranca.
5. Si hay base de datos: `await connectDb()` **antes** de `listen`.
6. `req` = lo que llega; `res` = cómo contestas.
7. **Una** respuesta por petición → `return` después de responder.
8. Express 5 captura solo los errores de funciones `async`.

👉 **Siguiente:** `02-rutas-y-router.md`
