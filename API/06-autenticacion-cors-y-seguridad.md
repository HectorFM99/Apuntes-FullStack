# 06 — AUTENTICACIÓN, CORS Y SEGURIDAD BÁSICA

> HTTP **no tiene memoria**: cada petición llega "sola". Para que el servidor sepa
> **quién eres**, en cada petición tienes que mandarle una **prueba**: una
> **cookie de sesión** o un **token**. Además, el navegador aplica una regla de
> seguridad (**CORS**) que decide qué webs pueden llamar a qué APIs.

---

## 6.1 Autenticación vs autorización

| Concepto | Pregunta | Si falla |
|----------|----------|----------|
| **Autenticación** (*authN*) | **¿Quién eres?** | `401 Unauthorized` |
| **Autorización** (*authZ*) | **¿Puedes hacer esto?** | `403 Forbidden` |

Ejemplo: entras con tu cuenta (autenticado ✅), pero intentas borrar el anuncio de
otro usuario (no autorizado ❌ → 403).

> 📌 La carpeta `autenticacion/` de tu bootcamp está **vacía** de momento. El
> ejemplo real de autenticación lo tienes en `ProyectoReproductor`, que usa
> **sesiones con cookie**.

---

## 6.2 Opción 1: sesiones con cookie (lo que usa tu reproductor)

### Cómo funciona

```
1. POST /api/auth/login  { email, password }
                  │
                  ▼
   Servidor comprueba la contraseña, crea un token aleatorio,
   lo guarda en la tabla "sessions" y responde:
   Set-Cookie: nightjar_session=8f3a...; HttpOnly; SameSite=Lax
                  │
                  ▼
2. El NAVEGADOR guarda la cookie solo.
                  │
                  ▼
3. En CADA petición siguiente el navegador la manda solo:
   GET /api/library/favorites
   Cookie: nightjar_session=8f3a...
                  │
                  ▼
   Servidor busca ese token en "sessions" → sabe que eres tú.
```

### Ejemplo REAL copiado — crear la sesión (`ProyectoReproductor/dist/server/index.js`)

```js
function createSession(userId, res) { const token = randomBytes(32).toString('hex'); run('INSERT INTO sessions(token,user_id,expires_at) VALUES(?,?,?)', [token, userId, Date.now() + 1209600000]); res.cookie('nightjar_session', token, { httpOnly: true, sameSite: 'lax', secure: process.env.NODE_ENV === 'production', maxAge: 1209600000 }); }
```

| Opción de la cookie | Qué hace |
|---------------------|----------|
| `httpOnly: true` | JavaScript **no** puede leerla (`document.cookie` no la ve) → protege contra robos por XSS |
| `sameSite: 'lax'` | No se envía en peticiones "raras" desde otras webs → ayuda contra CSRF |
| `secure: true` (en producción) | Solo viaja por **HTTPS** |
| `maxAge: 1209600000` | Caduca en 14 días (en milisegundos) |

### Ejemplo REAL copiado — proteger rutas (`ProyectoReproductor/dist/server/index.js`)

```js
function userFor(req) { const token = req.cookies?.nightjar_session; return token ? one('SELECT users.id, users.email FROM sessions JOIN users ON users.id=sessions.user_id WHERE sessions.token=? AND sessions.expires_at>?', [token, Date.now()]) : undefined; }
function session(req, res, next) { const user = userFor(req); if (!user)
    return res.status(401).json({ error: 'Inicia sesión para continuar.' }); res.locals.user = user; next(); }
```

Y se pone delante de las rutas privadas:

```js
app.delete('/api/library/favorites/:trackId', session, (req, res) => { ... });
```

### Ejemplo REAL copiado — "¿quién soy?" desde el cliente (`ProyectoReproductor/public/app.js`)

```js
(async () => {
    try {
        const user = await request('/api/auth/me');
        $('#account-email').textContent = user.email;
        const saved = await request('/api/library/favorites');
        favorites = new Set(saved.map(track => track.id));
        renderRoute();
    }
    catch {
        window.location.href = 'index.html';
    }
})();
```

Y lo contrario en `login.js`: si **ya** hay sesión, te manda directo a la app:

```js
(async () => {
    try {
        await request('/api/auth/me');
        window.location.href = 'app.html';
    }
    catch {
        // No active session, stay on landing/login page
    }
})();
```

> 📌 Fíjate que en el `fetch` del cliente **no aparece la cookie por ningún sitio**.
> El navegador la manda **automáticamente** porque el frontend y la API están en el
> **mismo dominio** (los sirve el mismo Express con `express.static('public')`).

### Logout

```js
app.post('/api/auth/logout', (req, res) => { const token = req.cookies?.nightjar_session; if (token)
    run('DELETE FROM sessions WHERE token=?', [token]); res.clearCookie('nightjar_session'); res.status(204).end(); });
```

