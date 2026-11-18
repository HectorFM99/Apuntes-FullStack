# 01 — INTRODUCCIÓN: NoSQL, DOCUMENTOS Y DOCKER

> **MongoDB** es una base de datos **NoSQL orientada a documentos**. En vez de guardar
> filas en tablas con columnas fijas, guarda **documentos** con forma de objeto JSON
> dentro de **colecciones**.

---

## 1.1 SQL vs NoSQL (MongoDB)

| | SQL (PostgreSQL, MySQL) | MongoDB (NoSQL documental) |
|---|---|---|
| Unidad de datos | **Fila** | **Documento** (objeto tipo JSON) |
| Agrupación | **Tabla** | **Colección** |
| Esquema | Fijo: columnas definidas antes (`CREATE TABLE`) | Flexible: cada documento puede tener campos distintos |
| Relaciones | Claves ajenas (`FOREIGN KEY`) + `JOIN` | Embeber datos dentro del documento, o guardar el id (referencia) |
| Integridad | La BD comprueba claves ajenas, tipos, `NOT NULL` | La comprueba **tu aplicación** (o validadores opcionales) |
| Lenguaje | SQL (`SELECT ... WHERE ...`) | Objetos JS (`find({ ... })`) |
| Ideal para | Datos muy relacionados y estrictos (pagos, contabilidad) | Datos semiestructurados, atributos variables, lectura de "todo de golpe" |

### Ejemplo REAL copiado (`mongo-rastro/mongo-rastro/respuestas.md`)

Tu propia respuesta de por qué elegir MongoDB para la web del Rastro:

```md
## A1. ¿Por qué MongoDB aquí?

1. **Atributos variables:** Evita múltiples columnas NULL (como ocurriría en SQL con 13+ atributos de diferentes categorías) y permite añadir nuevos atributos o categorías sin modificar el esquema.

2. **Modelo flexible:** Cada categoría y anuncio almacena sus propios campos dentro de atributos (ej. móviles, muebles).
```

Y la contraparte, cuándo **NO** usar Mongo:

```md
2. **Cuándo usar SQL:** Para datos estrictos y altamente relacionados (pagos, finanzas) que exigen transacciones y restricciones rígidas.
```

> 📌 No es "Mongo es mejor" o "SQL es mejor": es **elegir según los datos**.

---

## 1.2 Base de datos → colección → documento

```
Servidor MongoDB (contenedor Docker)
└── Base de datos "rastro"
    ├── Colección "anuncios"
    │   ├── { _id: ..., titulo: "Bicicleta...", precio: 300, ... }   ← documento
    │   └── { _id: ..., titulo: "Google Pixel 7", precio: 260, ... }
    ├── Colección "usuarios"
    ├── Colección "ofertas"
    └── Colección "valoraciones"
```

| Concepto Mongo | Equivalente SQL |
|---|---|
| Base de datos | Base de datos |
| Colección | Tabla |
| Documento | Fila |
| Campo | Columna |
| `_id` | Clave primaria |

> 📌 Las bases de datos y colecciones **se crean solas** la primera vez que insertas
> un documento. No hay `CREATE TABLE`.

### Un documento REAL (`mongo-rastro/mongo-rastro/rastro.js`)

```js
db.anuncios.insertOne({
  titulo: "Bicicleta de montaña Orbea",
  descripcion: "Revisada en taller.",
  precio: 300,
  categoria: "Deporte",
  estado: "DISPONIBLE",
  ciudad: "Castellón",
  fechaPublicacion: new Date("2026-10-27"),
  visitas: 0,
  vendedor: {
    id: ObjectId("660000000000000000000004"),
    nombre: "Hugo Sanz"
  },
  fotos: [{ url: "https://img.rastro.es/bici-1.jpg", orden: 1 }],
  atributos: { talla: "L", tipo: "montaña", pulgadasRueda: 29 }
})
```

**Fíjate en lo que cabe en UN documento:**

```
precio: 300                    → número
fechaPublicacion: new Date()   → fecha (tipo Date, no texto)
vendedor: { id, nombre }       → objeto ANIDADO (subdocumento)
fotos: [ {...} ]               → ARRAY de objetos
atributos: { talla, ... }      → campos libres que cambian según la categoría
```

Compáralo con el otro anuncio del mismo archivo (un móvil):

```js
  atributos: { marca: "Google", modelo: "Pixel 7", almacenamientoGB: 128, bateriaPct: 92 }
```

> 📌 La bici tiene `talla` y `pulgadasRueda`; el móvil tiene `almacenamientoGB`.
> **En la misma colección**, sin tocar ningún esquema. Esa es la flexibilidad de Mongo.

