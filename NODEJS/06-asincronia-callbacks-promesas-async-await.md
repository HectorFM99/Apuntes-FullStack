# 06 — ASINCRONÍA: CALLBACKS → PROMESAS → `async`/`await`

> Casi todo lo interesante en Node **tarda**: leer un archivo, consultar MongoDB,
> pedir datos a otra API con `fetch`... Como Node **no se queda esperando** (tema 01),
> necesitamos una forma de decir **"cuando termines, haz esto"**.
> Esa forma ha evolucionado en tres etapas: **callbacks**, **promesas** y
> **`async`/`await`**. Las tres hacen lo mismo; cada una es más cómoda que la anterior.

---

## 6.1 Síncrono vs. asíncrono

```js
// SÍNCRONO: cada línea espera a que termine la anterior
console.log("1");
console.log("2");
console.log("3");
// 1, 2, 3
```

```js
// ASÍNCRONO: la tarea lenta se "aparca" y el código sigue
console.log("1");
setTimeout(() => console.log("2 (después de 1 segundo)"), 1000);
console.log("3");
// 1, 3, 2
```

> 📌 **Idea clave:** el resultado de una operación asíncrona **no está disponible en
> la línea siguiente**. Hay que "engancharse" a cuando termine.

```js
// ❌ El error más típico del principiante
const res = fetch("https://dog.ceo/api/breeds/image/random");
console.log(res.message);   // undefined → res es una PROMESA, no los datos
```

---

## 6.2 Etapa 1: Callbacks

Un **callback** es una función que le pasas a otra para que la **llame cuando
termine**. (Ya los viste en `Apuntes/JAVASCRIPT/03-funciones.md` con
`addEventListener`).

### Convención de Node: *error-first callback*

En Node, los callbacks clásicos reciben **primero el error** y después el resultado:

```js
// Ejemplo propio
const fs = require("node:fs");

fs.readFile("datos.txt", "utf-8", (err, contenido) => {
  if (err) {
    console.error("No se pudo leer:", err.message);
    return;                          // ⚠️ importante salir
  }
  console.log("Contenido:", contenido);
});
```

```
(err, contenido) => { ... }
  ▲       ▲
  │       └── el resultado (si todo fue bien)
  └── null si todo fue bien; un objeto Error si falló
```

### Callbacks que SÍ aparecen en tus proyectos

No son de lectura de archivos, pero el concepto es el mismo: **"llama a esta función
cuando pase algo"**.

**Ejemplo REAL** (`api-rastro/api-rastro/src/index.ts`) — se ejecuta cuando el
servidor ya está escuchando:

```ts
app.listen(PORT, () => {
  console.log(`Servidor escuchando en http://localhost:${PORT}`);
});
```

**Ejemplo REAL** (`Async-hector/Advanced-JS-Asynchronous-code/index.js`) — se
ejecuta cuando el usuario hace clic:

```js
searchButton.addEventListener('click', () => {
    const username = searchInput.value.trim();
    if (username) getAndPrintGitHubUserProfile(username);
});
```

### 😱 El problema: *callback hell* (la pirámide de la muerte)

Cuando una tarea depende de otra, los callbacks se anidan:

```js
// ❌ Ejemplo propio: leer config → conectar BD → buscar usuario → guardar log
fs.readFile("config.json", "utf-8", (err, config) => {
  if (err) return console.error(err);
  conectarBD(JSON.parse(config), (err, db) => {
    if (err) return console.error(err);
    db.buscarUsuario("hector", (err, usuario) => {
      if (err) return console.error(err);
      fs.writeFile("log.txt", usuario.nombre, (err) => {
        if (err) return console.error(err);
        console.log("¡Hecho!");
      });
    });
  });
});
```

Problemas: difícil de leer, el manejo de errores se repite en cada nivel y es fácil
olvidarse de uno. Para solucionarlo nacieron las **promesas**.

---

## 6.3 Etapa 2: Promesas

Una **promesa** (`Promise`) es un objeto que representa **un valor que llegará en el
futuro**. Es como el **ticket** que te dan en un restaurante de comida rápida: aún no
tienes la hamburguesa, pero tienes la promesa de que te la darán (o de que te dirán
que se ha acabado).

### Estados de una promesa

```
                 ┌──────────────┐
                 │   PENDING    │  ← esperando
                 │  (pendiente) │
                 └──────┬───────┘
              ┌─────────┴─────────┐
              ▼                   ▼
     ┌────────────────┐   ┌────────────────┐
     │   FULFILLED    │   │    REJECTED    │
     │  (cumplida) ✅ │   │  (rechazada) ❌│
     │   → .then()    │   │   → .catch()   │
     └────────────────┘   └────────────────┘
