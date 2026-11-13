# 🧭 CONCLUSIONES — RENDER + SUPABASE

> Cierre de las carpetas `Render/` y `Supabase/`. No repite la teoría: la
> **condensa** en las ideas que de verdad tienes que llevarte, conectadas con tus
> proyectos (`nestjs-postgresql-supabase-render` y `express-mongo-atlas-render`)
> y con lo que se suele preguntar en una entrevista.

---

## 1. La foto final de tu arquitectura

```
┌─────────────┐  git push   ┌──────────┐  webhook  ┌───────────────────────────────┐
│ Tu PC       │ ──────────▶ │ GitHub   │ ────────▶ │ RENDER (Web Service, Frankfurt)│
│ Docker PG   │             │ sin .env │           │ npm install && npm run build   │
│ .env local  │             └──────────┘           │ npm run start:prod             │
└─────────────┘                                    │ PORT ← Render                  │
                                                   │ DATABASE_URL ← panel           │
     navegador / curl ──HTTPS──▶ https://<api>.onrender.com/api/books              │
                                                   └──────────────┬────────────────┘
                                                                  │ pg.Pool + TLS (no-verify)
                                                                  │ usuario postgres.<REF>
                                                                  ▼
                                        ┌──────────────────────────────────────────┐
                                        │ SUPABASE (eu-central-1 = Frankfurt)      │
                                        │ Supavisor · Session pooler :5432 (IPv4)  │
                                        │            ▼                             │
                                        │ PostgreSQL · public.books (+ RLS on)     │
                                        └──────────────────────────────────────────┘
```

**Render ejecuta tu código; Supabase guarda tus datos.** Cada uno resuelve una
mitad del problema y se unen con **una sola variable**: `DATABASE_URL`.

---

## 2. Las 10 ideas que tienes que llevarte

1. **Desplegar es ejecutar tu código en una máquina ajena.** Esa máquina solo conoce lo que hay en GitHub y lo que escribes en su panel. Si algo vive solo en tu PC (`.env`, la BD de Docker, archivos en `uploads/`), en producción **no existe**.

2. **Build y Start son dos momentos distintos.** `build` (una vez por despliegue) convierte TS en JS y necesita `devDependencies`; `start` (cada arranque) ejecuta `node dist/...` y debe ser ligero. Nunca `start:dev` ni `--watch` en producción.

3. **La configuración va en variables de entorno, no en el código.** El mismo código sirve en local y en producción porque todo lo que cambia (`PORT`, `DATABASE_URL`, `JWT_SECRET`, `CORS_ORIGIN`) se lee de `process.env`. Tu línea `if (existsSync('.env')) process.loadEnvFile();` es el ejemplo perfecto de código "consciente del entorno".

4. **El contenedor es efímero.** Disco y memoria se pierden en cada despliegue o reinicio. El estado vive fuera: Postgres (Supabase), Mongo (Atlas), archivos (Supabase Storage).

5. **Supabase es PostgreSQL.** Todo lo de la carpeta `POSTGRESQL` aplica. La prueba: tu `books.service.ts` funciona igual contra Docker que contra Supabase cambiando solo la cadena de conexión. No hay "lenguaje Supabase" que aprender para la forma A.

6. **La cadena de Supabase tiene tres decisiones dentro.** *Tipo* (direct / session / transaction), *red* (IPv6 vs IPv4) y *cifrado* (`sslmode`). Para un servidor persistente en Render: **Session pooler, puerto 5432, usuario `postgres.<REF>`, `?sslmode=no-verify`**.

7. **Un solo `Pool` por aplicación y escuchando `'error'`.** Las conexiones son caras y limitadas en el plan gratuito; el pooler cierra las inactivas. `DatabaseModule` con `useFactory` + `pool.on('error')` resuelve ambas cosas.

8. **Hay dos formas de usar Supabase** y la diferencia es **dónde está la seguridad**:
   - *Forma A* (la tuya): tu API decide; Supabase es solo la BD.
   - *Forma B*: el frontend habla con Supabase; **RLS** decide.

9. **RLS no es opcional si la API automática está encendida.** Toda tabla de `public` es accesible con la clave pública. Activa RLS en todas; sin políticas = cerrada al mundo, abierta para tu API (que entra como `postgres`).

10. **Los planes gratuitos duermen.** Render se duerme a los 15 min (*cold start* ~1 min, despierta solo); Supabase se **pausa** tras ~1 semana (hay que restaurarlo a mano). Antes de una demo: abre el panel de Supabase y llama a la API un minuto antes.

