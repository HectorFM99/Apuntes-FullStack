# 📋 CHEATSHEET — RENDER

> Chuleta rápida de comandos y código. Teoría en los documentos 01–06 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal

| Comando | Qué hace |
|---------|----------|
| `npm install && npm run build && npm start` | Reproducir **en local** lo que hará Render (Express) |
| `npm install && npm run build && npm run start:prod` | Lo mismo en NestJS |
| `git add . && git commit -m "feat: ..." && git push` | Desplegar (Auto-Deploy On Commit) |
| `curl -i https://<api>.onrender.com/api` | Comprobar que responde (y ver cabeceras/status) |
| `curl -i https://<api>.onrender.com/api/books` | Probar un endpoint con datos de la BD |
| `curl -i -X POST https://<api>.onrender.com/api/categories -H "Content-Type: application/json" -d '{"name":"cocina"}'` | Probar un POST |
| `node -e "console.log(require('crypto').randomBytes(48).toString('base64url'))"` | Generar un `JWT_SECRET` seguro |
| `npm install --include=dev && npm run build` | Build aunque haya `NODE_ENV=production` |

> 💡 Regla de oro: si `build` + `start` falla en tu PC, fallará en Render.

---

## 2. 📦 Scripts de `package.json`

```json
// express-mongo-atlas-render
"dev": "tsx watch --env-file=.env src/index.ts",   // solo dev carga el .env
"build": "tsc",                                    // TS → dist/
"start": "node dist/index.js"                      // lo que ejecuta Render
```

```json
// nestjs-postgresql-supabase-render
"build": "nest build",
"start:dev": "nest start --watch",                 // ❌ nunca en producción
"start:prod": "node dist/main"
```

```json
"engines": { "node": ">=22" }                      // fijar versión de Node
```

| Momento | Comando | Se ejecuta | Necesita |
|---------|---------|-----------|----------|
| **Build** | `npm install && npm run build` | 1 vez por despliegue | `devDependencies` (`typescript`, `@nestjs/cli`) |
| **Start** | `npm start` / `npm run start:prod` | En cada arranque | Solo `dependencies` |

---

## 3. 🧩 Código "consciente del entorno"

```ts
// Puerto: Render lo pone en PORT
const PORT = process.env.PORT ?? 3000;              // Express
await app.listen(process.env.PORT ?? 3000);         // NestJS
```

```ts
// .env solo si existe (NestJS main.ts)
import { existsSync } from 'node:fs';
if (existsSync('.env')) process.loadEnvFile();      // Node ≥ 20.12
```

```ts
// Fallar rápido si falta una variable (express db.ts)
const uri = process.env.MONGODB_URI;
if (!uri) throw new Error("Falta MONGODB_URI en el archivo .env");
```

```ts
// CORS configurable (mejora)
app.enableCors({ origin: process.env.CORS_ORIGIN ?? 'http://localhost:5173' });          // Nest
res.setHeader("Access-Control-Allow-Origin", process.env.CORS_ORIGIN ?? "http://localhost:5173"); // Express
```

```ts
// Que un aviso de Postgres no tumbe la API
pool.on('error', (error) => console.error('PostgreSQL:', error.message));
```

```ts
// Detectar Render (variables que pone Render)
if (process.env.RENDER) console.log('Desplegado en', process.env.RENDER_EXTERNAL_URL);
```

```ts
// Health check con BD (mejora, NestJS)
@Controller('health')
export class HealthController {
  constructor(private readonly pool: Pool) {}
  @Get() async check() { await this.pool.query('SELECT 1'); return { status: 'ok' }; }
}
```

---

## 4. 🖱️ Crear el Web Service (New → Web Service)

| Campo | NestJS + Supabase | Express + Atlas |
|-------|-------------------|-----------------|
| Name | `api-books-nest` | `api-products` |
| Runtime | Node | Node |
| Branch | `main` | `main` |
| Region | Frankfurt (junto a la BD) | La más cercana al cluster |
| Root Directory | `postgresql-render-atlas/nestjs-postgresql-supabase-render` | `mongo-atlas-render/express-mongo-atlas-render` |
| Build Command | `npm install && npm run build` | `npm install && npm run build` |
| Start Command | `npm run start:prod` | `npm start` |
| Instance Type | Free | Free |
| Variables | `DATABASE_URL` (Session pooler + `?sslmode=no-verify`) | `MONGODB_URI`, `DB_NAME=bootcamp_api` |
| Health Check Path | `/api` | `/` |

