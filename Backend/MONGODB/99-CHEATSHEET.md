# 📋 CHEATSHEET — MONGODB

> Chuleta rápida de comandos y código. Teoría en los documentos 01–07 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal

### Docker Compose (`practica-MongoDB/compose.yaml`)

```yaml
services:
  mongodb:
    image: mongo:8
    container_name: mongodb-practica
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin_dev_123
    ports:
      - "27017:27017"            # puerto PC : puerto contenedor
    volumes:
      - mongodb_data:/data/db    # los datos sobreviven
    restart: unless-stopped
volumes:
  mongodb_data:
```

| Comando | Qué hace |
|---------|----------|
| `docker compose up -d` | Arranca en segundo plano |
| `docker compose ps` | Ver si está corriendo |
| `docker compose stop` | Parar (los datos se quedan) |
| `docker compose down` | Parar y borrar el contenedor (el volumen se queda) |
| `docker compose down -v` | ⚠️ Borra también el volumen = **borra los datos** |
| `docker start mongodb-practica` | Arrancar un contenedor que ya existía |
| `docker exec -it mongodb-practica mongosh -u admin -p admin_dev_123 --authenticationDatabase admin` | Entrar en `mongosh` |
| `docker cp mongo/seed.js mongodb-practica:/tmp/seed.js` | Copiar un script dentro del contenedor |
| `docker exec mongodb-practica mongosh --quiet -u admin -p admin_dev_123 --authenticationDatabase admin /tmp/seed.js` | Ejecutar el script |
| `npm run seed` | Lo anterior en un script de `package.json` |

### Dentro de `mongosh`

```js
show dbs                     // lista las bases de datos
use rastro                   // cambia a (o "crea") la base rastro
show collections             // colecciones de la base actual
db.anuncios.find()           // todos los documentos
db.anuncios.countDocuments() // cuántos hay
db.anuncios.getIndexes()     // índices de la colección
db.anuncios.drop()           // ⚠️ borra la colección entera
```

### Script de datos repetible (`mongo/seed.js`)

```js
const database = db.getSiblingDB("bootcamp_api");   // en un script no vale "use"
database.products.deleteMany({});                    // vacía
database.products.insertMany([                       // y vuelve a llenar
  { name: "Teclado", price: 29.99, category: "perifericos" },
  { name: "Silla", price: 49.5, category: "muebles" }
]);
```

---

## 2. 🔗 Cadena de conexión (`.env`)

```env
MONGODB_URI=mongodb://admin:admin_dev_123@localhost:27017/?authSource=admin
DB_NAME=bootcamp_api
```

```
mongodb:// usuario : password @ host : 27017 / ?authSource=admin
```

---

## 3. ✍️ CRUD en `mongosh`

| Operación | Código |
|-----------|--------|
| Insertar uno | `db.anuncios.insertOne({ titulo: "Bici", precio: 120 })` |
| Insertar varios | `db.anuncios.insertMany([{...}, {...}])` |
| Leer todos | `db.anuncios.find()` |
| Leer con filtro | `db.anuncios.find({ ciudad: "Valencia", estado: "DISPONIBLE" })` (AND implícito) |
| Leer uno | `db.anuncios.findOne({ _id: ObjectId("670000000000000000000001") })` |
| Proyección | `db.anuncios.find({}, { titulo: 1, precio: 1, _id: 0 })` |
| Actualizar uno | `db.anuncios.updateOne({ _id: ObjectId("...") }, { $set: { precio: 80 } })` |
| Actualizar varios | `db.anuncios.updateMany({ categoria: "Coches" }, { $mul: { precio: 0.9 } })` |
| Actualizar y devolver | `db.anuncios.findOneAndUpdate(filtro, { $inc: { visitas: 1 } }, { returnDocument: "after" })` |
| Borrar uno | `db.anuncios.deleteOne({ _id: ObjectId("...") })` |
| Borrar varios | `db.anuncios.deleteMany({ titulo: /^Prueba/ })` |
| ⚠️ Borrar TODO | `db.anuncios.deleteMany({})` |

### Operadores de actualización

