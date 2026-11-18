# 04 — MIDDLEWARE, CORS Y MANEJO DE ERRORES

> Un **middleware** es una función que se ejecuta **en medio** del camino entre que llega
> la petición y se envía la respuesta. Express entero está hecho de middlewares en fila:
> cada uno hace algo y **pasa el turno** al siguiente con `next()`.

---

## 4.1 El concepto: una cadena de funciones

```ts
(req, res, next) => {
  // 1. hacer algo con req / res
  // 2. O BIEN responder (y la cadena se corta)
  //    O BIEN llamar a next() (y pasa al siguiente)
}
```

```
Petición
   │
   ▼
[logger] ──next()──► [CORS] ──next()──► [express.json()] ──next()──► [ruta] ──► res.json()
                        │
                        └── si es OPTIONS: res.sendStatus(204) y se CORTA aquí
```

| Una función de middleware puede... | Ejemplo |
|------------------------------------|---------|
| Leer/modificar `req` | `express.json()` rellena `req.body` |
| Modificar `res` | CORS añade cabeceras |
| **Responder** y cortar la cadena | El 404 final |
| Llamar a `next()` para seguir | El logger |
| Llamar a `next(err)` para saltar al manejador de errores | (Express 4) |

> ⚠️ Si un middleware **no responde y no llama a `next()`**, la petición se queda colgada
> para siempre.

---

## 4.2 `app.use()` — registrar middleware

```ts
app.use(funcion);              // se ejecuta en TODAS las peticiones
app.use("/api", funcion);      // solo en las que empiezan por /api
app.get("/x", mw, handler);    // solo en esa ruta (middleware de ruta)
```

> 📌 Las rutas (`app.get`, `router.post`...) también son middlewares: solo que se
> ejecutan si coinciden método y URL, y normalmente responden en vez de llamar a `next()`.

---

## 4.3 Middleware propio: el logger

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/src/index.ts`)

```ts
// 1. Registrar cada petición en la terminal
app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

En la terminal verás:

```
GET /api/books?country=chi
POST /api/books
PATCH /api/books/1
```

```ts
// ❌ Olvidar next(): NINGUNA petición llega a las rutas
app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
});

// ✅
app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

### Otro middleware propio (`ProyectoReproductor/server/index.ts`)

```ts
app.use((req,res,next)=>{ if(req.path.startsWith('/api/')) res.setHeader('Cache-Control','no-store'); next(); });
```

Formateado:

```ts
app.use((req, res, next) => {
  if (req.path.startsWith('/api/')) {
    res.setHeader('Cache-Control', 'no-store');   // que el navegador NO guarde en caché las respuestas de la API
  }
  next();
});
```

---

## 4.4 ⚠️ El ORDEN de los middlewares

Se ejecutan **en el orden en que los escribes**. Tus proyectos siguen siempre el mismo
esquema numerado:

### Ejemplo REAL copiado (`mongo-api-libros/.../src/index.ts`, comentarios originales)

```ts
// 1. Registrar cada petición en la terminal
app.use((req, _res, next) => { ... });

// 2. CORS: permitir peticiones desde nuestra página web
app.use((req, res, next) => { ... });

// 3. Leer bodies JSON
app.use(express.json());

// 4. Rutas
app.get("/", ...);
app.use("/api/products", productsRouter);
app.use("/api/categories", categoriesRouter);

// 5. Ninguna ruta ha respondido: 404
app.use((req, res) => { ... });