> 📌 `PORT` **no** hace falta crearla: Render la pone sola.

### Variables de entorno

```
DATABASE_URL=postgresql://postgres.<REF>:<PASSWORD>@aws-0-eu-central-1.pooler.supabase.com:5432/postgres?sslmode=no-verify
MONGODB_URI=mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net
DB_NAME=bootcamp_api
CORS_ORIGIN=https://<tu-front>.onrender.com
NODE_VERSION=22
JWT_SECRET=<generado con crypto>
```

| Dónde | Para qué |
|-------|----------|
| **Environment** | Variables del servicio (sin comillas) |
| **Environment Groups** | Variables compartidas entre servicios |
| **Secret Files** | Archivos secretos (certificados, `.env` completo) |
| `.env.example` en Git | Contrato: lista de variables a crear |

---

## 5. 🔁 Despliegues

| Acción | Cuándo |
|--------|--------|
| Auto-Deploy *On Commit* | Cada `git push` a la rama |
| *Deploy latest commit* | Forzar un redeploy |
| *Clear build cache & deploy* | Errores raros de dependencias |
| *Restart service* | Reiniciar sin reconstruir |
| **Rollback** (Events) | Volver a un despliegue que funcionaba |
| Cambiar una variable | Provoca redeploy (se leen al arrancar) |

### Blueprint (`render.yaml` en la raíz del repo)

```yaml
services:
  - type: web
    name: api-books-nest
    runtime: node
    region: frankfurt
    plan: free
    rootDir: postgresql-render-atlas/nestjs-postgresql-supabase-render
    buildCommand: npm install && npm run build
    startCommand: npm run start:prod
    healthCheckPath: /api
    envVars:
      - key: NODE_VERSION
        value: "22"
      - key: DATABASE_URL
        sync: false          # secreto: se rellena en el panel, no en Git
```

### Static Site (frontend Vite)

| Campo | Valor |
|-------|-------|
| Build Command | `npm install && npm run build` |
| Publish Directory | `dist` |
| Variables | `VITE_API_URL=https://<api>.onrender.com` (⚠️ pública) |
| Rewrite (SPA) | `/*` → `/index.html` |

---

## 6. 📜 Logs esperados

```
==> Using Node.js version 22.x.x          ← comprueba la versión
==> Running build command 'npm install && npm run build'...
==> Build successful 🎉
==> Running 'npm start'
> node dist/index.js
Conectado a MongoDB
Servidor escuchando en http://localhost:10000   ← normal: Render publica en https://<api>.onrender.com
==> Your service is live 🎉
```

---

## 7. 💤 Plan Free

| Característica | Valor |
|----------------|-------|
| RAM / CPU | 512 MB / 0,1 CPU |
| Horas | ~750 h/mes por workspace |
| Suspensión | Tras **15 min** sin peticiones |
| Cold start | ~30–60 s la primera petición |
| Disco persistente / Shell | ❌ / ❌ |
| Postgres de Render | Expira a los 30 días (→ Supabase) |
| Static Sites | No se duermen |

> 📌 Antes de una demo: abre la URL de la API **un minuto antes**.

---

## 8. 🩺 Método de diagnóstico

```
"Build failed"                   → BUILD   (Root Directory, devDeps, tipos, mayúsculas)
Build OK + "Exited with status 1"→ START   (dist/, .env, variables, Node)
"No open ports detected"         → PUERTO  (PORT fijo, host, o la app murió antes de listen)
Live pero 500                    → BD      (variables, red, tablas)
curl OK, navegador NO            → CORS
A veces tarda 50 s               → COLD START
```

> 💡 Lee el log **de abajo arriba**: el error real suele estar antes del "No open ports".

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `Could not read package.json` | Root Directory apuntando a la carpeta con `package.json` |
| `nest: not found` / `tsc: not found` | Quita `NODE_ENV=production` o usa `npm install --include=dev` |
| `Cannot find module './Books.service'` | Mayúsculas: Linux distingue, el import debe coincidir con el archivo |
| `No open ports detected` | `process.env.PORT ?? 3000`, sin host `localhost`; mira si la BD falló antes |
| `ENOENT ... open '.env'` | `if (existsSync('.env')) process.loadEnvFile();` |
| `ECONNREFUSED 127.0.0.1:5432` | Falta `DATABASE_URL` en Render (usa `localhost`) |
| `relation "books" does not exist` | Ejecuta `schema.sql` en Supabase: la BD de producción empieza vacía |
| `blocked by CORS policy` | `CORS_ORIGIN` con el dominio del frontend |
