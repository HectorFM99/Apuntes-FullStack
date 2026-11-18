# 06 — ERRORES COMUNES Y DIAGNÓSTICO

> Con Atlas, casi todos los errores caen en uno de cuatro grupos: **red** (la IP
> no está permitida), **autenticación** (usuario/contraseña/rol), **DNS** (el
> `+srv` no se resuelve) o **datos** (la API conecta pero no ve lo que esperas).

---

## 6.1 El método: ¿cuánto tarda en fallar?

```
¿Falla al instante con "bad auth"?           → AUTENTICACIÓN  → 6.3
¿Tarda ~30 s y da MongoServerSelectionError? → RED (IP)       → 6.2
¿Falla al instante con querySrv / ENOTFOUND? → DNS / URI      → 6.4
¿Conecta, pero devuelve [] o faltan datos?   → DATOS / DB_NAME→ 6.5
¿Conecta, pero "not authorized on ..."?      → ROL            → 6.3
```

> 💡 El **tiempo** es la mejor pista: un rechazo de contraseña es inmediato
> (el servidor te contesta "no"); un bloqueo de IP es silencioso (el cortafuegos
> no contesta nada y el driver espera los 30 s de `serverSelectionTimeoutMS`).

---

## 6.2 Red: `MongoServerSelectionError` tras ~30 segundos

```
MongoServerSelectionError: connection <monitor> to 18.x.x.x:27017 closed
    ...
  reason: TopologyDescription { type: 'ReplicaSetNoPrimary', servers: Map(3) {...} }
```

o bien `Server selection timed out after 30000 ms`.

| Causa | Cómo comprobarlo | Solución |
|-------|------------------|----------|
| Tu IP (local) no está en la lista | Atlas → Network Access avisa "current IP not added" | *Add Current IP Address* |
| Tu IP de casa ha cambiado | Ayer funcionaba, hoy no | Añadir la nueva IP |
| Render no está permitido | Solo falla desplegado | `0.0.0.0/0` o las *Outbound IPs* de Render |
| Red que bloquea el puerto 27017 (universidad, empresa, algunas wifis públicas) | Falla en esa red, funciona con el móvil compartiendo datos | Otra red / VPN |
| Cluster **pausado** | Atlas muestra el cluster como *Paused* | *Resume* |

> 📌 En Render, este error impide que tu `app.listen` se ejecute (porque
> `await connectDb()` va antes) y el log mostrará también "No open ports
> detected". El que importa es el `MongoServerSelectionError` de más arriba.

---

## 6.3 Autenticación y permisos

```
MongoServerError: bad auth : authentication failed      (code 8000)
```

| Causa | Solución |
|-------|----------|
| Contraseña incorrecta | Restablécela en *Database Access → Edit → Edit Password* |
| Usas tu **cuenta de Atlas** (email) en vez de un Database User | Crea un Database User (archivo `02`, 2.2) |
| Contraseña con `@ # / : %` sin codificar | `encodeURIComponent` o contraseña autogenerada |
| Dejaste `<db_password>` literal de la URI de ejemplo | Sustitúyelo, **sin** los `< >` |
| Comillas incluidas en el valor en Render | En el panel de Render el valor va **sin** comillas (en el `.env` local sí se toleran) |

```
MongoServerError: user is not allowed to do action [insert] on [bootcamp_api.products]   (code 13)
```

El usuario conecta pero su **rol** no permite esa operación:

- Rol `read` cuando necesitas `readWrite`.
- Rol `readWrite` sobre **otra** base (ej. `test`) mientras tu `DB_NAME` es `bootcamp_api`.

---

## 6.4 DNS y formato de la URI

```
Error: querySrv ENOTFOUND _mongodb._tcp.cluster0.xxxxx.mongodb.net
Error: querySrv ECONNREFUSED _mongodb._tcp.cluster0.xxxxx.mongodb.net
```

