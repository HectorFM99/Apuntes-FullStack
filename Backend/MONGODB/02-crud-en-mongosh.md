# 02 — CRUD EN MONGOSH

> **CRUD** = **C**reate (insertar), **R**ead (leer), **U**pdate (actualizar),
> **D**elete (borrar). En MongoDB cada operación es un **método de la colección**:
> `db.coleccion.metodo(...)`.

| Operación | SQL | MongoDB |
|---|---|---|
| Crear | `INSERT INTO` | `insertOne`, `insertMany` |
| Leer | `SELECT` | `find`, `findOne`, `countDocuments` |
| Actualizar | `UPDATE ... SET` | `updateOne`, `updateMany` (+ operadores `$set`, `$inc`...) |
| Borrar | `DELETE FROM` | `deleteOne`, `deleteMany` |

---

## 2.1 Insertar: `insertOne`

```js
db.coleccion.insertOne({ campo: valor, ... })
// → { acknowledged: true, insertedId: ObjectId("...") }
```

- Inserta **un** documento.
- Si no pones `_id`, Mongo lo genera y te lo devuelve en `insertedId`.
- Si la colección no existe, **se crea**.

### Ejemplo REAL copiado (`mongo-rastro/mongo-rastro/rastro.js`, ejercicio B2)

```js
db.anuncios.insertOne({
  titulo: "Google Pixel 7",
  descripcion: "Libre, con caja.",
  precio: 260,
  categoria: "Móviles",
  estado: "DISPONIBLE",
  ciudad: "Valencia",
  fechaPublicacion: new Date("2026-10-28"),
  visitas: 0,
  vendedor: {
    id: ObjectId("660000000000000000000005"),
    nombre: "Sara López"
  },
  fotos: [
    { url: "https://img.rastro.es/pixel-1.jpg", orden: 1 },
    { url: "https://img.rastro.es/pixel-2.jpg", orden: 2 }
  ],
  atributos: { marca: "Google", modelo: "Pixel 7", almacenamientoGB: 128, bateriaPct: 92 }
})
```

> 📌 Las fechas con `new Date(...)`, **no** como texto `"2026-10-28"`. Así luego
> puedes ordenarlas y compararlas bien.

---

## 2.2 Insertar varios: `insertMany`

```js
db.coleccion.insertMany([ {...}, {...}, {...} ])
// → { acknowledged: true, insertedIds: { '0': ObjectId(...), '1': ..., '2': ... } }
```

### Ejemplo REAL copiado (`mongo-api-libros/.../02b-express-mongodb/mongo/seed.js`)

```js
database.products.deleteMany({});
database.products.insertMany([
  { name: "Teclado", price: 29.99, category: "perifericos" },
  { name: "Ratón", price: 15, category: "perifericos" },
  { name: "Silla", price: 49.5, category: "muebles" }
]);

// Reto 2: categorías con nombre único
database.categories.deleteMany({});
database.categories.createIndex({ name: 1 }, { unique: true });
database.categories.insertMany([{ name: "perifericos" }, { name: "muebles" }]);

print(`products: ${database.products.countDocuments()} · categories: ${database.categories.countDocuments()}`);
```

> 📌 `print(...)` es el `console.log` de los scripts de `mongosh`.

---

## 2.3 Leer: `find` y `findOne`

```js
db.coleccion.find(filtro, proyeccion)      // → cursor con TODOS los que cumplen
db.coleccion.findOne(filtro, proyeccion)   // → el PRIMERO que cumple, o null
```

```js
db.anuncios.find()                              // todos
db.anuncios.find({ ciudad: "Valencia" })        // los de Valencia (igualdad)
db.anuncios.find({ ciudad: "Valencia", estado: "DISPONIBLE" })  // las 2 cosas (AND implícito)
db.anuncios.findOne({ _id: ObjectId("670000000000000000000001") })
```

> 📌 Varios campos en el mismo filtro = **AND**. `{ a: 1, b: 2 }` significa
> "a es 1 **y** b es 2".

### Ejemplo REAL copiado (`mongo-rastro/mongo-rastro/rastro.js`, B5.1)

