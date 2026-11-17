# 12 — ☁️ DESPLIEGUE Y CLOUD: RENDER, SUPABASE Y MONGODB ATLAS

> Un junior que ha **desplegado** algo destaca. Este documento resume lo que
> aprendiste en `../RENDER/`, `../SUPABASE/` y `../MONGODB-ATLAS/` en forma de
> preguntas de entrevista. La foto completa está en
> `../RENDER/CONCLUSIONES-Render-y-Supabase.md` y `../MONGODB-ATLAS/08-...`.

---

## 12.1 La arquitectura que has desplegado ⭐

```
 Navegador
   │  HTTPS
   ├──────────▶ Frontend React (Static Site: index.html + JS + CSS)
   │                   │ fetch(VITE_API_URL + '/api/...')
   ▼                   ▼
 API en RENDER (Web Service)  ── PORT, DATABASE_URL / MONGODB_URI, CORS_ORIGIN
   │                             (variables en el panel, nunca en GitHub)
   ├── pg.Pool + TLS ───────────▶ SUPABASE (PostgreSQL gestionado, Session pooler)
   │                              → nestjs-postgresql-supabase-render
   └── mongodb+srv:// + TLS ────▶ MONGODB ATLAS (replica set de 3 nodos, M0)
                                  → express-mongo-atlas-render
```

> 🗣️ "Render ejecuta mi código; Supabase o Atlas guardan mis datos. Se unen con una
> sola variable de entorno: la cadena de conexión. El código es el mismo que en
> local con Docker; solo cambia la configuración."

---

## 12.2 Conceptos de despliegue

| Concepto | Explicación corta |
|----------|-------------------|
| **PaaS** (Render, Heroku, Railway) | Subes el código y la plataforma lo construye, ejecuta y expone con HTTPS. No gestionas servidores |
| **DBaaS** (Atlas, Supabase, Neon) | Base de datos gestionada: backups, réplicas, TLS y actualizaciones los hace el proveedor |
| **BaaS** (Supabase, Firebase) | Backend "hecho": BD + API automática + Auth + Storage |
| **IaaS** (AWS EC2, máquinas virtuales) | Te dan la máquina; tú instalas y mantienes todo |
| **Serverless** (Edge Functions, Lambda) | Funciones que se ejecutan bajo demanda; pagas por ejecución |
| **Build vs Start** | *Build* una vez por despliegue (`npm run build`, compila TS, necesita devDependencies). *Start* en cada arranque (`node dist/...`), ligero |
| **Despliegue continuo** | Render se conecta a GitHub: cada `git push` a `main` → build y despliegue automáticos |
| **Contenedor efímero** | El disco se borra en cada despliegue/reinicio → el estado vive fuera (BD, Storage) |
| **Health check** | Ruta (`/api`) que la plataforma consulta para saber si la app está viva |
| **Cold start** | En el plan gratis, Render duerme la API tras 15 min sin tráfico; la primera petición tarda ~50 s |

---

## 12.3 Variables de entorno y secretos ⭐

| Pregunta | Respuesta |
|----------|-----------|
| ¿Por qué variables de entorno? | El **mismo código** funciona en local y producción; lo que cambia (`PORT`, `DATABASE_URL`, `JWT_SECRET`, `CORS_ORIGIN`) se lee de `process.env` (*twelve-factor app*) |
| ¿Por qué `process.env.PORT ?? 3000`? | Render asigna el puerto; si lo fijas, Render no detecta la app (*No open ports detected*) |
| ¿Dónde van los secretos? | `.env` en local (en `.gitignore`), panel de Render en producción. `.env.example` sin valores como contrato |
| ¿Y en el frontend? | `VITE_*` se **incrusta** en el JS al hacer build: es **público**. Nunca secretos ahí |
| ¿Qué haces si subes un secreto a GitHub? | Rotarlo **ya** (cambiar contraseña/clave). Borrarlo del repo no basta: queda en el historial |

---

## 12.4 Render

| Pregunta | Respuesta |
|----------|-----------|
| ¿Web Service vs Static Site? | Web Service: proceso Node escuchando en un puerto (tu API). Static Site: archivos de `dist/` servidos por CDN (tu React) |
| ¿Qué comandos configuras? | Build `npm install && npm run build`, Start `npm start` / `npm run start:prod`, Root Directory si el repo tiene varias carpetas |
| ¿Por qué falla en Render si en local va? | Mayúsculas en imports (Linux distingue), variables de entorno no puestas, `devDependencies` no instaladas, `localhost` en la conexión a BD |
| ¿Dónde miras cuando falla? | Pestaña **Logs**: primero el build, luego el arranque. El primer error es el importante |
| ¿Limitaciones del plan gratis? | Se duerme a los 15 min, horas limitadas, sin disco persistente |

---

## 12.5 Supabase ⭐

| Pregunta | Respuesta |
|----------|-----------|
| ¿Qué es Supabase? | Plataforma sobre **PostgreSQL** que añade API REST automática, Auth, Storage, Realtime y Edge Functions |
| ¿Cómo lo usaste tú? | Como **Postgres en la nube** para mi API NestJS: `pg.Pool` con `DATABASE_URL`, SQL parametrizado. El código es el mismo que contra Docker |
| ¿Direct connection vs pooler? | La directa usa IPv6 y Render no la alcanza (`ENETUNREACH`). El **Session pooler** (Supavisor, IPv4, puerto 5432, usuario `postgres.<REF>`) sí |
| ¿Y el SSL? | La conexión va cifrada; con `pg` usé `sslmode=no-verify` (cifra sin validar el certificado). Lo ideal: validar con el CA de Supabase |
| ¿Qué es RLS? | *Row Level Security*: políticas en Postgres que deciden qué filas puede ver/cambiar cada usuario (`auth.uid() = user_id`). **Obligatorio** si el frontend habla directo con Supabase |
| ¿Clave `anon` vs `service_role`? | `anon`/publishable: pública, va en el front, limitada por RLS. `service_role`/secret: se salta RLS, **solo** en el servidor |
| ¿Dos formas de usarlo? | **A**: tu API decide la seguridad y Supabase es solo la BD. **B**: el front usa `supabase-js` y RLS decide |
| ¿Plan gratis? | El proyecto se **pausa** tras ~1 semana sin actividad |

