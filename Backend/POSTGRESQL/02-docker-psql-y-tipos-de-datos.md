# 02 — POSTGRES CON DOCKER, PSQL Y TIPOS DE DATOS

> Antes de escribir SQL necesitas un **servidor de PostgreSQL** funcionando. En el
> bootcamp no lo instalas "a pelo" en Windows: lo levantas en un **contenedor
> Docker** con un archivo `compose.yaml`. Luego te conectas con un **cliente**
> (`psql`, pgAdmin, DBeaver, la extensión de VS Code...) o desde tu API en Node.

---

## 2.1 Cliente y servidor

```
 ┌─────────────────┐     puerto 5432     ┌──────────────────────────┐
 │  CLIENTES       │ ──────────────────► │  SERVIDOR PostgreSQL     │
 │  - psql         │                     │  (dentro de Docker)      │
 │  - pgAdmin      │ ◄────────────────── │                          │
 │  - tu API (pg)  │     filas / errores │  base de datos: academia │
 └─────────────────┘                     │   ├─ tabla planes        │
                                         │   ├─ tabla socios        │
                                         │   └─ ...                 │
                                         └──────────────────────────┘
```

- Un **servidor** puede tener **varias bases de datos** (`academia`, `library`...).
- Cada base de datos tiene **esquemas** (por defecto `public`) y dentro las **tablas**.
- Para conectarte necesitas: **host**, **puerto**, **usuario**, **contraseña** y **nombre de la BD**.

---

## 2.2 Docker Compose para Postgres

### Ejemplo REAL copiado (`practica-sql/compose.yaml`)

> 🔒 La contraseña real se ha sustituido por `TU_PASSWORD`.

```yaml
services:
  postgres:
    image: postgres:17
    container_name: postgres-sql
    environment:
      POSTGRES_DB: academia
      POSTGRES_USER: alumno
      POSTGRES_PASSWORD: TU_PASSWORD
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

**Línea a línea:**

| Línea | Qué hace |
|-------|----------|
| `image: postgres:17` | Usa la imagen oficial de PostgreSQL, **versión 17** |
| `container_name: postgres-sql` | Nombre del contenedor (para `docker exec`, `docker logs`...) |
| `POSTGRES_DB: academia` | Crea automáticamente una BD llamada `academia` la **primera vez** |
| `POSTGRES_USER: alumno` | Crea el usuario `alumno` (superusuario dentro del contenedor) |
| `POSTGRES_PASSWORD` | Su contraseña (**obligatoria** para la imagen oficial) |
| `ports: "5432:5432"` | `PUERTO_DE_TU_PC:PUERTO_DEL_CONTENEDOR` → accedes por `localhost:5432` |
| `volumes: postgres_data:...` | Guarda los datos en un **volumen**: si borras el contenedor, los datos **siguen** |
| `restart: unless-stopped` | Se vuelve a arrancar solo (al reiniciar Docker), salvo que lo pares tú |

### Ejemplo REAL copiado (`pruebas/compose.yaml`)

Es idéntico, pero la base de datos se llama **`library`**:

```yaml
    environment:
      POSTGRES_DB: library
      POSTGRES_USER: alumno
      POSTGRES_PASSWORD: TU_PASSWORD
```

### Ejemplo REAL copiado (`puta/api-books-nest/compose.yaml`)

```yaml
services:
  postgres:
    image: postgres:17
    container_name: practica-sql
    environment:
      POSTGRES_DB: academia
      POSTGRES_USER: alumno
      POSTGRES_PASSWORD: TU_PASSWORD
      DATABASE_URL: postgres://alumno:TU_PASSWORD@postgres:5432/academia
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

> 📌 Aquí aparece la **cadena de conexión** (`DATABASE_URL`). Es la forma de meter
> **todos** los datos de conexión en un solo texto. La usará tu API (archivo `07`).

### Anatomía de una `DATABASE_URL`

```
postgres://alumno:TU_PASSWORD@localhost:5432/academia
└──┬───┘   └─┬──┘ └────┬────┘ └───┬───┘ └┬─┘ └──┬───┘
protocolo  usuario  contraseña   host  puerto  base de datos
```

> ⚠️ **Ojo con el host:**
> - Desde **tu PC** (tu API con `npm run start:dev`) → `localhost`.
> - Desde **otro contenedor** del mismo compose → el **nombre del servicio** (`postgres`).
> Por eso en el compose pone `@postgres:5432` y en el `.env.example` del proyecto
> pone `@localhost:5432`.

### ⚠️ Conflicto de puertos y de nombres

Tus tres `compose.yaml` usan el **puerto 5432**, y dos usan el mismo
`container_name: postgres-sql`. Solo puede haber **uno arrancado a la vez**:

```bash
# ❌ Error típico si ya hay otro Postgres en el 5432:
# Bind for 0.0.0.0:5432 failed: port is already allocated

# ✅ Opción 1: para el otro antes
docker compose down          # (en la carpeta del otro proyecto)

# ✅ Opción 2: cambia el puerto de TU PC en uno de ellos
ports:
  - "5433:5432"              # ahora te conectas a localhost:5433
```

---

## 2.3 Comandos de Docker que vas a usar

```bash
docker compose up -d          # arranca en segundo plano (-d = detached)
docker compose ps             # ¿está corriendo?
docker compose logs -f        # ver los logs (útil si no arranca)
docker compose stop           # parar (conserva el contenedor)
docker compose down           # parar y BORRAR el contenedor (el volumen se queda)
docker compose down -v        # ⚠️ borra también el VOLUMEN = pierdes los datos
```

> 📌 **Las variables `POSTGRES_*` solo se aplican la primera vez** (cuando el
> volumen está vacío). Si cambias `POSTGRES_DB` o la contraseña después, no pasa
> nada... hasta que borres el volumen con `down -v`. Fuente de muchos
> "¡no me deja entrar con la contraseña nueva!".

---

## 2.4 `psql`: el cliente de terminal

`psql` viene **dentro** del contenedor, así que no tienes que instalar nada:

```bash
# Entrar a psql dentro del contenedor
docker exec -it postgres-sql psql -U alumno -d academia
#            │   └── nombre    │   └─ usuario └─ base de datos
#            └ interactivo     └ programa a ejecutar
```

Verás un prompt así:

```
academia=#
```

### Comandos "barra" de psql (meta-comandos, NO son SQL)

| Comando | Qué hace |
|---------|----------|
| `\l` | Lista las bases de datos |
| `\c library` | Cambia a la base de datos `library` |
| `\dt` | Lista las **tablas** |
| `\d clases` | **Describe** la tabla `clases` (columnas, tipos, restricciones) |
| `\di` | Lista los índices |
| `\du` | Lista los usuarios/roles |
| `\x` | Activa/desactiva la vista expandida (útil con filas anchas) |
| `\i archivo.sql` | Ejecuta un archivo `.sql` |
| `\?` | Ayuda de meta-comandos |
| `\h CREATE TABLE` | Ayuda de un comando SQL |
| `\q` | Salir |

### Ejemplo REAL copiado (`Proyecto-SQL/Proyecto-SQL/gimnasio.sql`)

```sql
--\dt y \d clases en psql, o Tables → Refresh en pgAdmin.
```

> 📌 Diferencias importantes:
> - Los **meta-comandos** (`\dt`) **no** llevan `;` y solo existen en `psql`.
> - Las sentencias **SQL** (`SELECT ...`) **siempre** terminan en `;`. Si te
>   olvidas, `psql` cambia el prompt a `academia-#` y se queda **esperando** el resto.

### Ejecutar un archivo `.sql` entero

```bash
# Desde tu PC, metiendo el archivo por la entrada estándar
docker exec -i postgres-sql psql -U alumno -d academia < gimnasio.sql
```

---

## 2.5 Comentarios en SQL

```sql
-- Comentario de una línea (lo más usado)

/* Comentario
   de varias líneas */
```

Tus proyectos los usan para numerar ejercicios (`gimnasio.sql`):

```sql
--Ejercicio B1

--B1.1
CREATE TABLE planes ( ... );
```

Y para explicar columnas (`api-books-nest/database/schema.sql`):

```sql
  id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- el id lo pone PostgreSQL
```

---

## 2.6 Tipos de datos de PostgreSQL

### Números

| Tipo | Rango / uso | Ejemplo en tus proyectos |
|------|-------------|--------------------------|
| `INTEGER` (`INT`) | Enteros hasta ±2.147 millones | `id`, `plazas`, `pages`, `plan_id` |
| `SMALLINT` | Enteros pequeños (±32.767) | — |
| `BIGINT` | Enteros enormes | — (pero `COUNT(*)` devuelve `bigint`, ver archivo `07`) |
| `NUMERIC(p, s)` | Decimal **exacto**: `p` dígitos en total, `s` decimales | `precio_mensual NUMERIC(6,2)` |
| `REAL` / `DOUBLE PRECISION` | Decimal **aproximado** (coma flotante) | — |

```sql
precio_mensual NUMERIC(6,2)
--                    │ └── 2 decimales
--                    └──── 6 dígitos en total → máximo 9999.99
```

> 📌 **Para dinero, siempre `NUMERIC`**, nunca `REAL`/`FLOAT`. Los flotantes tienen
> errores de redondeo (el famoso `0.1 + 0.2 = 0.30000000000000004` de JavaScript).

### Texto

| Tipo | Uso | Ejemplo en tus proyectos |
|------|-----|--------------------------|
| `VARCHAR(n)` | Texto con **máximo** `n` caracteres | `dni VARCHAR(9)`, `email VARCHAR(150)` |
| `TEXT` | Texto **sin límite** | `descripcion TEXT`, `password_hash TEXT` |
| `CHAR(n)` | Texto de longitud **fija** (rellena con espacios) | — (casi no se usa) |

