# 06 — 🗄️ BASES DE DATOS: SQL VS NOSQL

> Has trabajado con **PostgreSQL** (`Proyecto-SQL`, `api-books-nest`) y con **MongoDB**
> (`api-rastro`, `mongo-api-libros`). Eso es justo lo que te van a preguntar: **cuándo
> usar cada una y por qué**.

---

## 6.1 SQL vs NoSQL ⭐

| | **SQL (relacional)** | **NoSQL (documental, p. ej. MongoDB)** |
|---|---|---|
| Estructura | **Tablas** con filas y columnas | **Colecciones** de documentos (tipo JSON/BSON) |
| Esquema | **Fijo** y estricto (`CREATE TABLE`) | **Flexible** (cada documento puede variar) |
| Relaciones | Claves foráneas + `JOIN` | Documentos **embebidos** o **referencias** |
| Consistencia | Transacciones **ACID** fuertes | ACID en un documento (y multi-documento en versiones modernas) |
| Escalado | Típicamente **vertical** (máquina más potente) | Pensado para **horizontal** (*sharding*) |
| Lenguaje | SQL estándar | API propia (`find`, `aggregate`…) |
| Ejemplos | PostgreSQL, MySQL, SQLite | MongoDB, Redis, Cassandra, DynamoDB |

### ¿Cuándo cada una?

| Usa SQL si… | Usa NoSQL (Mongo) si… |
|-------------|-----------------------|
| Los datos están muy **relacionados** (socios, planes, reservas) | Los datos tienen forma **variable** (atributos distintos por producto) |
| Necesitas **integridad** fuerte (dinero, reservas) | Lees el documento **entero de una vez** |
| Haces consultas complejas con JOINs e informes | Prototipas rápido y el esquema cambia a menudo |

> 📌 Tus propios proyectos lo ilustran: el **gimnasio** (`Proyecto-SQL`) tiene socios,
> planes, tarjetas, monitores… muy relacionados → SQL. El **rastro** (`api-rastro`)
> tiene anuncios con `attributes: Record<string, unknown>` distintos según la
> categoría → Mongo encaja bien.

---

## 6.2 Claves y restricciones (SQL)

**Ejemplo real (`Proyecto-SQL/gimnasio.sql`):**

```sql
CREATE TABLE socios (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    dni VARCHAR(9) UNIQUE NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    fecha_alta DATE NOT NULL,
    plan_id INTEGER NOT NULL REFERENCES planes(id)   -- clave foránea
);

CREATE TABLE planes (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre VARCHAR(50) UNIQUE NOT NULL,
    precio_mensual NUMERIC(6,2) NOT NULL CHECK (precio_mensual >= 0)
);
```

| Restricción | Para qué |
|-------------|----------|
| `PRIMARY KEY` | Identifica cada fila de forma única (no nula) |
| `FOREIGN KEY` / `REFERENCES` | Garantiza que la relación apunta a algo que existe |
| `UNIQUE` | No se repite (email, DNI) |
| `NOT NULL` | Obligatorio |
| `CHECK` | Regla propia (precio ≥ 0) |
| `ON DELETE CASCADE` | Si borras el padre, se borran los hijos |

### Tipos de relación

| Relación | Cómo se modela | Ejemplo |
|----------|----------------|---------|
| **1:1** | FK con `UNIQUE` | Socio ↔ tarjeta (`socio_id UNIQUE`) |
| **1:N** | FK en la tabla "muchos" | Plan → muchos socios (`plan_id` en `socios`) |
| **N:M** | **Tabla intermedia** con dos FK | Monitores ↔ especialidades → tabla `monitor_especialidades` |

---

## 6.3 Normalización

Organizar las tablas para **evitar datos duplicados** y anomalías al actualizar.

```
❌ Sin normalizar
pedidos: id | cliente_nombre | cliente_email | producto1 | producto2 | ...
→ El email del cliente se repite en cada pedido. Si cambia, hay que tocar 100 filas.

✅ Normalizado
clientes: id | nombre | email
productos: id | nombre | precio
pedidos: id | cliente_id (FK) | fecha
pedido_productos: pedido_id (FK) | producto_id (FK) | cantidad
```

| Forma normal | En una frase |
|--------------|--------------|
| **1FN** | Valores **atómicos** (nada de listas en una celda, ni columnas repetidas) |
| **2FN** | 1FN + cada columna depende de **toda** la clave primaria |
| **3FN** | 2FN + no hay dependencias entre columnas que no son clave |

> 📌 A veces se **desnormaliza** a propósito (duplicar un dato) para leer más rápido.
> Es un *trade-off*: rendimiento de lectura vs. complejidad al actualizar.

---

## 6.4 JOINs ⭐

```
INNER JOIN  → solo las filas que coinciden en AMBAS tablas
LEFT JOIN   → TODAS las de la izquierda + las que coincidan de la derecha (si no, NULL)
RIGHT JOIN  → al revés
FULL JOIN   → todas de ambas
```

