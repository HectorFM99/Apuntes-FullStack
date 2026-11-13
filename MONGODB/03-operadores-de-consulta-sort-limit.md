# 03 — OPERADORES DE CONSULTA, SORT, LIMIT Y SKIP

> Con `{ campo: valor }` solo puedes buscar por **igualdad**. Para "mayor que",
> "uno de estos", "contiene el texto"... se usan **operadores**, que siempre empiezan
> por `$` y van **dentro** del campo: `{ campo: { $operador: valor } }`.

---

## 3.1 Operadores de comparación

| Operador | Significado | SQL equivalente |
|---|---|---|
| `$eq` | igual a | `=` |
| `$ne` | distinto de | `<>` / `!=` |
| `$gt` | mayor que (*greater than*) | `>` |
| `$gte` | mayor o igual (*greater than or equal*) | `>=` |
| `$lt` | menor que (*less than*) | `<` |
| `$lte` | menor o igual | `<=` |
| `$in` | está en esta lista | `IN (...)` |
| `$nin` | NO está en la lista | `NOT IN (...)` |

```js
db.anuncios.find({ precio: { $gt: 100 } })                 // precio > 100
db.anuncios.find({ estado: { $ne: "VENDIDO" } })           // estado != VENDIDO
db.anuncios.find({ ciudad: { $in: ["Valencia", "Castellón"] } })
```

### Ejemplos REALES copiados (`mongo-rastro/mongo-rastro/rastro.js`)

```js
// B5.3 — móviles con 128 GB o más
db.anuncios.find({ categoria: "Móviles", "atributos.almacenamientoGB": { $gte: 128 } })

// B5.4 — precio ENTRE 50 y 400 (dos operadores sobre el mismo campo)
db.anuncios.find({ precio: { $gte: 50, $lte: 400 } })
```

**Desglose de B5.4:**

```
{ precio: { $gte: 50, $lte: 400 } }
            └─ >= 50 ┘ └─ <= 400 ┘   → los dos a la vez (AND)  →  BETWEEN 50 AND 400
```

> 📌 Para un rango, pon **los dos operadores en el mismo objeto**. No escribas
> `{ precio: { $gte: 50 }, precio: { $lte: 400 } }`: una clave repetida en un
> objeto JS **pisa** a la anterior.

### Ejemplo REAL en la API (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
  if (req.query.maxPrice !== undefined) {
    const maxPrice = typeof req.query.maxPrice === "string" ? Number(req.query.maxPrice) : NaN;
    if (req.query.maxPrice === "" || Number.isNaN(maxPrice)) {
      res.status(400).json({ error: "maxPrice tiene que ser un número" });
      return;
    }
    filter.price = { $lte: maxPrice };
  }
```

> 📌 Fíjate en el `Number(...)`: lo que llega en la URL es **texto**. Tu respuesta
> en `api-rastro/respuestas.md` lo explica: *"`"380"` y `380` son tipos distintos,
> así que el filtro no coincide con ningún documento"*.

---

## 3.2 Operadores lógicos: `$and`, `$or`, `$not`, `$nor`

```js
// AND implícito (lo normal)
db.anuncios.find({ categoria: "Coches", precio: { $lt: 5000 } })

// AND explícito (igual que el anterior)
db.anuncios.find({ $and: [ { categoria: "Coches" }, { precio: { $lt: 5000 } } ] })

// OR: cumple UNA de las condiciones
db.anuncios.find({ $or: [ { ciudad: "Valencia" }, { precio: { $lt: 20 } } ] })

// NOR: no cumple NINGUNA
db.anuncios.find({ $nor: [ { estado: "VENDIDO" }, { estado: "RESERVADO" } ] })

// NOT: niega un operador de un campo
db.anuncios.find({ precio: { $not: { $gt: 100 } } })   // precio <= 100 o sin precio
```

*(Estos ejemplos son míos, no están en tus proyectos.)*

### ¿Cuándo hace falta `$and` explícito?

Cuando necesitas **dos `$or`** a la vez (porque no puedes repetir la clave `$or`):

```js
db.anuncios.find({
  $and: [
    { $or: [ { ciudad: "Valencia" }, { ciudad: "Castellón" } ] },
    { $or: [ { categoria: "Móviles" }, { precio: { $lt: 50 } } ] }
  ]
})
```

> 📌 Un `$or` sobre **el mismo campo** se escribe mejor con `$in`:
> `{ ciudad: { $in: ["Valencia", "Castellón"] } }`.

---

## 3.3 Buscar texto: `$regex` y expresiones regulares

```js
db.anuncios.find({ titulo: /nike/i })                            // literal regex de JS
db.anuncios.find({ titulo: { $regex: "nike", $options: "i" } })  // forma con operador (equivalente)
```

| Patrón | Significa |
|---|---|
| `/nike/` | contiene "nike" (distingue mayúsculas) |
| `/nike/i` | contiene "nike", **sin** distinguir mayúsculas (`i` = *insensitive*) |
| `/^Prueba/` | **empieza** por "Prueba" |
| `/\.jpg$/` | **termina** en ".jpg" |

### Ejemplos REALES copiados

**`mongo-rastro/mongo-rastro/rastro.js`:**

```js
// B5.5
db.anuncios.find({ titulo: /nike/i })

