# 01 — QUÉ ES UNA API Y CÓMO FUNCIONA HTTP

> Una **API** (*Application Programming Interface*) es una **puerta de entrada** que
> un programa ofrece para que **otros programas** le pidan datos o le manden hacer
> cosas. En web, esa conversación se hace con el protocolo **HTTP** y los datos
> viajan casi siempre en formato **JSON**.

---

## 1.1 La idea en una frase

```
Tu código  ──── "dame el post 1" ────▶  API (servidor)
Tu código  ◀─── { "id": 1, ... } ─────  API (servidor)
```

- **Tú no ves la base de datos** del servidor ni su código.
- Solo conoces **las direcciones (URLs)** a las que puedes llamar y **qué devuelven**.
- Ese "menú" de URLs se llama **endpoints** y suele venir en la **documentación**.

> 📌 Una API es como la **carta de un restaurante**: tú pides lo que pone en la
> carta (endpoints), la cocina (servidor) lo prepara y el camarero (HTTP) te lo trae.
> Nunca entras en la cocina.

### APIs que ya has usado en tus proyectos

| API | URL base | Proyecto |
|-----|----------|----------|
| JSONPlaceholder (API falsa de pruebas) | `https://jsonplaceholder.typicode.com` | `Prueba-Api/` |
| Dog CEO (perritos) | `https://dog.ceo/api` | `Async-hector/`, `Async-Hector-2/` |
| GitHub | `https://api.github.com` | `Async-hector/` |
| PokéAPI | `https://pokeapi.co/api/v2` | `Async-Hector-2/` |
| Rick and Morty | `https://rickandmortyapi.com/api` | `Async-Hector-2/` |
| Jamendo (música) | `https://api.jamendo.com/v3.0` | `ProyectoReproductor/` (desde el servidor) |
| **Tu propia API** de libros | `http://localhost:3000/api/books` | `API_Godines/.../api-books-claude/` |
| **Tu propia API** de rastro | `http://localhost:3000/api/listings` | `api-rastro/api-rastro/` |

---

## 1.2 Cliente y servidor

| Rol | Qué hace | Ejemplos en tus proyectos |
|-----|----------|---------------------------|
| **Cliente** | **Pide** cosas (hace la *petición*, *request*) | El navegador, tu `fetch`, `curl`, Postman |
| **Servidor** | **Responde** (devuelve la *respuesta*, *response*) | JSONPlaceholder, tu servidor Express |

```
┌──────────────┐   petición HTTP (request)    ┌──────────────┐
│   CLIENTE    │ ───────────────────────────▶ │   SERVIDOR   │
│ (navegador,  │                              │ (Express,    │
│  fetch, curl)│ ◀─────────────────────────── │  API pública)│
└──────────────┘   respuesta HTTP (response)  └──────────────┘
```

**Reglas importantes:**
- El **cliente siempre empieza** la conversación. El servidor no te escribe "porque sí".
- Cada petición es **independiente** (HTTP *no tiene memoria*, se dice que es
  **stateless**). Si el servidor tiene que saber quién eres, se lo tienes que
  recordar en **cada** petición (con una cookie o un token → ver `06`).
- Un mismo programa puede ser **las dos cosas**: en `ProyectoReproductor` tu servidor
  es **servidor** para tu `app.js`, y a la vez es **cliente** de la API de Jamendo.

```
app.js (navegador) ──▶ TU servidor Express ──▶ API de Jamendo
       cliente            servidor + cliente        servidor
```

---

## 1.3 Anatomía de una URL

```
https://jsonplaceholder.typicode.com/posts/2?userId=1&_limit=5#comentarios
└─┬─┘   └────────────┬─────────────┘└──┬──┘└────────┬────────┘└─────┬─────┘
protocolo         dominio (host)      ruta    query string     fragmento
                                     (path)   (parámetros)   (no llega al servidor)
```

Y en local, con tu propia API:

```
http://localhost:3000/api/books/1
└─┬┘   └───┬───┘└─┬┘└─────┬─────┘
proto   host   puerto    ruta
```

| Parte | Para qué sirve | Ejemplo |
|-------|----------------|---------|
| Protocolo | Cómo se habla (`http` o `https` cifrado) | `https` |
| Host | Qué máquina | `api.github.com`, `localhost` |
| Puerto | Qué "puerta" de esa máquina | `3000` (por defecto 80 en http y 443 en https) |
| Ruta (*path*) | Qué **recurso** quieres | `/posts/2`, `/api/books/1` |
| Query string | **Filtros/opciones** extra (`?clave=valor&clave2=valor2`) | `?country=chi&limit=5` |

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/src/routes/books-routes.ts`)

El comentario de tu ruta explica muy bien qué parámetros de query acepta:

```ts
/**
 * GET /api/books
 *
 * Texto (busca dentro del texto):  ?author=  ?country=  ?language=  ?title=  ?link=  ?imageLink=
 * Números (pages, year):           ?pages=  ?pages_gt=  ?pages_gte=  ?pages_lt=  ?pages_lte=
 *                                  ?year=1958  ?year_gt=  ?year_gte=  ?year_lt=  ?year_lte=
 * Límite de resultados:            ?limit=10
 *
 * Ejemplo: /api/books?country=chi&pages_gte=200&limit=5
 */