```sql
-- Socios con el nombre de su plan (solo los que tienen plan)
SELECT s.nombre, p.nombre AS plan
FROM socios s
INNER JOIN planes p ON p.id = s.plan_id;

-- TODOS los planes y cuántos socios tiene cada uno (incluidos los que tienen 0)
SELECT p.nombre, COUNT(s.id) AS total_socios
FROM planes p
LEFT JOIN socios s ON s.plan_id = p.id
GROUP BY p.nombre
ORDER BY total_socios DESC;
```

> 📌 `COUNT(s.id)` y no `COUNT(*)`: con LEFT JOIN, un plan sin socios tiene una fila con
> `s.id = NULL`, y `COUNT(columna)` no cuenta los NULL → da 0 ✅.

### Otras cláusulas que debes manejar

| Cláusula | Para qué |
|----------|----------|
| `WHERE` | Filtra **filas** antes de agrupar |
| `GROUP BY` | Agrupa para usar `COUNT`, `SUM`, `AVG`… |
| `HAVING` | Filtra **grupos** después de agrupar |
| `ORDER BY` | Ordena |
| `LIMIT` / `OFFSET` | Paginación |
| `ILIKE` | Búsqueda sin distinguir mayúsculas (PostgreSQL) |

```sql
-- WHERE vs HAVING
SELECT plan_id, COUNT(*) FROM socios
WHERE fecha_alta >= '2025-01-01'   -- filtra socios
GROUP BY plan_id
HAVING COUNT(*) > 10;              -- filtra planes con más de 10
```

---

## 6.5 Índices ⭐

Un **índice** es como el índice de un libro: permite encontrar filas **sin recorrer
toda la tabla**.

```sql
CREATE INDEX idx_socios_email ON socios(email);
```

```js
// MongoDB
db.listings.createIndex({ category: 1, price: -1 });
```

| ✅ Ventajas | ❌ Inconvenientes |
|------------|------------------|
| Búsquedas, `WHERE`, `JOIN`, `ORDER BY` mucho más rápidos | Ocupan espacio |
| | Las escrituras (`INSERT`/`UPDATE`) son algo más lentas |

> 📌 Las `PRIMARY KEY` y `UNIQUE` crean índice automáticamente. Indexa las columnas por
> las que **filtras o haces JOIN a menudo**. Para analizar una consulta lenta:
> `EXPLAIN ANALYZE` (SQL) o `.explain()` (Mongo).

---

## 6.6 Transacciones y ACID ⭐

Una **transacción** agrupa varias operaciones que deben ocurrir **todas o ninguna**.

```sql
BEGIN;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;
COMMIT;      -- ✅ se guardan las dos
-- ROLLBACK; -- ❌ si algo falla, se deshacen las dos
```

| Letra | Propiedad | Significa |
|-------|-----------|-----------|
| **A** | Atomicity (atomicidad) | Todo o nada |
| **C** | Consistency (consistencia) | La BD pasa de un estado válido a otro válido (se cumplen las restricciones) |
| **I** | Isolation (aislamiento) | Transacciones simultáneas no se pisan |
| **D** | Durability (durabilidad) | Lo confirmado sobrevive a un fallo/reinicio |

---

## 6.7 Inyección SQL (resumen, más en `07`)

```ts
// ❌ NUNCA concatenar lo que manda el usuario
pool.query(`SELECT * FROM books WHERE id = ${req.params.id}`);
// id = "1; DROP TABLE books" 💀

// ✅ Consultas parametrizadas (como en api-books-nest)
pool.query('SELECT ... FROM books WHERE id = $1', [id]);
```

---

## 6.8 MongoDB: embebido vs referenciado ⭐

```js
// EMBEBIDO: el dato va dentro del documento
{
  _id: ObjectId("..."),
  title: "Bicicleta vintage",
  photos: [ { url: "a.jpg", order: 1 }, { url: "b.jpg", order: 2 } ]
}

// REFERENCIADO: guardas el id y lo buscas aparte (o con $lookup)
{
  _id: ObjectId("..."),
  listingId: ObjectId("..."),   // → apunta a otro documento
  amount: 50
}
```

| Embeber cuando… | Referenciar cuando… |
|-----------------|---------------------|
| Relación 1:pocos (fotos de un anuncio) | Relación 1:muchos o N:M (ofertas, usuarios) |
| Siempre se leen **juntos** | Se consultan **por separado** |
| El dato hijo no tiene sentido solo | El hijo crece sin límite (máx. 16 MB por documento) |
| Pocas actualizaciones del dato embebido | El dato se actualiza a menudo y está compartido |

**Ejemplo real (`api-rastro/src/types.ts`)**: un `Listing` **embebe** las `photos` y
también un resumen del vendedor `seller: { id, name }` (patrón mixto: guardas la
referencia `id` + el `name` duplicado para no tener que buscar al usuario en cada
listado). Las ofertas (`Offer`) van en su propia colección.

