# 04 — AGREGACIONES E ÍNDICES

> **Agregación** = calcular cosas a partir de muchos documentos (contar, sumar,
> medias, agrupar...). Es el `GROUP BY` de MongoDB.
> **Índice** = una estructura que hace que las búsquedas sean rápidas (y que puede
> obligar a que un valor sea **único**).

---

## 4.1 El pipeline de agregación

```js
db.coleccion.aggregate([
  { $etapa1: { ... } },
  { $etapa2: { ... } },
  { $etapa3: { ... } }
])
```

Un **pipeline** (tubería) es un **array de etapas**. Los documentos entran por la
primera, cada etapa los transforma y pasa el resultado a la siguiente:

```
colección ──► $match ──► $group ──► $sort ──► $limit ──► resultado
             (filtra)   (agrupa)   (ordena)  (corta)
```

| Etapa | Qué hace | SQL parecido |
|---|---|---|
| `$match` | Filtra documentos (mismo filtro que `find`) | `WHERE` (o `HAVING` si va después de `$group`) |
| `$group` | Agrupa y calcula (`$sum`, `$avg`...) | `GROUP BY` |
| `$sort` | Ordena | `ORDER BY` |
| `$limit` / `$skip` | Corta / salta | `LIMIT` / `OFFSET` |
| `$project` | Elige, renombra o calcula campos | `SELECT a AS b` |
| `$lookup` | Trae documentos de otra colección | `JOIN` |
| `$unwind` | "Despliega" un array: un documento por elemento | — |
| `$count` | Cuenta los documentos que llegan | `COUNT(*)` |

> 📌 El **orden importa**: un `$match` al principio filtra antes y hace todo más rápido.

---

## 4.2 `$group`: agrupar y calcular

```js
{ $group: {
    _id: "$campoPorElQueAgrupas",       // ← la "clave" del grupo (con $)
    nombreResultado: { $acumulador: "$campo" }
} }
```

- `_id` es **obligatorio**: indica por qué agrupar. `_id: null` = un único grupo con todo.
- `"$categoria"` con `$` delante = "el **valor** del campo `categoria`".

| Acumulador | Qué calcula |
|---|---|
| `{ $sum: 1 }` | Cuenta documentos (suma 1 por cada uno) |
| `{ $sum: "$precio" }` | Suma un campo |
| `{ $avg: "$precio" }` | Media |
| `{ $min: "$precio" }` / `{ $max: ... }` | Mínimo / máximo |
| `{ $push: "$titulo" }` | Array con todos los valores |
| `{ $addToSet: "$ciudad" }` | Array sin repetidos |
| `{ $first: ... }` / `{ $last: ... }` | Primero / último (según el orden previo) |

### Ejemplos REALES copiados (`mongo-rastro/mongo-rastro/rastro.js`, ejercicio B6)

```js
// B6.1 — cuántos anuncios hay por categoría
db.anuncios.aggregate([
  { $group: { _id: "$categoria", cantidad: { $sum: 1 } } },
  { $sort: { cantidad: -1 } }
])

// B6.2 — precio medio por categoría, ordenado por nombre de categoría
db.anuncios.aggregate([
  { $group: { _id: "$categoria", precioMedio: { $avg: "$precio" } } },
  { $sort: { _id: 1 } }
])

// B6.3 — cuántos anuncios por ciudad
db.anuncios.aggregate([
  { $group: { _id: "$ciudad", cantidad: { $sum: 1 } } },
  { $sort: { cantidad: -1 } }
])
```

**Qué devuelve B6.1** (forma del resultado):

```js
[
  { _id: "Móviles", cantidad: 4 },
  { _id: "Coches", cantidad: 3 },
  ...
]
```

En SQL sería:

```sql
SELECT categoria, COUNT(*) AS cantidad
FROM anuncios
GROUP BY categoria
ORDER BY cantidad DESC;
```

---

## 4.3 `$match` + `$group` + `$sort` + `$limit`

### Ejemplo REAL copiado (B6.4) — valor en venta de cada vendedor

```js
// B6.4
db.anuncios.aggregate([
  { $match: { estado: "DISPONIBLE" } },
  {
    $group: {
      _id: "$vendedor.nombre",
      valorEnVenta: { $sum: "$precio" }
    }
  },
  { $sort: { valorEnVenta: -1 } }
])
```

**Paso a paso:**

```
1. $match  → solo los anuncios DISPONIBLES
2. $group  → un grupo por nombre de vendedor ("$vendedor.nombre", campo anidado)
             y suma sus precios
3. $sort   → del vendedor con más dinero en venta al que menos
```

### Ejemplo REAL copiado (B6.5) — la categoría con más visitas

```js
// B6.5
db.anuncios.aggregate([
  { $group: { _id: "$categoria", visitasTotales: { $sum: "$visitas" } } },
  { $sort: { visitasTotales: -1 } },
  { $limit: 1 }
])
```