---

## 1.3 BSON: el formato interno

MongoDB guarda los documentos en **BSON** (*Binary JSON*). Se parece a JSON pero
tiene **más tipos**:

| Tipo BSON | Ejemplo en `mongosh` | En JSON normal... |
|---|---|---|
| String | `"Valencia"` | igual |
| Number (int/double) | `300`, `29.99` | igual |
| Boolean | `true` | igual |
| Date | `new Date("2026-10-27")` | ❌ no existe (sería texto) |
| ObjectId | `ObjectId("6600...0004")` | ❌ no existe |
| Array | `[1, 2, 3]` | igual |
| Objeto | `{ talla: "L" }` | igual |
| null | `null` | igual |

> ⚠️ **Los tipos importan.** `300` (número) y `"300"` (texto) **no son lo mismo** para
> Mongo. Lo verás en el apartado 2.7 con un ejemplo real tuyo.

---

## 1.4 `_id` y `ObjectId`

Todo documento tiene un campo **`_id` único**. Si no lo pones, Mongo genera un
**`ObjectId`** automáticamente.

```js
ObjectId("670000000000000000000004")
//        └──────── 24 caracteres hexadecimales (12 bytes) ────────┘
```

- Un `ObjectId` real contiene la **fecha de creación** + datos aleatorios + contador,
  así que nunca se repite.
- En tus ejercicios se usan ids "bonitos" (`6600...0001`, `6700...0001`) para que sea
  fácil copiarlos: `66...` = usuarios, `67...` = anuncios, `68...` = ofertas.

### ❌/✅ El error más típico: buscar el id como texto

**Ejemplo REAL (`mongo-rastro/mongo-rastro/rastro.js`, ejercicio B4.6):**

```js
// B4.6
db.anuncios.findOne({ _id: "670000000000000000000003" })
```

Y tu explicación en `respuestas.md`:

```md
- **B4.6 (Id como texto):** Devuelve null al comparar un string con un ObjectId.
```

```js
// ❌ Texto: no encuentra nada → null
db.anuncios.findOne({ _id: "670000000000000000000003" })

// ✅ ObjectId: lo encuentra
db.anuncios.findOne({ _id: ObjectId("670000000000000000000003") })
```

> 📌 Esto mismo pasará en tu API: el id llega en la URL **como texto** y hay que
> convertirlo con `new ObjectId(id)`. Lo verás en el `06`.

### Comparar dos ObjectId (`api-rastro/api-rastro/respuestas.md`)

```md
Dos `ObjectId` son dos objetos distintos en memoria, aunque representen el mismo id, así que `===` compara referencias y daría `false`. Por eso se usa `listing.seller.id.equals(buyer._id)`, que compara el valor del id.
```

```ts
// ❌ Siempre false (son dos objetos distintos)
listing.seller.id === buyer._id

// ✅ Compara el valor
listing.seller.id.equals(buyer._id)
```

---

## 1.5 Arrancar MongoDB con Docker Compose

No instalas MongoDB en tu ordenador: lo levantas en un **contenedor Docker**.

### Ejemplo REAL copiado (`practica-MongoDB/compose.yaml`)

```yaml
services:
  mongodb:
    image: mongo:8
    container_name: mongodb-practica
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin_dev_123
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    restart: unless-stopped

volumes:
  mongodb_data:
```

**Línea a línea:**

| Línea | Qué hace |
|---|---|
| `image: mongo:8` | Usa la imagen oficial de MongoDB versión 8 |
| `container_name: mongodb-practica` | Nombre fijo del contenedor (para `docker start/exec`) |
| `MONGO_INITDB_ROOT_USERNAME/PASSWORD` | Crea el usuario administrador **la primera vez** que arranca |
| `ports: "27017:27017"` | Puerto de tu PC : puerto del contenedor. **27017** es el puerto por defecto de Mongo |
| `volumes: mongodb_data:/data/db` | Guarda los datos en un volumen → **no se pierden** al parar el contenedor |
| `restart: unless-stopped` | Se vuelve a arrancar solo (salvo que lo pares tú) |

> 📌 El `compose.yaml` de `mongo-api-libros/.../02b-express-mongodb/` es igual pero sin
> `restart` y con `container_name: practica-mongodb`.

### Comandos que vas a usar

```bash
docker compose up -d        # arranca en segundo plano
docker compose ps           # ver si está corriendo
docker compose stop         # parar (los datos se quedan)
docker compose down         # parar y borrar el contenedor (el volumen se queda)
docker compose down -v      # ⚠️ también borra el VOLUMEN = borra los datos
docker start mongodb-practica   # arrancar un contenedor que ya existía
```

