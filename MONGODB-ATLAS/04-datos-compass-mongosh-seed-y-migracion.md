# 04 — DATOS EN ATLAS: COMPASS, MONGOSH, SEED Y MIGRACIÓN

> Un cluster nuevo de Atlas está **vacío**. A diferencia de Postgres/Supabase,
> eso casi no da errores (Mongo crea colecciones al vuelo): simplemente tu
> `GET /api/products` devolverá `[]`. Aquí se ve cómo **mirar**, **cargar** y
> **mover** datos hacia Atlas.

---

## 4.1 Tres formas de ver tus datos

| Herramienta | Dónde | Para qué |
|-------------|-------|----------|
| **Data Explorer** (*Browse Collections*) | Panel web de Atlas | Mirar documentos, filtrar, editar a mano, crear índices. Cero instalación |
| **MongoDB Compass** | App de escritorio | Lo mismo con más potencia: constructor de agregaciones, *explain plan*, análisis de esquema |
| **`mongosh`** | Terminal | Scripts, automatizar, lo mismo que haces con Docker |

> 📌 Las tres usan una **cadena de conexión** con un Database User, y tu IP tiene
> que estar en Network Access (excepto Data Explorer, que va por el panel web).

### Conectar Compass

*Atlas → Connect → Compass* → copiar la URI → en Compass *New connection* → pegar
→ sustituir `<db_password>`.

Puedes guardar **dos conexiones** en Compass: `localhost:27017` (Docker) y
`cluster0` (Atlas), y comparar.

---

## 4.2 `mongosh` contra Atlas

### En tu PC, `mongosh` está dentro del contenedor

Tu proyecto ejecuta `mongosh` **dentro de Docker**:

### Ejemplo REAL (`express-mongo-atlas-render/package.json`, script `seed`)

```json
"seed": "docker cp mongo/seed.js mongodb-practica:/tmp/seed.js && docker exec mongodb-practica mongosh --quiet -u admin -p TU_PASSWORD --authenticationDatabase admin /tmp/seed.js"
```

Ese `mongosh` del contenedor **también puede conectarse a Atlas** (tiene acceso
a Internet). Solo hay que darle la URI en vez de usuario/contraseña locales:

```bash
# Abrir una consola contra Atlas usando el mongosh del contenedor
docker exec -it mongodb-practica mongosh "mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net/bootcamp_api"
```

```js
// Ya dentro: los mismos comandos de siempre (carpeta MONGODB/02)
show dbs
use bootcamp_api
db.products.find()
db.categories.getIndexes()
```

> 💡 Si instalas `mongosh` en Windows (*MongoDB Shell* desde mongodb.com), puedes
> hacer lo mismo sin Docker: `mongosh "<URI>"`.

---

## 4.3 Cargar tu `seed.js` en Atlas

### Ejemplo REAL (`express-mongo-atlas-render/mongo/seed.js`)

```js
// Datos de prueba para 02b. Se ejecuta con mongosh (ver README).
// Borra lo que hubiera y vuelve a empezar: se puede lanzar las veces que quieras.
/* global db, print */ // variables que pone mongosh (para que ESLint no se queje)

const database = db.getSiblingDB("bootcamp_api");

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

El script **no depende** de dónde se ejecute: usa `db.getSiblingDB("bootcamp_api")`
sobre la conexión que le des. Para lanzarlo contra Atlas:

```bash
docker cp mongo/seed.js mongodb-practica:/tmp/seed.js
docker exec mongodb-practica mongosh --quiet "mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net" /tmp/seed.js
# products: 3 · categories: 2
```

> ⚠️ El script empieza con `deleteMany({})`: **borra todo** lo que hubiera en esas
> colecciones. En local está bien; en Atlas, si ya tienes datos "de producción"
> creados por usuarios, **los perderías**. Antes de lanzarlo contra Atlas,
> asegúrate de a qué cluster apunta la URI.

### Variante: un script `seed:atlas` (NO está en tu proyecto)

```json
"seed:atlas": "docker cp mongo/seed.js mongodb-practica:/tmp/seed.js && docker exec mongodb-practica mongosh --quiet \"$MONGODB_URI\" /tmp/seed.js"
```

(En Windows/PowerShell la expansión de `$MONGODB_URI` funciona distinto; lo más
fiable es pegar la URI a mano en la terminal.)

---

## 4.4 Mover datos de tu Docker a Atlas: `mongodump` / `mongorestore`

Si en local ya tienes datos que quieres conservar:

```
Docker (localhost)  ──mongodump──▶  archivo .archive  ──mongorestore──▶  Atlas
```

```bash
# 1. Exportar la base bootcamp_api del contenedor local
docker exec mongodb-practica mongodump \
  -u admin -p TU_PASSWORD --authenticationDatabase admin \
  --db bootcamp_api --archive=/tmp/bootcamp.archive --gzip

