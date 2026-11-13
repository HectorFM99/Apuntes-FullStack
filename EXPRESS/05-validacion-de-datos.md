# 05 — VALIDACIÓN DE DATOS

> **Nunca te fíes de lo que llega del cliente.** Cualquiera puede mandar a tu API un
> `price: "gratis"`, un `id: "hola"` o un body vacío. Validar es comprobar **tipo**,
> **formato** y **reglas de negocio** antes de tocar los datos, y si algo falla, responder
> **400** con un mensaje claro.

---

## 5.1 Qué hay que validar

| Qué | Pregunta | Error típico |
|-----|----------|--------------|
| **Id de la URL** | ¿Tiene el formato correcto? | 400 `"El id no tiene un formato válido"` |
| **Query** | ¿Los números son números? | 400 `"maxPrice tiene que ser un número"` |
| **Body: presencia** | ¿Están los campos obligatorios? | 400 |
| **Body: tipo** | ¿`price` es `number`? ¿`title` es `string`? | 400 |
| **Body: formato/rango** | ¿Texto no vacío? ¿Precio ≥ 0? ¿Email válido? | 400 |
| **Existencia** | ¿Existe el recurso / el usuario referenciado? | 404 (o 400 si va en el body) |
| **Estado / reglas** | ¿Se puede hacer esto **ahora**? | 409 |

### El orden recomendado dentro de una ruta

```
1. Validar el id de la URL         → 400
2. Validar el body                 → 400
3. Buscar en la BD                 → 404
4. Comprobar reglas de negocio     → 409
5. Hacer la operación              → 200 / 201 / 204
```

> 📌 Valida **lo barato primero** (formato) y **lo caro después** (consultas a la BD).
> No tiene sentido ir a MongoDB con un id que ni siquiera tiene el formato correcto.

---

## 5.2 Validar el id

### Ids numéricos (`API_Godines/api-express/api-books-claude/src/routes/books-routes.ts`)

```ts
// Solo acepta dígitos: rechaza "0x1", "1.0", "1e2", " 1"...
function parseId(raw: string): number | undefined {
  return /^\d+$/.test(raw) ? Number(raw) : undefined;
}
```

```ts
router.delete("/:id", async (req, res) => {
  const id = parseId(req.params.id);

  if (id === undefined) {
    res.status(400).json({ error: "El id debe ser un número entero" });
    return;
  }

  const deletedBook = await storage.remove(id);

  if (!deletedBook) {
    sendNotFound(res, id);
    return;
  }

  res.json(deletedBook);
});
```

> 📌 `parseId` **valida y convierte** a la vez: devuelve el número o `undefined`. Es un
> patrón muy cómodo: "parsear, no solo validar".

### Ids de MongoDB (`mongo-api-libros/.../src/routes/products.ts`)

```ts
router.get("/:id", async (req, res) => {
  if (!ObjectId.isValid(req.params.id)) {
    res.status(400).json({ error: "El ID no tiene un formato válido" });
    return;
  }

  const product = await productsCollection().findOne({ _id: new ObjectId(req.params.id) });

  if (!product) {
    res.status(404).json({ error: "Producto no encontrado" });
    return;
  }

  res.json(product);
});
```

```ts
// ❌ Sin validar: new ObjectId("hola") LANZA un error → 500
const product = await productsCollection().findOne({ _id: new ObjectId(req.params.id) });

// ✅ Validar antes → 400 claro
if (!ObjectId.isValid(req.params.id)) { ... }
```

---

## 5.3 Validar el body: un error cada vez

El estilo más sencillo: comprobar campo a campo y **devolver el primer error**.

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
function isNonEmptyString(value: unknown): value is string {
  return typeof value === "string" && value.trim() !== "";
}

function isValidPrice(value: unknown): value is number {
  return typeof value === "number" && Number.isFinite(value) && value >= 0;
}

// ...