| Operador | Ejemplo | Qué hace |
|----------|---------|----------|
| `$set` | `{ $set: { estado: "RESERVADO" } }` | Cambia/crea un campo |
| `$unset` | `{ $unset: { descripcion: "" } }` | Quita un campo |
| `$inc` | `{ $inc: { visitas: 1 } }` | Suma (o resta con negativo) |
| `$mul` | `{ $mul: { precio: 0.9 } }` | Multiplica |
| `$push` | `{ $push: { fotos: { url: "f.jpg", orden: 4 } } }` | Añade al array |
| `$addToSet` | `{ $addToSet: { favoritos: id } }` | Añade si no está |
| `$pull` | `{ $pull: { favoritos: id } }` | Quita del array |
| Anidado | `{ $set: { "vendedor.nombre": "Ana" } }` | Notación punto **entre comillas** |

> ⚠️ `updateOne(filtro, { precio: 80 })` sin operador = error. Siempre `$set`.

---

## 4. 🔎 Operadores de consulta

| Operador | Ejemplo |
|----------|---------|
| `$eq` / `$ne` | `{ estado: { $ne: "VENDIDO" } }` |
| `$gt` `$gte` `$lt` `$lte` | `{ precio: { $gte: 50, $lte: 400 } }` |
| `$in` / `$nin` | `{ ciudad: { $in: ["Valencia", "Castellón"] } }` |
| `$or` | `{ $or: [ { ciudad: "Valencia" }, { precio: { $lt: 20 } } ] }` |
| `$and` | `{ $and: [ { categoria: "Coches" }, { precio: { $lt: 5000 } } ] }` |
| `$nor` | `{ $nor: [ { estado: "VENDIDO" }, { estado: "RESERVADO" } ] }` |
| `$not` | `{ precio: { $not: { $gt: 100 } } }` |
| `$regex` | `{ titulo: /nike/i }` o `{ titulo: { $regex: "nike", $options: "i" } }` |
| `$exists` | `{ "atributos.km": { $exists: true } }` |
| Anidado | `{ "atributos.km": { $lt: 100000 } }` |
| Array contiene | `{ favoritos: ObjectId("...") }` |
| `$size` | `{ fotos: { $size: 0 } }` |
| `$all` | `{ etiquetas: { $all: ["vintage", "madera"] } }` |
| `$elemMatch` | `{ fotos: { $elemMatch: { orden: 1, url: /\.jpg$/ } } }` |

### Ordenar, limitar, paginar, contar

```js
db.anuncios.find({ estado: "DISPONIBLE" }).sort({ precio: 1 })   // 1 asc, -1 desc
db.anuncios.find({}, { titulo: 1, visitas: 1, _id: 0 }).sort({ visitas: -1 }).limit(3)  // TOP 3
db.anuncios.find().sort({ publishedAt: -1 }).skip(20).limit(10)  // página 3 de 10 en 10
db.anuncios.countDocuments({ estado: "VENDIDO" })
```

---

## 5. 📊 Agregaciones

```js
db.anuncios.aggregate([
  { $match: { estado: "DISPONIBLE" } },                          // WHERE (primero = más rápido)
  { $group: { _id: "$vendedor.nombre", valorEnVenta: { $sum: "$precio" } } },  // GROUP BY
  { $sort: { valorEnVenta: -1 } },                               // ORDER BY
  { $limit: 5 }                                                  // LIMIT
])
```

| Etapa | SQL | | Acumulador | Calcula |
|-------|-----|---|------------|---------|
| `$match` | `WHERE` | | `{ $sum: 1 }` | Cuenta |
| `$group` | `GROUP BY` | | `{ $sum: "$precio" }` | Suma |
| `$sort` | `ORDER BY` | | `{ $avg: "$precio" }` | Media |
| `$limit` / `$skip` | `LIMIT` / `OFFSET` | | `{ $min }` / `{ $max }` | Mín / máx |
| `$project` | `SELECT a AS b` | | `{ $push: "$titulo" }` | Array de valores |
| `$lookup` | `JOIN` | | `{ $addToSet: "$ciudad" }` | Array sin repetidos |
| `$unwind` | — (array → 1 doc por elemento) | | `_id: null` | Un solo grupo con todo |
| `$count` | `COUNT(*)` | | | |

```js
// $lookup: traer el vendedor de otra colección
{ $lookup: { from: "usuarios", localField: "vendedor.id", foreignField: "_id", as: "vendedorCompleto" } },
{ $unwind: "$vendedorCompleto" }           // array de 1 → objeto
```

---

## 6. ⚡ Índices

