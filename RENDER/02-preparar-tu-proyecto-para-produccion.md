# 02 — PREPARAR TU PROYECTO PARA PRODUCCIÓN

> El 90 % de los despliegues fallidos no son culpa de Render: son cosas que en tu
> ordenador funcionaban "por casualidad" (un `.env` que existe, el puerto 3000
> fijo, TypeScript ejecutado al vuelo...). Este archivo es la **lista de
> requisitos** que tu código tiene que cumplir **antes** de pulsar *Deploy*.

---

## 2.1 Desarrollo vs producción: dos formas de ejecutar el mismo código

| | Desarrollo (tu PC) | Producción (Render) |
|---|-------------------|---------------------|
| Cómo se ejecuta TS | Al vuelo (`tsx watch`, `nest start --watch`) | **Compilado** a JS (`dist/`) y ejecutado con `node` |
| Reinicio al guardar | Sí (*watch*) | No: cada cambio = nuevo despliegue |
| Variables de entorno | Archivo `.env` | Panel **Environment** de Render |
| Puerto | `3000` (el que tú quieras) | El que diga `process.env.PORT` |
| Base de datos | Docker en `localhost` | Supabase / Atlas en Internet |
| CORS | `http://localhost:5173` | La URL real de tu frontend |

### Ejemplo REAL (`express-mongo-atlas-render/package.json`, scripts)

```json
"scripts": {
  "dev": "tsx watch --env-file=.env src/index.ts",
  "build": "tsc",
  "start": "node dist/index.js",
  "typecheck": "tsc --noEmit",
  "lint": "eslint ."
}
```

- `dev` → **solo en tu PC**: ejecuta TS directamente, recarga al guardar y carga el `.env` con `--env-file`.
- `build` → **Build Command** en Render: `tsc` traduce `src/*.ts` → `dist/*.js` (lo dice `"outDir": "dist"` en `tsconfig.json`).
- `start` → **Start Command** en Render: ejecuta el JS ya compilado. **Rápido** y sin dependencias de desarrollo.

### Ejemplo REAL (`nestjs-postgresql-supabase-render/package.json`, scripts)

```json
"build": "nest build",
"start": "nest start",
"start:dev": "nest start --watch",
"start:prod": "node dist/main"
```

> 📌 En NestJS el equivalente es: `nest build` → genera `dist/main.js`, y
> `start:prod` lo ejecuta con `node`. **No** uses `start:dev` en Render: el modo
> *watch* consume mucha más memoria (el plan Free tiene 512 MB) y no tiene sentido
> en un servidor.

---

## 2.2 El puerto: `process.env.PORT`

Render decide en qué puerto debe escuchar tu app y te lo dice con la variable
`PORT` (por defecto `10000`). Si tu app escucha en otro, Render no la encuentra:

```
==> No open ports detected, continuing to scan...
==> Port scan timeout reached, no open ports detected.
```

### Ejemplo REAL (`express-mongo-atlas-render/src/index.ts`)

