# 07 — PROBAR APIs: NAVEGADOR, curl, POSTMAN Y THUNDER CLIENT

> Antes de escribir el frontend (o mientras programas el backend) necesitas **hablar
> con la API directamente** para ver qué devuelve. Hay 4 herramientas típicas: el
> **navegador**, **curl** (terminal), **Postman** y **Thunder Client** (extensión de
> VS Code).

---

## 7.1 ¿Qué herramienta uso?

| Herramienta | Ventaja | Limitación |
|-------------|---------|------------|
| **Barra del navegador** | Cero instalación | Solo **GET** |
| **DevTools → Network** | Ves las peticiones **reales** de tu web | Solo lo que tu web ya hace |
| **Consola del navegador** | Pegas un `fetch` y listo | Le afecta **CORS** |
| **curl** | Rápido, se copia/pega, va en READMEs | Hay que aprender las opciones |
| **Postman** | Visual, colecciones, variables, tests | App aparte (y pide cuenta) |
| **Thunder Client** | Como Postman pero **dentro de VS Code** | Menos funciones avanzadas |

> 📌 curl, Postman y Thunder Client **no son navegadores** → **no les afecta
> CORS**. Si algo funciona en Postman pero no en tu web, sospecha de CORS (ver `06`).

---

## 7.2 El navegador

### Barra de direcciones (solo GET)

Pega una URL y verás el JSON:

```
https://jsonplaceholder.typicode.com/posts/1
https://dog.ceo/api/breeds/image/random
http://localhost:3000/api/books?country=chi&limit=5
http://localhost:3000/api/health
```

### Consola (F12)

Tu `Prueba-Api/README.md` lo proponía como la opción más rápida:

```md
### Opción C — Consola del navegador (la más rápida)
Abre https://jsonplaceholder.typicode.com en Chrome, pulsa F12, pestaña **Consola** y pega directamente el código del archivo.
```

```js
// Pega esto en la consola:
await (await fetch("https://jsonplaceholder.typicode.com/posts/2")).json();
```

### Pestaña Network (Red) — la más útil para depurar tu frontend

1. F12 → pestaña **Network** → filtro **Fetch/XHR**.
2. Usa tu web (busca una canción, guarda un favorito...).
3. Haz clic en la petición y mira:

| Sub-pestaña | Qué ves |
|-------------|---------|
| **Headers** | URL, **método**, **código de estado**, cabeceras de ida y vuelta (`Cookie`, `Set-Cookie`, `Content-Type`) |
| **Payload** | El **body** que mandaste |
| **Preview / Response** | Lo que respondió el servidor |
| **Timing** | Cuánto tardó |

> 📌 Si un botón "no hace nada", mira Network: ¿salió la petición? ¿con qué código
> volvió? ¿qué `{ error }` trae? En el 90% de los casos ahí está la respuesta.

---

## 7.3 curl (terminal)

`curl` viene instalado en Windows 10/11, macOS y Linux.

### Opciones que necesitas

| Opción | Qué hace |
|--------|----------|
| `-i` | Muestra también el **código de estado y las cabeceras** de la respuesta |
| `-X METODO` | Cambia el método (`POST`, `PATCH`, `DELETE`...) |
| `-H "Clave: valor"` | Añade una cabecera |
| `-d '...'` | Manda un body (y si no pones `-X`, pasa a ser POST) |
| `-v` | Modo "verbose": enseña **todo** lo que se envía y recibe |
| `-b` / `-c` | Mandar / guardar cookies |

### Ejemplo REAL copiado (`mongo-api-libros/02b-express-mongodb/02b-express-mongodb/README.md`)

Tienes en un README una batería de pruebas perfecta, **con el código esperado
comentado** al lado:

```bash
curl -i http://localhost:3000/api/products                          # 200, array
curl -i "http://localhost:3000/api/products?category=muebles"       # 200, solo muebles
curl -i "http://localhost:3000/api/products?search=TEC"             # 200, el teclado
curl -i http://localhost:3000/api/products/<ID>                     # 200
curl -i http://localhost:3000/api/products/abc                      # 400: formato incorrecto
curl -i http://localhost:3000/api/products/66f000000000000000000000 # 404: no existe
curl -i -X POST http://localhost:3000/api/products -H "Content-Type: application/json" -d '{"name":"Monitor","price":149.99}'   # 201
curl -i -X POST http://localhost:3000/api/products -H "Content-Type: application/json" -d '{"price":1}'                       # 400
curl -i -X PATCH http://localhost:3000/api/products/<ID> -H "Content-Type: application/json" -d '{"price":34.5}'              # 200
curl -i -X PATCH http://localhost:3000/api/products/<ID> -H "Content-Type: application/json" -d '{}'                          # 400
curl -i -X DELETE http://localhost:3000/api/products/<ID>                                                                    # 204

curl -i -X POST http://localhost:3000/api/categories -H "Content-Type: application/json" -d '{"name":"cocina"}'   # 201
curl -i -X POST http://localhost:3000/api/categories -H "Content-Type: application/json" -d '{"name":"cocina"}'   # 409
```