// 6. Ha ocurrido un error: respuesta JSON sin detalles internos
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => { ... });
```

| Paso | Por qué va ahí |
|------|----------------|
| 1. Logger | Primero, para registrar **todo**, incluso lo que luego falle |
| 2. CORS | Antes de las rutas, para que las cabeceras estén en **todas** las respuestas |
| 3. `express.json()` | Antes de las rutas, para que `req.body` esté listo |
| 4. Rutas | El trabajo de verdad |
| 5. 404 | Solo se llega si **ninguna** ruta respondió |
| 6. Errores | Siempre el **último** |

> 📌 Memoriza este orden. Es la plantilla de `index.ts` para cualquier API Express.

---

## 4.5 Middlewares que vienen con Express o con paquetes

| Middleware | Paquete | Qué hace |
|------------|---------|----------|
| `express.json()` | express | Lee bodies JSON → `req.body` |
| `express.urlencoded()` | express | Lee formularios HTML clásicos |
| `express.static("public")` | express | Sirve archivos estáticos (HTML, CSS, JS, imágenes) |
| `cookieParser()` | `cookie-parser` | Lee cookies → `req.cookies` |
| `cors()` | `cors` | CORS configurable (alternativa a hacerlo a mano) |

### Ejemplo REAL copiado (`ProyectoReproductor/server/index.ts`)

```ts
import cookieParser from 'cookie-parser';
// ...
app.use(express.json({ limit: '32kb' })); app.use(cookieParser()); app.use((req,res,next)=>{ ... }); app.use(express.static('public'));
```

- `express.static('public')` → si pides `/styles.css`, Express busca `public/styles.css` y
  lo envía. Así el **mismo servidor** sirve el frontend y la API.

---

## 4.6 CORS: dejar que tu web llame a la API

**El problema:** tu frontend está en `http://127.0.0.1:5500` (Live Server) y la API en
`http://localhost:3000`. Son **orígenes distintos**, y el navegador **bloquea** las
peticiones entre orígenes salvo que el servidor diga "este origen tiene permiso".

```
Navegador (127.0.0.1:5500) ── fetch ──► API (localhost:3000)
                                          │
              ◄── Access-Control-Allow-Origin: http://127.0.0.1:5500 ──┘
              ✅ El navegador deja pasar la respuesta
```

Además, para métodos como `PATCH`/`DELETE` o con `Content-Type: application/json`, el
navegador manda antes una petición de **comprobación** (`OPTIONS`, el "preflight").

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/src/index.ts`)

```ts
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
```

| Cabecera | Significado |
|----------|-------------|
| `Access-Control-Allow-Origin` | **Qué web** puede llamar a la API |
| `Access-Control-Allow-Methods` | **Qué métodos** se permiten |
| `Access-Control-Allow-Headers` | **Qué cabeceras** puede mandar (`Content-Type` para JSON) |

- Si es `OPTIONS` (preflight) → contesta `204` y **corta** (no hace falta ir a las rutas).
- Si no → `next()` y sigue.

> 📌 En `02b-express-mongodb` la línea de métodos es `"GET, POST, PATCH, DELETE"` (sin
> `PUT`), porque esa API no tiene rutas PUT. Permite **solo lo que usas**.

### Alternativa con el paquete `cors` (no está en tus proyectos)

```ts
// Ejemplo propio: npm install cors  (+ npm i -D @types/cors)
import cors from "cors";

app.use(cors({
  origin: "http://127.0.0.1:5500",
  methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
}));
```

Hace lo mismo que tu middleware a mano (incluido responder al `OPTIONS`).

```ts
// ❌ Permitir cualquier origen en una API con cookies/sesión
res.setHeader("Access-Control-Allow-Origin", "*");

