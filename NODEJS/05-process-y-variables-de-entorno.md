# 05 — `process` Y VARIABLES DE ENTORNO (`.env`)

> **`process`** es un objeto global de Node que representa **el programa que se está
> ejecutando**: desde él lees variables de entorno, argumentos de la terminal, la
> carpeta actual, o terminas el programa.
> Las **variables de entorno** sirven para sacar del código la **configuración** y los
> **secretos** (contraseñas, claves de API, URLs de bases de datos).

---

## 5.1 El objeto `process`

No hace falta importarlo: es **global**.

| Propiedad / método | Qué devuelve | Ejemplo |
|--------------------|--------------|---------|
| `process.env` | Objeto con las **variables de entorno** | `process.env.PORT` → `"3000"` |
| `process.cwd()` | Carpeta **desde la que** ejecutaste Node | `C:\...\ProyectoReproductor` |
| `process.argv` | Array con los **argumentos** de la terminal | `["node", "script.js", "Hector"]` |
| `process.exit(código)` | Termina el programa (`0` = bien, `1` = error) | `process.exit(1)` |
| `process.platform` | Sistema operativo | `"win32"` |
| `process.version` | Versión de Node | `"v22.11.0"` |
| `process.on("evento", fn)` | Escuchar eventos del proceso | `"uncaughtException"`, `"SIGINT"` |

### `process.argv` (ejemplo propio)

```js
// 📄 saludar.js
const nombre = process.argv[2] ?? "desconocido";
console.log(`Hola, ${nombre}`);
```

```bash
node saludar.js Hector
# Hola, Hector
```

```
process.argv = [
  "C:\\Program Files\\nodejs\\node.exe",   // [0] el ejecutable de node
  "C:\\...\\saludar.js",                   // [1] el script
  "Hector"                                 // [2] ← tu primer argumento
]
```

### `process.cwd()` — Ejemplo REAL (`ProyectoReproductor/server/index.ts`)

```ts
const dbPath = join(process.cwd(), relativePath);
// ...
app.get('*',(_req,res)=>res.sendFile(join(process.cwd(),'public','index.html')));
```

> 📌 `process.cwd()` = "la carpeta donde estaba la terminal cuando lanzaste el
> comando". Como siempre lanzas `npm run dev` desde la raíz del proyecto, las rutas
> funcionan. Si lo lanzaras desde otra carpeta, cambiarían.

---

## 5.2 ¿Qué es una variable de entorno?

Son **pares clave=valor** que viven **fuera** de tu código, en el sistema o en la
terminal, y que Node lee en `process.env`.

```bash
# PowerShell (Windows)
$env:PORT = "4000"; node index.js

# Git Bash / Mac / Linux
PORT=4000 node index.js
```

```js
console.log(process.env.PORT);   // "4000"
```

### ¿Por qué no escribirlo directamente en el código?

```js
// ❌ El secreto acaba en GitHub para siempre (aunque luego lo borres, queda en el historial)
const client = new MongoClient("mongodb://admin:MiContraseñaReal@servidor:27017");

// ✅ El código no conoce el secreto: lo lee del entorno
const client = new MongoClient(process.env.MONGODB_URI);
```

**Ventajas:**
1. 🔒 **Seguridad**: los secretos no se suben al repositorio.
2. 🔁 **Mismo código, distinta configuración**: en tu PC usas la BD local; en el
   servidor, la de producción. **No cambias ni una línea**.
3. 👥 Cada compañero puede tener sus propias credenciales.

> ⚠️ **Todo** lo que hay en `process.env` es **texto (string)** o `undefined`.
> `process.env.PORT` es `"3000"`, no `3000`.

---

## 5.3 El archivo `.env`

Escribir variables en la terminal cada vez es incómodo. Lo habitual es ponerlas en un
archivo **`.env`** en la raíz del proyecto:

```bash
# .env  (ejemplo propio, valores inventados)
PORT=3000
MONGODB_URI=mongodb://usuario:contraseña@localhost:27017/?authSource=admin
DB_NAME=rastro
```

**Reglas del formato:**
- Una variable por línea: `CLAVE=valor`
- Sin espacios alrededor del `=`
- Sin comillas (normalmente no hacen falta)
- `#` para comentarios
- Nombres en `MAYÚSCULAS_CON_GUIONES_BAJOS`

### 🔒 `.env` NUNCA va a Git

**Ejemplo REAL** — todos tus proyectos con `.env` lo ignoran:

```
# api-rastro/api-rastro/.gitignore
node_modules/
.env
dist
```

