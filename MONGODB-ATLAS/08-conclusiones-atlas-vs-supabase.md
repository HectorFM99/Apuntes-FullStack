# 08 — CONCLUSIONES: MONGODB ATLAS (Y COMPARATIVA CON SUPABASE)

> Cierre de la carpeta. Condensa lo importante y lo pone al lado de la otra
> opción de despliegue de base de datos que has usado: **Supabase**
> (`../SUPABASE/` y `../RENDER/CONCLUSIONES-Render-y-Supabase.md`).

---

## 1. La foto final

```
                         ┌─────────────────────── RENDER ───────────────────────┐
  navegador / curl ─────▶│ express-mongo-atlas-render                           │
                         │ npm install && npm run build → npm start             │
                         │ MONGODB_URI, DB_NAME (panel)   PORT (lo pone Render) │
                         └───────────────────────┬──────────────────────────────┘
                                                 │ mongodb+srv:// → DNS SRV → TLS
                                                 ▼
          ┌──────────────────────── MONGODB ATLAS · Proyecto ────────────────────────┐
          │ 🧱 IP Access List (0.0.0.0/0 o IPs de Render)                              │
          │ 🔐 Database User (readWrite@bootcamp_api)                                  │
          │ cluster0 (M0) = replica set  PRIMARY ⇄ SECONDARY ⇄ SECONDARY               │
          │ bootcamp_api.products · bootcamp_api.categories (índice único name_1)      │
          └────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Las 8 ideas que tienes que llevarte

1. **Atlas es MongoDB gestionado, no un backend.** Te da la base de datos (con réplicas, TLS, métricas); la API, la validación y la autenticación de usuarios las sigue poniendo tu Express en Render.

2. **Tu código no cambia entre Docker y Atlas.** `products.ts` y `categories.ts` son idénticos; solo cambia `MONGODB_URI`. Diseñar así (configuración fuera del código) es lo que hace desplegable un proyecto.

3. **Hay dos puertas: red y credenciales.** *Network Access* (IP Access List, cerrada por defecto) y *Database Access* (usuario, contraseña, rol). La mayoría de fallos son una de las dos; el **tiempo** que tarda en fallar te dice cuál (30 s = red, instantáneo = credenciales).

4. **El "cluster" son 3 servidores.** Un replica set con elección automática de primary. Por eso existen `retryWrites` y `w=majority`, y por eso en Atlas funcionan transacciones y change streams que tu Docker standalone no soporta.

5. **`mongodb+srv://` es un nombre, no un servidor.** El driver descubre los nodos por DNS, activa TLS solo y obtiene `authSource`/`replicaSet`. Sin puerto, sin `authSource` manual.

6. **Un `MongoClient` por aplicación, conectado antes de escuchar.** Es lo que ya hace tu `db.ts` + `index.ts`. Mejora recomendada: `serverSelectionTimeoutMS` corto y `appName`.

7. **Mínimo privilegio, sobre todo con `0.0.0.0/0`.** Usuario exclusivo para la API, rol `readWrite` solo sobre `bootcamp_api`, contraseña autogenerada, y la URI solo en Render y en tu `.env`.

8. **Réplicas ≠ backups.** M0 no tiene backups: `mongodump` es tu red de seguridad, y también la herramienta para llevar datos de Docker a Atlas.

---

## 3. Lo que tu proyecto ya hace bien con Atlas

| Práctica | Dónde |
|----------|-------|
| Falla con un mensaje claro si falta `MONGODB_URI` | `src/db.ts` |
| `DB_NAME` con valor por defecto | `src/db.ts` |
| Un único `MongoClient` reutilizado | `src/db.ts` |
| Índice único creado al arrancar (Atlas queda preparado sin pasos manuales) | `connectDb()` |
| No escucha si no hay base de datos | `start()` en `src/index.ts` |
| `PORT` de Render | `src/index.ts` |
| `.env` cargado solo en `dev`; `start` limpio para producción | `package.json` |
| Errores internos al log, mensaje genérico al cliente | middleware de errores |
| Comprueba `typeof === "string"` antes de meter `req.query` en un filtro | `src/routes/products.ts` |
| `E11000` → `409 Conflict` | `src/routes/categories.ts` |

## 4. Lo que mejoraría

| # | Mejora | Por qué |
|---|--------|---------|
| 1 | `serverSelectionTimeoutMS: 5000` y `appName` en el `MongoClient` | Fallar rápido y reconocer la app en Atlas |
| 2 | Quitar `MONGODB_USERNAME`/`MONGODB_PASSWORD` del `.env` y añadir `DB_NAME` | No duplicar secretos; configuración explícita |
| 3 | Usuario de BD exclusivo para Render con `readWrite@bootcamp_api` | Limitar el daño si se filtra la URI |
| 4 | Cambiar la contraseña del Database User si alguna vez se compartió el `.env` | Está en texto plano en tu disco |
| 5 | `CORS_ORIGIN` como variable | Hoy solo acepta `http://127.0.0.1:5500` |
| 6 | Root Directory real en el README | El README dice `02b-express-mongodb` |
| 7 | Índice en `products.category` | Tus filtros `?category=` harán `COLLSCAN` con muchos datos |
| 8 | `mongodump` periódico | M0 no tiene backups |