```ts
// Render nos dice el puerto en la variable PORT. En tu ordenador no existe: usamos 3000.
const PORT = process.env.PORT ?? 3000;
...
app.listen(PORT, () => {
  console.log(`Servidor escuchando en http://localhost:${PORT}`);
});
```

### Ejemplo REAL (`nestjs-postgresql-supabase-render/src/main.ts`)

```ts
await app.listen(process.env.PORT ?? 3000);
```

- `??` (*nullish coalescing*): si `process.env.PORT` es `undefined` (tu PC), usa `3000`.
- Ambos escuchan en **todas las interfaces** (`0.0.0.0`) por defecto, que es lo que Render necesita.

> ⚠️ Si alguna vez escribes `app.listen(PORT, 'localhost')` o `'127.0.0.1'`, la app
> solo aceptaría conexiones **desde dentro del propio contenedor** y Render no
> podría llegar a ella. Deja el host sin poner o usa `'0.0.0.0'`.

---

## 2.3 Variables de entorno: el `.env` NO viaja a Render

El `.env` contiene secretos, por eso está en `.gitignore`:

### Ejemplo REAL (`nestjs-postgresql-supabase-render/.gitignore`)

```
node_modules/
dist/
coverage/
.env
*.log
```

Consecuencia: **en Render no existe el archivo `.env`**. Las variables se escriben
en el panel *Environment* y Render las inyecta directamente en `process.env`
antes de arrancar tu proceso.

### El problema: código que **exige** que exista `.env`

`process.loadEnvFile()` (Node ≥ 20.12) lee un `.env`... y **lanza un error** si
el archivo no existe. En Render, la app se caería al arrancar.

### Ejemplo REAL — la solución (`nestjs-postgresql-supabase-render/src/main.ts`)

```ts
import { existsSync } from 'node:fs';
...
// En tu ordenador, lee el .env. En Render no hay .env: las variables las pone Render.
if (existsSync('.env')) process.loadEnvFile();
```

> 📌 Mismo código, dos entornos: en tu PC carga el `.env`; en Render se lo salta
> y usa las variables del panel. **Esta línea es la que hace desplegable el proyecto.**

En el proyecto de Express el problema ni aparece, porque el `.env` solo se carga
en el script `dev` (`tsx watch --env-file=.env`) y `start` no lo menciona.

### Fallar rápido si falta una variable

### Ejemplo REAL (`express-mongo-atlas-render/src/db.ts`)

```ts
const uri = process.env.MONGODB_URI;
if (!uri) {
  throw new Error("Falta MONGODB_URI en el archivo .env");
}
```

> 💡 Es una buena práctica: si olvidas poner la variable en Render, el log dirá
> exactamente **qué falta** en vez de un error críptico 30 segundos después.
> El proyecto de NestJS **no** lo hace: si falta `DATABASE_URL`, `pg` intentará
> conectarse a `localhost:5432` y fallará con `ECONNREFUSED` en la primera
> consulta. Mejora posible:

```ts
// database.module.ts — versión defensiva (NO está en tu proyecto)
function createPool() {
  if (!process.env.DATABASE_URL) throw new Error('Falta DATABASE_URL');
  const pool = new Pool({ connectionString: process.env.DATABASE_URL });
  pool.on('error', (error) => console.error('PostgreSQL:', error.message));
  return pool;
}
```

### `.env.example`: el contrato de variables

### Ejemplo REAL (`nestjs-postgresql-supabase-render/.env.example`)

```
DATABASE_URL=postgres://alumno:TU_PASSWORD@localhost:5432/bootcamp_api
```

### Ejemplo REAL (`express-mongo-atlas-render/.env.example`)

```
MONGODB_URI=mongodb://admin:TU_PASSWORD@localhost:27017/?authSource=admin
DB_NAME=bootcamp_api
```

> 📌 El `.env.example` **sí** se sube a Git. Es la lista de variables que tendrás
> que crear en Render (con los valores **de producción**, no los de localhost).

---

## 2.4 `dependencies` vs `devDependencies` en el build

- `dependencies`: lo que la app necesita **para ejecutarse** (`express`, `pg`, `@nestjs/core`...).
- `devDependencies`: lo que necesitas **para desarrollar o compilar** (`typescript`, `@nestjs/cli`, `tsx`, `eslint`, `@types/*`).

⚠️ **Trampa**: `npm run build` necesita `typescript` / `@nestjs/cli`, que son
`devDependencies`. Por eso el Build Command es `npm install && npm run build`:

- `npm install` (sin más) instala **también** las `devDependencies` → el build funciona ✅.
- Si añades en Render la variable `NODE_ENV=production`, `npm install` **omite** las
  `devDependencies` → `nest: not found` / `tsc: not found` ❌.

> 📌 Regla práctica: no pongas `NODE_ENV=production` hasta que entiendas este
> efecto, o usa `npm install --include=dev && npm run build`.

---

## 2.5 CORS: el frontend de producción tiene otra URL

### Ejemplo REAL (`nestjs-postgresql-supabase-render/src/main.ts`)

```ts
app.enableCors({ origin: 'http://localhost:5173' });
```

### Ejemplo REAL (`express-mongo-atlas-render/src/index.ts`)

```ts
res.setHeader("Access-Control-Allow-Origin", "http://127.0.0.1:5500");
```

Desplegada en Render, la API funciona perfectamente con `curl`, Postman o el
navegador abriendo la URL directamente... **pero** si tu frontend está en
`https://mi-web.onrender.com`, el navegador bloqueará las peticiones (el origen
permitido sigue siendo `localhost`).

Solución habitual: sacar el origen a una variable de entorno.

```ts
// main.ts — mejora (NO está en tu proyecto)
app.enableCors({
  origin: process.env.CORS_ORIGIN ?? 'http://localhost:5173',
});
```

> 📌 CORS lo aplica **el navegador**, no el servidor. Por eso `curl` nunca da
> errores de CORS y el navegador sí. (Ver `API/06-autenticacion-cors-y-seguridad.md`.)

---

## 2.6 Versión de Node

Render usa una versión de Node por defecto que puede **no** ser la tuya. Tus
proyectos dependen de cosas modernas:

- `process.loadEnvFile()` → Node **≥ 20.12**.
- El README del proyecto Express pide **Node 24 LTS**.

Formas de fijarla (de más a menos recomendada):

```json
// package.json
"engines": { "node": ">=22" }
```
```
# archivo .node-version en la raíz del servicio
22
```
```
# o variable de entorno en Render
NODE_VERSION=22
```

> 💡 En la primera línea del log de build Render dice qué versión está usando:
> `==> Using Node.js version 22.x.x`. Si no coincide con la tuya, empieza por ahí.

---

## 2.7 La conexión a la base de datos debe apuntar a Internet

En tu PC: `localhost:5432` (Docker). En Render, `localhost` es **el propio
contenedor de Render**, donde no hay ninguna base de datos.

```
TU PC                                      RENDER
API ──▶ localhost:5432 ──▶ Docker ✅      API ──▶ localhost:5432 ──▶ (nada) ❌ ECONNREFUSED
                                          API ──▶ aws-0-eu-central-1.pooler.supabase.com ✅
```

Por eso en Render la variable `DATABASE_URL` lleva la cadena de **Supabase**
(ver `Supabase/02-...`) y `MONGODB_URI` la de **MongoDB Atlas**.

---

## ✅ Checklist antes de desplegar

- [ ] `npm run build` funciona en tu PC **sin errores de TypeScript**.
- [ ] `npm start` / `npm run start:prod` arranca la versión compilada.
- [ ] Escuchas en `process.env.PORT ?? 3000`, sin fijar el host a `localhost`.
- [ ] El código **no exige** que exista un `.env`.
- [ ] Sabes qué variables necesitas (mira tu `.env.example`) y tienes sus valores **de producción**.
- [ ] La BD de producción es accesible desde Internet (Supabase / Atlas con *Network Access* abierto).
- [ ] El origen de CORS sirve para tu frontend real (o está en una variable).
- [ ] La versión de Node está fijada.
- [ ] El código está **subido a GitHub** (Render no ve tu disco).