→ Borra la sesión de la base de datos **y** le dice al navegador que borre la cookie.

---

## 6.3 Opción 2: tokens (JWT) con el header `Authorization`

*(No está en tus proyectos todavía: ejemplo propio)*

Un **JWT** (*JSON Web Token*) es un texto con 3 partes separadas por puntos:

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI0MiIsImVtYWlsIjoiaEBtYWlsLmNvbSJ9.Xk3...firma
└──── header ──────┘ └─────────────── payload (datos) ─────────────┘ └ firma ┘
```

- El **payload** lleva datos como `{ "sub": "42", "email": "h@mail.com", "exp": 1767225600 }`.
- Está codificado en **Base64**, **no cifrado**: cualquiera puede leerlo.
  **Nunca metas contraseñas** en un JWT.
- La **firma** la hace el servidor con un secreto. Si alguien cambia el payload, la
  firma ya no cuadra y el servidor lo rechaza.

### Flujo

```js
// 1. Login: el servidor devuelve el token en el body
const res = await fetch("/api/auth/login", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ email, password }),
});
const { token } = await res.json();

// 2. El cliente lo guarda (en memoria, o localStorage con cuidado)
localStorage.setItem("token", token);

// 3. Lo manda EN CADA petición, a mano, en el header Authorization
const favoritos = await fetch("/api/library/favorites", {
  headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
});
```

### Cookies de sesión vs JWT

| | Cookie de sesión (tu reproductor) | JWT en `Authorization` |
|-|-----------------------------------|------------------------|
| ¿Dónde se guarda? | Cookie (el navegador) | Donde tú decidas (memoria, `localStorage`) |
| ¿Quién la manda? | El navegador, **solo** | **Tú**, a mano en cada `fetch` |
| ¿El servidor guarda algo? | Sí (tabla `sessions`) | No necesariamente (verifica la firma) |
| Cerrar sesión | Borrar la sesión → efecto inmediato | Difícil: el token vale hasta que caduca |
| Riesgo principal | CSRF (mitigado con `SameSite`) | XSS si está en `localStorage` |
| Ideal para | Web + API en el mismo dominio | Apps móviles, APIs para terceros, varios servicios |

### API keys

Otra forma muy común en APIs públicas: una **clave** que identifica a tu
**aplicación** (no a un usuario). Jamendo usa un `client_id`:

```js
url.search = new URLSearchParams({ client_id: process.env.JAMENDO_CLIENT_ID, format: 'json', limit: '30', search: q, audioformat: 'mp32', include: 'musicinfo' }).toString();
```

---

## 6.4 Secretos: nunca en el frontend

Tu `ProyectoReproductor/replica/backend.md` lo dice en la primera línea:

```md
- [x] Jamendo official public API route; credentials stay server-side.
```

```
❌  app.js (navegador) ──── client_id ────▶ Jamendo
    Cualquiera que abra las DevTools ve tu clave.

✅  app.js ──▶ TU servidor (lee process.env.JAMENDO_CLIENT_ID) ──▶ Jamendo
    La clave nunca sale del servidor.
```

Los secretos van en un archivo `.env` que **no se sube** a Git, y se comparte un
`.env.example` **vacío** como plantilla.

### Ejemplo REAL copiado (`ProyectoReproductor/.env.example`)

```env
JAMENDO_CLIENT_ID=
SESSION_SECRET=
PORT=3000
DATABASE_URL=file:./data/nightjar.sqlite
```

### Ejemplo REAL copiado (`api-rastro/api-rastro/.env.example`)

```env
MONGODB_URI=mongodb://usuario:contraseña@localhost:27017/?authSource=admin
DB_NAME=rastro
```

> 📌 Si la clave falta, tu servidor responde un **503** con un mensaje claro en vez
> de romperse:
>
> ```js
> if (!process.env.JAMENDO_CLIENT_ID)
>     return res.status(503).json({ error: 'Falta configurar el acceso al catálogo de Jamendo.' });
> ```

---

## 6.5 Contraseñas: nunca en texto plano

### Ejemplo REAL copiado (`ProyectoReproductor/dist/server/index.js`)

```js
function hashPassword(password, salt = randomBytes(16).toString('hex')) { return `${salt}:${scryptSync(password, salt, 64).toString('hex')}`; }
function passwordMatches(password, stored) { const [salt, hash] = stored.split(':'); const actual = scryptSync(password, salt, 64), expected = Buffer.from(hash, 'hex'); return actual.length === expected.length && timingSafeEqual(actual, expected); }
```

- Se guarda un **hash** (una "huella" que no se puede deshacer) con una **sal**
  aleatoria, no la contraseña.
- Al hacer login se vuelve a calcular el hash y se compara.

### Mensaje de error genérico en el login

```js
return res.status(401).json({ error: 'Correo o contraseña incorrectos.' });
```

```js
// ❌ Le dices a un atacante qué emails existen
"Ese email no está registrado"
"Contraseña incorrecta"