> 📌 Esa es la forma correcta de probar una API: **un caso bueno y varios malos**
> por endpoint (formato incorrecto → 400, no existe → 404, duplicado → 409...).
> Fíjate en el 409 de la última línea: es el **mismo** POST repetido → POST **no**
> es idempotente.

### Qué ves con `-i`

```
HTTP/1.1 404 Not Found
X-Powered-By: Express
Content-Type: application/json; charset=utf-8

{"error":"No existe ningún libro con id 9999"}
```

### La misma batería para tu API de libros *(ejemplo propio)*

```bash
curl -i http://localhost:3000/api/books                                  # 200
curl -i "http://localhost:3000/api/books?country=chi&pages_gte=200&limit=5"  # 200
curl -i http://localhost:3000/api/books/1                                # 200
curl -i http://localhost:3000/api/books/abc                              # 400
curl -i http://localhost:3000/api/books/9999                             # 404
curl -i -X PATCH http://localhost:3000/api/books/1 -H "Content-Type: application/json" -d '{"pages":210}'  # 200
curl -i -X PATCH http://localhost:3000/api/books/1 -H "Content-Type: application/json" -d '{}'            # 400
curl -i -X DELETE http://localhost:3000/api/books/1                      # 200 + libro borrado
```

### ⚠️ curl en Windows PowerShell

```powershell
# En PowerShell 5.1, "curl" es un ALIAS de Invoke-WebRequest (¡otra herramienta!)
curl -i http://localhost:3000/api/books        # ❌ error raro de parámetros

# ✅ Usa curl.exe para el curl de verdad
curl.exe -i http://localhost:3000/api/books
```

Y las comillas del JSON dan guerra en PowerShell/CMD. Lo más cómodo:

```bash
# ✅ Usa Git Bash (las comillas simples funcionan como en los ejemplos)
# ✅ O guarda el body en un archivo y mándalo con @
curl.exe -i -X POST http://localhost:3000/api/books -H "Content-Type: application/json" -d "@libro.json"
```

### Probar sesiones con cookies *(ejemplo propio con tu reproductor)*

```bash
# 1. Login guardando la cookie en cookies.txt
curl -i -c cookies.txt -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"yo@mail.com","password":"una-contraseña-larga"}'

# 2. Peticiones privadas mandando la cookie
curl -i -b cookies.txt http://localhost:3000/api/auth/me
curl -i -b cookies.txt http://localhost:3000/api/library/favorites

# 3. Sin cookie → 401
curl -i http://localhost:3000/api/library/favorites
```

### Con token (JWT) *(ejemplo propio)*

```bash
curl -i http://localhost:3000/api/perfil -H "Authorization: Bearer eyJhbGciOi..."
```

---

## 7.4 Postman