```

Una vez cumplida o rechazada, **ya no cambia**.

### Consumir una promesa: `.then()` y `.catch()`

```js
// Ejemplo propio con fetch (fetch devuelve una promesa)
fetch("https://dog.ceo/api/breeds/image/random")
  .then((res) => res.json())             // res.json() TAMBIÉN devuelve una promesa
  .then((data) => console.log(data.message))
  .catch((err) => console.error("Error:", err.message))
  .finally(() => console.log("Terminado (bien o mal)"));
```

**Puntos clave:**
- `.then(fn)` → se ejecuta cuando la promesa **se cumple**; recibe el valor.
- Si dentro de un `.then` **devuelves** otra promesa, el siguiente `.then` espera a esa.
  Así se **encadenan** en vez de anidarse.
- Un único `.catch` al final captura el error de **cualquier** paso anterior.
- `.finally` se ejecuta siempre.

### Ejemplo REAL de `.then()` (`Async-hector/Advanced-JS-Asynchronous-code/index.js`)

```js
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

> 📌 Aquí se mezclan las dos etapas: `.then(res => res.json())` dentro del `map`
> (promesas) y `await Promise.all(...)` (async/await). Es totalmente válido.

### El callback hell, rehecho con promesas (ejemplo propio)

```js
// ✅ Plano en vez de pirámide, y un solo catch
readFile("config.json", "utf-8")
  .then((config) => conectarBD(JSON.parse(config)))
  .then((db) => db.buscarUsuario("hector"))
  .then((usuario) => writeFile("log.txt", usuario.nombre))
  .then(() => console.log("¡Hecho!"))
  .catch((err) => console.error(err));
```

### Crear una promesa tú mismo (ejemplo propio)

Casi nunca lo harás (las librerías ya devuelven promesas), pero ayuda a entenderlas:

```js
function esperar(ms) {
  return new Promise((resolve, reject) => {
    if (ms < 0) {
      reject(new Error("El tiempo no puede ser negativo"));   // ❌ rechazar
      return;
    }
    setTimeout(() => resolve(`Han pasado ${ms} ms`), ms);    // ✅ cumplir
  });
}

esperar(1000).then((mensaje) => console.log(mensaje));     // "Han pasado 1000 ms"
```

```
new Promise((resolve, reject) => { ... })
              ▲         ▲
              │         └── llámala si algo va MAL   → activa .catch
              └── llámala si todo va BIEN → activa .then
```

### Convertir un callback en promesa: `util.promisify` / `fs/promises`

```js
// Opción 1: Node ya trae versiones con promesas
import { readFile } from "node:fs/promises";

// Opción 2: convertir cualquier función con callback error-first
import { promisify } from "node:util";
import fs from "node:fs";
const readFilePromesa = promisify(fs.readFile);
```

---

## 6.4 Etapa 3: `async` / `await`

`async`/`await` es **azúcar sintáctico** sobre las promesas: hace que el código
asíncrono **se lea como si fuera síncrono**, de arriba abajo.

```js
async function nombre() {
  const valor = await algoQueDevuelvePromesa();   // espera AQUÍ sin bloquear Node
  return valor;
}
```

**Reglas:**
1. `await` solo se puede usar **dentro de una función `async`** (o en el nivel
   superior de un ES Module, tema 03).
2. `await promesa` → **pausa esa función** hasta que la promesa se cumple y te da el
   **valor**. El resto de Node sigue funcionando.
3. Una función `async` **siempre devuelve una promesa**, aunque hagas `return 5`.

### Ejemplo REAL copiado (`Async-hector/Advanced-JS-Asynchronous-code/index.js`)

```js
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
```

**Desglose de `getAllBreeds`:**

```
await fetch(url)        → espera la RESPUESTA HTTP (cabeceras, status...)
await res.json()        → espera a que llegue y se lea el CUERPO, y lo convierte a objeto
Object.keys(data.message) → { affenpinscher: [], african: [], ... } → ["affenpinscher", "african", ...]
return ...              → la promesa de getAllBreeds se cumple con ese array
```