---

## 3. Mapa de responsabilidades

| Pregunta | Render | Supabase | Tu código |
|----------|:------:|:--------:|:---------:|
| ¿Dónde se ejecuta la API? | ✅ | | |
| ¿Quién compila TS? | ✅ (Build Command) | | `tsc` / `nest build` |
| ¿Qué puerto? | ✅ (`PORT`) | | `process.env.PORT ?? 3000` |
| ¿Dónde viven los datos? | | ✅ | |
| ¿Quién valida el body? | | (CHECK en BD) | ✅ DTO + `ValidationPipe` |
| ¿Quién protege del SQL injection? | | | ✅ `$1, $2` |
| ¿Quién decide permisos? | | ✅ si forma B (RLS) | ✅ si forma A (guards/servicio) |
| ¿HTTPS? | ✅ (automático) | ✅ (TLS en la conexión) | |
| ¿CORS? | | | ✅ `enableCors` |
| ¿Backups? | | ✅ en planes de pago / `pg_dump` | |
| ¿Logs? | ✅ (pestaña Logs) | ✅ (logs de Postgres/API) | ✅ `console.error` |

---

## 4. Lo que tus proyectos ya hacen bien

| Práctica | Dónde |
|----------|-------|
| `.env` fuera de Git, `.env.example` como contrato | `.gitignore` de ambos proyectos |
| `.env` opcional para que arranque en Render | `nestjs.../src/main.ts` |
| Puerto desde `process.env.PORT` | `main.ts` y `index.ts` |
| Scripts separados de desarrollo y producción | `package.json` de ambos |
| Un único `Pool` inyectable | `database.module.ts` |
| `pool.on('error')` para no caer | `database.module.ts` |
| Fallar rápido si falta la variable | `express.../src/db.ts` |
| SQL parametrizado (`$1`) | `books.service.ts` |
| Capa de traducción BD → API (`toBook`) | `book.ts` |
| Errores genéricos al cliente, detalle al log | middleware de errores de Express |
| Postgres local en Docker idéntico al de producción | `compose.yaml` (Postgres 17) |
| Session pooler + `sslmode=no-verify` documentado | `README.md` |

---

## 5. Lo que mejoraría antes de llamarlo "producción"

| # | Mejora | Por qué | Esfuerzo |
|---|--------|---------|----------|
| 1 | `CORS_ORIGIN` como variable de entorno | Hoy solo acepta `localhost`: un frontend desplegado sería bloqueado | Bajo |
| 2 | `ALTER TABLE books ENABLE ROW LEVEL SECURITY;` en Supabase | Cierra la API automática que hoy podría saltarse tu NestJS | Bajo |
| 3 | Comprobar `DATABASE_URL` al arrancar (como hace `db.ts` con Mongo) | Error claro en vez de `ECONNREFUSED` en la primera petición | Bajo |
| 4 | Fijar versión de Node (`engines` / `NODE_VERSION`) | `process.loadEnvFile()` exige Node ≥ 20.12 | Bajo |
| 5 | `?sslmode=no-verify` también en el `.env` local de Supabase | Hoy esa conexión viaja sin cifrar | Bajo |
| 6 | Health Check Path en Render (`/api`) | Reinicio automático si se cuelga; despliegues más seguros | Bajo |
| 7 | Migraciones (CLI de Supabase o un ORM) en vez de `schema.sql` suelto | Esquema versionado y reproducible entre entornos | Medio |
| 8 | Verificación SSL completa con el CA de Supabase | Protege contra suplantación del servidor | Medio |
| 9 | `render.yaml` (Blueprint) | Configuración del despliegue versionada en Git | Medio |
| 10 | Rotar la contraseña de la BD y los secretos si alguna vez se compartieron | La `DATABASE_URL` es superusuario | Bajo |

---

## 6. Guía rápida de diagnóstico (de un vistazo)