### Operaciones básicas de Mongo

| SQL | MongoDB |
|-----|---------|
| `SELECT * FROM books WHERE pages < 150` | `db.books.find({ pages: { $lt: 150 } })` |
| `INSERT` | `insertOne` / `insertMany` |
| `UPDATE ... SET` | `updateOne({ _id }, { $set: { ... } })` |
| `DELETE` | `deleteOne` / `deleteMany` |
| `GROUP BY` + `COUNT` | `aggregate([{ $group: ... }])` |
| `JOIN` | `aggregate([{ $lookup: ... }])` |

---

## 6.9 Otras preguntas rápidas

| Pregunta | Respuesta corta |
|----------|-----------------|
| ¿`DELETE` vs `TRUNCATE` vs `DROP`? | Borra filas (con WHERE) / vacía la tabla entera / elimina la tabla |
| ¿Qué es un ORM? | Mapea tablas a objetos (Prisma, TypeORM, Sequelize). Mongoose es un ODM |
| ¿Qué es un *seed*? | Script que mete datos iniciales (como `mongo/seed.js` de `mongo-api-libros`) |
| ¿Qué es una migración? | Script versionado que cambia el esquema de la BD |
| ¿Qué es el *connection pool*? | Conexiones reutilizables (el `Pool` de `pg` en tu API Nest) |
| ¿Problema N+1? | Hacer 1 consulta + N consultas en bucle en vez de un JOIN / `$in` |

---

## 6.10 Bases de datos en la nube (Supabase y MongoDB Atlas) 🆕

> Detalle completo en `12-despliegue-y-cloud-render-supabase-atlas.md`.

| Pregunta | Respuesta corta |
|----------|-----------------|
| ¿Qué es una DBaaS? | Base de datos gestionada por un proveedor: backups, réplicas, TLS y actualizaciones incluidos |
| ¿Qué usaste para Postgres en producción? | **Supabase**, conectado desde mi API NestJS en Render con un `pg.Pool` por el **Session pooler** |
| ¿Y para Mongo? | **MongoDB Atlas** (M0), conectado desde Express con `mongodb+srv://` |
| ¿Cambió tu código al pasar de Docker a la nube? | No: solo la cadena de conexión en una variable de entorno |
| ¿Qué es un replica set? | 3 nodos con los mismos datos (primary + 2 secondaries) y elección automática si cae el primary. Permite transacciones en Mongo |
| ¿Qué es un *connection pooler*? | Intermediario que reparte pocas conexiones reales a la BD entre muchos clientes (Supavisor en Supabase) |
| ¿Qué es RLS? | Row Level Security: políticas por fila en Postgres. Clave en Supabase si el front accede directamente |
| ¿Cómo migras datos de local a la nube? | `pg_dump`/`psql` o el SQL Editor (Postgres); `mongodump`/`mongorestore` o el *seed* con `mongosh` (Mongo) |

---

## 🎯 Cómo responderlo en la entrevista

**¿SQL o NoSQL?**
> "Depende de los datos. SQL tiene esquema fijo, relaciones con claves foráneas y JOINs
> y transacciones ACID: lo uso cuando los datos están muy relacionados o necesito
> integridad, como en el proyecto del gimnasio. MongoDB guarda documentos flexibles: va
> bien cuando la estructura varía o leo el documento completo, como los anuncios de mi
> API del rastro, donde cada categoría tiene atributos distintos."

**¿Qué es la normalización?**
> "Organizar las tablas para no duplicar datos, separando entidades y relacionándolas
> con claves foráneas. Así, si un dato cambia, lo cambio en un solo sitio."

**¿INNER JOIN vs LEFT JOIN?**
> "INNER devuelve solo las filas con coincidencia en ambas tablas. LEFT devuelve todas
> las de la izquierda y, si no hay coincidencia, NULL en las columnas de la derecha.
> Por ejemplo, para listar todos los planes con su número de socios, incluidos los que
> tienen cero, uso LEFT JOIN."

**¿Qué es un índice?**
> "Una estructura que acelera las búsquedas, como el índice de un libro. Mejora
> lecturas, filtros y JOINs, pero ocupa espacio y ralentiza un poco las escrituras, así
> que indexo las columnas por las que más consulto."

**¿Qué es ACID?**
> "Las propiedades de una transacción: atomicidad, todo o nada; consistencia, siempre
> estados válidos; aislamiento, las transacciones concurrentes no se interfieren; y
> durabilidad, lo confirmado no se pierde. El ejemplo típico es una transferencia
> bancaria."

**¿Embeber o referenciar en Mongo?**
> "Embebo cuando son pocos datos que se leen siempre juntos, como las fotos de un
> anuncio. Referencio cuando la relación crece mucho o se consulta por separado, como
> las ofertas."
