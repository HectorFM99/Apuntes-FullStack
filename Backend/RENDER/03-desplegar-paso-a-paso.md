# 03 — DESPLEGAR PASO A PASO

> Con el proyecto preparado (archivo `02`), crear el servicio en Render es
> rellenar un formulario. Aquí se explica **cada campo**: qué significa y qué
> valor lleva en tus dos proyectos.

---

## 3.1 Antes de empezar

1. Cuenta en **render.com** (lo más cómodo: *Sign in with GitHub*).
2. Tu proyecto en un **repositorio de GitHub** (público o privado).
3. Permiso para que Render lea ese repo: *Configure GitHub App* → elegir repos.
4. La **cadena de conexión de producción** a mano (Supabase o Atlas).

---

## 3.2 Crear el Web Service

**Dashboard → New → Web Service → elegir el repositorio.**

### Los campos, uno a uno

| Campo | Qué es | `nestjs-postgresql-supabase-render` | `express-mongo-atlas-render` |
|-------|--------|-------------------------------------|------------------------------|
| **Name** | Nombre del servicio. Forma la URL `https://<name>.onrender.com` | `api-books-nest` (ejemplo) | `api-products` (ejemplo) |
| **Language / Runtime** | Entorno de ejecución | `Node` | `Node` |
| **Branch** | Rama que se despliega | `main` | `main` |
| **Region** | Dónde está físicamente el servidor | **Frankfurt (EU Central)** (ver 3.4) | La más cercana a tu cluster de Atlas |
| **Root Directory** | Subcarpeta del repo donde está el `package.json` | La carpeta del proyecto, si el repo tiene varias | Ídem |
| **Build Command** | Se ejecuta una vez por despliegue | `npm install && npm run build` | `npm install && npm run build` |
| **Start Command** | Arranca el servidor | `npm run start:prod` | `npm start` |
| **Instance Type** | Potencia (RAM/CPU) y precio | `Free` | `Free` |
| **Environment Variables** | Lo que va a `process.env` | `DATABASE_URL` | `MONGODB_URI`, `DB_NAME` |

### Ejemplo REAL (`nestjs-postgresql-supabase-render/README.md`)

```markdown
| Campo de Render | Valor |
|---|---|
| Root Directory | `03b-nestjs-postgresql` (si el proyecto está en una subcarpeta) |
| Build Command | `npm install && npm run build` |
| Start Command | `npm run start:prod` |
| Variables | `DATABASE_URL` (la del *Session pooler* de Supabase, terminada en `?sslmode=no-verify`) |
```

### Ejemplo REAL (`express-mongo-atlas-render/README.md`)

```markdown
| Campo de Render | Valor |
|---|---|
| Root Directory | `02b-express-mongodb` (si el proyecto está en una subcarpeta) |
| Build Command | `npm install && npm run build` |
| Start Command | `npm start` |
| Variables | `MONGODB_URI` (la de MongoDB Atlas) y `DB_NAME=bootcamp_api` |

`npm run build` compila TypeScript a `dist/` y `npm start` ejecuta `node dist/index.js`.
El puerto sale de `process.env.PORT` (Render lo pone solo; en tu ordenador es 3000).
```

> 📌 Fíjate en que **no** hay variable `PORT`: Render la pone sola. Si la añades
> tú, no pasa nada, pero no hace falta.

---

## 3.3 El **Root Directory** (monorepos)

Si tu repositorio tiene esta forma:

```
Bootcamp-FullStack/           ← raíz del repo en GitHub
├── mongo-atlas-render/
│   └── express-mongo-atlas-render/
│       ├── package.json       ← aquí está el proyecto
│       └── src/
└── postgresql-render-atlas/
    └── nestjs-postgresql-supabase-render/
        ├── package.json       ← y aquí otro
        └── src/
```

Render, por defecto, busca `package.json` en la **raíz del repo** → no lo encuentra:

```
npm error enoent Could not read package.json
```

Solución: **Root Directory** = `postgresql-render-atlas/nestjs-postgresql-supabase-render`.
Desde ese momento, todos los comandos se ejecutan **dentro** de esa carpeta, y
Render solo redespliega cuando cambian archivos de esa carpeta.

> 💡 Puedes tener **varios servicios** de Render apuntando al **mismo repo**, cada
> uno con su Root Directory. Así despliegas las dos APIs desde un único repo.

---

## 3.4 Elegir la región: cerca de tu base de datos

Cada petición a tu API hace (normalmente) una o varias consultas a la BD. Si la
API está en Oregón (EE. UU.) y la BD en Frankfurt, **cada consulta cruza el
Atlántico** (~100-150 ms de ida y vuelta):