| Síntoma | Primera sospecha |
|---------|------------------|
| `Could not read package.json` | Root Directory |
| `nest: not found` / `tsc: not found` | `NODE_ENV=production` sin devDependencies |
| `Cannot find module './X'` solo en Render | Mayúsculas (Linux distingue) |
| `No open ports detected` | `PORT` fijo, host `localhost`, o la app murió antes de `listen` |
| `ECONNREFUSED 127.0.0.1:5432` | `DATABASE_URL` no puesta en Render |
| `ENETUNREACH` / dirección `2a05:...` | Conexión directa (IPv6) → usa el pooler |
| `self-signed certificate in certificate chain` | `sslmode=require` con `pg` → `no-verify` |
| `relation "books" does not exist` | No ejecutaste `schema.sql` en Supabase |
| `Tenant or user not found` | Usuario sin `.<REF>` o cadena de otro proyecto |
| CORS en el navegador, curl OK | `origin` sigue siendo `localhost` |
| Primera petición tarda ~50 s | Render Free dormido (normal) |
| Todo falla tras semanas sin uso | Proyecto de Supabase **pausado** |
| `supabase-js` no devuelve filas, sin error | RLS sin política que lo permita |

---

## 7. ¿Cuándo elegir qué? (criterio, no receta)

| Situación | Recomendación |
|-----------|---------------|
| Proyecto del bootcamp / portfolio con NestJS o Express | **Forma A**: API en Render + Supabase como Postgres. Demuestra que sabes backend |
| MVP rápido, poco tiempo, sobre todo CRUD + login | **Forma B**: frontend + `supabase-js` + Auth + RLS |
| Necesitas claves de terceros (Stripe, OpenAI) o webhooks | Tu API en Render **o** Edge Functions |
| Subida de archivos | Supabase Storage (nunca el disco de Render) |
| Datos muy relacionales, `JOIN`, transacciones | Postgres (Supabase) mejor que Mongo (Atlas) |
| Documentos flexibles sin esquema fijo | MongoDB Atlas (tu otro proyecto) |
| Algo con usuarios reales y dinero | Planes de pago: sin *sleep*, con backups |

---

## 8. Preguntas de entrevista (con respuesta corta)

**¿Qué diferencia hay entre IaaS, PaaS y BaaS?**
IaaS te da una máquina (EC2); PaaS te da dónde ejecutar tu código (Render); BaaS te da el backend hecho (Supabase).

**¿Por qué no subes el `.env` a GitHub y cómo llegan entonces las variables a producción?**
Contiene secretos. En producción se configuran en el panel del proveedor (Render → Environment), que las inyecta en `process.env` al arrancar.

**¿Por qué tu API escucha en `process.env.PORT`?**
Porque el puerto lo decide la plataforma; si escuchas en otro, el balanceador no encuentra tu app.

**¿Qué es un connection pooler y por qué lo usas con Supabase?**
Un intermediario que reparte muchas conexiones de clientes entre pocas conexiones reales a Postgres, que son caras. Además, el pooler de Supabase tiene IPv4, necesaria desde Render.

**Session vs transaction pooling.**
Session: una conexión real por cliente mientras esté conectado (servidores con `Pool`). Transaction: por transacción (serverless); rompe prepared statements y `SET`.

**¿Qué es Row Level Security?**
Políticas de Postgres que añaden un filtro automático por fila según quién consulta (`auth.uid()`). Permiten exponer la BD al frontend con seguridad.

**¿La clave `anon` de Supabase es un secreto?**
No. Es pública; la seguridad la da RLS. La secreta (`service_role`) y la `DATABASE_URL` nunca deben salir del servidor.

**¿Qué pasa con los archivos que tu API guarda en disco en Render?**
Se pierden al redesplegar: el sistema de archivos es efímero. Hay que usar almacenamiento de objetos (Supabase Storage, S3).

**¿Por qué tu API y tu base de datos están en la misma región?**
Cada consulta hace un viaje de ida y vuelta; entre continentes son ~150 ms por consulta.

**¿Cómo evitas SQL injection con `pg`?**
Consultas parametrizadas (`$1`, `$2`) — los valores viajan separados del SQL y nunca se interpretan como código.

---

## 9. En una frase

> **Render convierte tu repositorio en un servicio público, Supabase convierte
> PostgreSQL en un servicio gestionado, y lo que los une —variables de entorno,
> un pool de conexiones bien configurado y una seguridad pensada en el lugar
> correcto— es exactamente lo que diferencia "funciona en mi máquina" de
> "funciona en producción".**

---

📚 Para profundizar: `Render/00-INDICE.md` · `Supabase/00-INDICE.md` ·
`POSTGRESQL/07-postgres-desde-node-con-pg.md` · `NESTJS/06-base-de-datos-y-configuracion.md` ·
`API/06-autenticacion-cors-y-seguridad.md`
