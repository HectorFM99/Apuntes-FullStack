# 🍃 CARPETA MONGODB — ÍNDICE DE ESTUDIO

Teoría de **MongoDB** explicada con ejemplos reales **copiados y pegados de tus
proyectos** del Bootcamp: consultas en `mongosh`, Docker y la API con Express +
TypeScript usando el driver oficial.

> ⚠️ Los proyectos **NO** se han modificado. Solo se han copiado fragmentos de su código.
> 🔒 No se ha copiado ningún archivo `.env` (solo los `.env.example`).

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a aprender |
|---|---------|--------------------|
| 1 | `01-introduccion-nosql-documentos-y-docker.md` | NoSQL vs SQL, base de datos/colección/documento, BSON, `ObjectId`, Docker Compose, `mongosh` |
| 2 | `02-crud-en-mongosh.md` | `insertOne`/`insertMany`, `find`/`findOne`, proyección, `updateOne`/`updateMany` con `$set`, `$inc`, `$push`..., `deleteOne`/`deleteMany` |
| 3 | `03-operadores-de-consulta-sort-limit.md` | `$gt`, `$lte`, `$in`, `$and`, `$or`, `$regex`, campos anidados, arrays, `sort`, `limit`, `skip`, `countDocuments` |
| 4 | `04-agregaciones-e-indices.md` | Pipeline `aggregate`: `$match`, `$group`, `$sort`, `$project`, `$limit`, `$lookup`; índices simples, compuestos y únicos, `explain` |
| 5 | `05-modelado-de-datos.md` | Embeber vs referenciar, desnormalización, copias históricas, integridad sin claves ajenas |
| 6 | `06-mongodb-con-node-express-typescript.md` | Driver `mongodb`: conexión, colecciones tipadas, `ObjectId.isValid`, 400/404/409, `findOneAndUpdate`, error `11000`, Mongoose |
| 7 | `07-ejemplos-completos-de-proyectos.md` | Archivos enteros copiados de tus proyectos |
| 📋 | `99-CHEATSHEET.md` | Chuleta de comandos y código para consultar rápido |

---

## 🗂️ Proyectos de los que se han extraído los ejemplos

| Proyecto | Archivos | Temas que cubre |
|----------|----------|-----------------|
| `mongo-rastro/mongo-rastro/` | `rastro.js` | Consultas en `mongosh`: CRUD, operadores, índices, agregaciones, `$push`, `$inc`, `$addToSet`, `$pull`, `$mul` |
| `mongo-rastro/mongo-rastro/` | `respuestas.md` | Modelado (embeber/aparte), errores típicos (E11000, tipos, `ObjectId` como texto) |
| `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/` | `compose.yaml`, `.env.example`, `mongo/seed.js` | Docker, cadena de conexión, script de datos con `mongosh` |
| `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/` | `src/db.ts`, `src/index.ts`, `src/routes/products.ts`, `src/routes/categories.ts` | API CRUD con el driver: `find`, `$regex`, `insertOne`, `findOneAndUpdate`, `deleteOne`, índice único → 409 |
| `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/` | `src/soluciones/*.ts` | `updateOne` + `matchedCount`, convertir `_id` → `id` con `toHexString()` |
| `practica-MongoDB/` | `compose.yaml` | Contenedor `mongo:8` con usuario, volumen y `restart` |
| `api-rastro/api-rastro/` | `src/db.ts`, `src/types.ts`, `src/routes/listings.ts`, `src/routes/offers.ts`, `respuestas.md` | Colecciones tipadas, `Filter<T>`, `.project()`, `.sort()`, `aggregate` en la API, `$inc` de visitas, `ObjectId.equals()` |

---

## 🚀 Orden de estudio recomendado

1. `01` → qué es MongoDB y cómo arrancarlo (sin esto no puedes practicar).
2. `02` → CRUD en `mongosh`: lo que más vas a escribir.
3. `03` → filtros potentes: operadores, ordenar y paginar.
4. `04` → agregaciones (el "GROUP BY" de Mongo) e índices.
5. `05` → **modelado**: la parte más "de pensar" (y la que más preguntan).
6. `06` → llevarlo todo a una API real con Express + TypeScript.
7. `07` → código real completo de tus proyectos.

---

## 💡 La idea clave en 10 segundos

```sql
-- SQL (PostgreSQL): tablas con filas y columnas fijas
SELECT name, price FROM products WHERE price < 30;
```
```js
// MongoDB (mongosh): colecciones con DOCUMENTOS tipo JSON
db.products.find({ price: { $lt: 30 } }, { name: 1, price: 1 })
```
```ts
// MongoDB desde Node (driver oficial): lo mismo pero con await
const products = await getDb().collection("products").find({ price: { $lt: 30 } }).toArray();
```

> 📌 En MongoDB **las consultas son objetos JavaScript**. Si sabes escribir un objeto
> `{ clave: valor }`, ya sabes escribir el 80% de las consultas.
