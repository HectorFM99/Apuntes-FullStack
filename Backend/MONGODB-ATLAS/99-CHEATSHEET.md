# 📋 CHEATSHEET — MONGODB ATLAS

> Chuleta rápida de comandos y código. Teoría en los documentos 01–08 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal

| Comando | Qué hace |
|---------|----------|
| `docker exec -it mongodb-practica mongosh "mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net/bootcamp_api"` | Consola `mongosh` contra Atlas (usando el mongosh del contenedor) |
| `mongosh "<URI>"` | Lo mismo con *MongoDB Shell* instalado en Windows |
| `npm run dev` | API local contra Atlas (`tsx watch --env-file=.env src/index.ts`) |
| `npm run build` / `npm start` | `tsc` → `dist/` / `node dist/index.js` (lo que ejecuta Render) |
| `node --env-file=.env probar-atlas.mjs` | Script de diagnóstico de la conexión (§7) |
| `curl -i https://<api>.onrender.com/api/products` | Probar la API desplegada |

### Cargar el `seed.js` en Atlas

```bash
docker cp mongo/seed.js mongodb-practica:/tmp/seed.js                      # copia el script
docker exec mongodb-practica mongosh --quiet "<URI_ATLAS>" /tmp/seed.js     # lo ejecuta contra Atlas
```

```json
"seed:atlas": "docker cp mongo/seed.js mongodb-practica:/tmp/seed.js && docker exec mongodb-practica mongosh --quiet \"$MONGODB_URI\" /tmp/seed.js"
```

### Migrar de Docker a Atlas / backup manual

```bash
# 1. Exportar la base local
docker exec mongodb-practica mongodump \
  -u admin -p TU_PASSWORD --authenticationDatabase admin \
  --db bootcamp_api --archive=/tmp/bootcamp.archive --gzip

# 2. Importar en Atlas
docker exec mongodb-practica mongorestore \
  --uri "mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net" \
  --archive=/tmp/bootcamp.archive --gzip --drop
```

| Flag | Qué hace |
|------|----------|
| `--archive=...` | Todo en un único archivo |
| `--gzip` | Comprimido |
| `--drop` | ⚠️ Borra cada colección en destino antes de restaurar |
| `--db` | Solo esa base |

> 📌 `mongodump` copia también los **índices**. En M0 no hay backups automáticos: `mongodump` contra Atlas = tu backup.

---

## 2. 🏗️ Montar Atlas (pasos en el panel)

```
1. Crear proyecto → Create cluster → M0 (Free) → proveedor + región (Frankfurt/eu)
2. Database Access → Add New Database User → usuario + contraseña autogenerada
   → Rol: readWrite sobre bootcamp_api (Specific Privileges)
3. Network Access → Add IP Address → tu IP  y/o  0.0.0.0/0 (Render Free)
4. Connect → Drivers → copiar la URI mongodb+srv://
5. Pegar en .env (local) y en Render → Environment (producción)
```

| Rol | Úsalo para |
|-----|------------|
| `readWrite@bootcamp_api` | ✅ Tu API en producción (puede crear índices) |
| `read@bootcamp_api` | Solo lectura (informes) |
| `readWriteAnyDatabase` | Lo que pone el asistente por defecto (demasiado amplio) |
| `atlasAdmin` | Administración, nunca en una API |

| IP Access List | Cuándo |
|----------------|--------|
| *Add Current IP Address* | Desarrollo desde tu PC |
| `0.0.0.0/0` | Render Free (IPs de salida variables). La seguridad queda en usuario + TLS |

---

## 3. 🔗 La URI

```env
# Local (Docker)
MONGODB_URI=mongodb://admin:TU_PASSWORD@localhost:27017/?authSource=admin
# Atlas
MONGODB_URI="mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net"
DB_NAME=bootcamp_api
```

```
mongodb+srv:// <USUARIO>:<PASSWORD> @ cluster0.<ID>.mongodb.net / [bd] ? retryWrites=true & w=majority & appName=Cluster0
```

| Parte | Docker | Atlas |
|-------|--------|-------|
| Protocolo | `mongodb://` | `mongodb+srv://` (DNS SRV → 3 nodos) |
| Puerto | `:27017` | ❌ Sin puerto |
| `authSource` | `admin` (manual) | Lo da el DNS (TXT) |
| TLS | No | Sí, automático |

| Opción | Qué hace |
|--------|----------|
| `retryWrites=true` | Reintenta una vez escrituras fallidas (por defecto ya `true`) |
| `w=majority` | Confirma cuando la mayoría de nodos tiene el dato |
| `appName=api-products` | Identifica tu app en las métricas de Atlas |
| `readPreference=secondaryPreferred` | Leer de secondaries (datos algo atrasados) |
| `maxPoolSize` | Máx. conexiones del pool (defecto 100) |
| `serverSelectionTimeoutMS` | Espera antes de fallar (defecto 30000) |

```js
encodeURIComponent('p@ss#1')   // 'p%40ss%231' → caracteres especiales en la contraseña
```

---

## 4. 🟢 Código de conexión (`express-mongo-atlas-render`)

