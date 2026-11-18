# 02 — MÉTODOS HTTP Y CÓDIGOS DE ESTADO

> El **método** es el **verbo** de la petición (qué quieres hacer) y el **código de
> estado** es la **respuesta corta** del servidor (cómo ha ido). Si dominas estas
> dos tablas, entiendes el 80% de cualquier API.

---

## 2.1 Los 5 métodos que vas a usar siempre

| Método | Qué hace | ¿Lleva body? | Ejemplo en tus proyectos |
|--------|----------|--------------|--------------------------|
| `GET` | **Leer** / obtener | ❌ No | `GET /api/books/1` |
| `POST` | **Crear** algo nuevo | ✅ Sí | `POST /api/books` |
| `PUT` | **Sustituir** un recurso **entero** | ✅ Sí (todos los campos) | `PUT /api/books/1` |
| `PATCH` | **Modificar** solo **algunos** campos | ✅ Sí (solo lo que cambia) | `PATCH /api/listings/:id` `{ "price": 340 }` |
| `DELETE` | **Borrar** | ❌ Normalmente no | `DELETE /api/library/favorites/:trackId` |

> 📌 `fetch(url)` sin opciones hace **siempre un GET**. Para cualquier otro método
> tienes que escribir `method: "POST"` (o el que sea).

---

## 2.2 CRUD ↔ métodos HTTP

**CRUD** son las 4 operaciones básicas sobre datos: **C**reate, **R**ead, **U**pdate,
**D**elete.

| CRUD | Método HTTP | Ruta típica | Respuesta típica |
|------|-------------|-------------|------------------|
| **C**reate | `POST` | `/api/books` | `201 Created` + el recurso creado |
| **R**ead (todos) | `GET` | `/api/books` | `200 OK` + array |
| **R**ead (uno) | `GET` | `/api/books/:id` | `200 OK` + objeto (o `404`) |
| **U**pdate (entero) | `PUT` | `/api/books/:id` | `200 OK` + recurso actualizado |
| **U**pdate (parcial) | `PATCH` | `/api/books/:id` | `200 OK` + recurso actualizado |
| **D**elete | `DELETE` | `/api/books/:id` | `204 No Content` o `200` + el borrado |

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/docs/prd-books.md`)

Tu documento de requisitos ya describía este mapeo:

```md
## Funcionalidades
1. Listar libros, con filtros por autor, género y precio máximo.
2. Ver un libro por su id.
3. Crear un libro (título, autor y precio obligatorios).
4. Editar y borrar un libro:
   - `PUT /api/books/:id`: sustituye el libro entero (todos los campos obligatorios).
   - `PATCH /api/books/:id`: modifica solo los campos enviados (al menos uno).
   - `DELETE /api/books/:id`: borra el libro y lo devuelve (200).