```js
db.usuarios.createIndex({ email: 1 }, { unique: true })              // simple único
db.valoraciones.createIndex({ "autor.id": 1, anuncioId: 1 }, { unique: true })  // pareja única
db.anuncios.createIndex({ categoria: 1, precio: 1 })                  // compuesto
db.anuncios.getIndexes()                                              // listar
db.anuncios.dropIndex("categoria_1_precio_1")                         // borrar
db.anuncios.find({ categoria: "Coches" }).sort({ precio: 1 }).explain("executionStats")
```

| En `explain` | Significa |
|--------------|-----------|
| `IXSCAN` | ✅ Usa índice |
| `COLLSCAN` | ❌ Recorre toda la colección |
| `SORT` | Ordena en memoria |
| `totalDocsExamined` ≈ `nReturned` | ✅ Ideal |

> 📌 Regla del prefijo: `{ categoria: 1, precio: 1 }` sirve para filtrar por `categoria`, no solo por `precio`.

---

## 7. 🟢 Driver de Node (`mongodb`) con TypeScript

```bash
npm install mongodb          # ya trae los tipos
```

```json
"dev": "tsx watch --env-file=.env src/index.ts"
```

```ts
// src/db.ts
import { Db, MongoClient } from "mongodb";
const client = new MongoClient(process.env.MONGODB_URI!, { serverSelectionTimeoutMS: 5000 });
let db: Db;

export async function connectDb() {
  await client.connect();
  db = client.db(process.env.DB_NAME ?? "rastro");
  await db.collection("categories").createIndex({ name: 1 }, { unique: true }); // idempotente
}
export const getDb = () => db;
export const productsCollection = () => getDb().collection<Product>("products"); // tipada
```

```ts
// src/index.ts — conectar ANTES de escuchar
await connectDb();
app.listen(PORT);

app.get("/api/health", async (_req, res) => {
  await getDb().command({ ping: 1 });                 // comprueba la BD
  res.json({ status: "ok", db: getDb().databaseName });
});
```

### Patrones de rutas

```ts
import { ObjectId, MongoServerError } from "mongodb";

if (!ObjectId.isValid(req.params.id)) return res.status(400).json({ error: "ID no válido" });
const _id = new ObjectId(req.params.id);                              // texto → ObjectId

const list = await productsCollection().find(filter).project({ name: 1 }).sort({ price: 1 }).toArray();
const one  = await productsCollection().findOne({ _id });             // null → 404
const { insertedId } = await productsCollection().insertOne(doc);     // → 201
const updated = await productsCollection().findOneAndUpdate(
  { _id }, { $set: changes }, { returnDocument: "after" });           // null → 404
const { deletedCount } = await productsCollection().deleteOne({ _id }); // 0 → 404, si no 204

// Filtros desde req.query
if (typeof req.query.search === "string")
  filter.name = { $regex: req.query.search, $options: "i" };

// Clave duplicada (índice único) → 409
const isDuplicateKey = (err: unknown) => err instanceof MongoServerError && err.code === 11000;
```

### Mongoose (alternativa con esquemas)

```ts
import mongoose from "mongoose";
await mongoose.connect(process.env.MONGODB_URI!);
const Product = mongoose.model("Product", new mongoose.Schema({ name: String, price: Number }));
await Product.find({ price: { $lt: 30 } });
```

---

## 8. 🧩 Modelado (resumen)

| Embeber (dentro) | Referenciar (`_id`) |
|------------------|---------------------|
| Pocos datos, se leen juntos (fotos del anuncio) | Muchos o crecen sin límite (ofertas) |
| Una lectura, escritura atómica | Se consultan por separado |
| Copias "foto del momento" (precio en la oferta) | Datos que cambian y deben estar al día |

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `findOne({ _id: "6700..." })` devuelve `null` | Usar `ObjectId("6700...")` |
| `Authentication failed` | Falta `--authenticationDatabase admin` / `?authSource=admin`, o el volumen tiene otra contraseña (`down -v` borra datos) |
| `E11000 duplicate key error` | Índice único: responder **409** en la API |
| Update sin efecto o error "update document requires atomic operators" | Usar `$set`, `$inc`… |
| `BSONError: input must be a 24 character hex string` | Validar con `ObjectId.isValid` → 400 |
| `vendedor.id: ...` da error de sintaxis | Campo anidado entre comillas: `"vendedor.id"` |
| `MongoServerSelectionError` / `ECONNREFUSED 27017` | El contenedor no está arrancado: `docker compose up -d` |
| Consulta lenta, `COLLSCAN` en `explain` | Crear el índice adecuado |