router.post("/", async (req, res) => {
  const body = req.body ?? {};

  for (const field of ["title", "description", "category", "city"] as const) {
    if (!isNonEmptyString(body[field])) {
      res.status(400).json({ error: `${field} es obligatorio y tiene que ser un texto no vacío` });
      return;
    }
  }

  if (!isValidPrice(body.price)) {
    res.status(400).json({ error: "price es obligatorio y tiene que ser un número mayor o igual que 0" });
    return;
  }

  if (
    body.attributes !== undefined &&
    (typeof body.attributes !== "object" || body.attributes === null || Array.isArray(body.attributes))
  ) {
    res.status(400).json({ error: "attributes tiene que ser un objeto" });
    return;
  }

  if (typeof body.sellerId !== "string" || !ObjectId.isValid(body.sellerId)) {
    res.status(400).json({ error: "sellerId no tiene un formato válido" });
    return;
  }

  const seller = await usersCollection().findOne({ _id: new ObjectId(body.sellerId) });
  if (!seller) {
    res.status(400).json({ error: "No existe ningún usuario con ese sellerId" });
    return;
  }

  // ... crear el anuncio
});
```

**Cosas a aprender de aquí:**

1. **Bucle sobre campos** con la misma regla → no repites 4 `if` iguales.
2. `value.trim() !== ""` → `"   "` (solo espacios) **no** cuenta como texto.
3. `Number.isFinite(value)` → rechaza `NaN` e `Infinity`.
4. Comprobar que algo es un **objeto de verdad**: `typeof x === "object"` es `true` también
   para `null` y para arrays, por eso hay que descartarlos.
5. El `sellerId` existe en la BD → si no, **400** (no 404), porque el fallo está en un
   **dato del body**, no en la URL.

### `value is string` — type guards de TypeScript

```ts
function isNonEmptyString(value: unknown): value is string { ... }
//                                         ▲
//             "si devuelvo true, TypeScript puede tratar value como string"
```

```ts
const title: unknown = req.body.title;

if (isNonEmptyString(title)) {
  title.toUpperCase();   // ✅ TypeScript ya sabe que es string
}
```

---

## 5.4 Validar el body: devolver una función de validación reutilizable

Cuando POST y PATCH validan casi lo mismo, se saca a una función.

### Ejemplo REAL copiado (`mongo-api-libros/.../src/routes/products.ts`)

```ts
// Devuelve un mensaje de error, o null si los datos son válidos.
// partial = true: los campos pueden faltar (para PATCH).
export function validateProduct(
  body: { name?: unknown; price?: unknown; category?: unknown },
  partial: boolean
): string | null {
  const { name, price, category } = body;

  if (name !== undefined || !partial) {
    if (typeof name !== "string" || name.trim() === "") {
      return "El nombre debe ser un texto no vacío";
    }
  }

  if (price !== undefined || !partial) {
    if (typeof price !== "number" || !Number.isFinite(price) || price < 0) {
      return "El precio debe ser un número igual o mayor que cero";
    }
  }

  // Ejercicio 5: la categoría es opcional, pero si llega tiene que ser un texto
  if (category !== undefined) {
    if (typeof category !== "string" || category.trim() === "") {
      return "La categoría debe ser un texto no vacío";
    }
  }

  return null;
}
```

**La clave está en `partial`:**

| | `name` no llega | `name` llega mal | `name` llega bien |
|-|-----------------|------------------|-------------------|
| **POST** (`partial = false`) | ❌ error | ❌ error | ✅ |
| **PATCH** (`partial = true`) | ✅ (no se cambia) | ❌ error | ✅ |

`name !== undefined || !partial` se lee: "valida `name` **si ha llegado** o **si es
obligatorio**".

**Uso en las rutas:**

```ts
router.post("/", async (req, res) => {
  const body = req.body ?? {};
  const error = validateProduct(body, false);   // todo obligatorio

  if (error) {
    res.status(400).json({ error });
    return;
  }
  // ...
});

router.patch("/:id", async (req, res) => {
  // ...
  const body = req.body ?? {};

  if (body.name === undefined && body.price === undefined && body.category === undefined) {
    res.status(400).json({ error: "Debes enviar al menos un campo para modificar" });
    return;
  }

  const error = validateProduct(body, true);    // solo lo que llega
  // ...
});
```

> 📌 En PATCH, además, se comprueba que llegue **al menos un campo**. Un `PATCH` con `{}`
> no tiene sentido.

---

## 5.5 Validar el body: todos los errores a la vez

Más cómodo para el usuario de un formulario: le dices **todo** lo que está mal de una vez.

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/src/routes/books-routes.ts`)