```

> 📌 **Todo lo que viene en la URL llega como TEXTO.** `?pages_gte=200` llega al
> servidor como `"200"` (string), no como `200`. Lo explicaste tú mismo en
> `api-rastro/api-rastro/respuestas.md` (Pregunta 2):
>
> > Porque todo lo que viene en la URL llega como **texto**: `req.query` es
> > `{ price: "380" }`, con el `"380"` como string. En MongoDB el precio está
> > guardado como número (`380`), y `"380"` y `380` son tipos distintos, así que el
> > filtro no coincide con ningún documento.

### Construir una URL con parámetros sin equivocarte

Copiado de `ProyectoReproductor/dist/server/index.js` (tu servidor llamando a Jamendo):

```js
const url = new URL('https://api.jamendo.com/v3.0/tracks/');
url.search = new URLSearchParams({ client_id: process.env.JAMENDO_CLIENT_ID, format: 'json', limit: '30', search: q, audioformat: 'mp32', include: 'musicinfo' }).toString();
```

```js
// ❌ Concatenar a mano: si "q" lleva espacios, & o tildes, se rompe la URL
const url = "https://api.ejemplo.com/tracks?search=" + q + "&limit=30";

// ✅ URLSearchParams codifica los caracteres raros por ti
const params = new URLSearchParams({ search: "rock & roll", limit: "30" });
params.toString();   // "search=rock+%26+roll&limit=30"
```

---

## 1.4 Anatomía de una PETICIÓN HTTP (request)

Una petición HTTP es, literalmente, **texto** que viaja por la red:

```http
POST /posts HTTP/1.1                         ← línea de inicio: MÉTODO + RUTA
Host: jsonplaceholder.typicode.com           ← cabeceras (headers)
Content-Type: application/json
Accept: application/json
                                              ← línea en blanco
{"title":"Aprendiendo JavaScript","body":"Mi primera petición POST","userId":1}
                                              ↑ cuerpo (body)
```

| Parte | Qué es | Ejemplo |
|-------|--------|---------|
| **Método** | El **verbo**: qué quieres hacer | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` (ver `02`) |
| **Ruta** | **Sobre qué** recurso | `/posts`, `/api/books/1` |
| **Headers** | Metadatos: tipo de datos, autenticación, cookies... | `Content-Type: application/json` |
| **Body** | Los **datos** que mandas (solo en POST/PUT/PATCH normalmente) | `{"title": "..."}` |

### Ejemplo REAL copiado (`Prueba-Api/prueba2.js`)

Así se ve esa misma petición escrita con `fetch`:

```js
const response = await fetch(
  "https://jsonplaceholder.typicode.com/posts",
  {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      title: "Aprendiendo JavaScript",
      body: "Mi primera petición POST",
      userId: 1
    })
  }
);
```

- `method` → el verbo.
- `headers` → las cabeceras.
- `body` → el cuerpo, **convertido a texto** con `JSON.stringify`.

---

## 1.5 Anatomía de una RESPUESTA HTTP (response)

```http
HTTP/1.1 201 Created                          ← línea de estado: CÓDIGO + texto
Content-Type: application/json; charset=utf-8 ← cabeceras
Content-Length: 85

{                                             ← cuerpo
  "title": "Aprendiendo JavaScript",
  "body": "Mi primera petición POST",
  "userId": 1,
  "id": 101
}
```

| Parte | Qué es |
|-------|--------|
| **Código de estado** | Número de 3 cifras que dice **cómo ha ido** (`200`, `201`, `404`, `500`... ver `02`) |
| **Headers** | Tipo de contenido, cookies que te pone el servidor (`Set-Cookie`), caché... |
| **Body** | Los datos que te devuelve (JSON, HTML, una imagen...) |

> Esa respuesta exacta (con `"id": 101`) es la que documentaste en
> `Prueba-Api/README.md`: *"El `id: 101` lo pone la API (los posts 1–100 ya
> existen). Ojo: JSONPlaceholder es una API de prueba, **no guarda realmente nada**."*

---

## 1.6 Headers (cabeceras) más importantes

| Header | Dirección | Para qué |
|--------|-----------|----------|
| `Content-Type` | petición y respuesta | **Qué formato** tiene el body (`application/json`, `text/html`...) |
| `Accept` | petición | Qué formato **quiero recibir** |
| `Authorization` | petición | Credenciales, p. ej. `Bearer <token>` (ver `06`) |
| `Cookie` | petición | Las cookies que el navegador manda solo |
| `Set-Cookie` | respuesta | El servidor le dice al navegador "guarda esta cookie" |
| `Cache-Control` | respuesta | Si se puede cachear o no |
| `Access-Control-Allow-Origin` | respuesta | **CORS**: qué webs pueden llamarme (ver `06`) |