| Causa | Solución |
|-------|----------|
| Errata en el nombre del cluster | Copia la URI otra vez desde *Connect* |
| Tu DNS (router, VPN, red corporativa) no resuelve registros SRV | Cambia el DNS del equipo a uno público (ej. `8.8.8.8` / `1.1.1.1`) o usa la URI **sin** `+srv` que ofrece Atlas (con los 3 hosts) |
| Pusiste puerto en una URI `+srv` (`...mongodb.net:27017`) | Con `+srv` **no** se pone puerto |

```
MongoParseError: Invalid scheme, expected connection string to start with "mongodb://" or "mongodb+srv://"
```

→ La variable está vacía o tiene espacios/comillas raras. Comprueba que
`process.env.MONGODB_URI` llega bien (tu `db.ts` ya avisa si falta).

---

## 6.5 Conecta, pero los datos no son los esperados

| Síntoma | Causa | Solución |
|---------|-------|----------|
| `GET /api/products` → `[]` | Atlas está vacío | Cargar `seed.js` o `mongorestore` (archivo `04`) |
| Los datos están en Atlas pero la API no los ve | La API usa otra base: `DB_NAME` distinto, o los insertaste en `test` | Data Explorer → mira en qué base están; alinea `DB_NAME` |
| En Data Explorer aparece una base `test` inesperada | Insertaste sin elegir base (la URI no la lleva y usaste `client.db()` sin nombre) | Tu código siempre usa `DB_NAME`; revisa scripts sueltos |
| El 409 por categoría duplicada no salta | No existe el índice único en Atlas | Tu `connectDb()` lo crea al arrancar; si falló, mira el log. Si ya había duplicados, `createIndex` **falla** con `E11000` → limpia duplicados primero |
| Las búsquedas van lentas | Faltan índices | `explain()` y Performance Advisor (archivo `04`, 4.5) |

---

## 6.6 Otros errores

| Error | Significado |
|-------|-------------|
| `MongoNetworkError: ... ECONNRESET` puntual | Corte de red o cambio de primary; `retryWrites` reintenta las escrituras. Si es constante, revisa la red |
| `MongoServerError: you are over your space quota` | Llegaste a los 512 MB de M0 |
| `MongoServerError: Document failed validation` | Has activado `$jsonSchema` y el documento no cumple (archivo `04`, 4.6) |
| `Transaction numbers are only allowed on a replica set member` | Usas transacciones contra tu Docker standalone (en Atlas funcionan) |
| `E11000 duplicate key error` | Índice único: en tu API se traduce a **409** (es esperado, no un fallo) |

---

## 6.7 Herramienta de diagnóstico rápida

Un script mínimo que aísla el problema de tu API:

```js
// probar-atlas.mjs  →  node --env-file=.env probar-atlas.mjs
import { MongoClient } from "mongodb";

const client = new MongoClient(process.env.MONGODB_URI, { serverSelectionTimeoutMS: 5000 });
try {
  await client.connect();
  const db = client.db(process.env.DB_NAME ?? "bootcamp_api");
  console.log("✅ Conectado");
  console.log("Ping:", await db.command({ ping: 1 }));
  console.log("Colecciones:", (await db.listCollections().toArray()).map((c) => c.name));
} catch (err) {
  console.error("❌", err.name, "-", err.message);
} finally {
  await client.close();
}
```

| Salida | Grupo |
|--------|-------|
| `MongoServerSelectionError` en 5 s | Red |
| `MongoServerError bad auth` | Autenticación |
| `querySrv ENOTFOUND` | DNS / URI |
| `✅ Conectado` + `Colecciones: []` | Datos: base vacía o `DB_NAME` equivocado |

---

## ✅ Resumen

1. **¿Cuánto tarda en fallar?** Instantáneo = credenciales/DNS; 30 s = red.
2. Red → *Network Access*. Credenciales → *Database Access*. DNS → copiar URI / cambiar DNS.
3. Si conecta pero no ves datos → `DB_NAME` y Data Explorer.
4. Prueba la URI aislada antes de culpar a la API.
