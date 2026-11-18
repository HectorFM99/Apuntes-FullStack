# 03 — LA CADENA DE CONEXIÓN `mongodb+srv://` Y EL DRIVER

> La URI de Atlas parece la de Docker con un `+srv` de más. Ese `+srv` cambia
> bastante cómo se conecta el driver: la URI no apunta a **un servidor**, sino a
> un **nombre** del que el driver descubre los 3 servidores del replica set.

---

## 3.1 Local vs Atlas, lado a lado

### Ejemplo REAL (`express-mongo-atlas-render/.env.example`)

```
MONGODB_URI=mongodb://admin:TU_PASSWORD@localhost:27017/?authSource=admin
DB_NAME=bootcamp_api
```

### La forma REAL de tu `.env` contra Atlas (credenciales ocultas)

```
MONGODB_URI="mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net"
```

### La que te ofrece Atlas en *Connect → Drivers* (forma completa)

```
mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
```

### Desmontada

```
mongodb+srv:// <USUARIO> : <PASSWORD> @ cluster0.<ID>.mongodb.net / bootcamp_api ? retryWrites=true & w=majority & appName=Cluster0
└─protocolo──┘ └─usuario┘   └contraseña┘  └──── nombre DNS del cluster ┘ └─BD (opcional)┘ └────────────── opciones ──────────────────┘
```

| Parte | Docker (local) | Atlas |
|-------|----------------|-------|
| Protocolo | `mongodb://` | `mongodb+srv://` |
| Host | `localhost:27017` (un servidor, con puerto) | `cluster0.<ID>.mongodb.net` (un **nombre**, **sin** puerto) |
| Usuario | `admin` (root del contenedor) | Un *Database User* |
| `authSource` | `admin` (hay que ponerlo) | Lo indica el DNS (TXT) → no hace falta |
| TLS | No | **Sí**, activado automáticamente por `+srv` |
| Base en la URI | No (usas `DB_NAME`) | Opcional (tú usas `DB_NAME`) |

---

## 3.2 Qué hace el `+srv` por dentro

Con `mongodb+srv://`, el driver hace **consultas DNS** antes de conectar:

```
1. DNS SRV  _mongodb._tcp.cluster0.<ID>.mongodb.net
   → ac-xxx-shard-00-00.<ID>.mongodb.net:27017
   → ac-xxx-shard-00-01.<ID>.mongodb.net:27017      (los 3 nodos)
   → ac-xxx-shard-00-02.<ID>.mongodb.net:27017

2. DNS TXT  cluster0.<ID>.mongodb.net
   → authSource=admin&replicaSet=atlas-xxxx-shard-0  (opciones por defecto)

3. Conecta con TLS a los 3, descubre cuál es el PRIMARY, y manda ahí las escrituras.
```

Ventajas:

- La URI es **corta** y **no cambia** aunque Atlas sustituya un nodo por otro.
- `tls=true` va implícito: **no** puedes conectarte a Atlas sin cifrar.
- El driver conoce **todos** los nodos: si cambia el primary, se reconecta solo.

> ⚠️ Si tu red o tu DNS no resuelven registros SRV (algunas redes corporativas,
> VPNs o DNS antiguos), verás `querySrv ENOTFOUND` / `ECONNREFUSED`. Atlas también
> ofrece una URI "clásica" (`mongodb://` con los 3 hosts escritos) para esos casos
> (ver `06`).

---

## 3.3 Las opciones de la URI

| Opción | Valor típico | Qué hace |
|--------|--------------|----------|
| `retryWrites` | `true` | Reintenta **una vez** una escritura que falló por un problema transitorio (cambio de primary, corte de red). Por defecto ya es `true` en drivers modernos |
| `w` | `majority` | *Write concern*: la escritura se confirma cuando la tiene la **mayoría** de nodos. Evita perder datos si cae el primary justo después |
| `appName` | `Cluster0` / `api-products` | Nombre que aparece en los logs y métricas de Atlas: útil para saber **qué app** hace cada conexión |
| `readPreference` | `primary` (defecto) / `secondaryPreferred` | De qué nodo leer. Leer de secondaries reparte carga pero puedes leer datos **ligeramente atrasados** |
| `maxPoolSize` | `100` (defecto) | Máximo de conexiones del pool por servidor |
| `serverSelectionTimeoutMS` | `30000` (defecto) | Cuánto esperar a encontrar un servidor válido antes de fallar |

> 📌 Tu URI real no lleva opciones. Funciona porque `retryWrites=true` es el
> valor por defecto del driver y `authSource`/`replicaSet` vienen del DNS. Aun
> así, es buena idea añadir `appName=api-products` para reconocer tu API en el panel.

### Caracteres especiales en la contraseña

La contraseña va **dentro** de una URL. Si contiene `@ : / ? # [ ] %`, rompe el
formato:

```
mongodb+srv://api:p@ss#1@cluster0...      ❌ ¿dónde acaba la contraseña?
mongodb+srv://api:p%40ss%231@cluster0...  ✅ codificada
```