# 2. Importarla en Atlas (desde el mismo contenedor)
docker exec mongodb-practica mongorestore \
  --uri "mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net" \
  --archive=/tmp/bootcamp.archive --gzip --drop
```

| Flag | Significado |
|------|-------------|
| `--archive` | Todo en un único archivo (en vez de una carpeta) |
| `--gzip` | Comprimido |
| `--drop` | Borra cada colección en destino antes de restaurarla (⚠️ destructivo) |

> 📌 `mongodump` copia **también los índices** (como el único de `categories`).
> Y al revés sirve como **backup manual**: en M0 no hay backups automáticos,
> así que `mongodump` contra Atlas es tu copia de seguridad.

Otras herramientas: `mongoexport`/`mongoimport` (JSON/CSV legibles, pero sin
índices ni tipos BSON exactos) y la opción **Import** de Compass.

---

## 4.5 Índices en Atlas

Tu API crea su índice al arrancar (ver `03`, 3.4). Puedes comprobarlo en
**Data Explorer → colección `categories` → Indexes**:

```
_id_      { _id: 1 }        (siempre existe)
name_1    { name: 1 }  UNIQUE
```

Y en Atlas tienes ayuda extra para detectar índices que faltan:

- **Performance Advisor** (más completo en clusters dedicados): sugiere índices según las consultas lentas reales.
- **Query Profiler / Query Insights**: consultas lentas con su tiempo.
- En Compass: pestaña **Explain Plan** de una consulta → `COLLSCAN` (recorre toda la colección) vs `IXSCAN` (usa índice). Ver `../MONGODB/04`.

```js
// Tu consulta con category=muebles → sin índice hace COLLSCAN
db.products.find({ category: "muebles" }).explain("executionStats")

// Un índice la convierte en IXSCAN
db.products.createIndex({ category: 1 })
```

> 💡 Con 3 documentos da igual; con 100 000, la diferencia es enorme. Y en M0, con
> recursos compartidos, las consultas sin índice se notan antes.

---

## 4.6 Validación de esquema en la base de datos

MongoDB no obliga a un esquema, pero **puede**. Tu API valida en
`validateProduct()`; puedes añadir una segunda línea de defensa en la propia
colección (útil si otros scripts o personas escriben en Atlas directamente):

```js
// En mongosh contra Atlas — NO está en tu proyecto
db.runCommand({
  collMod: "products",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "price"],
      properties: {
        name:     { bsonType: "string", minLength: 1 },
        price:    { bsonType: ["double", "int", "decimal"], minimum: 0 },
        category: { bsonType: "string" }
      }
    }
  },
  validationAction: "error"   // "warn" solo registra
});
```

```js
db.products.insertOne({ name: "", price: -1 })
// MongoServerError: Document failed validation
```

> 📌 Es el equivalente a los `CHECK` y `NOT NULL` de Postgres. Validar en la API
> da buenos mensajes al usuario; validar en la BD protege los datos aunque la API
> tenga un fallo.

---

## ✅ Resumen

- Data Explorer (web), Compass (escritorio) y `mongosh` (terminal) ven el mismo cluster.
- El `mongosh` de tu contenedor Docker puede conectarse a Atlas pasándole la URI.
- `seed.js` sirve tal cual para Atlas, pero **borra** las colecciones antes.
- `mongodump`/`mongorestore` = migrar de Docker a Atlas y hacer backups manuales.
- Índices: compruébalos en Atlas; `explain` para ver `COLLSCAN` vs `IXSCAN`.
- `$jsonSchema` = validación dentro de la base de datos.