> 📌 En Postgres `VARCHAR(n)` y `TEXT` rinden **igual**. Usa `VARCHAR(n)` cuando el
> límite sea una **regla** (un DNI tiene 9 caracteres) y `TEXT` cuando no lo haya.
> ⚠️ Los textos van entre **comillas simples** `'Ana'`. Las comillas dobles
> `"columna"` son para **nombres** de tablas/columnas, no para valores.

### Fechas y horas

| Tipo | Guarda | Ejemplo en tus proyectos |
|------|--------|--------------------------|
| `DATE` | Solo fecha `2026-11-02` | `fecha_alta DATE`, `fecha_reserva DATE` |
| `TIME` | Solo hora `09:00` | — |
| `TIMESTAMP` | Fecha + hora **sin** zona horaria | `fecha_hora TIMESTAMP` (clases) |
| `TIMESTAMPTZ` | Fecha + hora **con** zona horaria | `created_at TIMESTAMPTZ` (books) |
| `INTERVAL` | Duración (`'2 hours'`, `'7 days'`) | — |

```sql
-- Formato ISO: AAAA-MM-DD y AAAA-MM-DD HH:MM
INSERT INTO clases (nombre, sala, fecha_hora, plazas, monitor_id)
VALUES ('Spinning', 'Sala 1', '2026-11-02 09:00', 12, 1);
```

> 📌 Para fechas de **eventos reales** (cuándo se creó algo) usa `TIMESTAMPTZ`:
> Postgres guarda el instante en UTC y lo convierte a tu zona al leerlo. Es lo que
> hace tu `schema.sql` de `api-books-nest` con `created_at`.

### Otros tipos

| Tipo | Uso | Ejemplo |
|------|-----|---------|
| `BOOLEAN` | `TRUE` / `FALSE` / `NULL` | `activa BOOLEAN NOT NULL` (tarjetas) |
| `UUID` | Identificadores aleatorios | `id UUID DEFAULT gen_random_uuid()` |
| `JSONB` | JSON guardado en binario (consultable) | `atributos JSONB` |
| `TEXT[]` | Arrays (propio de Postgres) | `etiquetas TEXT[]` |

> 📌 `JSONB` es lo más parecido a lo que hacías en Mongo con
> `attributes: Record<string, unknown>` en `api-rastro`. Úsalo para datos **realmente
> variables**, no como excusa para no diseñar tablas.

### `NULL`: la ausencia de valor

```sql
telefono VARCHAR(15),       -- sin NOT NULL → puede quedar vacío (NULL)
```

- `NULL` **no es** `0` ni `''`: significa "**no se sabe / no hay**".
- **Nunca** se compara con `=`:

```sql
-- ❌ Nunca devuelve filas (NULL = NULL es "desconocido", no TRUE)
SELECT * FROM monitores WHERE telefono = NULL;

-- ✅ Correcto
SELECT * FROM monitores WHERE telefono IS NULL;
SELECT * FROM monitores WHERE telefono IS NOT NULL;
```

---

## 2.7 Conversión de tipos (cast)

```sql
SELECT '42'::int;              -- texto → entero (sintaxis propia de Postgres)
SELECT CAST('42' AS INTEGER);  -- lo mismo, sintaxis estándar
SELECT NOW()::date;            -- timestamp → solo la fecha
```

### Ejemplo REAL copiado (`puta/api-books-nest/src/books/books.service.ts`)

```sql
SELECT COUNT(*)::text AS total FROM books
```

```sql
WHERE ($1::text IS NULL OR author ILIKE '%' || $1 || '%')
  AND ($2::int IS NULL OR pages <= $2)
```

> 📌 `$1::text` le dice a Postgres **qué tipo** es el parámetro. Hace falta porque
> si le llega `NULL`, Postgres no puede adivinar si era un texto o un número.

---

## ✅ Resumen rápido

1. Postgres corre como **servidor** en Docker; te conectas con un **cliente**.
2. `compose.yaml`: `image`, `POSTGRES_DB/USER/PASSWORD`, `ports`, **volumen** para no perder datos.
3. `docker compose up -d` / `down` / `down -v` (⚠️ borra datos).
4. `docker exec -it <contenedor> psql -U <usuario> -d <bd>`.
5. `\dt` tablas, `\d tabla` describe, `\q` salir. SQL siempre con `;`.
6. Tipos clave: `INTEGER`, `NUMERIC(p,s)` (dinero), `VARCHAR(n)`, `TEXT`, `DATE`, `TIMESTAMPTZ`, `BOOLEAN`.
7. `NULL` se compara con `IS NULL`, nunca con `=`.
8. Texto con comillas **simples**. `::tipo` para convertir.

👉 **Siguiente:** `03-ddl-crear-tablas-y-restricciones.md`