> 📌 `$sort: -1` + `$limit: 1` = "el máximo". Es el patrón **TOP N**.

---

## 4.4 `$project`: dar forma al resultado

Después de `$group`, el nombre del grupo queda en `_id`, que es poco claro. Con
`$project` lo renombras y redondeas:

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
router.get("/stats", async (_req, res) => {
  const stats = await listingsCollection()
    .aggregate([
      { $group: { _id: "$category", count: { $sum: 1 }, averagePrice: { $avg: "$price" } } },
      { $project: { _id: 0, category: "$_id", count: 1, averagePrice: { $round: ["$averagePrice", 2] } } },
      { $sort: { category: 1 } },
    ])
    .toArray();

  res.json(stats);
});
```

**Desglose del `$project`:**

```
_id: 0                                   → quita _id
category: "$_id"                         → crea "category" con el valor que había en _id
count: 1                                 → deja count tal cual
averagePrice: { $round: ["$averagePrice", 2] }   → media redondeada a 2 decimales
```

Forma del resultado de `GET /api/listings/stats` (los números son de ejemplo):

```json
[
  { "count": 3, "category": "Coches", "averagePrice": 8433.33 },
  { "count": 4, "category": "Móviles", "averagePrice": 312.5 }
]
```

> ⚠️ Fíjate: esta ruta `/stats` va **antes** de `/:id` en el router. Tu reto 3 de
> `api-rastro/respuestas.md` explica por qué: si va después, Express toma `"stats"`
> como un id y responde 400.

---

## 4.5 `$lookup`: "JOIN" entre colecciones

*(No aparece en tus proyectos: tu modelo del Rastro copia el nombre del vendedor
dentro del anuncio precisamente para **no** necesitar `$lookup`. Ejemplo mío.)*

```js
db.anuncios.aggregate([
  { $match: { estado: "DISPONIBLE" } },
  {
    $lookup: {
      from: "usuarios",           // colección de la que traer datos
      localField: "vendedor.id",  // campo en anuncios
      foreignField: "_id",        // campo en usuarios
      as: "vendedorCompleto"      // nombre del nuevo campo (SIEMPRE es un array)
    }
  },
  { $unwind: "$vendedorCompleto" },  // array de 1 elemento → objeto
  { $project: { titulo: 1, precio: 1, "vendedorCompleto.email": 1, "vendedorCompleto.ciudad": 1 } }
])
```

```
$lookup  → añade vendedorCompleto: [ { _id, nombre, email, ciudad, ... } ]
$unwind  → convierte ese array en un objeto (o duplica el doc si hubiera varios)
```

> 📌 `$lookup` funciona, pero si lo necesitas en **cada** consulta frecuente, quizá
> deberías copiar (desnormalizar) los campos que más lees. Lo verás en el `05`.

### `$unwind` solo (ejemplo mío): contar fotos totales

```js
db.anuncios.aggregate([
  { $unwind: "$fotos" },                 // un documento por cada foto
  { $group: { _id: null, totalFotos: { $sum: 1 } } }
])
```

---

## 4.6 Índices: qué son y por qué importan

Sin índice, para buscar `{ categoria: "Coches" }` Mongo tiene que **mirar todos los
documentos** (*COLLSCAN*). Con un índice sobre `categoria`, va **directo** (*IXSCAN*),
como el índice de un libro.

```js
db.coleccion.createIndex({ campo: 1 })                  // ascendente
db.coleccion.createIndex({ campo: -1 })                 // descendente
db.coleccion.createIndex({ a: 1, b: 1 })                // compuesto
db.coleccion.createIndex({ email: 1 }, { unique: true }) // único
db.coleccion.getIndexes()                               // ver los índices
db.coleccion.dropIndex("nombre_1")                      // borrar uno
```

> 📌 `_id` **siempre** tiene un índice único automático.

| ✅ Ventaja | ❌ Coste |
|---|---|
| Lecturas y `sort` mucho más rápidos | Ocupan espacio |
| Pueden garantizar unicidad | Cada `insert`/`update` tiene que actualizar el índice |

> 📌 Crea índices para los campos por los que **filtras u ordenas a menudo**, no
> para todos.

---

## 4.7 Ejemplos REALES de índices (`mongo-rastro/mongo-rastro/rastro.js`, B3)

```js
// Ejercicio B3 — Índices

db.usuarios.createIndex({ email: 1 }, { unique: true })


db.valoraciones.createIndex(
  { "autor.id": 1, anuncioId: 1 },
  { unique: true }
)


db.anuncios.createIndex({ categoria: 1, precio: 1 })


