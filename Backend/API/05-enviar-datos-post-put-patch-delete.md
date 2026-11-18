# 05 — ENVIAR DATOS CON `fetch`: POST, PUT, PATCH Y DELETE

> Para todo lo que **no** sea leer (GET), `fetch` necesita un **segundo argumento**:
> un objeto de opciones con el `method`, los `headers` y el `body`.

---

## 5.1 La plantilla

```js
const response = await fetch(url, {
  method: "POST",                                   // o "PUT", "PATCH", "DELETE"
  headers: { "Content-Type": "application/json" },  // "te mando JSON"
  body: JSON.stringify(datos),                      // objeto → texto JSON
});
```

| Opción | Obligatoria | Para qué |
|--------|-------------|----------|
| `method` | Sí (si no, es GET) | El verbo |
| `headers` | Sí cuando mandas JSON | Que el servidor sepa **cómo leer** el body |
| `body` | En POST/PUT/PATCH | Los datos, **como texto** |
| `credentials` | A veces | Mandar cookies a **otro** dominio (ver `06`) |

---

## 5.2 POST — crear

### Ejemplo REAL copiado (`Prueba-Api/prueba2.js`)

```js
// PASO 5: Ejecuta crearPublicacion() y localiza el objeto devuelto.
// Es un POST: la "página" del recurso lleva el body (title, body, userId).
// JSONPlaceholder responde 201 Created y devuelve el objeto creado
// con un id asignado (101). Ojo: es una API de prueba, NO guarda nada.

async function crearPublicacion() {
  try {
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

    if (!response.ok) {
      throw new Error(`Error HTTP: ${response.status}`);
    }

    const data = await response.json();
    console.log("Publicación creada:", data);
  } catch (error) {
    console.error("No se pudo crear la publicación:", error.message);
  }
}

crearPublicacion();
```

Respuesta (de tu `Prueba-Api/README.md`):

```json
{
  "title": "Aprendiendo JavaScript",
  "body": "Mi primera petición POST",
  "userId": 1,
  "id": 101
}
```

> 📌 Ojo: aquí `body` aparece **dos veces** con significados distintos: la
> **opción** `body` de `fetch` (el cuerpo de la petición) y el **campo** `body` del
> post (el texto de la publicación). No te líes.

### Ejemplo REAL copiado — POST desde un formulario (`ProyectoReproductor/public/login.js`)

```js
for (const [id, endpoint] of [['login-form', 'login'], ['register-form', 'register']]) {
    $(`#${id}`).addEventListener('submit', async (event) => {
        event.preventDefault();
        const form = event.currentTarget;
        const data = new FormData(form);
        feedback(form);
        try {
            await request(`/api/auth/${endpoint}`, {
                method: 'POST',
                body: JSON.stringify({ email: data.get('email'), password: data.get('password') })
            });
            window.location.href = 'app.html';
        }
        catch (error) {
            feedback(form, error.message, true);
        }
    });
}
```

**Lo que hay que aprender de aquí:**
1. `event.preventDefault()` → evita que el formulario recargue la página.
2. `new FormData(form)` + `data.get('email')` → lee los campos del formulario.
3. Se construye un objeto y se manda con `JSON.stringify`.
4. Si va bien → redirige. Si falla → muestra el mensaje de error del servidor.

> 📌 El mismo bucle sirve para **dos** endpoints (`login` y `register`) porque los
> dos reciben el mismo body. Buen ejemplo de no repetir código.

---

## 5.3 PUT — sustituir entero

*(Ejemplo propio contra tu API de libros `API_Godines/.../api-books-claude`)*

```js
async function sustituirLibro(id, libroCompleto) {
  const response = await fetch(`http://localhost:3000/api/books/${id}`, {
    method: "PUT",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(libroCompleto),
  });

  if (!response.ok) {
    const body = await response.json().catch(() => ({}));
    throw new Error(body.error || JSON.stringify(body.errors) || `Error ${response.status}`);
  }

  return response.json();
}