```js
// Ejercicio B5
db.anuncios.find(
  { estado: "DISPONIBLE", ciudad: "Valencia" },
  { titulo: 1, precio: 1, _id: 0 }
).sort({ precio: 1 })
```

En SQL sería:

```sql
SELECT titulo, precio FROM anuncios
WHERE estado = 'DISPONIBLE' AND ciudad = 'Valencia'
ORDER BY precio ASC;
```

---

## 2.4 Proyección: elegir qué campos devolver

El **segundo** argumento de `find` dice qué campos quieres:

```js
{ titulo: 1, precio: 1 }          // SOLO titulo y precio (+ _id, que sale siempre)
{ titulo: 1, precio: 1, _id: 0 }  // titulo y precio, SIN _id
{ descripcion: 0, fotos: 0 }      // TODO menos descripcion y fotos
```

| Valor | Significado |
|---|---|
| `1` | Incluir |
| `0` | Excluir |

> ⚠️ No mezcles `1` y `0` en la misma proyección (excepto `_id: 0`, que sí se permite).

### Ejemplos REALES (`mongo-rastro/mongo-rastro/rastro.js`)

```js
// B5.9 — qué usuarios tienen este anuncio en favoritos (solo su nombre)
db.usuarios.find(
  { favoritos: ObjectId("670000000000000000000008") },
  { nombre: 1, _id: 0 }
)

// B5.11 — los 3 anuncios más vistos
db.anuncios.find({}, { titulo: 1, visitas: 1, _id: 0 }).sort({ visitas: -1 }).limit(3)
```

> 📌 `find({})` = filtro vacío = **todos**. Lo necesitas cuando solo quieres poner
> proyección.

---

## 2.5 Actualizar: `updateOne` / `updateMany` con operadores

```js
db.coleccion.updateOne(filtro, { $operador: { campo: valor } })   // el PRIMERO que cumpla
db.coleccion.updateMany(filtro, { $operador: { campo: valor } })  // TODOS los que cumplan
// → { matchedCount: 1, modifiedCount: 1, ... }
```

### ❌/✅ ¡Siempre con operador!

**Ejemplo REAL (`mongo-rastro/mongo-rastro/rastro.js`, B4.5):**

```js
// B4.5
db.anuncios.updateOne(
  { _id: ObjectId("670000000000000000000005") },
  { precio: 80 }
)
```

Tu respuesta en `respuestas.md`:

```md
- **B4.5 (Update sin operador):** Falla al requerir operadores atómicos como $set.
```

```js
// ❌ Error: "Update document requires atomic operators"
db.anuncios.updateOne({ _id: ObjectId("670000000000000000000005") }, { precio: 80 })

// ✅ Correcto
db.anuncios.updateOne({ _id: ObjectId("670000000000000000000005") }, { $set: { precio: 80 } })
```

> 📌 Si quisieras **reemplazar el documento entero** existe `replaceOne(filtro, docNuevo)`.
> Pero casi siempre quieres `$set`.

### Tabla de operadores de actualización

| Operador | Qué hace | Ejemplo |
|---|---|---|
| `$set` | Pone un valor (crea el campo si no existe) | `{ $set: { estado: "VENDIDO" } }` |
| `$unset` | Elimina un campo | `{ $unset: { atributos: "" } }` |
| `$inc` | Suma (o resta con negativo) | `{ $inc: { visitas: 1 } }` |
| `$mul` | Multiplica | `{ $mul: { precio: 0.9 } }` |
| `$push` | Añade al final de un array | `{ $push: { fotos: {...} } }` |
| `$addToSet` | Añade al array **solo si no está** | `{ $addToSet: { favoritos: id } }` |
| `$pull` | Quita del array los que coincidan | `{ $pull: { favoritos: id } }` |
| `$rename` | Cambia el nombre de un campo | `{ $rename: { titulo: "title" } }` |

---

## 2.6 Ejemplos REALES de cada operador

**Archivo:** `mongo-rastro/mongo-rastro/rastro.js` *(copiado tal cual, ejercicio B7)*

### `$push` — añadir una foto al array