```js
encodeURIComponent('p@ss#1')   // 'p%40ss%231'
```

> 💡 Lo más sencillo: usa el botón *Autogenerate Secure Password* de Atlas, que
> genera contraseñas solo alfanuméricas.

---

## 3.4 Tu código de conexión

### Ejemplo REAL (`express-mongo-atlas-render/src/db.ts`)

```ts
import { Db, MongoClient } from "mongodb";

const uri = process.env.MONGODB_URI;
if (!uri) {
  throw new Error("Falta MONGODB_URI en el archivo .env");
}

const client = new MongoClient(uri);

let db: Db;

export async function connectDb() {
  await client.connect();
  db = client.db(process.env.DB_NAME ?? "bootcamp_api");

  // Reto 2: índice único para que no se repita el nombre de una categoría.
  // Si ya existe, no hace nada.
  await db.collection("categories").createIndex({ name: 1 }, { unique: true });

  console.log("Conectado a MongoDB");
}

export function getDb() {
  return db;
}
```

Leído en clave Atlas:

| Línea | Qué pasa con Atlas |
|-------|--------------------|
| `if (!uri) throw ...` | En Render, si olvidas la variable, el log lo dice claramente |
| `new MongoClient(uri)` | **Un solo cliente** para toda la app. Lleva dentro un **pool** de conexiones a los 3 nodos. Crear un cliente por petición agotaría el límite de conexiones del M0 |
| `await client.connect()` | Resuelve el SRV, abre TLS, autentica. Si la IP no está permitida o la contraseña es incorrecta, **falla aquí** |
| `client.db(DB_NAME ?? "bootcamp_api")` | No crea nada todavía: la base aparece en Atlas con el primer documento (o índice) |
| `createIndex(..., { unique: true })` | Se ejecuta en **cada arranque**. Es idempotente (si ya existe, no hace nada) y garantiza que Atlas tenga el índice aunque nunca ejecutaras `seed.js` allí |

> 📌 Gracias a ese `createIndex` en el arranque, el reto del 409 por nombre
> duplicado funciona en Atlas **sin pasos manuales**. Es una forma sencilla de
> "migración" de índices.

### Conectar antes de escuchar

### Ejemplo REAL (`express-mongo-atlas-render/src/index.ts`)

```ts
async function start() {
  await connectDb();

  app.listen(PORT, () => {
    console.log(`Servidor escuchando en http://localhost:${PORT}`);
  });
}

start();
```

- ✅ Si Atlas no es accesible, la API **no arranca** → Render marca el despliegue como fallido y mantiene la versión anterior.
- ⚠️ Pero tarda **30 s** en rendirse (el `serverSelectionTimeoutMS` por defecto) y en Render verás antes "No open ports detected". El error real (`MongoServerSelectionError`) está más arriba en el log.

### Mejoras posibles (NO están en tu proyecto)

```ts
const client = new MongoClient(uri, {
  appName: "api-products",          // identificar la app en Atlas
  serverSelectionTimeoutMS: 5000,   // fallar en 5 s en vez de 30
  maxPoolSize: 10,                  // de sobra para M0 y una instancia de Render
});

// Cerrar limpio cuando Render para el servicio (redeploy, sleep)
process.on("SIGTERM", async () => {
  await client.close();
  process.exit(0);
});
```

> 💡 El proyecto `api-rastro` de tus apuntes ya usa un timeout corto (ver
> `../MONGODB/06`, apartado 6.2, "Variante").

---

## 3.5 El pool de conexiones y los límites de M0

```
           Render (1 instancia)
   ┌──────────── MongoClient ───────────┐
   │  pool: hasta maxPoolSize por nodo  │
   └──┬───────────────┬──────────────┬──┘
      ▼               ▼              ▼
   PRIMARY        SECONDARY      SECONDARY     (+ conexiones de monitorización)
```

- El driver abre conexiones **bajo demanda** hasta `maxPoolSize` y las reutiliza.
- Además mantiene conexiones de **monitorización** con cada nodo (para saber quién es el primary).
- M0 tiene un límite de conexiones simultáneas (≈500). Con **un** `MongoClient` por instancia no te acercas. Lo que lo agota: crear un `MongoClient` en cada petición, o funciones *serverless* que no reutilizan el cliente.

> 📌 Es la misma lección que con `pg.Pool` en Supabase: **un único cliente/pool
> por aplicación**, creado al arrancar.

---

## ✅ Resumen

- `mongodb+srv://` = un **nombre DNS** que el driver resuelve en los 3 nodos; TLS implícito; `authSource` y `replicaSet` vienen del DNS.
- `retryWrites=true` y `w=majority` protegen frente a cambios de primary.
- Codifica caracteres especiales de la contraseña (o usa una autogenerada).
- Un solo `MongoClient`, conectar **antes** de `listen`, y mejor con `serverSelectionTimeoutMS` corto.
- Tu `createIndex` en el arranque deja Atlas preparado sin pasos manuales.
