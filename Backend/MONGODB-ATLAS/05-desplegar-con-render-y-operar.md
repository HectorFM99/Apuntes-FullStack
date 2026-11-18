# 05 — DESPLEGAR CON RENDER Y OPERAR ATLAS

> El recorrido completo: de "funciona en mi PC con Docker" a "funciona en
> `https://<api>.onrender.com` contra Atlas", y lo que hay que vigilar después.
> Los detalles de Render (build, start, logs, plan Free) están en `../RENDER/`;
> aquí se ve la parte que toca a Atlas.

---

## 5.1 La secuencia completa

```
 1. Atlas: cluster M0 (región = la de Render)                      → archivo 02
 2. Atlas: Database User con readWrite@bootcamp_api                → archivo 02
 3. Atlas: Network Access → tu IP + 0.0.0.0/0                      → archivo 02
 4. Local: probar la API contra Atlas (cambiar MONGODB_URI en .env)→ 5.2
 5. Atlas: cargar datos (seed.js o mongorestore)                   → archivo 04
 6. GitHub: push (sin .env)
 7. Render: Web Service + variables MONGODB_URI y DB_NAME          → 5.3
 8. Probar con curl la URL pública                                  → 5.4
```

---

## 5.2 Paso intermedio: tu API local contra Atlas

Antes de desplegar, conecta tu API **local** a Atlas. Si falla aquí, fallará en
Render, y en local los errores son más fáciles de ver.

```
# .env (local) — temporalmente apuntando a Atlas
MONGODB_URI=mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net/?appName=api-products
DB_NAME=bootcamp_api
```

```bash
npm run dev
# Conectado a MongoDB
# Servidor escuchando en http://localhost:3000

curl -i http://localhost:3000/api/products
```

| Resultado | Diagnóstico |
|-----------|-------------|
| `Conectado a MongoDB` y `[]` | ✅ Conecta, pero Atlas está vacío → archivo `04` |
| `Conectado a MongoDB` y tus productos | ✅ Todo listo para desplegar |
| ~30 s de espera y `MongoServerSelectionError` | 🧱 Tu IP no está en Network Access |
| `bad auth : authentication failed` | 🔐 Usuario o contraseña mal (o caracteres sin codificar) |

> 💡 Mantén **dos** archivos de ejemplo mentales: `.env` con Docker para el día a
> día (rápido, sin límites, sin Internet) y la URI de Atlas solo cuando quieras
> probar contra la nube. Así no llenas Atlas de datos de prueba.

---

## 5.3 Render: los campos

### Ejemplo REAL (`express-mongo-atlas-render/README.md`, sección de despliegue)

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

> ⚠️ El Root Directory del README (`02b-express-mongodb`) es el nombre original
> del ejercicio. En tu repo la carpeta es `mongo-atlas-render/express-mongo-atlas-render`:
> pon la ruta **real** dentro de tu repositorio.

| Variable en Render | Valor |
|--------------------|-------|
| `MONGODB_URI` | La de Atlas, con el usuario de producción (`api-render`) |
| `DB_NAME` | `bootcamp_api` (opcional: el código ya usa ese valor por defecto, pero explícito es mejor) |
| `NODE_VERSION` | La de tu PC (el README pide Node 24 LTS) |

### Por qué `npm start` no necesita el `.env`

### Ejemplo REAL (`express-mongo-atlas-render/package.json`, scripts)

```json
"dev": "tsx watch --env-file=.env src/index.ts",
"build": "tsc",
"start": "node dist/index.js",
```

Solo `dev` carga el `.env` (`--env-file`). `start` ejecuta el JS compilado y lee
`process.env` tal cual → en Render, las variables del panel. No hace falta el
truco `existsSync('.env')` del proyecto NestJS.

---

## 5.4 Comprobar el despliegue

En los logs de Render debes ver:

```
==> Running 'npm start'
> node dist/index.js
Conectado a MongoDB
Servidor escuchando en http://localhost:10000
==> Your service is live 🎉
```

> 📌 El mensaje dice `localhost:10000` porque el `console.log` está escrito así;
> Render escucha en el puerto 10000 de todas las interfaces y lo publica en
> `https://<api>.onrender.com`. No es un error.

```bash
curl -i https://<api>.onrender.com/api/products                       # 200
curl -i -X POST https://<api>.onrender.com/api/categories \
  -H "Content-Type: application/json" -d '{"name":"cocina"}'          # 201
curl -i -X POST https://<api>.onrender.com/api/categories \
  -H "Content-Type: application/json" -d '{"name":"cocina"}'          # 409 (índice único en Atlas)
```