```js
// B7.1
db.anuncios.updateOne(
  { _id: ObjectId("670000000000000000000003") },
  { $push: { fotos: { url: "https://img.rastro.es/a3-4.jpg", orden: 4 } } }
)
```

### `$inc` — sumar una visita

```js
// B7.2
db.anuncios.updateOne(
  { _id: ObjectId("670000000000000000000005") },
  { $inc: { visitas: 1 } }
)
```

> 📌 `$inc` es **atómico**: si 100 personas abren el anuncio a la vez, se suman las
> 100. Si hicieras "leer visitas → sumar 1 en JS → guardar", podrías perder visitas.

### `$pull` y `$addToSet` — quitar y añadir favoritos

```js
// B7.3
db.usuarios.updateOne(
  { _id: ObjectId("660000000000000000000005") },
  { $pull: { favoritos: ObjectId("670000000000000000000003") } }
)
db.usuarios.updateOne(
  { _id: ObjectId("660000000000000000000005") },
  { $addToSet: { favoritos: ObjectId("670000000000000000000002") } }
)
```

Y en el archivo se repite el `$addToSet` a propósito: la **segunda vez** da
`modifiedCount: 0`, porque el id ya estaba en el array.

```js
// $push      → [a, b, b]   (permite duplicados)
// $addToSet  → [a, b]      (no duplica)
```

### `$set` — cambiar el estado (oferta aceptada → anuncio reservado)

```js
// B7.4
db.ofertas.updateOne(
  { _id: ObjectId("680000000000000000000003") },
  { $set: { estado: "ACEPTADA" } }
)
db.anuncios.updateOne(
  { _id: ObjectId("670000000000000000000001") },
  { $set: { estado: "RESERVADO" } }
)
```

### `$mul` con `updateMany` — rebajar un 10% todos los coches

```js
// B7.5
db.anuncios.updateMany(
  { categoria: "Coches" },
  { $mul: { precio: 0.9 } }
)
```

### `$set` sobre campos anidados — cambiar un nombre copiado en muchas colecciones

```js
// B7.6
db.usuarios.updateOne(
  { _id: ObjectId("660000000000000000000002") },
  { $set: { nombre: "Luis M. Martín" } }
)


db.anuncios.updateMany(
  { "vendedor.id": ObjectId("660000000000000000000002") },
  { $set: { "vendedor.nombre": "Luis M. Martín" } }
)


db.ofertas.updateMany(
  { "comprador.id": ObjectId("660000000000000000000002") },
  { $set: { "comprador.nombre": "Luis M. Martín" } }
)


db.valoraciones.updateMany(
  { "autor.id": ObjectId("660000000000000000000002") },
  { $set: { "autor.nombre": "Luis M. Martín" } }
)
```

> 📌 `"vendedor.nombre"` con **comillas y punto** = "el campo `nombre` dentro de
> `vendedor`". Esto es el **precio de desnormalizar** (copiar el nombre en varios
> sitios): cuando cambia, hay que actualizarlo en todos. Lo verás en el `05`.

---

## 2.7 Cuidado con los tipos al insertar

**Ejemplo REAL (`mongo-rastro/mongo-rastro/rastro.js`, B4.4):**

```js
// B4.4
db.anuncios.insertOne({
  titulo: "Prueba lámpara",
  precio: "75",
  categoria: "Muebles",
  estado: "DISPONIBLE",
  ciudad: "Valencia",
  fechaPublicacion: new Date(),
  visitas: 0,
  vendedor: { id: ObjectId("660000000000000000000001"), nombre: "Ana García" },
  fotos: [],
  atributos: { material: "metal" }
})


db.anuncios.find(
  { precio: { $lt: 100 } },
  { titulo: 1, precio: 1, _id: 0 }
)
```

Tu respuesta:

```md
- **B4.4 (Precio como texto):** Se inserta, pero la consulta numérica $lt no lo encuentra por tipado estricto.
```

```js
// ❌ Mongo lo acepta... pero ya no aparece en las búsquedas por precio
{ precio: "75" }

// ✅
{ precio: 75 }
```