```

Y así quedan las rutas en `books-routes.ts` (solo las "firmas"):

```ts
router.get("/", async (req, res) => {        // R: listar
router.get("/:id", async (req, res) => {     // R: uno
router.post("/", async (req, res) => {       // C: crear
router.put("/:id", async (req, res) => {     // U: sustituir
router.patch("/:id", async (req, res) => {   // U: modificar
router.delete("/:id", async (req, res) => {  // D: borrar
```

---

## 2.3 PUT vs PATCH (la duda más típica)

Libro guardado:

```json
{ "id": 1, "title": "Things Fall Apart", "author": "Chinua Achebe", "pages": 209, "year": 1958 }
```

```http
PATCH /api/books/1
{ "pages": 210 }
```
→ Resultado: **solo cambia `pages`**, el resto se queda igual.

```http
PUT /api/books/1
{ "pages": 210 }
```
→ Resultado en tu API: **400**, porque PUT **sustituye el libro entero** y faltan
todos los demás campos obligatorios.

### Ejemplo REAL copiado (`books-routes.ts`)

```ts
/**
 * PATCH /api/books/:id
 *
 * Modifica solo los campos que se envían; el resto del libro no cambia.
 * Cada campo enviado se valida con las mismas reglas que en POST y PUT.
 * El id de la URL manda: si el body trae un id, se ignora. Los campos desconocidos también se ignoran.
 * 400 si el id no es un número entero, no se envía ningún campo editable o alguno no es válido.
 * 404 si no existe.
 *
 * Body (todos opcionales, al menos uno): { author, country, imageLink, language, link, title, pages, year }
 *
 * Ejemplo: PATCH /api/books/1  { "pages": 210 }
 */
```

Y la clave está en la validación con `partial`:

```ts
// En PATCH solo se valida lo que llega
const shouldCheck = (field: string) => !partial || body[field] !== undefined;
```

| | PUT | PATCH |
|-|-----|-------|
| Qué mandas | **Todo** el recurso | **Solo** lo que cambia |
| Campos que no mandas | Se pierden / error | Se quedan como estaban |
| Analogía | Cambiar la rueda **entera** | Cambiar **solo la válvula** |

> 📌 En el día a día se usa mucho más `PATCH`. Fíjate que en `api-rastro` solo hay
> `PATCH` (cambiar el precio, aceptar una oferta), nunca `PUT`.

---

## 2.4 Idempotencia y métodos "seguros"

**Idempotente** = hacer la **misma petición 1 vez o 10 veces** deja el servidor
**igual** que haciéndola una sola vez.

| Método | ¿Seguro? (no cambia datos) | ¿Idempotente? | Por qué |
|--------|----------------------------|---------------|---------|
| `GET` | ✅ (en teoría) | ✅ | Solo lee |
| `PUT` | ❌ | ✅ | Sustituir por lo mismo 10 veces = mismo resultado |
| `DELETE` | ❌ | ✅ | Borrado 1 vez o 10 → sigue borrado (las siguientes darán 404, pero el **estado** es el mismo) |
| `PATCH` | ❌ | ⚠️ Depende | `{ "price": 340 }` sí; "suma 1 a las visitas" no |
| `POST` | ❌ | ❌ | Cada POST crea **otro** recurso nuevo |

```js
// POST 3 veces → 3 libros distintos (ids 4, 5 y 6)
await fetch("/api/books", { method: "POST", body: ... });
await fetch("/api/books", { method: "POST", body: ... });
await fetch("/api/books", { method: "POST", body: ... });

// DELETE 3 veces → el libro 1 sigue borrado (1ª: 200, 2ª y 3ª: 404)
await fetch("/api/books/1", { method: "DELETE" });
```

> ⚠️ **Curiosidad de tu propia API:** en `api-rastro/.../listings.ts`, el
> `GET /api/listings/:id` **suma una visita** cada vez que se llama:
>
> ```ts
> const listing = await listingsCollection().findOneAndUpdate(
>   { _id: new ObjectId(id) },
>   { $inc: { views: 1 } },
>   { returnDocument: "after" },
> );
> ```
>
> Es un uso muy común (contador de visitas), pero técnicamente ese GET **no es
> "seguro"**: cambia datos. Por eso los navegadores o proxys que repiten GETs
> podrían inflar las visitas.

> 📌 ¿Por qué importa? Si se corta la conexión y **no sabes** si tu petición llegó,
> un método idempotente se puede **reintentar sin miedo**. Un POST no (podrías
> crear el pedido dos veces → el famoso "cobrado dos veces").

---

## 2.5 Códigos de estado: las 5 familias

| Familia | Significado | Frase para recordar |
|---------|-------------|---------------------|
| **1xx** | Informativo | "Espera, sigo..." (casi nunca lo verás) |
| **2xx** | ✅ Éxito | "Todo bien" |
| **3xx** | ↪️ Redirección | "Eso está en otro sitio" |
| **4xx** | ❌ Error **del cliente** | "**Tú** lo has pedido mal" |
| **5xx** | 💥 Error **del servidor** | "**Yo** (servidor) me he roto" |

---

## 2.6 Los códigos que de verdad usas

### 2xx — Éxito

| Código | Nombre | Cuándo | Ejemplo real |
|--------|--------|--------|--------------|
| `200` | OK | GET, PUT, PATCH, DELETE que devuelve algo | `res.json(book)` (Express pone 200 por defecto) |
| `201` | Created | Se ha **creado** algo con POST | `res.status(201).json(newBook)` |
| `204` | No Content | Ha ido bien pero **no hay body** | `DELETE` de `listings.ts` y de favoritos del reproductor |

### 3xx — Redirección

| Código | Nombre | Cuándo |
|--------|--------|--------|
| `301` | Moved Permanently | La URL ha cambiado para siempre |
| `302` / `307` | Found / Temporary Redirect | Redirección temporal (p. ej. tras un login) |
| `304` | Not Modified | "Lo que tienes en caché sigue valiendo" |

> 📌 `fetch` **sigue las redirecciones solo**; normalmente ni te enteras.

### 4xx — Error del cliente

| Código | Nombre | Cuándo | Ejemplo real |
|--------|--------|--------|--------------|
| `400` | Bad Request | Datos mal formados o inválidos | `"El id debe ser un número entero"` |
| `401` | Unauthorized | **No estás identificado** (falta login) | `"Inicia sesión para continuar."` |
| `403` | Forbidden | Estás identificado pero **no tienes permiso** | *(no aparece en tus proyectos)* |
| `404` | Not Found | El recurso **no existe** | `"No existe ningún libro con id ${id}"` |
| `409` | Conflict | Petición correcta pero **choca con el estado actual** | `"No se puede ofertar: el anuncio está SOLD"` |
| `422` | Unprocessable Entity | Alternativa a 400 para errores de validación | *(tú usas 400)* |
| `429` | Too Many Requests | Has hecho demasiadas peticiones (*rate limit*) | Las APIs públicas lo usan para limitarte (GitHub, sin token, responde `403` o `429` al pasar del límite por hora) |

### 5xx — Error del servidor

| Código | Nombre | Cuándo | Ejemplo real |
|--------|--------|--------|--------------|
| `500` | Internal Server Error | Excepción no controlada | `"Error interno del servidor"` |
| `502` | Bad Gateway | Tu servidor llamó a **otra** API y falló | Cuando Jamendo no responde bien |
| `503` | Service Unavailable | El servicio no está disponible/configurado | `"Falta configurar el acceso al catálogo de Jamendo."` |

### Ejemplo REAL copiado — un endpoint con muchos códigos (`ProyectoReproductor/dist/server/index.js`)

```js
app.post('/api/auth/register', (req, res) => { const email = String(req.body?.email ?? '').trim().toLowerCase(), password = String(req.body?.password ?? ''); if (!/^\S+@\S+\.\S+$/.test(email) || password.length < 10 || password.length > 128)
    return res.status(400).json({ error: 'Introduce un correo válido y una contraseña de 10 a 128 caracteres.' }); const id = randomBytes(16).toString('hex'); try {
    run('INSERT INTO users(id,email,password_hash) VALUES(?,?,?)', [id, email, hashPassword(password)]);
```
```js
    return res.status(409).json({ error: 'Ya existe una cuenta con ese correo.' });
} createSession(id, res); res.status(201).json({ id, email }); });
```

| Situación | Código |
|-----------|--------|
| Email mal escrito o contraseña corta | `400` |
| El email ya está registrado | `409` |
| Cuenta creada | `201` |

---

## 2.7 400 vs 404 vs 409 (cómo elegir)

Tu explicación en `api-rastro/api-rastro/respuestas.md` (Pregunta 5) es perfecta:

> Porque la petición está bien hecha: los ids tienen buen formato, existen y la
> cantidad es válida. El problema no es lo que envía el cliente, sino que choca con
> el **estado actual** del anuncio (ya está `SOLD`). `400` es para peticiones mal
> formadas; `409` es para peticiones correctas que entran en conflicto con el estado
> del recurso.

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/routes/offers.ts`)

```ts
if (typeof amount !== "number" || !Number.isFinite(amount) || amount <= 0) {
  res.status(400).json({ error: "amount tiene que ser un número mayor que 0" });
  return;
}
```
```ts
if (listing.status !== "AVAILABLE") {
  res.status(409).json({ error: `No se puede ofertar: el anuncio está ${listing.status}` });
  return;
}
```

### Árbol de decisión

```
¿La petición está bien formada (JSON válido, tipos correctos, id con buen formato)?
├── NO  → 400 Bad Request
└── SÍ
    ¿Sabe el servidor quién eres (si hace falta)?
    ├── NO → 401 Unauthorized
    └── SÍ
        ¿Tienes permiso para esto?
        ├── NO → 403 Forbidden
        └── SÍ
            ¿Existe el recurso?
            ├── NO → 404 Not Found
            └── SÍ
                ¿Choca con el estado actual (duplicado, ya vendido...)?
                ├── SÍ → 409 Conflict
                └── NO → 200 / 201 / 204 ✅
```

### Ojo con el orden: id inválido (400) antes que no existe (404)

Copiado de `books-routes.ts`:

```ts
router.get("/:id", async (req, res) => {
  const id = parseId(req.params.id);

  if (id === undefined) {
    res.status(400).json({ error: "El id debe ser un número entero" });
    return;
  }

  const book = await storage.findById(id);

  if (!book) {
    sendNotFound(res, id);
    return;
  }

  res.json(book);
});
```

| Petición | Respuesta |
|----------|-----------|
| `GET /api/books/abc` | `400` (el id ni siquiera es un número) |
| `GET /api/books/9999` | `404` (es un número, pero no existe) |
| `GET /api/books/1` | `200` + el libro |

---

## 2.8 ❌/✅ Errores comunes con códigos

```js
// ❌ Devolver 200 con un error dentro: el cliente cree que todo fue bien
res.json({ error: "No existe el libro" });

// ✅ El código ya dice que ha fallado
res.status(404).json({ error: "No existe el libro" });
```

```js
// ❌ 201 sin devolver nada útil
res.status(201).end();

// ✅ Devuelve el recurso creado (con su id nuevo) para que el cliente lo use
res.status(201).json(newBook);
```

```js
// ❌ 204 CON body (el 204 significa "sin contenido": el body se ignora)
res.status(204).json({ ok: true });

// ✅
res.status(204).end();
```

```js
// ❌ 500 con detalles internos (contraseñas, rutas de archivos, la query de la BD)
res.status(500).json({ error: err.stack });

// ✅ Mensaje genérico al cliente y el detalle solo en la consola del servidor
console.error(err);
res.status(500).json({ error: "Error interno del servidor" });
```

> 📌 La última es exactamente lo que hace tu `api-rastro/src/index.ts`: según tu
> `respuestas.md` (Pregunta 1), el manejador de errores *"responde un mensaje
> genérico sin detalles internos"*.
