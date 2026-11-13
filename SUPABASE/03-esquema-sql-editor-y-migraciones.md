# 03 — ESQUEMA, SQL EDITOR Y MIGRACIONES

> Un proyecto nuevo de Supabase es un Postgres **vacío**. Tu API espera encontrar
> la tabla `books`; si no la creas, la primera petición en Render responderá 500
> con `relation "books" does not exist`. Aquí se explica cómo llevar el esquema a
> Supabase y, sobre todo, cómo **mantenerlo** sincronizado con tu código.

---

## 3.1 El punto de partida: tu `schema.sql`

### Ejemplo REAL (`nestjs-postgresql-supabase-render/database/schema.sql`)

```sql
CREATE TABLE books (
  id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  author VARCHAR(100) NOT NULL,
  pages INTEGER NOT NULL CHECK (pages > 0),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

INSERT INTO books (title, author, pages)
VALUES
  ('Cien años de soledad', 'Gabriel García Márquez', 471),
  ('El principito', 'Antoine de Saint-Exupéry', 96),
  ('Crónica de una muerte anunciada', 'Gabriel García Márquez', 122);
```

Repaso rápido (ver carpeta `POSTGRESQL/03`):

| Fragmento | Significado |
|-----------|-------------|
| `GENERATED ALWAYS AS IDENTITY` | Id autoincremental estándar SQL (sustituye a `SERIAL`). `ALWAYS` = no puedes inventarte el id en un `INSERT` |
| `CHECK (pages > 0)` | La BD rechaza libros con 0 páginas aunque la API fallara al validar |
| `TIMESTAMPTZ ... DEFAULT NOW()` | Fecha con zona horaria, rellenada sola |

### En local, Docker lo ejecutaba por ti

### Ejemplo REAL (`nestjs-postgresql-supabase-render/compose.yaml`, fragmento)

```yaml
    volumes:
      - postgres_data:/var/lib/postgresql/data
      # Carga el esquema la primera vez que se crea la base
      - ./database/schema.sql:/docker-entrypoint-initdb.d/schema.sql:ro
```

La imagen oficial de Postgres ejecuta los `.sql` de `/docker-entrypoint-initdb.d/`
**la primera vez** que crea la base. Supabase **no** tiene nada parecido: lo
tienes que ejecutar tú.

---

## 3.2 Opción 1: SQL Editor (lo más directo)

**Dashboard → SQL Editor → New query** → pegar el contenido de `schema.sql` → **Run**.

```
✅ Success. No rows returned
```

Comprobación:

```sql
SELECT * FROM books;               -- los 3 libros
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_name = 'books';        -- debe salir en el schema "public"
```

> 📌 El SQL Editor se conecta como un rol con permisos de administrador: es como
> tu `psql` de Docker, pero en el navegador. Las consultas que escribas se pueden
> guardar como *snippets*.

> ⚠️ Al crear una tabla en `public` por SQL, Supabase te avisará de que **no tiene
> RLS activado**. Para tu API (forma A) no es un problema funcional, pero sí de
> seguridad: la API automática de Supabase la expondría. Lee `05` y añade:
>
> ```sql
> ALTER TABLE books ENABLE ROW LEVEL SECURITY;
> ```
>
> Tu API conecta como `postgres` (dueño/superusuario) y **no** se ve afectada por RLS.

---

## 3.3 Opción 2: Table Editor (visual)

**Dashboard → Table Editor → New table**: formulario tipo hoja de cálculo para
crear columnas, tipos, claves foráneas y editar filas a mano.

| Ventaja | Inconveniente |
|---------|---------------|
| Rápido para explorar y para ver datos | Los cambios **no quedan en tu repositorio** |
| Activa RLS por defecto al crear tablas | Difícil de repetir igual en otro proyecto (staging, otro alumno...) |

> 📌 Úsalo para **mirar** y para tocar datos de prueba. Para **definir** el
> esquema, mejor SQL guardado en Git.

---

## 3.4 Opción 3: migraciones con la CLI de Supabase (lo profesional)

### El problema que resuelven

Tu esquema **evoluciona**: mañana añades `user_id` a `books`, pasado una tabla
`reviews`... Necesitas que **tu PC, Supabase de producción y cualquier
compañero** tengan exactamente el mismo esquema, y saber **en qué orden** se
aplicaron los cambios.

Una **migración** es un archivo SQL con un cambio, con fecha en el nombre, que
se aplica **una sola vez** y en orden:

```
supabase/migrations/
├── 20261101100000_create_books.sql        ← CREATE TABLE books ...
├── 20261105120000_add_user_id_to_books.sql← ALTER TABLE books ADD COLUMN user_id ...
└── 20261110090000_create_reviews.sql      ← CREATE TABLE reviews ...
```

Supabase guarda en una tabla interna cuáles ya se han aplicado, así que
`db push` solo ejecuta las **nuevas**.

### Flujo con la CLI (NO está en tu proyecto)

```bash
npx supabase login                          # abre el navegador para autenticarte
npx supabase init                           # crea la carpeta supabase/ en tu proyecto
npx supabase link --project-ref <REF>       # enlaza con tu proyecto remoto

npx supabase migration new create_books     # crea supabase/migrations/<fecha>_create_books.sql
# ...pegas ahí el CREATE TABLE de schema.sql...

npx supabase db push                        # aplica las migraciones pendientes en Supabase
```

Y si alguien ya había creado cosas desde el panel:

```bash
npx supabase db pull                        # genera una migración con el estado remoto actual
```

### Datos de prueba: `seed.sql`

Los `INSERT` de ejemplo no son "estructura": van en `supabase/seed.sql`, que se
carga en el entorno **local** (`supabase db reset`) pero no en producción.

> 💡 Mismo concepto que las migraciones de **TypeORM / Prisma / Knex** en NestJS:
> el esquema vive versionado junto al código que lo usa.

---

## 3.5 Entorno local: Docker propio vs `supabase start`

| | Tu `compose.yaml` (lo que usas) | `npx supabase start` |
|---|---|---|
| Qué levanta | Solo Postgres 17 | **Todo** Supabase en Docker: Postgres, Studio, Auth, PostgREST, Storage... |
| Peso | 1 contenedor | ~10 contenedores |
| Para la forma A (API con `pg`) | ✅ Suficiente | Funciona, pero sobra |
| Para la forma B (`supabase-js`, Auth, RLS) | ❌ No tienes Auth ni API | ✅ Necesario |

> 📌 Para tu proyecto el `compose.yaml` es la elección acertada: tu API solo
> necesita **un Postgres**, y el de Docker y el de Supabase son el mismo motor.

> ⚠️ Pequeña diferencia a tener en cuenta: en local tu base se llama
> `bootcamp_api` y tu usuario `alumno`; en Supabase la base es `postgres` y el
> usuario `postgres`. Como el código no menciona ninguno de los dos (todo va en
> `DATABASE_URL`), no te afecta.

---

## 3.6 Schemas en Supabase: dónde van tus tablas

Un *schema* de Postgres es como una carpeta de tablas dentro de la base.

| Schema | De quién es | ¿Lo tocas? |
|--------|-------------|------------|
| `public` | **Tuyo**. Aquí van `books`, etc. Es el que expone la API automática | ✅ |
| `auth` | Supabase Auth (`auth.users`...) | Solo leer / referenciar con FK |
| `storage` | Supabase Storage (`storage.objects`, `storage.buckets`) | Solo vía políticas |
| `extensions`, `realtime`, `graphql`, `vault`... | Internos | ❌ |

```sql
-- Un FK hacia los usuarios de Supabase Auth (solo si usas Supabase Auth)
ALTER TABLE books ADD COLUMN user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE;
```

> 💡 Si quisieras que una tabla **nunca** saliera en la API automática, podrías
> crearla en un schema propio (`CREATE SCHEMA private;`) que no esté expuesto.

---

## 3.7 Copias de seguridad y exportación

- **Plan Free**: sin backups automáticos descargables. Haz los tuyos:

```bash
pg_dump "postgresql://postgres.<REF>:<PASSWORD>@aws-0-eu-central-1.pooler.supabase.com:5432/postgres" \
  --schema=public --no-owner > backup.sql
# o con la CLI:
npx supabase db dump -f backup.sql
```

- Ese `backup.sql` se puede cargar en **cualquier** Postgres (tu Docker, Render, AWS RDS…). Esto es lo que significa que Supabase "no te ata".

---

## ✅ Resumen

- Supabase empieza **vacío**: ejecuta tu `schema.sql` (SQL Editor) antes del primer despliegue.
- Table Editor para **ver**, SQL para **definir**.
- Para proyectos que evolucionan: **migraciones** con la CLI (`migration new` + `db push`) versionadas en Git.
- Tus tablas van en `public`; activa RLS en ellas aunque tu API no lo necesite.
- `pg_dump` = tu salida de emergencia de la plataforma.