// limpieza de los anuncios de prueba
db.anuncios.deleteMany({ titulo: /^Prueba/ })
```

**`mongo-api-libros/.../02b-express-mongodb/src/routes/products.ts`** (ejercicio 6):

```ts
  // Ejercicio 6: buscar por nombre sin distinguir mayúsculas
  if (typeof req.query.search === "string") {
    filter.name = { $regex: req.query.search, $options: "i" };
  }
```

> 📌 En la API se usa la forma `{ $regex, $options }` porque el texto viene de una
> variable (`req.query.search`). Con `GET /api/products?search=TEC` encuentra el
> "Teclado".

> ⚠️ Un `$regex` que no empieza por `^` **no puede usar bien los índices**: recorre
> toda la colección. Para buscadores serios existe el índice de texto (`$text`).

---

## 3.4 Campos anidados (notación punto)

Para filtrar por un campo **dentro** de un objeto, usa `"objeto.campo"` **entre comillas**:

### Ejemplos REALES copiados (`mongo-rastro/mongo-rastro/rastro.js`)

```js
// B5.2 — coches con menos de 100.000 km
db.anuncios.find({ categoria: "Coches", "atributos.km": { $lt: 100000 } })

// B5.6 — anuncios de un vendedor concreto
db.anuncios.find({ "vendedor.id": ObjectId("660000000000000000000002") })

// B5.8 — ofertas pendientes de un anuncio
db.ofertas.find({ "anuncio.id": ObjectId("670000000000000000000001"), estado: "PENDIENTE" })
```

```js
// ❌ Sin comillas: error de sintaxis en JS
db.anuncios.find({ vendedor.id: ObjectId("...") })

// ✅
db.anuncios.find({ "vendedor.id": ObjectId("...") })
```

### ¿Y si el documento no tiene ese campo?

Tu respuesta B5.12 (`respuestas.md`):

```md
Los documentos sin el campo consultado (ej. atributos.km) simplemente no cumplen el filtro y son omitidos sin generar errores.
```

Para buscar **si existe** o no un campo:

```js
db.anuncios.find({ "atributos.km": { $exists: true } })    // los que tienen km
db.anuncios.find({ descripcion: { $exists: false } })      // los que NO tienen descripción
```

---

## 3.5 Consultas sobre arrays

### Buscar un valor dentro de un array

Si el campo es un array, `{ campo: valor }` significa **"el array contiene ese valor"**:

**Ejemplo REAL (`mongo-rastro/mongo-rastro/rastro.js`, B5.9):**

```js
db.usuarios.find(
  { favoritos: ObjectId("670000000000000000000008") },
  { nombre: 1, _id: 0 }
)
```

> 📌 `favoritos` es un array de ids. No hace falta ningún operador especial: Mongo
> mira **dentro** del array.

### Tamaño del array: `$size`

**Ejemplo REAL (B5.7) — anuncios sin fotos:**

```js
db.anuncios.find({ fotos: { $size: 0 } })
```

### Otros operadores de arrays (ejemplos míos)

```js
// Contiene TODOS estos valores
db.anuncios.find({ etiquetas: { $all: ["vintage", "madera"] } })

// Algún elemento cumple VARIAS condiciones a la vez
db.anuncios.find({ fotos: { $elemMatch: { orden: 1, url: /\.jpg$/ } } })