db.usuarios.getIndexes()
db.valoraciones.getIndexes()
db.anuncios.getIndexes()
```

**Qué hace cada uno:**

| Índice | Tipo | Para qué |
|---|---|---|
| `usuarios { email: 1 }` unique | Simple **único** | No puede haber dos usuarios con el mismo email |
| `valoraciones { "autor.id": 1, anuncioId: 1 }` unique | Compuesto **único** | Un usuario solo puede valorar **una vez** cada anuncio (la **pareja** no se repite) |
| `anuncios { categoria: 1, precio: 1 }` | Compuesto | Acelera "anuncios de X categoría ordenados por precio" |

### El índice compuesto y la "regla del prefijo"

Un índice `{ categoria: 1, precio: 1 }` sirve para:

```js
find({ categoria: "Coches" })                        // ✅ (usa el prefijo "categoria")
find({ categoria: "Coches" }).sort({ precio: 1 })    // ✅ filtra Y ordena con el índice
find({ categoria: "Coches", precio: { $lt: 5000 } }) // ✅
find({ precio: { $lt: 5000 } })                      // ❌ no empieza por categoria
```

> 📌 Un índice compuesto se usa **de izquierda a derecha**. Por eso el orden de los
> campos importa.

---

## 4.8 `explain`: comprobar si se usa el índice

### Ejemplo REAL copiado (`mongo-rastro/mongo-rastro/rastro.js`)

```js
db.anuncios
  .find({ categoria: "Coches" })
  .sort({ precio: 1 })
  .explain("executionStats")
```

En la salida busca:

| Dato | Qué significa |
|---|---|
| `stage: "IXSCAN"` | ✅ Ha usado un índice |
| `stage: "COLLSCAN"` | ❌ Ha recorrido toda la colección |
| `stage: "SORT"` | Ha tenido que ordenar en memoria (el índice no sirvió para ordenar) |
| `totalDocsExamined` | Documentos que ha mirado |
| `nReturned` | Documentos devueltos |

> 📌 Lo ideal: `totalDocsExamined` ≈ `nReturned`. Si examina 10.000 para devolver 3,
> falta un índice.

---

## 4.9 Índices únicos y el error E11000

### Ejemplos REALES (`mongo-rastro/mongo-rastro/rastro.js`, B4.1 y B4.2)

```js
// Ejercicio B4

db.usuarios.insertOne({
  nombre: "Ana Copia",
  email: "ana@rastro.es",
  ciudad: "Valencia",
  fechaAlta: new Date(),
  favoritos: [],
  valoracionMedia: 0,
  numValoraciones: 0
})

// B4.2
db.valoraciones.insertOne({
  anuncioId: ObjectId("670000000000000000000004"),
  autor: { id: ObjectId("660000000000000000000004"), nombre: "Hugo Sanz" },
  valoradoId: ObjectId("660000000000000000000003"),
  estrellas: 1,
  comentario: "Cambio de opinión",
  fecha: new Date()
})
```

Tus respuestas:

```md
- **B4.1 (Email duplicado):** Falla (E11000) por el índice único en email.

- **B4.2 (Valoración duplicada):** Falla (E11000) por el índice único compuesto (autor.id + anuncioId).
```

```
MongoServerError: E11000 duplicate key error collection: rastro.usuarios
index: email_1 dup key: { email: "ana@rastro.es" }
```

### En la API: E11000 → 409 Conflict

En `02b-express-mongodb` el índice único se crea al arrancar (`src/db.ts`):

```ts
  // Reto 2: índice único para que no se repita el nombre de una categoría.
  // Si ya existe, no hace nada.
  await db.collection("categories").createIndex({ name: 1 }, { unique: true });
```

Y se captura el código `11000` en `src/routes/categories.ts`:

```ts
// ¿Es el error de "clave duplicada" del índice único?
function isDuplicateKey(err: unknown): boolean {
  return err instanceof MongoServerError && err.code === 11000;
}
```

> 📌 `createIndex` es **idempotente**: si el índice ya existe igual, no hace nada.
> Por eso se puede llamar en cada arranque. Lo verás completo en el `06`.

---

## ✅ Resumen rápido

1. `aggregate([...etapas])`: los documentos pasan por cada etapa en orden.
2. `$match` (filtrar, mejor al principio), `$group` (agrupar), `$sort`, `$limit`, `$project`.
3. En `$group`, `_id: "$campo"` es la clave; `$sum: 1` cuenta, `$sum/$avg/$min/$max: "$campo"` calculan.
4. `"$campo"` con `$` = el **valor** del campo.
5. `$lookup` = JOIN (resultado siempre en array) → `$unwind` para aplanarlo.
6. `createIndex({ campo: 1 })` acelera; `{ unique: true }` impide duplicados (error **E11000**).
7. Índice compuesto: el orden de campos importa (prefijo de izquierda a derecha).
8. `.explain("executionStats")`: `IXSCAN` bien, `COLLSCAN` mal.

👉 **Siguiente:** `05-modelado-de-datos.md`