```
# mongo-api-libros/02b-express-mongodb/02b-express-mongodb/.gitignore
node_modules
dist
.env
```

---

## 5.4 `.env.example`: la plantilla que SÍ se sube

Si `.env` no está en Git, ¿cómo sabe un compañero qué variables necesita? Con un
**`.env.example`**: mismas claves, **sin valores reales**.

### Ejemplo REAL copiado (`ProyectoReproductor/.env.example`)

```
JAMENDO_CLIENT_ID=
SESSION_SECRET=
PORT=3000
DATABASE_URL=file:./data/nightjar.sqlite
```

### Ejemplo REAL copiado (`api-rastro/api-rastro/.env.example`)

```
MONGODB_URI=mongodb://usuario:contraseña@localhost:27017/?authSource=admin
DB_NAME=rastro
```

> 📌 Flujo al clonar un proyecto:
> ```bash
> npm install
> cp .env.example .env      # copiar la plantilla
> # editar .env y rellenar los valores reales
> npm run dev
> ```

| | `.env` | `.env.example` |
|---|--------|----------------|
| Valores | **Reales** (secretos) | Vacíos o de ejemplo |
| ¿A Git? | ❌ Nunca | ✅ Sí |
| ¿Lo lee Node? | ✅ Sí | ❌ No, es documentación |

---

## 5.5 Cargar el `.env`: tres formas

Node **no** lee el `.env` automáticamente. Hay que pedírselo.

### Forma 1: paquete `dotenv` (la clásica)

```bash
npm install dotenv
```

**Ejemplo REAL** (`ProyectoReproductor/server/index.ts`, **primera línea**):

```ts
import 'dotenv/config';
import express from 'express';
// ...
```

```js
// Equivalente en CommonJS:
require("dotenv").config();
```

> ⚠️ Tiene que ir **lo primero**. Si otro archivo lee `process.env` antes de que
> `dotenv` cargue, encontrará `undefined`.

### Forma 2: flag `--env-file` (Node 20.6+, sin instalar nada)

**Ejemplo REAL** (`api-rastro/api-rastro/package.json`):

```json
"dev": "tsx watch --env-file=.env src/index.ts",
```

Y el mismo en `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/package.json`:

```json
"dev": "tsx watch --env-file=.env src/index.ts",
```

> 📌 Por eso `api-rastro` y `mongo-api-libros` **no** tienen `dotenv` en sus
> dependencias: el propio Node carga el archivo al arrancar. Con `node` directamente
> sería: `node --env-file=.env dist/index.js`.

### Forma 3: `process.loadEnvFile()` (Node 21.7+, ejemplo propio)

```js
process.loadEnvFile();          // carga ./.env
console.log(process.env.PORT);
```

| Forma | Necesita instalar | Dónde se pone | Usada en |
|-------|-------------------|---------------|----------|
| `import 'dotenv/config'` | `dotenv` | Primera línea del código | `ProyectoReproductor` |
| `--env-file=.env` | Nada (Node ≥ 20.6) | Script de `package.json` | `api-rastro`, `mongo-api-libros` |
| `process.loadEnvFile()` | Nada (Node ≥ 21.7) | Primera línea del código | — |

---

## 5.6 Leer variables con valor por defecto: `??` y `||`

Como una variable puede **no existir**, conviene dar un valor por defecto.

### Ejemplo REAL (`ProyectoReproductor/server/index.ts`)

```ts
const dbUrl = process.env.DATABASE_URL ?? 'file:./data/nightjar.sqlite';
// ...
const port=Number(process.env.PORT||3000);app.listen(port,()=>console.log(`Nightjar disponible en http://localhost:${port}`));
```

### Ejemplo REAL (`mongo-api-libros/02b-express-mongodb/02b-express-mongodb/src/db.ts`)

```ts
const uri = process.env.DATABASE_URL || process.env.MONGODB_URI;
// ...
  db = client.db(process.env.DB_NAME ?? "bootcamp_api");
```

| Operador | Usa el valor por defecto cuando la variable es... |
|----------|--------------------------------------------------|
| `??` | `undefined` o `null` (no existe) |
| `\|\|` | Cualquier valor *falsy*: `undefined`, `null`, `""` (vacía), `0`... |

```js
// En .env:  PORT=
process.env.PORT ?? 3000     // ""   ← la variable EXISTE pero vacía → no usa el defecto
process.env.PORT || 3000     // 3000 ← "" es falsy → usa el defecto
```

> 📌 Para variables de entorno, `||` suele ser más seguro, porque en el `.env` es
> fácil dejar una clave **vacía** (como `JAMENDO_CLIENT_ID=` en `.env.example`).

### Convertir tipos

```ts
// Ejemplo REAL: el puerto llega como string → se convierte a número
const port = Number(process.env.PORT || 3000);