---

## 12.6 MongoDB Atlas ⭐

| Pregunta | Respuesta |
|----------|-----------|
| ¿Qué es Atlas? | MongoDB gestionado en la nube (AWS/GCP/Azure). El M0 es gratis |
| ¿Qué es un cluster / replica set? | 3 servidores con los mismos datos: un **primary** (escrituras) y dos **secondaries**. Si cae el primary, se elige otro automáticamente |
| ¿Qué dos "puertas" hay que abrir? | **Network Access** (IP Access List; `0.0.0.0/0` para Render en el plan gratis) y **Database Access** (usuario + contraseña + rol) |
| ¿Qué tiene de especial `mongodb+srv://`? | Es un **nombre DNS**, no un servidor: el driver descubre los 3 nodos por DNS SRV, activa TLS solo y obtiene `authSource` |
| ¿`retryWrites` y `w=majority`? | Reintentar una vez escrituras fallidas por cortes; confirmar cuando la mayoría de nodos tiene el dato |
| ¿Cómo distingues un fallo de red de uno de credenciales? | Red: tarda ~30 s (`MongoServerSelectionError`). Credenciales: instantáneo (`bad auth`) |
| ¿Contraseña con `@` o `#`? | Va dentro de una URL: hay que codificarla (`encodeURIComponent`) |
| ¿Cómo llevaste los datos de Docker a Atlas? | `mongodump` / `mongorestore`, o ejecutando el `seed.js` con `mongosh` contra la URI de Atlas |

---

## 12.7 ¿Supabase o Atlas? (pregunta de criterio)

| Si… | Elijo |
|-----|-------|
| Datos muy relacionados, JOINs, transacciones, integridad | **Supabase** (PostgreSQL) |
| Documentos flexibles, cada elemento con campos distintos | **Atlas** (MongoDB) |
| Quiero Auth, Storage y API sin escribir backend (MVP) | **Supabase** (forma B con RLS) |
| Ya tengo una API propia | Cualquiera: es "solo" la base de datos |

> 🗣️ "No hay una mejor en general: depende de la forma de los datos. Lo bueno es que
> en ambos casos mi código no dependía del proveedor, solo de la cadena de conexión."

---

## 12.8 CORS en producción ⭐

```
Local:  front http://localhost:5173  → API http://localhost:3000   → CORS_ORIGIN=http://localhost:5173
Render: front https://<front>.onrender.com → API https://<api>.onrender.com → CORS_ORIGIN=https://<front>.onrender.com
```

- CORS lo **aplica el navegador** y se configura en el **servidor** (cabecera
  `Access-Control-Allow-Origin` o `enableCors` en Nest).
- Con `curl` o Postman funciona y desde el navegador no → casi siempre es CORS.
- No uses `*` si envías cookies/credenciales.

---

## 12.9 Diagnóstico rápido (para contar que sabes depurar)

| Síntoma | Primera sospecha |
|---------|------------------|
| `No open ports detected` | Puerto fijo en vez de `process.env.PORT` |
| `ECONNREFUSED 127.0.0.1` | Falta la variable de conexión en Render (usa `localhost`) |
| `ENETUNREACH` con Supabase | Conexión directa IPv6 → usa el pooler |
| `self-signed certificate` | `sslmode` → `no-verify` con `pg` |
| `MongoServerSelectionError` tras 30 s | IP Access List de Atlas |
| `bad auth` | Usuario/contraseña de Atlas (o contraseña sin codificar) |
| `blocked by CORS policy` | `CORS_ORIGIN` sin el dominio del front |
| Primera petición muy lenta | Render dormido (normal en el plan gratis) |
| Todo falla tras semanas sin uso | Proyecto de Supabase pausado |

---

## 🎯 Cómo responderlo en la entrevista

**¿Has desplegado algo?**
> "Sí. Desplegué una API de NestJS en Render conectada a PostgreSQL en Supabase, y una
> API de Express conectada a MongoDB Atlas. Render compila con `npm run build` y
> arranca con `node dist/...`; las credenciales van en variables de entorno del panel,
> nunca en GitHub. Cada push a main se despliega solo."

**¿Qué problemas tuviste al desplegar?**
> (Respuesta STAR, ver `10`. ⚠️ Cuenta **lo que te pasó de verdad**; esto es un
> ejemplo basado en los errores de `../SUPABASE/02` y `../RENDER/05`.) "Al conectar Render con Supabase la conexión directa fallaba
> con `ENETUNREACH`. Investigando vi que usa IPv6 y Render no lo soporta, así que cambié
> al Session pooler, que es IPv4, y ajusté el SSL. Aprendí a leer los logs y a separar
> si un fallo es de red, de credenciales o de configuración."

**¿Cómo gestionas la configuración entre entornos?**
> "Con variables de entorno: `.env` local ignorado por Git, `.env.example` como
> plantilla, y las variables reales en el panel de Render. El código lee
> `process.env` y falla rápido con un mensaje claro si falta alguna."

**¿Qué es RLS?**
> "Row Level Security de PostgreSQL: políticas que filtran qué filas puede leer o
> modificar cada usuario. En Supabase es imprescindible si el frontend accede
> directamente a la base de datos con la clave pública."