> 📌 **Los dos `await` son necesarios**: `fetch` devuelve una promesa y `res.json()`
> devuelve **otra** promesa.

### La misma función en las tres etapas

```js
// 1️⃣ Callback (estilo inventado, fetch no funciona así)
getRandomDogCb((err, url) => {
  if (err) return console.error(err);
  console.log(url);
});

// 2️⃣ Promesas
function getRandomDogThen() {
  return fetch('https://dog.ceo/api/breeds/image/random')
    .then(res => res.json())
    .then(data => data.message);
}

// 3️⃣ async/await (Ejemplo REAL de tu proyecto)
async function getRandomDog() {
    const res = await fetch('https://dog.ceo/api/breeds/image/random');
    const data = await res.json();
    return data.message;
}
```

### Usar el resultado de una función `async`

```js
// ❌ Sin await → obtienes la promesa, no el valor
const url = getRandomDog();
console.log(url);           // Promise { <pending> }

// ✅ Con await (dentro de otra función async)
const url2 = await getRandomDog();
console.log(url2);          // "https://images.dog.ceo/breeds/..."

// ✅ O con .then
getRandomDog().then(url => console.log(url));
```

---

## 6.5 Encadenar pasos que dependen unos de otros

Con `await`, cuando un dato depende del anterior, simplemente escribes un paso
debajo de otro.

### Ejemplo REAL copiado (`Async-Hector-2/advanced-js-bikram-async-await/index.js`)

```js
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
```

**Flujo:**

```
getRandomCharacterInfo()
  │
  ├─ await getRandomCharacter()      → 1ª petición: un personaje al azar
  │      └─ character.episode = ["https://.../episode/1", ...]
  │
  ├─ await fetch(episodes[0])        → 2ª petición: NECESITA la URL de la 1ª
  │
  └─ return { img, name, episodes, firstEpisode, dateEpisode }
```

> 📌 Una función `async` puede llamar a **otra** función `async` con `await`
> (`await getRandomCharacter()`). Así construyes funciones pequeñas y las combinas.

### Ejemplo REAL: `await` condicional (`Async-Hector-2/.../index.js`)

```js
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
```

> 📌 Si no te pasan un pokémon, **primero** pide uno aleatorio y **después** pide su
> imagen. Con callbacks esto sería bastante más lioso.

---

## 6.6 Varias promesas a la vez: `Promise.all`

Si las peticiones **no dependen** unas de otras, no tiene sentido esperarlas **una
detrás de otra**: lánzalas todas y espera a que acaben todas.

```js
// ❌ En SERIE: 3 peticiones × 1 s = 3 s
const a = await fetch(url1);
const b = await fetch(url2);
const c = await fetch(url3);

// ✅ En PARALELO: ~1 s en total
const [a2, b2, c2] = await Promise.all([fetch(url1), fetch(url2), fetch(url3)]);
```

### Ejemplo REAL copiado (`Async-hector/Advanced-JS-Asynchronous-code/index.js`)

```js
async function fetchGithubUsers(userNames) {
    const promises = userNames.map(name =>
        fetch(`https://api.github.com/users/${name}`).then(res => res.json())
    );
    const users = await Promise.all(promises);
    // ...
    return users;
}
```

**Paso a paso:**

```
userNames = ["HectorFM99", "octocat"]
       │
       ▼  .map(...)  → lanza un fetch por usuario (TODOS a la vez)
promises = [Promise<usuario1>, Promise<usuario2>]
       │
       ▼  await Promise.all(promises)  → espera a que TODAS terminen