// ✅ Lista concreta de orígenes de confianza
res.setHeader("Access-Control-Allow-Origin", "http://127.0.0.1:5500");
```

> 📌 `ProyectoReproductor` **no necesita CORS**: el frontend lo sirve el propio Express con
> `express.static('public')`, así que todo está en el **mismo origen**.

---

## 4.7 Middleware de ruta: proteger rutas con sesión

Un middleware se puede poner **solo en algunas rutas**, entre la URL y el handler.

### Ejemplo REAL copiado (`ProyectoReproductor/server/index.ts`)

```ts
function userFor(req: express.Request) { const token = req.cookies?.nightjar_session; return token ? one<{id:string;email:string}>('SELECT users.id, users.email FROM sessions JOIN users ON users.id=sessions.user_id WHERE sessions.token=? AND sessions.expires_at>?',[token,Date.now()]) : undefined; }
function session(req: express.Request, res: express.Response, next: express.NextFunction) { const user=userFor(req); if(!user)return res.status(401).json({error:'Inicia sesión para continuar.'}); res.locals.user=user; next(); }
```

Formateado y comentado:

```ts
// Busca el usuario a partir de la cookie de sesión
function userFor(req: express.Request) {
  const token = req.cookies?.nightjar_session;
  return token
    ? one<{ id: string; email: string }>(
        'SELECT users.id, users.email FROM sessions JOIN users ON users.id=sessions.user_id WHERE sessions.token=? AND sessions.expires_at>?',
        [token, Date.now()]
      )
    : undefined;
}

// MIDDLEWARE: deja pasar solo si hay sesión válida
function session(req: express.Request, res: express.Response, next: express.NextFunction) {
  const user = userFor(req);
  if (!user) return res.status(401).json({ error: 'Inicia sesión para continuar.' });  // corta
  res.locals.user = user;    // guarda el usuario para la ruta
  next();                    // sigue
}
```

Uso:

```ts
app.get('/api/library/:kind', session, (req, res) => {
  // aquí YA sabes que hay usuario: res.locals.user.id
  const rows = many<{track_json:string}>(`SELECT track_json FROM ${kind} WHERE user_id=? ...`, [res.locals.user.id]);
  // ...
});
```

```
GET /api/library/favorites
   │
   ▼
session ──¿hay cookie válida?── NO ──► 401 { error: 'Inicia sesión para continuar.' }
   │
   SÍ → res.locals.user = { id, email } → next()
   │
   ▼
handler → usa res.locals.user.id
```

> 📌 `res.locals` es un objeto vacío que vive **durante una petición**: sirve para pasar
> datos de un middleware a la ruta.

> 📌 Esta idea (un middleware que comprueba si estás autenticado) es la base de cualquier
> sistema de login: con JWT, con sesiones, con lo que sea.

---

## 4.8 El 404 final: "ninguna ruta ha respondido"

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/index.ts`)

```ts
app.use("/api/listings", listingsRouter);
app.use("/api/offers", offersRouter);

app.use((req, res) => {
  res.status(404).json({ error: `No existe la ruta ${req.method} ${req.url}` });
});
```

Sin este middleware, Express responde un HTML feo `Cannot GET /api/loquesea`. Con él,
la API **siempre** responde JSON:

```json
{ "error": "No existe la ruta GET /api/loquesea" }
```

---

## 4.9 El manejador de errores (4 parámetros)