> 📌 MongoDB **no valida** tipos por defecto. Por eso en la API (`06`) validamos
> con `typeof price !== "number"` antes de insertar.

### Referencias "fantasma" (B4.3)

```js
// B4.3
db.anuncios.insertOne({
  titulo: "Prueba fantasma",
  precio: 10,
  categoria: "Libros",
  estado: "DISPONIBLE",
  ciudad: "Valencia",
  fechaPublicacion: new Date(),
  visitas: 0,
  vendedor: { id: ObjectId("660000000000000000000099"), nombre: "Nadie" },
  fotos: [],
  atributos: {}
})
```

```md
- **B4.3 (Vendedor inexistente):** Se inserta sin error; MongoDB no valida referencias automáticamente.
```

---

## 2.8 Borrar: `deleteOne` / `deleteMany`

```js
db.coleccion.deleteOne(filtro)    // borra el PRIMERO que cumpla
db.coleccion.deleteMany(filtro)   // borra TODOS los que cumplan
// → { acknowledged: true, deletedCount: N }
```

### Ejemplo REAL copiado (`mongo-rastro/mongo-rastro/rastro.js`)

```js
db.anuncios.deleteMany({ titulo: /^Prueba/ })
```

Borra todos los anuncios cuyo título **empieza** por "Prueba" (los de los
ejercicios B4.3 y B4.4). `/^Prueba/` es una expresión regular (ver `03`).

```js
// Uno por _id
db.anuncios.deleteOne({ _id: ObjectId("670000000000000000000009") })
```

### ⚠️ Peligro

```js
db.anuncios.deleteMany({})   // ❌ ¡BORRA TODA LA COLECCIÓN!
db.anuncios.drop()           // ❌ borra la colección entera (y sus índices)
```

> 📌 Antes de un `deleteMany`, lanza un `find` con **el mismo filtro** y comprueba
> qué vas a borrar.

---

## 2.9 `findOneAndUpdate`: actualizar Y devolver el documento

`updateOne` solo te dice **cuántos** cambió. Si quieres el documento resultante:

```js
db.anuncios.findOneAndUpdate(
  { _id: ObjectId("670000000000000000000005") },
  { $inc: { visitas: 1 } },
  { returnDocument: "after" }      // "before" (por defecto) o "after"
)
```

> 📌 Lo usarás mucho en la API (`06`): un `GET /api/listings/:id` de `api-rastro`
> suma una visita y devuelve el anuncio en **una sola operación**.

---

## 2.10 Errores comunes en el CRUD

| Error | Causa | Solución |
|---|---|---|
| `Update document requires atomic operators` | `updateOne(f, { precio: 80 })` | Usa `{ $set: { precio: 80 } }` |
| `findOne` devuelve `null` | Buscas el `_id` como texto | `ObjectId("...")` |
| No encuentra por precio | Guardaste `"75"` (texto) | Guarda números como números |
| `E11000 duplicate key error` | Índice único (ver `04`) | Cambia el valor o captura el error |
| `matchedCount: 0` | El filtro no coincide con nada | Revisa el filtro con `find` |
| `modifiedCount: 0` pero `matchedCount: 1` | El valor ya era ese | Normal (p. ej. `$addToSet` repetido) |

---

## ✅ Resumen rápido

1. `insertOne({...})` / `insertMany([...])` → crean (y la colección si no existe).
2. `find(filtro, proyeccion)` / `findOne(...)` → leen. `{a: 1, b: 2}` = AND.
3. Proyección: `{ campo: 1 }` incluir, `{ campo: 0 }` excluir, `_id: 0` para quitarlo.
4. `updateOne`/`updateMany` **siempre con operador**: `$set`, `$inc`, `$mul`, `$push`, `$addToSet`, `$pull`, `$unset`.
5. `"a.b"` (con comillas) para tocar campos anidados.
6. `deleteOne`/`deleteMany` → cuidado con el filtro vacío `{}`.
7. Mongo **no valida** tipos ni referencias por defecto: valida tú.

👉 **Siguiente:** `03-operadores-de-consulta-sort-limit.md`