// Booleanos (ejemplo propio): "true"/"false" son strings
const debug = process.env.DEBUG === "true";
```

---

## 5.7 Validar variables obligatorias al arrancar

Si falta una variable **imprescindible**, lo mejor es **fallar en seguida** con un
mensaje claro, en vez de tener un error raro más tarde.

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/db.ts`)

```ts
const uri = process.env.MONGODB_URI;
if (!uri) {
  throw new Error("Falta MONGODB_URI en el archivo .env");
}

const client = new MongoClient(uri, { serverSelectionTimeoutMS: 5000 });
```

### Ejemplo REAL copiado (`mongo-api-libros/02b-express-mongodb/02b-express-mongodb/src/db.ts`)

```ts
const uri = process.env.DATABASE_URL || process.env.MONGODB_URI;
if (!uri) {
  throw new Error("Falta DATABASE_URL o MONGODB_URI en el archivo .env");
}
```

> 📌 Además de avisar, el `if (!uri)` le dice a TypeScript que a partir de ahí `uri`
> es `string` y no `string | undefined`.

### Ejemplo REAL: variable opcional que desactiva una función

En `ProyectoReproductor`, si no hay clave de la API de música, el servidor sigue
funcionando pero esa ruta responde con un error controlado:

```ts
if(!process.env.JAMENDO_CLIENT_ID)return res.status(503).json({error:'Falta configurar el acceso al catálogo de Jamendo.'});
```

---

## 5.8 `NODE_ENV`: desarrollo vs. producción

Convención: `NODE_ENV` vale `"development"` o `"production"`.

### Ejemplo REAL (`ProyectoReproductor/server/index.ts`)

```ts
res.cookie('nightjar_session',token,{httpOnly:true,sameSite:'lax',secure:process.env.NODE_ENV==='production',maxAge:1209600000});
```

> 📌 `secure: true` obliga a usar HTTPS. En tu PC (`http://localhost`) no hay HTTPS,
> así que solo se activa **en producción**.

---

## 5.9 `process.exit` y eventos del proceso (ejemplos propios)

```js
// Terminar con error si falta algo
if (!process.env.MONGODB_URI) {
  console.error("Falta MONGODB_URI");
  process.exit(1);          // 1 = terminó con error
}

// Hacer limpieza al pulsar Ctrl + C
process.on("SIGINT", async () => {
  console.log("Cerrando servidor...");
  await client.close();     // cerrar conexión a la BD
  process.exit(0);          // 0 = todo bien
});
```

> 📌 `throw new Error(...)` al arrancar (como en tus `db.ts`) tiene un efecto parecido:
> si nadie lo captura, Node imprime el error y termina con código `1`.

---

## 5.10 Errores comunes

| Problema | Causa | Solución |
|----------|-------|----------|
| `process.env.X` es `undefined` | No se ha cargado el `.env` | `import 'dotenv/config'` o `--env-file=.env` |
| Sigue siendo `undefined` | El `.env` no está en la carpeta desde la que ejecutas | Ejecuta desde la raíz del proyecto |
| `dotenv` carga tarde | El `import 'dotenv/config'` no es la primera línea | Muévelo arriba del todo |
| `"3000" + 1 = "30001"` | Las variables son **strings** | `Number(process.env.PORT)` |
| Subiste el `.env` a GitHub | Faltaba en `.gitignore` | **Cambia las contraseñas** (ya son públicas), añade `.env` al `.gitignore` y `git rm --cached .env` |
| Cambiaste el `.env` y no se nota | El proceso ya estaba arrancado | Reinicia el servidor (el `.env` se lee **al arrancar**; el modo *watch* no siempre lo vigila) |

---

## ✅ Resumen rápido

1. `process` es global: `process.env`, `process.cwd()`, `process.argv`, `process.exit()`.
2. Configuración y secretos en **variables de entorno**, nunca en el código.
3. `.env` con los valores reales → **en `.gitignore`**; `.env.example` como plantilla → **sí a Git**.
4. Cargar el `.env`: `import 'dotenv/config'` (primera línea) o `--env-file=.env`.
5. Siempre son **strings**: convierte con `Number(...)` o compara con `=== "true"`.
6. Valores por defecto con `??` o `||`; variables obligatorias → `throw new Error(...)` al arrancar.

👉 **Siguiente:** `06-asincronia-callbacks-promesas-async-await.md`
