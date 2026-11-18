# ☁️🍃 CARPETA MONGODB ATLAS — ÍNDICE DE ESTUDIO

Teoría de **MongoDB Atlas**, la base de datos MongoDB **en la nube**: qué es por
dentro, cómo crear el cluster, cómo conectar tu API desde tu PC y desde
**Render**, cómo llevar tus datos y cómo operarlo. Ejemplos reales **copiados de
tu proyecto** `mongo-atlas-render/express-mongo-atlas-render`.

> 📌 Esta carpeta **no** repite el lenguaje de MongoDB (CRUD, operadores,
> agregaciones, modelado, driver): eso está en `../MONGODB/`. Aquí se trata
> **dónde vive** la base de datos y **cómo se llega a ella** en producción.
>
> ⚠️ Los proyectos **NO** se han modificado. Solo se han copiado fragmentos de su código.
> 🔒 No se ha copiado ningún `.env`. La URI de Atlas aparece como
> `mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net`.

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a aprender |
|---|---------|--------------------|
| 1 | `01-que-es-atlas-y-su-arquitectura.md` | DBaaS, organización → proyecto → cluster, **replica set** (3 nodos), tiers M0/Flex/Dedicated, regiones |
| 2 | `02-crear-cluster-usuarios-y-acceso-de-red.md` | Crear el M0 paso a paso, **usuarios de base de datos** y roles, **Network Access** (IP Access List, `0.0.0.0/0`) |
| 3 | `03-cadena-de-conexion-srv-y-driver.md` | Anatomía de `mongodb+srv://`, DNS SRV, TLS, `retryWrites`, `w=majority`, opciones de `MongoClient`, tu `db.ts` |
| 4 | `04-datos-compass-mongosh-seed-y-migracion.md` | Data Explorer, Compass, `mongosh` contra Atlas, ejecutar tu `seed.js`, `mongodump`/`mongorestore` de Docker a Atlas, índices, validación de esquema |
| 5 | `05-desplegar-con-render-y-operar.md` | Render + Atlas de punta a punta, monitorización, alertas, backups, límites del plan Free |
| 6 | `06-errores-comunes-y-diagnostico.md` | `bad auth`, `MongoServerSelectionError`, `querySrv ENOTFOUND`, timeouts de 30 s... |
| 7 | `07-ejemplos-completos-de-proyectos.md` | Tu proyecto Express + Atlas entero, archivo por archivo |
| 8 | `08-conclusiones-atlas-vs-supabase.md` | Conclusiones y comparativa con la otra opción (Supabase/PostgreSQL) |
| 📋 | `99-CHEATSHEET.md` | Chuleta de comandos y código para consultar rápido |

---

## 🗂️ Proyecto del que se han extraído los ejemplos

| Proyecto | Archivo | Temas que cubre |
|----------|---------|-----------------|
| `mongo-atlas-render/express-mongo-atlas-render/` | `src/db.ts` | `MongoClient` con `MONGODB_URI`, `DB_NAME` con valor por defecto, índice único al arrancar |
| ″ | `src/index.ts` | `await connectDb()` **antes** de `app.listen`, `PORT` de Render, middleware de errores |
| ″ | `src/routes/products.ts`, `categories.ts` | El CRUD que funciona igual en Docker y en Atlas; error `11000` → 409 |
| ″ | `mongo/seed.js` | Datos de prueba (hoy se ejecutan con `docker exec`; aquí: cómo lanzarlos contra Atlas) |
| ″ | `compose.yaml`, `.env.example` | El "gemelo" local de Atlas (Mongo 8 en Docker) |
| ″ | `.env` (solo su **forma**) | URI `mongodb+srv://` sin base de datos ni opciones; variables sobrantes |
| ″ | `README.md` | Tabla de despliegue en Render (`MONGODB_URI`, `DB_NAME`) |

---

## 🚀 Orden de estudio recomendado

1. `01` → qué te está dando Atlas (sin esto, "cluster" y "replica set" son palabras vacías).
2. `02` → las **dos puertas** que hay que abrir: usuario y red. El 80 % de los fallos están aquí.
3. `03` → entender la URI que pegas en `MONGODB_URI`.
4. `04` → meter datos en Atlas y moverlos desde tu Docker.
5. `05` → el despliegue completo con Render y el día a día.
6. `06` → cuando falla, dónde mirar.
7. `07` → repaso con el código real.
8. `08` → la foto final y la comparación con Supabase.

---

## 💡 La idea en una imagen

```
 DESARROLLO (tu PC)                              PRODUCCIÓN
 ┌──────────────────────────┐                    ┌──────────────────────────┐
 │ API Express (tsx watch)  │                    │ API Express en RENDER    │
 │ MONGODB_URI=mongodb://   │                    │ MONGODB_URI=mongodb+srv://│
 │   admin:...@localhost    │                    │   <USUARIO>:...@cluster0 │
 └────────────┬─────────────┘                    └────────────┬─────────────┘
              ▼                                               ▼  TLS, puerto 27017
 ┌──────────────────────────┐          ┌────────────────────────────────────────┐
 │ Docker: mongo:8          │          │ MONGODB ATLAS · cluster0 (M0)          │
 │ 1 servidor, sin TLS      │          │ ┌────────┐ ┌──────────┐ ┌──────────┐   │
 │ contenedor mongodb-      │          │ │PRIMARY │→│SECONDARY │ │SECONDARY │   │
 │ practica                 │          │ └────────┘ └──────────┘ └──────────┘   │
 └──────────────────────────┘          │  🔐 Database Users · 🧱 IP Access List  │
                                       └────────────────────────────────────────┘
        MISMO CÓDIGO (products.ts, categories.ts) — SOLO CAMBIA LA URI
```

> 📌 Igual que con Supabase: **tu código no sabe** si habla con Docker o con
> Atlas. Lo único que cambia es una variable de entorno. Lo que sí cambia es
> todo lo que hay **alrededor**: usuarios, red, TLS, réplicas y límites.