Un middleware con **cuatro** parámetros `(err, req, res, next)` es especial: Express solo
lo llama cuando **ha ocurrido un error**.

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/src/index.ts`)

```ts
// 6. Ha ocurrido un error: respuesta JSON sin detalles internos
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  if (err instanceof SyntaxError) {
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }

  console.error(err);
  res.status(500).json({ error: "Error interno del servidor" });
});
```

| Caso | Qué pasa |
|------|----------|
| El cliente manda un JSON mal escrito (`{ "title": `) | `express.json()` lanza un `SyntaxError` → **400** |
| Cualquier otro error (bug, BD caída...) | Se escribe en la terminal (`console.error`) y se responde **500** genérico |

> ⚠️ **Los 4 parámetros son obligatorios**, aunque no uses `next`. Express distingue el
> manejador de errores **contando los parámetros**. Con 3, lo trataría como middleware normal.

> 🔒 **Nunca** devuelvas `err.message` o el stack al cliente: puede revelar rutas de
> archivos, consultas de la BD, etc. Al cliente un mensaje genérico; el detalle, a la
> terminal.

```ts
// ❌ Filtra detalles internos
res.status(500).json({ error: err.message, stack: err.stack });

// ✅
console.error(err);
res.status(500).json({ error: "Error interno del servidor" });
```

---

## 4.10 ¿Cómo llegan los errores al manejador?

| Forma | Express 4 | Express 5 |
|-------|-----------|-----------|
| `throw` en función **síncrona** | ✅ | ✅ |
| `throw` / promesa rechazada en función **`async`** | ❌ (se pierde; hay que `try/catch` + `next(err)`) | ✅ automático |
| `next(err)` | ✅ | ✅ |

### Ejemplo REAL: relanzar el error que no sabes tratar (`mongo-api-libros/.../src/routes/categories.ts`)

```ts
router.post("/", async (req, res) => {
  const { name } = req.body ?? {};
  const error = validateName(name);

  if (error) {
    res.status(400).json({ error });
    return;
  }

  const newCategory: Category = { name: name.trim() };

  try {
    const result = await categoriesCollection().insertOne(newCategory);
    res.status(201).json({ _id: result.insertedId, ...newCategory });
  } catch (err) {
    if (isDuplicateKey(err)) {
      res.status(409).json({ error: "Ya existe una categoría con ese nombre" });
      return;
    }
    throw err; // cualquier otro error, al middleware de errores (500)
  }
});
```

```
insertOne falla
   │
   ├─ ¿Es "clave duplicada" (código 11000)?  → 409, lo tratas TÚ
   └─ ¿Otra cosa?                             → throw err → Express 5 → manejador → 500
```

> 📌 Patrón importante: en el `catch` **solo** trata los errores que **esperas**; el
> resto, `throw err` para que los gestione el manejador global.

### El mismo patrón en Express 4 (ejemplo propio)

```ts
router.post("/", async (req, res, next) => {
  try {
    const result = await categoriesCollection().insertOne(newCategory);
    res.status(201).json(result);
  } catch (err) {
    next(err);      // en Express 4 hay que pasarlo a mano
  }
});
```

### `try/catch` para errores de otra API (`ProyectoReproductor/server/index.ts`)

```ts
app.get('/api/tracks/featured',session,async(req,res)=>{if(!process.env.JAMENDO_CLIENT_ID)return res.status(503).json({error:'Falta configurar el acceso al catálogo de Jamendo.'});try{ /* fetch a Jamendo */ }catch{res.status(502).json({error:'No se pudo consultar Jamendo. Comprueba la conexión e inténtalo de nuevo.'});}});
```

Aquí el `catch` responde directamente con **502** (falla un servicio externo), en vez de
dejar que llegue al manejador como 500.

---

## 4.11 Errores comunes con middleware

| Error | Causa | Solución |
|-------|-------|----------|
| Todas las peticiones se cuelgan | Un middleware sin `next()` | Llama a `next()` o responde |
| `req.body` es `undefined` | `express.json()` después de las rutas | Muévelo antes |
| Error de CORS en la consola del navegador | Falta la cabecera, origen mal escrito o no respondes al `OPTIONS` | Revisa el middleware de CORS |
| El manejador de errores no se ejecuta | Tiene 3 parámetros o está antes de las rutas | 4 parámetros y al final |
| Error en ruta `async` deja la petición colgada | Estás en Express 4 | `try/catch` + `next(err)` |
| El 404 salta en rutas que existen | El 404 está antes de las rutas | Al final, antes del de errores |

---

## ✅ Resumen rápido

1. Middleware = `(req, res, next) => {}`; o responde o llama a `next()`.
2. Se ejecutan **en orden**: logger → CORS → `express.json()` → rutas → 404 → errores.
3. CORS: cabeceras `Access-Control-Allow-*` + responder `204` a `OPTIONS`.
4. Middleware de ruta: `app.get("/x", session, handler)`; pasa datos con `res.locals`.
5. Manejador de errores = **4 parámetros**, el último de todos, sin filtrar detalles.
6. Express 5 lleva solo los errores `async` al manejador; en el `catch`, trata lo esperado y `throw err` lo demás.

👉 **Siguiente:** `05-validacion-de-datos.md`