await sustituirLibro(1, {
  author: "Chinua Achebe",
  country: "Nigeria",
  imageLink: "images/things-fall-apart.jpg",
  language: "English",
  link: "https://en.wikipedia.org/wiki/Things_Fall_Apart",
  title: "Things Fall Apart",
  pages: 210,
  year: 1958,
});
```

> 📌 Hay que mandar **todos** los campos. Recuerda la regla del PRD:
> *"`PUT /api/books/:id`: sustituye el libro entero (todos los campos obligatorios)."*

---

## 5.4 PATCH — modificar solo algunos campos

*(Ejemplo propio)*

```js
await fetch("http://localhost:3000/api/books/1", {
  method: "PATCH",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ pages: 210 }),
});
```

### Ejemplo con tu API de rastro: cambiar precio y aceptar oferta

Lo que esperan las rutas (copiado de `api-rastro/.../listings.ts` y `offers.ts`):

```ts
const price = req.body?.price;
if (!isValidPrice(price)) {
  res.status(400).json({ error: "price es obligatorio y tiene que ser un número mayor o igual que 0" });
  return;
}
```
```ts
if (req.body?.status !== "ACCEPTED") {
  res.status(400).json({ error: 'status tiene que ser "ACCEPTED"' });
  return;
}
```

Y cómo las llamarías desde el cliente *(ejemplo propio)*:

```js
// Bajar el precio de un anuncio
await fetch(`http://localhost:3000/api/listings/${listingId}`, {
  method: "PATCH",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ price: 340 }),
});

// Aceptar una oferta
await fetch(`http://localhost:3000/api/offers/${offerId}`, {
  method: "PATCH",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ status: "ACCEPTED" }),
});
```

### ❌/✅ El tipo de dato importa

```js
// ❌ El precio como texto → tu API responde 400
body: JSON.stringify({ price: "340" })

// ❌ Viene de un <input>: .value SIEMPRE es string
body: JSON.stringify({ price: input.value })

// ✅ Convertir antes de enviar
body: JSON.stringify({ price: Number(input.value) })
```

> 📌 Pregunta 6 de tu `respuestas.md`: la API comprueba *"que `price`/`amount` sean
> números (y no el texto `"75"`)"*. El body JSON **sí** conserva tipos (a
> diferencia de la query string), así que manda números como números.

---

## 5.5 DELETE — borrar

### Ejemplo REAL copiado (`ProyectoReproductor/public/app.js`)

```js
async function toggleFavorite(track, button) {
    try {
        if (favorites.has(track.id)) {
            await request(`/api/library/favorites/${track.id}`, { method: 'DELETE' });
            favorites.delete(track.id);
            button.classList.remove('is-saved');
            button.setAttribute('aria-label', `Guardar ${track.name}`);
            if (activeView === 'favorites') {
                currentTracks = currentTracks.filter(t => t.id !== track.id);
                renderFilteredResults();
            }
        }
        else {
            await request(`/api/library/favorites/${track.id}`, { method: 'POST', body: JSON.stringify({ track }) });
            favorites.add(track.id);
            button.classList.add('is-saved');
            button.setAttribute('aria-label', `Quitar ${track.name}`);
        }
    }
    catch (error) {
        setStatus(error.message, true);
    }
}
```

**Fíjate:**
- La **misma URL** (`/api/library/favorites/:id`) con **dos métodos** distintos:
  `POST` guarda, `DELETE` quita. Eso es REST.
- El DELETE **no lleva body**: el id va en la URL.
- **Primero** se espera a que el servidor confirme y **después** se cambia la
  interfaz (`favorites.delete`, `classList.remove`). Si el servidor falla, la UI no
  miente.

### El servidor responde 204 (copiado de `ProyectoReproductor/dist/server/index.js`)

```js
app.delete('/api/library/favorites/:trackId', session, (req, res) => { run('DELETE FROM favorites WHERE user_id=? AND track_id=?', [res.locals.user.id, String(req.params.trackId)]); res.status(204).end(); });
```

```js
// ❌ Con un 204 no hay body: esto lanza "Unexpected end of JSON input"
const data = await response.json();