```ts
// src/db.ts
import { Db, MongoClient } from "mongodb";

const uri = process.env.MONGODB_URI;
if (!uri) throw new Error("Falta MONGODB_URI en el archivo .env");   // fallar rápido

const client = new MongoClient(uri, {
  appName: "api-products",          // (mejora) identificar la app
  serverSelectionTimeoutMS: 5000,   // (mejora) fallar en 5 s en vez de 30
  maxPoolSize: 10,                  // (mejora) suficiente para M0
});
let db: Db;

export async function connectDb() {
  await client.connect();
  db = client.db(process.env.DB_NAME ?? "bootcamp_api");
  await db.collection("categories").createIndex({ name: 1 }, { unique: true }); // idempotente
}
export const getDb = () => db;
```

```ts
// src/index.ts
const PORT = process.env.PORT ?? 3000;        // Render pone PORT

async function start() {
  await connectDb();                          // si Atlas falla, la API no arranca
  app.listen(PORT, () => console.log(`Servidor escuchando en http://localhost:${PORT}`));
}
start();

process.on("SIGTERM", async () => {           // (mejora) cierre limpio al parar Render
  await client.close();
  process.exit(0);
});
```

---

## 5. 🚀 Desplegar con Render

| Campo de Render | Valor |
|-----------------|-------|
| Root Directory | `mongo-atlas-render/express-mongo-atlas-render` (ruta real en tu repo) |
| Build Command | `npm install && npm run build` |
| Start Command | `npm start` |
| `MONGODB_URI` | URI de Atlas (usuario `api-render`), **sin comillas** en el panel |
| `DB_NAME` | `bootcamp_api` |
| `NODE_VERSION` | `24` |

```json
"dev": "tsx watch --env-file=.env src/index.ts",   // solo dev lee el .env
"build": "tsc",
"start": "node dist/index.js"                      // lee las variables del panel
```

```bash
curl -i https://<api>.onrender.com/api/products                    # 200
curl -i -X POST https://<api>.onrender.com/api/categories \
  -H "Content-Type: application/json" -d '{"name":"cocina"}'       # 201, y repetido → 409
```

Logs esperados: `Conectado a MongoDB` → `Servidor escuchando...` → `Your service is live 🎉`.

---

## 6. 🛠️ Datos, índices y validación en Atlas

```js
// mongosh contra Atlas: los mismos comandos de siempre
show dbs
use bootcamp_api
db.products.find()
db.categories.getIndexes()
db.products.createIndex({ category: 1 })                             // evitar COLLSCAN
db.products.find({ category: "muebles" }).explain("executionStats")  // IXSCAN ✅

// Validación de esquema (≈ CHECK / NOT NULL de Postgres)
db.runCommand({
  collMod: "products",
  validator: { $jsonSchema: {
    bsonType: "object",
    required: ["name", "price"],
    properties: {
      name:  { bsonType: "string", minLength: 1 },
      price: { bsonType: ["double", "int", "decimal"], minimum: 0 }
    }
  } },
  validationAction: "error"          // "warn" solo registra
});
```

| Herramienta | Para qué |
|-------------|----------|
| **Data Explorer** (web) | Ver/editar documentos e índices |
| **Compass** (escritorio) | GUI completa, pegando la URI |
| **Performance Advisor** | Sugiere índices |
| **Metrics / Alerts** | Conexiones, operaciones, espacio |

| Límites M0 | |
|------------|---|
| Almacenamiento | 512 MB |
| Backups | ❌ (usa `mongodump`) |
| Replica set | ✅ 3 nodos (transacciones OK) |

---

## 7. 🩺 Script de diagnóstico

```js
// probar-atlas.mjs  →  node --env-file=.env probar-atlas.mjs
import { MongoClient } from "mongodb";
const client = new MongoClient(process.env.MONGODB_URI, { serverSelectionTimeoutMS: 5000 });
try {
  await client.connect();
  const db = client.db(process.env.DB_NAME ?? "bootcamp_api");
  console.log("✅ Conectado", await db.command({ ping: 1 }));
  console.log("Colecciones:", (await db.listCollections().toArray()).map((c) => c.name));
} catch (err) {
  console.error("❌", err.name, "-", err.message);
} finally {
  await client.close();
}
```

```
Falla al instante "bad auth"          → CREDENCIALES
Tarda ~30 s MongoServerSelectionError → RED (IP Access List)
querySrv ENOTFOUND                    → DNS / URI
Conecta pero []                       → DB_NAME / base vacía
not authorized on ...                 → ROL
```

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `MongoServerSelectionError` tras ~30 s | Network Access: añade tu IP o `0.0.0.0/0` (Render) |
| `bad auth : authentication failed` | Usuario de **Database Access** (no tu email), contraseña sin `< >`, codificada, sin comillas en Render |
| `querySrv ENOTFOUND` / `ECONNREFUSED` | URI mal copiada, puerto en `+srv`, o DNS que no resuelve SRV (usa `8.8.8.8` o la URI sin `+srv`) |
| `user is not allowed to do action [insert]` | Rol `readWrite` sobre la base correcta (`bootcamp_api`) |
| `GET /api/products` → `[]` | Atlas vacío (`seed`/`mongorestore`) o `DB_NAME` distinto; aparece base `test` |
| `createIndex` falla con `E11000` | Ya había duplicados: límpialos antes |
| `MongoParseError: Invalid scheme` | `MONGODB_URI` vacía o con espacios/comillas raras |
| `you are over your space quota` | Llegaste a los 512 MB de M0 |