### ⚠️ Error típico: `Authentication failed`

Del `README.md` de `02b-express-mongodb`:

```md
Si sale `Authentication failed`, es que el volumen de datos ya existía sin contraseña. Bórralo con `docker compose down -v` y vuelve a arrancar.
```

> 📌 Las variables `MONGO_INITDB_*` solo se aplican cuando el volumen está **vacío**.
> Si el volumen ya tenía datos de antes, el usuario no se crea.

---

## 1.6 `mongosh`: la consola de MongoDB

`mongosh` es la shell interactiva de Mongo. Es JavaScript: puedes usar variables,
`new Date()`, bucles...

### Entrar en `mongosh` dentro del contenedor

```bash
docker exec -it mongodb-practica mongosh -u admin -p admin_dev_123 --authenticationDatabase admin
```

### Comandos básicos

```js
show dbs                    // lista las bases de datos
use rastro                  // cambia a (o "crea") la base de datos rastro
show collections            // colecciones de la base actual
db                          // muestra en qué base estás
db.anuncios.find()          // todos los documentos de "anuncios"
db.anuncios.countDocuments()// cuántos hay
exit                        // salir
```

> 📌 Tu archivo `rastro.js` empieza justamente así:
>
> ```js
> // use rastro
> ```
>
> `db` es "la base de datos actual"; `db.anuncios` es "la colección anuncios".

### Ejecutar un archivo `.js` con `mongosh` (script de datos)

**Ejemplo REAL copiado (`mongo-api-libros/.../02b-express-mongodb/package.json`):**

```json
"seed": "docker cp mongo/seed.js mongodb-practica:/tmp/seed.js && docker exec mongodb-practica mongosh --quiet -u admin -p admin_dev_123 --authenticationDatabase admin /tmp/seed.js"
```

```
docker cp mongo/seed.js mongodb-practica:/tmp/seed.js   → copia el script DENTRO del contenedor
docker exec mongodb-practica mongosh ... /tmp/seed.js   → lo ejecuta con mongosh
```

Y el script (`mongo/seed.js`) usa `db.getSiblingDB` para elegir base de datos (en un
script no puedes escribir `use`):

```js
const database = db.getSiblingDB("bootcamp_api");

database.products.deleteMany({});
database.products.insertMany([
  { name: "Teclado", price: 29.99, category: "perifericos" },
  { name: "Ratón", price: 15, category: "perifericos" },
  { name: "Silla", price: 49.5, category: "muebles" }
]);
```

> 📌 `deleteMany({})` + `insertMany` = script **repetible**: lo lanzas cuantas veces
> quieras y siempre deja los mismos datos.

---

## 1.7 Cadena de conexión (URI)

Para conectarte desde una app (Node, Compass...) se usa una **URI**:

**`.env.example` REAL (`mongo-api-libros/.../02b-express-mongodb/.env.example`):**

```env
MONGODB_URI=mongodb://admin:admin_dev_123@localhost:27017/?authSource=admin
DATABASE_URL=mongodb://admin:admin_dev_123@localhost:27017/?authSource=admin
DB_NAME=bootcamp_api
```

```
mongodb://  admin : admin_dev_123 @ localhost : 27017 / ?authSource=admin
└protocolo┘ └user┘ └─ password ─┘ └─ host ─┘ └puerto┘   └─ BD donde está el usuario
```

**`.env.example` de `api-rastro/api-rastro/`** (sin contraseña real, como debe ser):

```env
MONGODB_URI=mongodb://usuario:contraseña@localhost:27017/?authSource=admin
DB_NAME=rastro
```

> ⚠️ El `.env` real **nunca** se sube a Git (está en `.gitignore`). Se sube el
> `.env.example` como plantilla y cada uno hace `cp .env.example .env`.

---

## ✅ Resumen rápido

1. MongoDB = **documentos** (objetos tipo JSON) dentro de **colecciones** dentro de **bases de datos**.
2. Esquema **flexible**: cada documento puede tener campos distintos.
3. Internamente es **BSON**: tiene `Date` y `ObjectId`, y **los tipos importan**.
4. `_id` es único; si no lo das, Mongo crea un `ObjectId`. Búscalo con `ObjectId("...")`, no con texto.
5. Se arranca con `docker compose up -d` (puerto **27017**, volumen para no perder datos).
6. `mongosh` = consola JS: `use`, `show collections`, `db.coleccion.find()`.
7. La app se conecta con una URI `mongodb://user:pass@host:27017/?authSource=admin` guardada en `.env`.

👉 **Siguiente:** `02-crud-en-mongosh.md`