users = [{...usuario1}, {...usuario2}]   ← en el MISMO orden que userNames
```

### Variantes de `Promise`

| Método | Se cumple cuando... | Si una falla... |
|--------|---------------------|-----------------|
| `Promise.all([...])` | **Todas** se cumplen → array de resultados | ❌ Rechaza en cuanto **una** falla |
| `Promise.allSettled([...])` | Todas terminan (bien o mal) | ✅ Nunca rechaza; te dice cuál falló |
| `Promise.race([...])` | La **primera** que termine (bien o mal) | Depende de la primera |
| `Promise.any([...])` | La **primera que se cumpla** | Solo rechaza si **todas** fallan |

```js
// Ejemplo propio con allSettled
const resultados = await Promise.allSettled([getRandomDog(), getAllBreeds()]);
resultados.forEach(r => {
  if (r.status === "fulfilled") console.log("OK:", r.value);
  else console.log("Falló:", r.reason.message);
});
```

---

## 6.7 Manejo de errores: `try` / `catch`

Si una promesa **se rechaza** dentro de un `await`, se lanza un error, igual que un
`throw`. Se captura con `try/catch`.

```js
// Ejemplo propio
async function getRandomDogSeguro() {
  try {
    const res = await fetch('https://dog.ceo/api/breeds/image/random');

    if (!res.ok) {                                   // ⚠️ fetch NO falla con 404/500
      throw new Error(`HTTP ${res.status}`);
    }

    const data = await res.json();
    return data.message;
  } catch (err) {
    console.error("No se pudo obtener el perro:", err.message);
    return null;                                     // valor "seguro"
  } finally {
    console.log("Petición terminada");
  }
}
```

> ⚠️ **Trampa de `fetch`:** solo **rechaza** si hay un error de **red** (sin
> internet, dominio que no existe...). Un `404` o un `500` **no** salta al `catch`:
> hay que comprobar `res.ok` (o `res.status`) a mano.

### Ejemplo REAL copiado: `try/catch` + `res.ok` + timeout (`ProyectoReproductor/server/index.ts`)

Ruta `/api/tracks/search` (original en una sola línea; aquí **formateada** para
leerla y con un mensaje de error abreviado con `/* ... */`; la lógica es la misma):

```ts
app.get('/api/tracks/search', session, async (req, res) => {
  const q = String(req.query.q ?? '').trim();
  if (!q || q.length > 100) return res.status(400).json({ error: 'Escribe una búsqueda de hasta 100 caracteres.' });
  if (!process.env.JAMENDO_CLIENT_ID) return res.status(503).json({ error: 'Falta configurar el acceso al catálogo de Jamendo.' });
  try {
    const url = new URL('https://api.jamendo.com/v3.0/tracks/');
    url.search = new URLSearchParams({ client_id: process.env.JAMENDO_CLIENT_ID, format: 'json', limit: '30', search: q, audioformat: 'mp32', include: 'musicinfo' }).toString();
    const response = await fetch(url, { signal: AbortSignal.timeout(10000) });
    if (!response.ok) throw new Error();
    const payload = await response.json() as { headers?: { status?: string; error_message?: string }; results?: Array<Record<string, unknown>> };
    if (payload.headers?.status === 'failed') return res.status(502).json({ error: /* ... */ 'Jamendo no pudo completar la búsqueda.' });
    res.json((payload.results ?? []).map(t => ({ id: String(t.id), name: t.name, artist: t.artist_name, album: t.album_name, image: t.album_image, audio: t.audio, duration: t.duration })));
  } catch {
    res.status(502).json({ error: 'No se pudo consultar Jamendo. Comprueba la conexión e inténtalo de nuevo.' });
  }
});
```

**Lo que hay que aprender de aquí:**

| Línea | Para qué |
|-------|----------|
| `async (req, res) => {...}` | La ruta es `async` porque dentro hay `await` |
| `AbortSignal.timeout(10000)` | Si la API tarda más de 10 s, se cancela (rechaza la promesa) |
| `if (!response.ok) throw new Error();` | Convierte un 4xx/5xx en un error para que vaya al `catch` |
| `catch { res.status(502)... }` | Cualquier fallo → respuesta controlada, el servidor **no se cae** |

### Ejemplo REAL: `try/catch` síncrono para un fallo esperado (`ProyectoReproductor/server/index.ts`)

```ts
try{run('INSERT INTO users(id,email,password_hash) VALUES(?,?,?)',[id,email,hashPassword(password)]);}catch{return res.status(409).json({error:'Ya existe una cuenta con ese correo.'});}
```

> 📌 Si el email ya existe, la base de datos lanza un error (columna `UNIQUE`), y se
> transforma en un `409 Conflict` con un mensaje claro.

### ❌ / ✅ Errores típicos con `async`

```js
// ❌ Olvidar await: el try/catch NO captura el error (la promesa se rechaza "después")
try {
  getRandomDog();
} catch (err) { /* nunca llega aquí */ }

// ✅
try {
  await getRandomDog();
} catch (err) { console.error(err); }
```

```js
// ❌ await dentro de forEach: forEach NO espera
userNames.forEach(async (name) => {
  await guardar(name);
});
console.log("¿Terminado?");   // se imprime ANTES de guardar nada