// ✅ Mismo mensaje en los dos casos
"Correo o contraseña incorrectos."
```

---

## 6.6 CORS — por qué el navegador "bloquea" tu fetch

**Origen** = protocolo + dominio + puerto. Estos son **orígenes distintos**:

```
http://localhost:5500   (Live Server con tu index.html)
http://localhost:3000   (tu API Express)
```

La **política del mismo origen** dice: una web solo puede **leer** respuestas de su
**mismo origen**... **salvo** que el servidor dé permiso con cabeceras **CORS**
(*Cross-Origin Resource Sharing*).

### El error típico (en la consola del navegador)

```
Access to fetch at 'http://localhost:3000/api/books' from origin
'http://localhost:5500' has been blocked by CORS policy: No
'Access-Control-Allow-Origin' header is present on the requested resource.
```

### Puntos clave

- CORS lo aplica **el navegador**, no el servidor. Por eso **con `curl`, Postman o
  Node funciona** y en el navegador no.
- Se arregla **en el servidor** (añadiendo cabeceras), **nunca** en el `fetch` del
  cliente.
- Las APIs públicas que usaste (JSONPlaceholder, Dog CEO, PokéAPI, GitHub) ya
  responden con `Access-Control-Allow-Origin: *`, por eso tus ejercicios funcionaban
  desde un `index.html`.
- Tu `ProyectoReproductor` **no necesita CORS**: el HTML y la API salen del mismo
  servidor y del mismo puerto.

### Cómo se arregla en Express *(ejemplo propio; no está en tus proyectos)*

```bash
npm install cors
```

```js
import cors from "cors";

app.use(cors({ origin: "http://localhost:5500" }));   // solo permites tu frontend
// app.use(cors());   // permite a TODO el mundo (vale para APIs públicas o pruebas)
```

### Preflight (la petición `OPTIONS`)

Para peticiones "no simples" (por ejemplo con `Content-Type: application/json`,
o métodos `PUT`/`PATCH`/`DELETE`), el navegador manda **antes** una petición
`OPTIONS` preguntando permiso:

```
OPTIONS /api/books/1                     ← "¿puedo hacer un PATCH con JSON?"
Origin: http://localhost:5500
Access-Control-Request-Method: PATCH
Access-Control-Request-Headers: content-type

204 No Content                           ← "sí"
Access-Control-Allow-Origin: http://localhost:5500
Access-Control-Allow-Methods: GET,POST,PUT,PATCH,DELETE
Access-Control-Allow-Headers: content-type
```

→ Si ves en la pestaña **Network** una petición `OPTIONS` antes de la tuya, es
esto. La librería `cors` la responde por ti.

### CORS + cookies

Si frontend y API están en **orígenes distintos** y usas cookies:

```js
// Cliente: hay que pedir explícitamente que mande las cookies
fetch("http://localhost:3000/api/auth/me", { credentials: "include" });

// Servidor: hay que permitirlo (y el origin NO puede ser "*")
app.use(cors({ origin: "http://localhost:5500", credentials: true }));
```

---

## 6.7 Otras medidas de seguridad que ya aplicas

| Medida | Dónde | Código |
|--------|-------|--------|
| Limitar tamaño del body | `ProyectoReproductor` | `app.use(express.json({ limit: '32kb' }));` |
| Validar longitud de búsquedas | `ProyectoReproductor` | `if (!q \|\| q.length > 100) return res.status(400)...` |
| Timeout al llamar a otra API | `ProyectoReproductor` | `fetch(url, { signal: AbortSignal.timeout(10000) })` |
| Queries parametrizadas (anti SQL injection) | `ProyectoReproductor` | `run('DELETE FROM sessions WHERE token=?', [token])` |
| No filtrar errores internos | `api-rastro` | `res.status(500).json({ error: "Error interno del servidor" })` |
| Validar ids antes de usarlos | `api-rastro` | `if (!ObjectId.isValid(id)) ... 400` |
| No dejar que el cliente fije campos sensibles | `api-rastro` | `status: "AVAILABLE"`, `views: 0` |

Y lo que tu propio checklist (`ProyectoReproductor/replica/backend.md`) deja
pendiente — buena lista de "siguientes pasos":

```md
- [ ] Email verification, password reset, and rate limiting.
- [ ] CSRF protections and account deletion.
```

> 📌 **Rate limiting** = limitar cuántas peticiones puede hacer una IP por minuto
> (responde `429 Too Many Requests`). Evita que alguien pruebe 1 millón de
> contraseñas en tu `/login`.