```ts
// Campos de texto: obligatorios al crear o sustituir un libro, opcionales al modificarlo (PATCH)
const TEXT_FIELDS = ["author", "country", "imageLink", "language", "link", "title"] as const;

// Todos los campos que se pueden editar (el id nunca)
const EDITABLE_FIELDS = [...TEXT_FIELDS, "pages", "year"] as const;

/**
 * Devuelve un error por cada campo no válido (vacío si todo está bien).
 *
 * @param body - Body de la petición, sin validar.
 * @param options.partial - Si es true (PATCH), los campos ausentes no se validan;
 *   si es false (POST y PUT), todos son obligatorios.
 * @returns Un objeto campo -> mensaje de error.
 */
function validateBook(body: Query, { partial = false } = {}): Record<string, string> {
  const errors: Record<string, string> = {};

  // En PATCH solo se valida lo que llega
  const shouldCheck = (field: string) => !partial || body[field] !== undefined;

  for (const field of TEXT_FIELDS) {
    if (!shouldCheck(field)) continue;

    const value = body[field];
    if (typeof value !== "string" || value.trim() === "") {
      errors[field] = "Debe ser un texto no vacío";
    }
  }

  // isSafeInteger rechaza valores enormes como 1e300 (isInteger los aceptaría)
  if (shouldCheck("pages") && (!Number.isSafeInteger(body.pages) || (body.pages as number) <= 0)) {
    errors.pages = "Debe ser un número entero mayor que 0";
  }

  // Los años negativos están permitidos (a. C.), los futuros no
  if (
    shouldCheck("year") &&
    (!Number.isSafeInteger(body.year) || (body.year as number) > new Date().getFullYear())
  ) {
    errors.year = "Debe ser un número entero no posterior al año actual";
  }

  return errors;
}
```

**Uso:**

```ts
  const errors = validateBook(body);

  if (Object.keys(errors).length > 0) {      // ¿hay algún error?
    res.status(400).json({ errors });
    return;
  }
```

**Respuesta ejemplo** a `POST /api/books` con `{ "title": "", "pages": -3 }`:

```json
{
  "errors": {
    "author": "Debe ser un texto no vacío",
    "country": "Debe ser un texto no vacío",
    "imageLink": "Debe ser un texto no vacío",
    "language": "Debe ser un texto no vacío",
    "link": "Debe ser un texto no vacío",
    "title": "Debe ser un texto no vacío",
    "pages": "Debe ser un número entero mayor que 0",
    "year": "Debe ser un número entero no posterior al año actual"
  }
}
```

| Estilo | Proyecto | Ventaja |
|--------|----------|---------|
| Primer error (`string \| null`) | `02b-express-mongodb`, `api-rastro` | Más simple de escribir |
| Todos los errores (`Record<campo, mensaje>`) | `api-books-claude` | Mejor experiencia en formularios |

---

## 5.6 Después de validar: limpiar y construir el objeto

Validar no es suficiente: **no guardes `req.body` tal cual**. Construye tú el objeto con
**solo** los campos permitidos (así nadie te cuela un `id`, un `isAdmin: true`...).

### Ejemplo REAL copiado (`api-books-claude/src/routes/books-routes.ts`)

```ts
// Construye los datos del libro (sin id) a partir de un body ya validado
function toBookData(body: Query): BookData {
  return {
    // Se guarda recortado, igual que se valida
    author: (body.author as string).trim(),
    country: (body.country as string).trim(),
    imageLink: (body.imageLink as string).trim(),
    language: (body.language as string).trim(),
    link: (body.link as string).trim(),
    title: (body.title as string).trim(),
    pages: body.pages as number,
    year: body.year as number,
  };
}
```

```ts
// ❌ Se guarda TODO lo que mande el cliente (incluido un "id" falso o campos basura)
await collection.insertOne(req.body);

// ✅ Solo los campos que tú decides, ya limpios
await storage.create(toBookData(body));
```

### Para PATCH: solo los campos que llegan (`mongo-api-libros/.../src/routes/products.ts`)

```ts
  // Solo los campos que han llegado
  const changes: Partial<Product> = {};
  if (body.name !== undefined) changes.name = body.name.trim();
  if (body.price !== undefined) changes.price = body.price;
  if (body.category !== undefined) changes.category = body.category.trim();
```

### Valores por defecto (`mongo-api-libros/.../src/routes/products.ts`)

```ts
  const newProduct: Product = {
    name: body.name.trim(),
    price: body.price,
    // Ejercicio 5: si no llega categoría, "sin-categoria"
    category: body.category?.trim() ?? "sin-categoria"
  };
```

### Campos que pone el servidor, no el cliente (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
  const newListing: Listing = {
    title: body.title,
    description: body.description,
    price: body.price,
    category: body.category,
    status: "AVAILABLE",          // ← siempre empieza disponible
    city: body.city,
    publishedAt: new Date(),      // ← la fecha la pone el servidor
    views: 0,                     // ← empieza en 0
    seller: { id: seller._id, name: seller.name },
    photos: [],
    attributes: body.attributes ?? {},
  };