// ✅ Lo que hace tu función request
return response.status === 204 ? undefined : response.json();
```

> 📌 Tu API de libros, en cambio, **devuelve el libro borrado** con `200`
> (*"`DELETE /api/books/:id`: borra el libro y lo devuelve (200)"*). Las dos
> opciones son válidas; lo importante es **documentarlo** y que el cliente lo sepa.

---

## 5.6 POST sin body

### Ejemplo REAL copiado (`ProyectoReproductor/public/app.js`)

```js
$('#logout-button').addEventListener('click', async () => {
    await request('/api/auth/logout', { method: 'POST' });
    audio.pause();
    window.location.href = 'index.html';
});
```

→ Cerrar sesión no necesita datos: el servidor sabe quién eres por la **cookie**
(ver `06`). Se usa POST (y no GET) porque **cambia** algo en el servidor (borra la
sesión).

> ⚠️ Nunca hagas acciones que cambian datos con **GET** (`/logout`,
> `/delete?id=1`...). Los navegadores precargan enlaces, los buscadores los siguen...
> y podrían ejecutar la acción sin querer.

---

## 5.7 Otros formatos de body

| Qué mandas | `body` | `Content-Type` |
|------------|--------|----------------|
| JSON (lo normal) | `JSON.stringify(obj)` | `application/json` (lo pones tú) |
| Formulario con archivos | `new FormData(form)` | **No lo pongas**: el navegador lo pone solo (`multipart/form-data` con su *boundary*) |
| Formulario clásico | `new URLSearchParams({ a: "1" })` | `application/x-www-form-urlencoded` (automático) |

```js
// ❌ Mandar un objeto sin stringify: llega "[object Object]"
fetch(url, { method: "POST", body: { title: "Hola" } });

// ❌ stringify pero SIN Content-Type: Express no lo parsea y req.body está vacío
fetch(url, { method: "POST", body: JSON.stringify({ title: "Hola" }) });

// ✅
fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ title: "Hola" }),
});
```

> 📌 La segunda ❌ es un error **muy** típico: `express.json()` solo lee el body si
> el `Content-Type` es `application/json`. Si no, `req.body` llega `undefined` y tu
> API responde `400` diciendo que faltan todos los campos.

---

## 5.8 Un pequeño "cliente" de tu API de libros

*(Ejemplo propio: junta todo lo de este tema en un objeto reutilizable)*

```js
const API = "http://localhost:3000/api/books";

async function request(url, options = {}) {
  const response = await fetch(url, {
    ...options,
    headers: { "Content-Type": "application/json", ...(options.headers || {}) },
  });

  if (!response.ok) {
    const body = await response.json().catch(() => ({}));
    const message = body.error || (body.errors && Object.values(body.errors).join(", "));
    throw new Error(message || `Error HTTP ${response.status}`);
  }

  return response.status === 204 ? undefined : response.json();
}

const booksApi = {
  list: (filters = {}) => request(`${API}?${new URLSearchParams(filters)}`),
  get: (id) => request(`${API}/${id}`),
  create: (book) => request(API, { method: "POST", body: JSON.stringify(book) }),
  replace: (id, book) => request(`${API}/${id}`, { method: "PUT", body: JSON.stringify(book) }),
  update: (id, changes) => request(`${API}/${id}`, { method: "PATCH", body: JSON.stringify(changes) }),
  remove: (id) => request(`${API}/${id}`, { method: "DELETE" }),
};

// Uso
const chinos = await booksApi.list({ country: "chi", limit: 5 });
const nuevo = await booksApi.create({ title: "Mi libro", /* ...resto de campos */ });
await booksApi.update(nuevo.id, { pages: 300 });
await booksApi.remove(nuevo.id);
```

---

## 5.9 Resumen: método → qué poner en `fetch`

| Acción | `method` | `headers` | `body` | Respuesta típica |
|--------|----------|-----------|--------|------------------|
| Leer | *(nada)* | — | — | `200` + JSON |
| Crear | `"POST"` | `Content-Type: application/json` | `JSON.stringify(nuevo)` | `201` + creado |
| Sustituir | `"PUT"` | `Content-Type: application/json` | `JSON.stringify(completo)` | `200` + actualizado |
| Modificar | `"PATCH"` | `Content-Type: application/json` | `JSON.stringify(cambios)` | `200` + actualizado |
| Borrar | `"DELETE"` | — | — | `204` (sin body) o `200` + borrado |
