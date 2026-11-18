# ⚡ CARPETA SUPABASE — ÍNDICE DE ESTUDIO

Teoría de **Supabase** explicada desde la base: qué es por dentro, cómo lo usas
tú (como **PostgreSQL en la nube** para tu API NestJS) y cómo se usa como
**backend completo** (API automática, Auth, RLS, Storage, Realtime).

> ⚠️ Los proyectos **NO** se han modificado. Solo se han copiado fragmentos de su código.
> 🔒 No se ha copiado ningún `.env` real. Las cadenas de conexión aparecen con
> `<REF>` (identificador del proyecto) y `<PASSWORD>`.

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a aprender |
|---|---------|--------------------|
| 1 | `01-que-es-supabase-y-su-arquitectura.md` | Qué es un BaaS, las piezas internas (Postgres, PostgREST, Auth, Storage, Realtime, Edge Functions), las **dos formas** de usar Supabase |
| 2 | `02-conectarse-a-postgres-pooler-ssl-e-ipv6.md` | Cadenas de conexión: directa vs **Session pooler** vs Transaction pooler, IPv4/IPv6, SSL y `sslmode=no-verify`, `pg.Pool` |
| 3 | `03-esquema-sql-editor-y-migraciones.md` | Llevar tu `schema.sql` a Supabase, SQL Editor, Table Editor, CLI y migraciones, entorno local |
| 4 | `04-api-automatica-claves-y-supabase-js.md` | La API REST generada, claves `anon`/`publishable` vs `service_role`/`secret`, `@supabase/supabase-js` |
| 5 | `05-auth-y-row-level-security.md` | Usuarios, JWT, `auth.uid()`, **RLS** y políticas, comparado con tu Auth hecha a mano |
| 6 | `06-storage-realtime-y-edge-functions.md` | Archivos, cambios en tiempo real y funciones serverless |
| 7 | `07-ejemplos-completos-de-proyectos.md` | Tu proyecto NestJS + Supabase entero, y cómo se vería la misma API con `supabase-js` |
| 📋 | `99-CHEATSHEET.md` | Chuleta de comandos y código para consultar rápido |

📌 Las **conclusiones conjuntas** de Render + Supabase están en
`../CONCLUSIONES-Render-y-Supabase.md`.

---

## 🗂️ Proyectos de los que se han extraído los ejemplos

| Proyecto | Archivo | Temas que cubre |
|----------|---------|-----------------|
| `postgresql-render-atlas/nestjs-postgresql-supabase-render/` | `src/database/database.module.ts` | `Pool` de `pg` con `DATABASE_URL` de Supabase |
| ″ | `src/books/books.service.ts` | CRUD con SQL parametrizado contra Supabase |
| ″ | `src/books/book.ts` | `BookRow` (snake_case) → `Book` (camelCase) |
| ″ | `database/schema.sql` | La tabla `books` que hay que crear en Supabase |
| ″ | `compose.yaml` | Postgres local en Docker (el "gemelo" de desarrollo de Supabase) |
| ″ | `README.md` | Usar el **Session pooler** con `?sslmode=no-verify` |
| ″ | `.env` (solo la **forma**, sin contraseña) | Host `aws-0-eu-central-1.pooler.supabase.com`, usuario `postgres.<REF>` |

---

## 🚀 Orden de estudio recomendado

1. `01` → la foto completa: qué es Supabase y **qué parte usas tú**.
2. `02` → lo que necesitas para que tu proyecto funcione (conexión). **Imprescindible.**
3. `03` → cómo crear y evolucionar las tablas en Supabase.
4. `04` → la "otra forma" de usar Supabase: sin escribir backend.
5. `05` → la pieza de seguridad que lo hace posible: **RLS**.
6. `06` → los extras que lo convierten en "backend completo".
7. `07` → repaso con código real y comparativa.

---

## 💡 La idea en una imagen

```
                       ┌──────────────────── SUPABASE (un proyecto) ─────────────────────┐
                       │                                                                 │
 Tu API NestJS ───────────── DATABASE_URL (pg) ─────────────▶  ┌──────────────────────┐  │
 (en Render)           │   ← lo que usas TÚ                     │                      │  │
                       │                                        │     PostgreSQL       │  │
 Frontend React ──▶ https://<REF>.supabase.co ─▶ API Gateway ─┬▶│  (tablas, RLS,       │  │
 (supabase-js)         │   ← la "otra forma"                  │ │   funciones...)      │  │
                       │                     ┌────────────────┤ └──────────────────────┘  │
                       │                     │ /rest/v1    PostgREST (API REST automática) │
                       │                     │ /auth/v1    Auth (usuarios, JWT)            │
                       │                     │ /storage/v1 Storage (archivos)              │
                       │                     │ /realtime   Realtime (WebSockets)           │
                       │                     │ /functions  Edge Functions (Deno)           │
                       │                     └─────────────────────────────────────────────┘
                       └─────────────────────────────────────────────────────────────────┘
```

> 📌 Supabase **es PostgreSQL**. Todo lo que sabes de la carpeta `POSTGRESQL`
> (tablas, `JOIN`, índices, transacciones) funciona tal cual. Lo demás son
> servicios construidos **alrededor** de ese Postgres.