---

## 5. Atlas vs Supabase: las dos opciones de despliegue de datos

| | **MongoDB Atlas** | **Supabase** |
|---|---|---|
| Motor | MongoDB (documentos, NoSQL) | PostgreSQL (tablas, SQL) |
| Qué es | **DBaaS**: base de datos gestionada | **BaaS**: Postgres + API automática + Auth + Storage + Realtime |
| Tu proyecto | `express-mongo-atlas-render` | `nestjs-postgresql-supabase-render` |
| Librería en tu API | `mongodb` (driver oficial) | `pg` |
| Cadena | `mongodb+srv://user:pass@cluster0.<ID>.mongodb.net` | `postgresql://postgres.<REF>:pass@aws-0-...pooler.supabase.com:5432/postgres` |
| Problema típico desde Render | IP no permitida → timeout | IPv6 → usar el **pooler**; SSL → `no-verify` |
| TLS | Siempre, automático con `+srv` | Opcional; con `pg` hay que configurar `sslmode` |
| Cortafuegos por IP | Cerrado por defecto | Abierto por defecto |
| Alta disponibilidad en Free | ✅ Replica set de 3 nodos | Una instancia |
| Pooling | En el driver (`MongoClient`) | En el driver (`pg.Pool`) + Supavisor |
| Esquema | Flexible (opcional `$jsonSchema`) | Rígido (`CREATE TABLE`, `CHECK`) |
| Relaciones | Embeber o referenciar (`$lookup`) | Claves foráneas y `JOIN` |
| Permisos por fila | ❌ (los pone tu API) | ✅ RLS |
| API para el frontend sin backend | ❌ | ✅ PostgREST + `supabase-js` |
| Auth de usuarios finales | ❌ | ✅ |
| Almacenamiento Free | 512 MB | 500 MB |
| Inactividad en Free | Pausa tras un periodo largo sin conexiones | Pausa tras ~1 semana |
| Backups en Free | ❌ → `mongodump` | ❌ → `pg_dump` |
| Salir de la plataforma | `mongodump` → cualquier MongoDB | `pg_dump` → cualquier Postgres |

### ¿Cuándo elegir cada una?

| Situación | Elección |
|-----------|----------|
| Datos con estructura variable (catálogos con atributos distintos, logs, eventos) | **Atlas** |
| Datos muy relacionados (usuarios ↔ reservas ↔ clases), informes con `JOIN`/`GROUP BY` | **Supabase** |
| Quieres login, archivos y tiempo real sin programarlos | **Supabase** |
| Ya tienes tu API (Express/Nest) y solo necesitas dónde guardar | Cualquiera: elige por el **modelo de datos** |
| Búsqueda de texto avanzada | Atlas Search / full-text de Postgres |
| Transacciones complejas entre muchas entidades | Supabase (Postgres), aunque Atlas también las soporta |

> 📌 La conclusión de fondo: **Render es la misma pieza en los dos casos** (ejecuta
> tu API); lo que eliges es **el modelo de datos**. El proceso de despliegue
> —configuración en variables, un único cliente/pool, abrir la red justa,
> credenciales mínimas, región cercana— es **idéntico** con Atlas y con Supabase.

---

## 6. Preguntas de entrevista (respuesta corta)

**¿Qué es MongoDB Atlas?**
El servicio de MongoDB gestionado en la nube: clusters con réplicas, TLS, cortafuegos por IP, métricas y backups (en planes de pago).

**¿Qué es un replica set y para qué sirve?**
Un grupo de servidores con los mismos datos: un primary acepta escrituras y los secondaries se copian. Si el primary cae, se elige otro. Da alta disponibilidad, no backups.

**¿Qué diferencia hay entre `mongodb://` y `mongodb+srv://`?**
`+srv` usa DNS para descubrir los nodos del cluster y las opciones por defecto, y activa TLS automáticamente; no lleva puerto.

**Tu API en Render no conecta con Atlas y tarda 30 s en fallar. ¿Qué miras?**
Network Access: la IP de salida de Render no está permitida.

**¿Qué significa `w=majority`?**
Que una escritura solo se confirma cuando la tienen la mayoría de nodos, así no se pierde si cae el primary.

**¿Por qué no creas un `MongoClient` en cada petición?**
Cada cliente abre su propio pool de conexiones a los 3 nodos; agotarías el límite de conexiones y sumarías latencia de TLS/autenticación a cada petición.

**¿Cómo evitas la inyección NoSQL?**
Validando tipos antes de construir el filtro (que un campo sea `string`, no un objeto como `{ "$ne": "" }`).

**¿Atlas o Supabase?**
Depende del modelo de datos: documentos flexibles → Atlas; relaciones y SQL, o necesitar Auth/Storage hechos → Supabase.

---

## 7. En una frase

> **Atlas pone tu MongoDB en Internet con réplicas y cifrado de serie; tú solo
> tienes que abrir la puerta justa (red), dar la llave justa (usuario y rol) y
> darle a tu API una única variable: `MONGODB_URI`.**
