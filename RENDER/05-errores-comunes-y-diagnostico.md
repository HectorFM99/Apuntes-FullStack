# 05 — ERRORES COMUNES Y DIAGNÓSTICO

> Cuando un despliegue falla, la pregunta clave es **en qué fase** falló
> (archivo `01`, apartado 1.3). Cada fase tiene sus errores típicos y su sitio
> donde mirar.

---

## 5.1 El método: localizar la fase

```
¿El log dice "Build failed"?               → Fase BUILD   → 5.2
¿Build OK pero "Exited with status 1"?     → Fase START   → 5.3
¿"No open ports detected"?                 → PUERTO       → 5.4
¿Live, pero las peticiones dan 500?        → BD/CÓDIGO    → 5.5
¿Live y curl va bien, pero el navegador no?→ CORS         → 5.6
¿Va, pero a veces tarda 50 segundos?       → COLD START   → archivo 04
```

> 💡 Regla de oro: **reproduce en local en modo producción** antes de pelearte con
> Render:
>
> ```bash
> npm install && npm run build && npm run start:prod
> ```
>
> Si falla en tu PC, fallará en Render. Si funciona en tu PC y no en Render, la
> diferencia está casi siempre en **variables**, **versión de Node** o **red**.

---

## 5.2 Fallos de BUILD

| Mensaje en el log | Causa | Solución |
|-------------------|-------|----------|
| `Could not read package.json` | Root Directory mal puesto | Apunta a la carpeta que contiene `package.json` |
| `npm error Missing script: "build"` | No hay script `build` | Añádelo (`"build": "tsc"` / `"nest build"`) |
| `sh: nest: not found` / `tsc: not found` | `devDependencies` no instaladas (¿`NODE_ENV=production`?) | Quita esa variable o usa `npm install --include=dev` |
| `error TS2322: Type 'x' is not assignable...` | Error de tipos que en dev ignorabas | Arréglalo; prueba con `npm run typecheck` / `npm run build` en local |
| `Cannot find module './Books.service'` | Mayúsculas: Windows no distingue, **Linux sí** | El `import` debe coincidir **exactamente** con el nombre del archivo |
| `npm ERR! ERESOLVE` | Conflicto de versiones de dependencias | Sube un `package-lock.json` coherente; prueba `npm ci` en local |

> ⚠️ El de **mayúsculas** es muy típico viniendo de Windows: `import { BooksService }
> from './Books.service'` funciona en tu PC aunque el archivo sea `books.service.ts`,
> pero Render corre en **Linux**, donde son nombres distintos.

---

## 5.3 Fallos al ARRANCAR (Start Command)

| Mensaje | Causa | Solución |
|---------|-------|----------|
| `Error: Cannot find module '/opt/render/project/src/dist/main'` | El build no generó `dist/` o el Start apunta a otro sitio | Comprueba `outDir` en `tsconfig` y el script `start` |
| `ENOENT: no such file or directory, open '.env'` | `process.loadEnvFile()` sin comprobar si existe | `if (existsSync('.env')) process.loadEnvFile();` (tu `main.ts` ya lo hace) |
| `Error: Falta MONGODB_URI en el archivo .env` | Variable no creada en Render | Pestaña Environment → añadirla |
| `process.loadEnvFile is not a function` | Node antiguo (< 20.12) | Fija `NODE_VERSION` / `engines` |
| `SyntaxError: Cannot use import statement outside a module` | Ejecutas `.ts` o hay lío ESM/CommonJS | Ejecuta el JS compilado; revisa `"type"` en `package.json` y `module` en `tsconfig` |
| `JavaScript heap out of memory` | 512 MB del plan Free (¿usando `start:dev`/watch?) | Usa `start:prod`; no ejecutes `tsc`/`nest` en el Start |

---

## 5.4 "No open ports detected"

```
==> No open ports detected, continuing to scan...
==> Port scan timeout reached, no open ports detected. Bind your service to at least one port.
```

Causas:

1. Puerto fijo: `app.listen(3000)` → usa `process.env.PORT ?? 3000`.
2. Host fijo: `app.listen(PORT, 'localhost')` → quita el host.
3. **La app nunca llega a `listen`**: está bloqueada antes. Ojo en Express:

### Ejemplo REAL (`express-mongo-atlas-render/src/index.ts`)

```ts
async function start() {
  await connectDb();      // ← si Atlas no responde, se queda aquí ~30 s y lanza error

  app.listen(PORT, () => {
    console.log(`Servidor escuchando en http://localhost:${PORT}`);
  });
}

start();
```

Si Atlas rechaza la conexión (IP no permitida, contraseña mal), `connectDb()`
falla y **nunca** se llega a `app.listen` → Render ve "no open ports". El error
real está **más arriba** en el log (`MongoServerSelectionError`, `bad auth`...).

> 📌 Esto es un diseño **correcto** ("si no hay BD, no arranco"), pero hay que
> saber leer el log de abajo arriba. Además, como `start()` no tiene `.catch`,
> el error sale como *unhandled rejection* y Node termina con código 1, que es lo
> que queremos para que Render lo marque como fallido.

---

## 5.5 Live, pero responde 500 (problemas de base de datos)

Mira la pestaña **Logs** en el momento de la petición:

| Error en el log | Significado | Solución |
|-----------------|-------------|----------|
| `connect ECONNREFUSED 127.0.0.1:5432` | `DATABASE_URL` vacía o apunta a `localhost` | Pon la cadena de Supabase en Render |
| `connect ENETUNREACH 2a05:...:5432` | Usas la conexión **directa** de Supabase (IPv6) | Usa el **Session pooler** (`...pooler.supabase.com`) |
| `self-signed certificate in certificate chain` | `sslmode=require` y `pg` intenta verificar el certificado | `?sslmode=no-verify` (o configurar el CA, ver `Supabase/02`) |
| `password authentication failed for user "postgres"` | Contraseña mal o usuario sin el `.<REF>` del pooler | Usuario `postgres.<REF>`; si la contraseña tiene `@ # /`, codifícala (`encodeURIComponent`) |
| `Tenant or user not found` | Usuario/host del pooler no coinciden con el proyecto | Copia la cadena de nuevo desde *Connect* |
| `relation "books" does not exist` | La tabla no existe **en Supabase** (solo en tu Docker) | Ejecuta `database/schema.sql` en el SQL Editor de Supabase |
| `MongoServerSelectionError` | Atlas no deja entrar a la IP de Render | *Network Access* → `0.0.0.0/0` |
| `remaining connection slots are reserved` / `max clients reached` | Demasiadas conexiones abiertas | Un único `Pool` por app (ya lo haces con `DatabaseModule`); baja `max` |

> 📌 El error `relation "books" does not exist` es el clásico: **la base de datos
> de producción empieza vacía**. Tu `compose.yaml` cargaba `schema.sql`
> automáticamente en Docker, pero Supabase no sabe nada de ese archivo.

---

## 5.6 Funciona con curl, falla en el navegador: CORS

```
Access to fetch at 'https://api-books-nest.onrender.com/api/books'
from origin 'https://mi-web.onrender.com' has been blocked by CORS policy:
The 'Access-Control-Allow-Origin' header has a value 'http://localhost:5173'
that is not equal to the supplied origin.
```

El mensaje lo dice todo: la API sigue permitiendo solo `localhost:5173`
(`app.enableCors({ origin: 'http://localhost:5173' })`). Solución en `02`,
apartado 2.5 (variable `CORS_ORIGIN`).

---

## 5.7 Otros despistes

- **Cambié una variable y no hace nada** → redespliega/reinicia: se leen al arrancar.
- **Hice push y no se despliega** → ¿rama correcta? ¿Auto-Deploy activado? ¿El cambio está **fuera** del Root Directory? (Render ignora cambios fuera de él).
- **Mi API "pierde" datos** → ¿guardas en memoria o en disco? El contenedor es efímero (`01`, 1.3).
- **Los tiempos son malos siempre, no solo al despertar** → ¿región de Render distinta de la de la BD? (`03`, 3.4).

---

## ✅ Resumen

1. Identifica la **fase** (build / start / puerto / BD / CORS).
2. Reproduce en local con `build` + `start:prod`.
3. Lee el log **desde el primer error**, no el último mensaje.
4. Casi todo es: Root Directory, variables, versión de Node, IPv6/SSL de Supabase, o CORS.