App de escritorio (https://www.postman.com) para probar APIs con interfaz gráfica.

### Hacer una petición

1. **New → HTTP Request**.
2. Elige el **método** en el desplegable (`GET`, `POST`...).
3. Escribe la **URL**: `http://localhost:3000/api/books`.
4. Pestaña **Params**: añade query params en una tabla (los pone en la URL por ti).
5. Pestaña **Headers**: `Content-Type: application/json` (Postman lo pone solo si
   eliges body *raw → JSON*).
6. Pestaña **Body** → **raw** → **JSON** y escribe:
   ```json
   { "price": 340 }
   ```
7. **Send**. Abajo verás el **status** (`200 OK`), el **tiempo** y el **body**.

### Lo que lo hace potente

| Función | Para qué |
|---------|----------|
| **Collections** | Guardar todas las peticiones de una API ordenadas (una carpeta por recurso) |
| **Environments / variables** | `{{baseUrl}}/api/books` → cambias de `localhost` a producción en un clic |
| **Authorization** | Pestaña para poner `Bearer Token`, Basic Auth, API key... |
| **Cookies** | Guarda las cookies como un navegador → el login funciona solo |
| **Tests** | Scripts que comprueban la respuesta automáticamente |
| **Code** (`</>`) | Te genera el `fetch` o el `curl` equivalente |
| **Import** | Pegas un `curl` y lo convierte en petición |

### Un test sencillo en Postman *(ejemplo propio)*

```js
pm.test("Devuelve 404 si no existe", () => {
  pm.response.to.have.status(404);
});

pm.test("Trae un mensaje de error", () => {
  const body = pm.response.json();
  pm.expect(body.error).to.include("No existe");
});
```

---

## 7.5 Thunder Client (dentro de VS Code)

Extensión de VS Code (busca **"Thunder Client"** en Extensiones). Aparece un icono
de rayo ⚡ en la barra lateral.

1. **New Request**.
2. Método + URL igual que en Postman.
3. Pestañas **Query**, **Headers**, **Auth**, **Body** (elige **JSON**).
4. **Send** → respuesta a la derecha con status, tiempo y JSON.
5. Puedes guardar las peticiones en **Collections** y usar **Env** para variables.

> 📌 Ventaja: no sales del editor. Tienes el código del servidor a la izquierda y la
> petición a la derecha.

### Alternativa: archivos `.http` (extensión REST Client) *(ejemplo propio)*

Un archivo de texto que vive **en el repo** junto al código:

```http
### Listar libros
GET http://localhost:3000/api/books?limit=3

### Crear libro con errores (esperado: 400)
POST http://localhost:3000/api/books
Content-Type: application/json

{ "title": "" }

### Bajar precio de un anuncio
PATCH http://localhost:3000/api/listings/66f0a1b2c3d4e5f601234567
Content-Type: application/json

{ "price": 340 }
```

Encima de cada `###` aparece **"Send Request"**.

---

## 7.6 Comprobar que la API está viva: `/api/health`

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/index.ts`)

```ts
app.get("/api/health", async (_req, res) => {
  await getDb().command({ ping: 1 });
  res.json({ status: "ok", db: getDb().databaseName });
});
```

Lo primero que pruebas al arrancar:

```bash
curl -i http://localhost:3000/api/health
# 200 {"status":"ok","db":"rastro"}
```

Y tu `respuestas.md` (Pregunta 1) documenta qué pasa si la base de datos está caída:

> Con MongoDB parado, `GET /api/health` tarda unos 5 segundos (el
> `serverSelectionTimeoutMS`) y devuelve **`500`** con
> `{ "error": "Error interno del servidor" }`.

### Ver las peticiones que llegan al servidor

Copiado de `api-rastro/api-rastro/src/index.ts` — un *logger* mínimo:

```ts
app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

→ En la terminal del servidor verás `GET /api/listings?city=Madrid`,
`PATCH /api/offers/...`, etc. Muy útil para saber si tu petición **llegó**.

---

## 7.7 Cómo leer una respuesta cuando algo falla

| Ves... | Probable causa | Mira... |
|--------|----------------|---------|
| `TypeError: Failed to fetch` | Servidor apagado, URL/puerto mal, o **CORS** | ¿Está arrancado? ¿Consola: error de CORS? |
| `400` | Body o parámetros mal | El `{ error }` / `{ errors }` de la respuesta |
| `400` "faltan campos" pero los mandaste | Falta `Content-Type: application/json` | Headers de la petición |
| `401` | No hay sesión/token | Cookie o `Authorization` |
| `404` con HTML `Cannot GET ...` | La **ruta** no existe (no el recurso) | Método y URL exactos |
| `404` con JSON `{ error }` | La ruta existe, el **recurso** no | El id |
| `409` | Conflicto de estado (duplicado, ya vendido) | El estado actual del recurso |
| `500` | Error en el servidor | **La terminal del servidor**, no el navegador |
| `Unexpected token < in JSON` | Te ha llegado **HTML** en vez de JSON | Network → Response |
| `Unexpected end of JSON input` | Body vacío (p. ej. **204**) y haces `.json()` | Comprueba el status antes |

---

## 7.8 Tests automáticos (lo que hacen los bikrams)

Los ejercicios `Async-hector` y `Async-Hector-2` traen un `js/index.spec.js` que
**comprueba tus funciones** automáticamente. Del `readme.md` de `Async-hector`:

```md
Estas pruebas cubren un espectro de dificultad incremental, dotada de tests que cubrirán la correcta resolución para ayudarte a encontrar posibles errores en tu código.
```

```md
(Abre index.html para ver el resultado de los tests)
```

→ El siguiente nivel es escribir **tus propios tests** de API (con herramientas como
*Vitest* + *Supertest* en el servidor), que hacen lo mismo que tu batería de `curl`
pero automáticamente cada vez que cambias código. Tu CI de
`api-rastro/api-rastro/.github/workflows/ci.yml` ya ejecuta `htmlhint`, `lint` y
`typecheck` en cada PR; añadir un paso `npm test` con esos tests de API sería el
siguiente escalón.