### Ejemplo REAL copiado — leer un header (`ProyectoReproductor/public/app.js`)

```js
const contentType = response.headers.get('content-type') || '';
if (!contentType.includes('application/json')) {
    await response.text().catch(() => '');
    throw new Error(response.ok ? 'Respuesta no válida del servidor.' : `Error del servidor (${response.status})`);
}
```

> 📌 Antes de hacer `response.json()`, tu reproductor comprueba que el servidor
> **de verdad** ha devuelto JSON. Si te devuelve una página HTML de error,
> `response.json()` explotaría con un mensaje raro (`Unexpected token <`).

### Ejemplo REAL copiado — poner un header desde el servidor (`ProyectoReproductor/dist/server/index.js`)

```js
app.use((req, res, next) => { if (req.path.startsWith('/api/'))
    res.setHeader('Cache-Control', 'no-store'); next(); });
```

→ Todas las respuestas de `/api/...` llevan `Cache-Control: no-store` para que el
navegador **no guarde copias** de datos privados (tus favoritos, tu historial...).

---

## 1.7 JSON — el idioma de las APIs

**JSON** (*JavaScript Object Notation*) es **texto** con forma de objeto de JS.

```json
{
  "id": 1,
  "author": "Chinua Achebe",
  "country": "Nigeria",
  "imageLink": "images/things-fall-apart.jpg",
  "language": "English",
  "pages": 209,
  "title": "Things Fall Apart",
  "year": 1958
}
```
*(Copiado del primer libro de `API_Godines/api-express/api-books-claude/src/data/books.json`)*

### Reglas del JSON (más estrictas que un objeto JS)

```js
// ❌ Esto es un objeto JS válido, pero NO es JSON válido
{ title: 'Hola', pages: 10, }

// ✅ JSON válido
{ "title": "Hola", "pages": 10 }
```

- Las **claves** van **siempre** entre comillas **dobles**.
- Los textos, con comillas **dobles** (nunca simples).
- **Sin coma final**.
- Tipos permitidos: string, number, boolean, `null`, array, objeto.
  **No** hay `undefined`, ni funciones, ni fechas (las fechas viajan como texto
  `"2026-11-09T10:00:00.000Z"`).

### Convertir: `JSON.stringify` y `JSON.parse`

```js
const libro = { title: "Fairy tales", pages: 784 };

const texto = JSON.stringify(libro);   // objeto → texto
// '{"title":"Fairy tales","pages":784}'

const deVuelta = JSON.parse(texto);    // texto → objeto
deVuelta.pages;                        // 784
```

| Cuándo | Qué usas |
|--------|----------|
| **Enviar** datos en el body de un `fetch` | `JSON.stringify(objeto)` |
| **Leer** la respuesta de un `fetch` | `await response.json()` (hace el `JSON.parse` por ti) |

### Ejemplo REAL copiado — guardar y leer JSON (`ProyectoReproductor/dist/server/index.js`)

Tu servidor guarda la canción entera como texto JSON en SQLite y la convierte de
vuelta al leerla:

```js
run('INSERT OR REPLACE INTO favorites(user_id,track_id,track_json) VALUES(?,?,?)', [res.locals.user.id, String(req.params.trackId), JSON.stringify(track)]);
```

```js
res.json(rows.map(row => JSON.parse(row.track_json)));
```

### ¿Y si el JSON que llega está mal escrito?

Copiado de `api-rastro/api-rastro/src/index.ts` (el manejador de errores):

```ts
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  if (err instanceof SyntaxError) {
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }

  console.error(err);
  res.status(500).json({ error: "Error interno del servidor" });
});
```

→ Si el cliente manda `{"price": 380,}` (coma final), `express.json()` no puede
parsearlo y la API responde **400** con un mensaje claro.

---

## 1.8 Resumen visual

```
            ┌──────────────── PETICIÓN ────────────────┐
fetch(url, {│ method:  "POST"                          │
            │ headers: { "Content-Type": "application/json" }
            │ body:    JSON.stringify({ ... })         │
            └──────────────────────────────────────────┘
                              │
                              ▼
                        🖥️ SERVIDOR
                              │
                              ▼
            ┌──────────────── RESPUESTA ───────────────┐
response →  │ status:  201                             │
            │ headers: content-type: application/json  │
            │ body:    await response.json()           │
            └──────────────────────────────────────────┘
```

> 📌 **Checklist mental para cualquier API:** ¿qué **URL**? ¿qué **método**? ¿necesita
> **headers** (tipo de contenido, autenticación)? ¿lleva **body**? ¿qué **código**
> me devuelve y qué **JSON**?