// ✅ for...of sí espera (en serie)
for (const name of userNames) {
  await guardar(name);
}

// ✅ o en paralelo con map + Promise.all
await Promise.all(userNames.map((name) => guardar(name)));
```

```js
// ❌ Promesa rechazada sin capturar → en Node moderno TERMINA el proceso
//    "UnhandledPromiseRejection"
start();          // si start() rechaza y nadie lo captura...

// ✅ Capturar en el punto de entrada (ejemplo propio)
start().catch((err) => {
  console.error("No se pudo arrancar:", err);
  process.exit(1);
});
```

> 📌 En tu `api-rastro` se hace `start();` sin `.catch`. Si MongoDB no está
> encendido, `connectDb()` rechaza, Node muestra el error y el proceso termina. Para
> un proyecto de clase es aceptable; en producción conviene el `.catch` con un mensaje
> claro.

---

## 6.8 Async en el backend: el patrón que repetirás siempre

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/db.ts`)

```ts
export async function connectDb() {
  await client.connect();
  db = client.db(process.env.DB_NAME ?? "rastro");
  console.log(`Conectado a MongoDB (base de datos ${db.databaseName})`);
}
```

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/index.ts`)

```ts
app.get("/api/health", async (_req, res) => {
  await getDb().command({ ping: 1 });
  res.json({ status: "ok", db: getDb().databaseName });
});

// ...

async function start() {
  await connectDb();

  app.listen(PORT, () => {
    console.log(`Servidor escuchando en http://localhost:${PORT}`);
  });
}

start();
```

**Orden garantizado gracias a `await`:**

```
1. start()
2.   await connectDb()       → espera a que MongoDB conteste
3.   app.listen(...)         → SOLO entonces empieza a aceptar peticiones
4.     callback → "Servidor escuchando en http://localhost:3000"
```

### Ejemplo REAL: métodos que devuelven `Promise` aunque no esperen nada

**Archivo:** `API_Godines/api-express/api-books-claude/src/services/books-storage.ts`
(el comentario es del propio archivo)

```ts
/**
 * Los métodos devuelven `Promise` aunque la versión en memoria no lo necesite:
 * así el contrato ya sirve para almacenamientos asíncronos como una base de datos.
 */
```

```ts
// books-storage-in-memory.ts
async findAll(): Promise<Book[]> {
  // Copia: quien la reciba no puede modificar el almacenamiento desde fuera
  return [...this.books];
}
```

> 📌 Una función `async` **siempre** devuelve una promesa. Si mañana cambias el array
> en memoria por MongoDB (que sí es asíncrono), las rutas que hacen
> `await storage.findAll()` **no cambian**.

---

## 6.9 Tabla resumen de las tres etapas

| | Callbacks | Promesas | `async`/`await` |
|---|-----------|----------|-----------------|
| Sintaxis | `fn(args, (err, data) => {})` | `fn().then().catch()` | `const data = await fn()` |
| Errores | `if (err)` en cada nivel | Un `.catch()` | `try/catch` |
| Varias en paralelo | Muy difícil | `Promise.all` | `await Promise.all` |
| Legibilidad | 😱 Pirámide | 🙂 Cadena | 😃 Como código normal |
| Dónde lo verás | APIs antiguas de Node, eventos, `listen` | Librerías, `fetch().then` | **Todo el código moderno** |

---

## ✅ Resumen rápido

1. Una operación asíncrona **no** da su resultado en la línea siguiente.
2. **Callback**: función que se llama al terminar; en Node, `(err, data)`.
3. **Promesa**: objeto con estados *pending* → *fulfilled* / *rejected*; `.then` / `.catch`.
4. **`async`/`await`**: escribe código asíncrono como si fuera síncrono; `await` solo dentro de `async`.
5. Una función `async` **siempre** devuelve una promesa → quien la llama necesita `await` o `.then`.
6. `fetch` necesita **dos** `await` (`fetch` y `res.json()`) y **no** salta al `catch` con 404/500 → `res.ok`.
7. Tareas independientes → `await Promise.all([...])`.
8. Errores con `try/catch`; nunca dejes promesas rechazadas sin capturar.
9. ❌ `await` dentro de `forEach` → usa `for...of` o `Promise.all`.

👉 **Siguiente:** `07-herramientas-tsx-nodemon-typescript-eslint.md`