// Campo de un objeto dentro del array (notación punto también vale)
db.anuncios.find({ "fotos.orden": 4 })
```

---

## 3.6 Contar: `countDocuments`

**Ejemplo REAL (B5.10):**

```js
db.anuncios.countDocuments({ estado: "VENDIDO" })
```

Y en `seed.js` (sin filtro = todos):

```js
print(`products: ${database.products.countDocuments()} · categories: ${database.categories.countDocuments()}`);
```

---

## 3.7 Ordenar, limitar y saltar: `sort`, `limit`, `skip`

Se **encadenan** detrás de `find(...)`:

```js
db.coleccion.find(filtro).sort({ campo: 1 }).skip(n).limit(m)
```

| Método | Qué hace | SQL |
|---|---|---|
| `.sort({ campo: 1 })` | Orden **ascendente** (menor → mayor, A → Z) | `ORDER BY campo ASC` |
| `.sort({ campo: -1 })` | Orden **descendente** | `ORDER BY campo DESC` |
| `.limit(n)` | Como máximo `n` resultados | `LIMIT n` |
| `.skip(n)` | Se salta los `n` primeros | `OFFSET n` |

### Ejemplos REALES copiados (`mongo-rastro/mongo-rastro/rastro.js`)

```js
// B5.1 — los disponibles de Valencia, del más barato al más caro
db.anuncios.find(
  { estado: "DISPONIBLE", ciudad: "Valencia" },
  { titulo: 1, precio: 1, _id: 0 }
).sort({ precio: 1 })

// B5.11 — TOP 3 por visitas
db.anuncios.find({}, { titulo: 1, visitas: 1, _id: 0 }).sort({ visitas: -1 }).limit(3)
```

### En la API (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
  const listings = await listingsCollection()
    .find(filter)
    .project(LIST_FIELDS)
    .sort({ publishedAt: -1 })
    .toArray();
```

> 📌 `sort({ publishedAt: -1 })` = **los más nuevos primero** (orden típico de un
> listado de anuncios).

### Paginación con `skip` + `limit` (ejemplo mío)

```js
const porPagina = 10;
const pagina = 3;    // empezando en 1

db.anuncios.find()
  .sort({ fechaPublicacion: -1 })
  .skip((pagina - 1) * porPagina)   // se salta 20
  .limit(porPagina)                 // devuelve del 21 al 30
```

> ⚠️ Ordena **siempre** cuando pagines. Sin `sort`, el orden no está garantizado y
> podrías ver el mismo documento en dos páginas.

### Orden por varios campos

```js
db.anuncios.find().sort({ categoria: 1, precio: -1 })
// primero por categoría A→Z; dentro de cada categoría, del más caro al más barato
```

---

## 3.8 Chuleta: de SQL a MongoDB

| SQL | MongoDB |
|---|---|
| `WHERE precio = 300` | `{ precio: 300 }` |
| `WHERE precio > 300` | `{ precio: { $gt: 300 } }` |
| `WHERE precio BETWEEN 50 AND 400` | `{ precio: { $gte: 50, $lte: 400 } }` |
| `WHERE ciudad IN ('A','B')` | `{ ciudad: { $in: ["A", "B"] } }` |
| `WHERE a = 1 AND b = 2` | `{ a: 1, b: 2 }` |
| `WHERE a = 1 OR b = 2` | `{ $or: [ { a: 1 }, { b: 2 } ] }` |
| `WHERE titulo ILIKE '%nike%'` | `{ titulo: /nike/i }` |
| `WHERE titulo LIKE 'Prueba%'` | `{ titulo: /^Prueba/ }` |
| `WHERE descripcion IS NULL` | `{ descripcion: null }` (o `$exists: false`) |
| `SELECT titulo, precio` | proyección `{ titulo: 1, precio: 1, _id: 0 }` |
| `ORDER BY precio DESC` | `.sort({ precio: -1 })` |
| `LIMIT 3 OFFSET 6` | `.skip(6).limit(3)` |
| `SELECT COUNT(*) WHERE ...` | `.countDocuments({ ... })` |

---

## ✅ Resumen rápido

1. Operadores dentro del campo: `{ precio: { $gte: 50, $lte: 400 } }`.
2. Comparación: `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin`.
3. Lógicos: AND implícito, `$or: [ ... ]`, `$and`, `$nor`, `$not`.
4. Texto: `/texto/i` o `{ $regex: texto, $options: "i" }`; `^` = empieza por.
5. Anidados: `"vendedor.id"` con comillas. `$exists` para saber si hay campo.
6. Arrays: `{ favoritos: id }` busca dentro; `$size`, `$all`, `$elemMatch`.
7. `.sort({ c: 1 / -1 })`, `.limit(n)`, `.skip(n)`; `countDocuments(filtro)`.
8. Lo que viene de la URL es **texto**: conviértelo antes de comparar números.

👉 **Siguiente:** `04-agregaciones-e-indices.md`
