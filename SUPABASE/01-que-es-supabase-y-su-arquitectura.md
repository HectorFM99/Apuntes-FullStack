# 01 — QUÉ ES SUPABASE Y SU ARQUITECTURA

> Supabase se suele presentar como "la alternativa open-source a Firebase". La
> frase es cierta pero esconde lo importante: **Supabase es un PostgreSQL de
> verdad** al que le han puesto alrededor un conjunto de servicios (API, login,
> archivos, tiempo real) para que puedas construir una aplicación **sin escribir
> backend**... o, como haces tú, usarlo simplemente como **Postgres en la nube**.

---

## 1.1 El problema que resuelve

Para tener una app con usuarios y datos necesitas, como mínimo:

| Pieza | Hecho a mano (lo que has aprendido) | En Supabase |
|-------|-------------------------------------|-------------|
| Base de datos | Postgres en Docker / servidor | ✅ Postgres gestionado (backups, actualizaciones) |
| API CRUD | NestJS/Express: controller + service + SQL | ✅ Generada automáticamente desde las tablas (PostgREST) |
| Registro / login | `argon2` + JWT + guards a mano | ✅ Supabase Auth (email, magic link, Google, GitHub...) |
| Permisos ("solo veo mis datos") | `if (book.userId !== user.id) throw ...` en el servicio | ✅ **Row Level Security** en la propia BD |
| Subida de archivos | `multer` + disco/S3 | ✅ Supabase Storage |
| Tiempo real | `socket.io` a mano | ✅ Supabase Realtime |
| Lógica a medida | Endpoints propios | ✅ Edge Functions / funciones SQL |

> 📌 Saber hacerlo **a mano** primero (como en el bootcamp) es lo que te permite
> entender **qué** te está dando Supabase hecho y **cuándo** te conviene o no.

---

## 1.2 Firebase vs Supabase: la diferencia de fondo

| | Firebase (Google) | Supabase |
|---|---|---|
| Base de datos | Firestore: **NoSQL**, documentos (parecido a MongoDB) | **PostgreSQL**: relacional, SQL |
| Relaciones / `JOIN` | No hay; se duplican datos | Sí, con FK y `JOIN` |
| Consultas | API propia, limitada | SQL completo |
| Código | Cerrado | **Open source** (puedes autoalojarlo) |
| Salir de la plataforma | Difícil (formato propio) | `pg_dump` y te llevas tu Postgres a cualquier sitio |

> 💡 Esa última fila es clave: con Supabase **no quedas atrapado**. Tu proyecto
> NestJS lo demuestra: el mismo código funciona contra el Postgres de Docker y
> contra Supabase cambiando **solo** la `DATABASE_URL`.

---

## 1.3 Las piezas internas (todas open source)

```
                         https://<REF>.supabase.co
                                    │
                           ┌────────▼────────┐
                           │  API Gateway    │  ← comprueba la apikey y enruta
                           └──┬───┬───┬───┬──┘
          ┌───────────────────┘   │   │   └───────────────────┐
          ▼                       ▼   ▼                       ▼
   ┌─────────────┐      ┌──────────┐ ┌──────────┐     ┌──────────────┐
   │ PostgREST   │      │  Auth    │ │ Storage  │     │  Realtime    │
   │ /rest/v1    │      │ /auth/v1 │ │/storage/v1│    │  /realtime   │
   │ tablas→REST │      │ (GoTrue) │ │ archivos │     │ (WebSockets) │
   └──────┬──────┘      └────┬─────┘ └────┬─────┘     └──────┬───────┘
          │                  │            │                   │
          └──────────────────┴─────┬──────┴───────────────────┘
                                   ▼
               ┌─────────────────────────────────────────┐
               │              POSTGRESQL                  │
               │  schema public   → TUS tablas (books...) │
               │  schema auth     → usuarios (auth.users) │
               │  schema storage  → metadatos de archivos │
               │  + RLS, funciones, triggers, extensiones │
               └─────────────────────────────────────────┘
                                   ▲
               Supavisor (pooler) ─┘  ← por aquí entra TU API con `pg`
```

| Pieza | Qué hace | Detalle técnico |
|-------|----------|-----------------|
| **PostgreSQL** | Guarda todo | Postgres estándar + extensiones (`pgcrypto`, `uuid-ossp`, `pgvector`, `postgis`...) |
| **PostgREST** | Convierte cada tabla/vista del schema `public` en endpoints REST | `GET /rest/v1/books?author=ilike.*garcia*` |
| **Auth** (GoTrue) | Registro, login, tokens JWT | Los usuarios viven en la tabla `auth.users` |
| **Storage** | Guarda archivos en *buckets* | Metadatos en Postgres, archivos en almacenamiento tipo S3 |
| **Realtime** | Envía cambios de la BD por WebSocket | Escucha el log de replicación de Postgres |
| **Edge Functions** | Tu código TS/JS ejecutado en servidores de Supabase | Corren en **Deno**, no en Node |
| **Supavisor** | *Connection pooler* | Reparte muchas conexiones de clientes entre pocas conexiones reales a Postgres |
| **Studio** | El panel web (Table Editor, SQL Editor...) | También es open source |