```
Usuario (España) ──▶ API en Oregón ──(150 ms)──▶ BD en Frankfurt
                                    ◀─(150 ms)──
 Un GET que hace 3 consultas ≈ +450 ms solo de "viaje"

Usuario (España) ──▶ API en Frankfurt ──(1-2 ms)──▶ BD en Frankfurt  ✅
```

En tu proyecto, la cadena de Supabase apunta a
`aws-0-eu-central-1.pooler.supabase.com` → **eu-central-1 = Frankfurt** → la
región correcta en Render es **Frankfurt (EU Central)**.

> ⚠️ La región de un servicio de Render **no se puede cambiar** después; habría
> que crear otro servicio.

---

## 3.5 Primer despliegue: qué verás en los logs

```
==> Cloning from https://github.com/<usuario>/<repo>
==> Checking out commit a1b2c3d in branch main
==> Using Node.js version 22.x.x
==> Running build command 'npm install && npm run build'...
added 712 packages in 18s
> api-books-nest@0.0.1 build
> nest build
==> Build successful 🎉
==> Deploying...
==> Running 'npm run start:prod'
> node dist/main
[Nest] LOG [NestFactory] Starting Nest application...
[Nest] LOG [RoutesResolver] BooksController {/api/books}:
[Nest] LOG [NestApplication] Nest application successfully started
==> Your service is live 🎉
```

Y para comprobarlo:

```bash
curl -i https://api-books-nest.onrender.com/api          # Hello World (AppController)
curl -i https://api-books-nest.onrender.com/api/books    # 200 con los libros de Supabase
```

> 📌 `/api` delante de todo viene de `app.setGlobalPrefix('api')` en `main.ts`.

---

## 3.6 Auto-deploy: desplegar es hacer `git push`

Por defecto **Auto-Deploy = On Commit**:

```
git add . && git commit -m "feat: filtro por autor" && git push
        │
        ▼
GitHub avisa a Render (webhook) → nuevo despliegue automático del último commit
```

Otras opciones del menú **Manual Deploy**:

| Acción | Cuándo usarla |
|--------|---------------|
| *Deploy latest commit* | Auto-deploy está desactivado o quieres forzar un redeploy |
| *Clear build cache & deploy* | Errores raros de dependencias ("en mi PC funciona") |
| *Restart service* | Reiniciar **sin** reconstruir (ej. tras cambiar una variable) |
| **Rollback** (en *Events*) | Volver a un despliegue anterior que funcionaba |

> 💡 Cambiar una **variable de entorno** también provoca un nuevo despliegue (o
> te pregunta si quieres hacerlo): las variables solo se leen al arrancar el proceso.

---

## 3.7 Infraestructura como código: `render.yaml` (Blueprint)

En vez de rellenar el formulario a mano, puedes describir el servicio en un
archivo `render.yaml` en la **raíz del repo** y crearlo con *New → Blueprint*.
**Tus proyectos no lo usan**; esto sería el equivalente:

```yaml
# render.yaml (ejemplo, NO está en tus proyectos)
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
        sync: false        # secreto: Render te lo pide en el panel, NO va en el repo

  - type: web
    name: api-products
    runtime: node
    region: frankfurt
    plan: free
    rootDir: mongo-atlas-render/express-mongo-atlas-render
    buildCommand: npm install && npm run build
    startCommand: npm start
    envVars:
      - key: DB_NAME
        value: bootcamp_api
      - key: MONGODB_URI
        sync: false
```

Ventajas:

- La configuración queda **versionada en Git** (sabes quién cambió qué).
- Puedes recrear todo el entorno en otra cuenta con un clic.
- Revisable en un *pull request* como cualquier código.

> ⚠️ `sync: false` es la forma de declarar una variable **secreta** sin escribir su
> valor en el repo. Nunca pongas contraseñas en `render.yaml`.

---

## 3.8 Desplegar el frontend (Static Site) — para completar el cuadro

Si tienes un frontend con Vite:

| Campo | Valor |
|-------|-------|
| Build Command | `npm install && npm run build` |
| Publish Directory | `dist` |
| Variables | `VITE_API_URL=https://api-books-nest.onrender.com/api` |

Y en la API, `CORS_ORIGIN=https://mi-web.onrender.com` (ver `02`, apartado 2.5).

> ⚠️ Las variables `VITE_*` se **incrustan** en el JS durante el build: son
> **públicas**. Nunca pongas ahí un secreto (`DATABASE_URL`, `JWT_SECRET`...).

---

## ✅ Resumen

- Web Service = repo + rama + **Root Directory** + **Build** + **Start** + **variables**.
- Región: la **misma** que tu base de datos (Supabase eu-central-1 → Frankfurt).
- `git push` = nuevo despliegue (auto-deploy). Hay *rollback* si algo sale mal.
- `render.yaml` permite guardar la configuración como código.