Y en Atlas → **Data Explorer**, el documento `cocina` aparece en `categories`.

---

## 5.5 Monitorización en Atlas

En el cluster → **Metrics** (y *Real Time* en planes superiores):

| Métrica | Qué te dice | Qué mirar |
|---------|-------------|-----------|
| **Connections** | Conexiones abiertas | Si sube sin parar → estás creando clientes de más |
| **Opcounters** | Operaciones por segundo (insert, query, update...) | Picos = tráfico; en M0 hay techo |
| **Network** | Bytes de entrada/salida | Consultas que devuelven demasiados datos (falta proyección o `limit`) |
| **Data Size** | Lo que ocupan tus datos | Cerca de 512 MB en M0 → toca limpiar o subir de tier |

### Alertas

**Project → Alerts**: Atlas avisa por email (u otras integraciones) cuando, por
ejemplo, las conexiones superan un umbral, el almacenamiento se acerca al límite
o hay un cambio de primary. Algunas vienen activadas por defecto.

### Relación con el *sleep* de Render

Cuando Render duerme tu API (15 min sin tráfico), el proceso se para y sus
conexiones a Atlas se cierran. Al despertar, `connectDb()` vuelve a conectar:
eso **suma** al *cold start* (resolver SRV + TLS + autenticación ≈ 1-2 s).

---

## 5.6 Backups

| Plan | Backups |
|------|---------|
| **M0 (Free)** | ❌ Sin backups en la nube → `mongodump` manual (archivo `04`, 4.4) |
| Flex | Snapshots diarios |
| Dedicated | **Cloud Backups** con snapshots programados y restauración a un momento concreto (*point-in-time*) |

> ⚠️ El replica set **no es un backup**: si haces `deleteMany({})` por error, la
> orden se replica a los 3 nodos en milisegundos. Las réplicas te protegen de que
> **se caiga un servidor**, no de **tus errores**.

---

## 5.7 Límites del plan Free que conviene recordar

> ⚠️ Compruébalos en la documentación; cambian con el tiempo.

| Límite | Valor aproximado | Efecto |
|--------|------------------|--------|
| Almacenamiento | 512 MB | Al llegar, las escrituras fallan |
| Conexiones | ~500 simultáneas | Un `MongoClient` por app está muy lejos |
| Operaciones | Throughput limitado (recursos compartidos) | Picos fuertes se ralentizan |
| Clusters M0 | 1 por proyecto | Otro M0 = otro proyecto |
| Inactividad | Atlas puede **pausar** un M0 sin conexiones durante mucho tiempo (del orden de 60 días) | Hay que reanudarlo desde el panel |
| Backups | No | `mongodump` |

---

## 5.8 Seguridad en producción: resumen práctico

- [ ] Usuario de BD **exclusivo** para Render, con `readWrite` **solo** en `bootcamp_api`.
- [ ] Contraseña autogenerada; la URI **solo** en el panel de Render y en tu `.env` (ignorado por Git).
- [ ] Si usas `0.0.0.0/0`, lo anterior es **obligatorio**.
- [ ] Nunca devolver errores internos de Mongo al cliente (tu middleware ya responde `"Error interno del servidor"`).
- [ ] Validar entradas (`validateProduct`) — evita, entre otras cosas, **inyección de operadores** (que alguien envíe `{ "$gt": "" }` en vez de un texto).
- [ ] Si una URI se filtra: Atlas → Database Access → **borrar o cambiar la contraseña** de ese usuario → actualizar Render.

> 📌 Sobre la **inyección NoSQL**: en tu `products.ts`, `filter.category = req.query.category`
> solo se asigna si `typeof req.query.category === "string"`. Esa comprobación
> evita que alguien mande `?category[$ne]=x` y lo convierta en un operador.
> Es el equivalente Mongo a los `$1` de Postgres.

---

## ✅ Resumen

- Orden: cluster → usuario → red → probar en local → datos → push → Render → curl.
- En Render: `MONGODB_URI`, `DB_NAME` y Root Directory **real**.
- `npm start` no necesita `.env`; el `.env` solo lo carga `dev`.
- Vigila **Connections** y **Data Size**; configura alertas.
- M0 no tiene backups: `mongodump`. Las réplicas no son backup.