> 📌 Lo importante: **todo pasa por Postgres**. Los usuarios son filas de
> `auth.users`, los permisos son políticas RLS de Postgres, el tiempo real lee el
> log de Postgres. Por eso decimos que en Supabase "la base de datos es el backend".

---

## 1.4 Las DOS formas de usar Supabase

Esta es la idea más importante de toda la carpeta.

### Forma A — "Postgres gestionado" (la que usa tu proyecto)

```
Navegador ──HTTP──▶ TU API (NestJS en Render) ──SQL (pg)──▶ Postgres de Supabase
                    │ valida (DTO)                          (solo como base de datos)
                    │ autentica (JWT propio)
                    │ decide permisos
                    └ escribe el SQL ($1, $2...)
```

- Usas **solo** la base de datos. Ignoras PostgREST, Auth, Storage...
- Te conectas con la **cadena de conexión** (`DATABASE_URL`) y la librería `pg`.
- La seguridad la pone **tu API**: el navegador nunca toca la BD.
- Ventaja: control total, lo aprendido en NestJS/Express se aplica tal cual, y puedes cambiar de proveedor sin tocar código.

### Forma B — "Backend as a Service"

```
Navegador ──HTTPS + apikey + JWT──▶ API de Supabase ──▶ Postgres
(supabase-js)                      (PostgREST, Auth...)   (RLS decide qué filas
                                                           puede ver cada usuario)
```

- El **frontend** habla directamente con Supabase usando `@supabase/supabase-js`.
- No hay API propia (o solo para casos especiales: Edge Functions).
- La seguridad la pone **la base de datos** con **Row Level Security**. Sin RLS, cualquiera con la clave pública podría leer/borrar todo.
- Ventaja: muchísimo menos código. Inconveniente: lógica repartida entre SQL/políticas y frontend, y más dependencia de Supabase.

### Comparativa

| | Forma A (tu proyecto) | Forma B (BaaS) |
|---|---|---|
| Quién habla con Postgres | Tu API, con `pg` | PostgREST (y el resto de servicios) |
| Credencial | `DATABASE_URL` (secreta, en Render) | `apikey` pública + JWT del usuario |
| Dónde está la validación | DTO + `ValidationPipe` | Restricciones SQL (`CHECK`, `NOT NULL`) + políticas |
| Dónde están los permisos | Código del servicio / guards | Políticas **RLS** |
| Necesitas Render | Sí (para la API) | Solo para el frontend (o ni eso) |
| ¿RLS obligatorio? | No es la defensa principal, pero recomendable activarlo igualmente (ver `05`) | **Sí, imprescindible** |

> 💡 No son excluyentes: es muy común usar **Supabase Auth** desde el frontend y
> además tener una API propia que verifica el JWT de Supabase. Elige según el
> proyecto; lo que no puedes es usar la forma B sin entender RLS.

---

## 1.5 Proyecto, región y plan gratuito

Al crear un proyecto en `supabase.com/dashboard` eliges:

- **Nombre** y **contraseña de la base de datos** (la del usuario `postgres`; guárdala, es la de tu `DATABASE_URL`).
- **Región**: dónde vive físicamente la BD. Tu proyecto está en **eu-central-1 (Frankfurt)** — se ve en el host `aws-0-eu-central-1.pooler.supabase.com`. Elige la misma zona que tu API de Render (ver `Render/03`, apartado 3.4).

Supabase asigna al proyecto un identificador, el **project ref** (`<REF>`), que
aparece en la URL (`https://<REF>.supabase.co`) y en el usuario del pooler
(`postgres.<REF>`).

> ⚠️ Límites del plan Free (consúltalos en `supabase.com/pricing`, cambian):

| Recurso | Free (aprox.) |
|---------|---------------|
| Proyectos activos | 2 |
| Base de datos | 500 MB |
| Storage | 1 GB |
| Usuarios activos/mes (Auth) | 50 000 |
| **Pausa por inactividad** | El proyecto se **pausa** tras ~1 semana sin actividad |
| Backups automáticos | ❌ (solo planes de pago) |

> 📌 La **pausa** es el equivalente al *cold start* de Render, pero peor: un
> proyecto pausado **no despierta solo** con una petición; hay que entrar al panel
> y pulsar *Restore*. Si tu API de Render empieza a fallar tras semanas sin usarla,
> mira primero si el proyecto de Supabase está pausado.

---

## 1.6 ¿Por qué Supabase y no la Postgres de Render?

Render también ofrece Postgres, pero en el plan Free **expira a los 30 días**.
Supabase Free no expira (solo se pausa por inactividad) y además trae panel SQL,
Table Editor y el resto de servicios. Por eso la combinación típica de bootcamp es:

```
Render (API, gratis)  +  Supabase (Postgres, gratis)  [+ Render Static Site (frontend)]
```

---

## ✅ Resumen

- Supabase = **PostgreSQL** + servicios alrededor (PostgREST, Auth, Storage, Realtime, Edge Functions).
- Todo se apoya en Postgres: usuarios, permisos y archivos son tablas y políticas.
- **Forma A** (tu proyecto): tu API conecta con `pg` → Supabase como Postgres gestionado.
- **Forma B**: el frontend usa `supabase-js` → la seguridad depende de **RLS**.
- Free: 500 MB, se **pausa** tras una semana sin uso.