```

> 📌 `status`, `publishedAt` y `views` **no** se leen del body: si el cliente pudiera
> mandarlos, podría publicar un anuncio con 1 millón de visitas.

---

## 5.7 Reglas de negocio (409 Conflict)

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/routes/offers.ts`, PATCH)

```ts
  if (req.body?.status !== "ACCEPTED") {
    res.status(400).json({ error: 'status tiene que ser "ACCEPTED"' });
    return;
  }

  const offerId = new ObjectId(id);

  const offer = await offersCollection().findOne({ _id: offerId });
  if (!offer) {
    res.status(404).json({ error: "No existe ninguna oferta con ese id" });
    return;
  }

  if (offer.status !== "PENDING") {
    res.status(409).json({ error: `No se puede aceptar: la oferta está ${offer.status}` });
    return;
  }

  const listing = await listingsCollection().findOne({ _id: offer.listing.id });
  if (!listing || listing.status !== "AVAILABLE") {
    res.status(409).json({ error: "No se puede aceptar: el anuncio ya no está disponible" });
    return;
  }
```

Aquí se ve el orden completo: **400** (body) → **404** (no existe) → **409** (estado).

### Unicidad con índice de la BD (`mongo-api-libros/.../src/routes/categories.ts`)

```ts
// ¿Es el error de "clave duplicada" del índice único?
function isDuplicateKey(err: unknown): boolean {
  return err instanceof MongoServerError && err.code === 11000;
}
```

En vez de "buscar si ya existe y luego insertar" (que puede fallar si dos peticiones
llegan a la vez), se deja que **MongoDB** lo garantice con un índice único, y se
traduce su error a un **409**.

---

## 5.8 Validación en el reproductor (estilo compacto)

### Ejemplo REAL copiado (`ProyectoReproductor/server/index.ts`, formateado)

```ts
app.post('/api/auth/register', (req, res) => {
  const email = String(req.body?.email ?? '').trim().toLowerCase(),
        password = String(req.body?.password ?? '');

  if (!/^\S+@\S+\.\S+$/.test(email) || password.length < 10 || password.length > 128)
    return res.status(400).json({ error: 'Introduce un correo válido y una contraseña de 10 a 128 caracteres.' });
  // ...
});
```

- `String(x ?? '')` → convierte cualquier cosa a texto (si no llega, `''`).
- `.trim().toLowerCase()` → **normaliza** el email (`" Hector@Mail.com "` → `"hector@mail.com"`).
- Regex simple de email: algo `@` algo `.` algo, sin espacios.
- Contraseña con longitud **mínima y máxima** (la máxima evita hashear textos enormes).

---

## 5.9 Librerías de validación (no están en tus proyectos)

En proyectos grandes se usan librerías como **Zod** para no escribir los `if` a mano:

```ts
// Ejemplo propio con Zod (npm install zod)
import { z } from "zod";

const productSchema = z.object({
  name: z.string().trim().min(1, "El nombre debe ser un texto no vacío"),
  price: z.number().nonnegative("El precio debe ser un número igual o mayor que cero"),
  category: z.string().trim().min(1).optional(),
});

router.post("/", async (req, res) => {
  const result = productSchema.safeParse(req.body ?? {});
  if (!result.success) {
    res.status(400).json({ errors: result.error.flatten().fieldErrors });
    return;
  }
  const data = result.data;   // ya validado y tipado
  // ...
});
```

> 📌 Primero aprende a validar a mano (como en tus proyectos): así entiendes qué hace una
> librería por dentro.

---

## ✅ Resumen rápido

1. Valida id → body → existencia → reglas de negocio, en ese orden.
2. `/^\d+$/` para ids numéricos; `ObjectId.isValid()` para MongoDB.
3. Tipo **y** contenido: `typeof x === "string" && x.trim() !== ""`, `Number.isFinite(n)`.
4. Una función `validateX(body, partial)` sirve para POST/PUT (todo obligatorio) y PATCH (solo lo que llega).
5. Nunca guardes `req.body` tal cual: construye el objeto con los campos permitidos.
6. Lo que decide el servidor (`status`, fechas, contadores) no se lee del body.
7. 400 datos malos · 404 no existe · 409 conflicto con el estado.

👉 **Siguiente:** `06-arquitectura-por-capas-y-base-de-datos.md`
