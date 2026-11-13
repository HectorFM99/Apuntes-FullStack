# 06 — BASE DE DATOS (PostgreSQL con `pg`) Y CONFIGURACIÓN

> Tu API guarda los libros en **PostgreSQL**. Para hablar con la base de datos usa la
> librería **`pg`** (sin ORM), con un **`Pool`** de conexiones que Nest inyecta en los
> servicios. La dirección de la BD se lee de una **variable de entorno**
> (`DATABASE_URL`) que viene del archivo **`.env`**.

---

## 6.1 Piezas que intervienen

```
compose.yaml      → levanta PostgreSQL en Docker
database/schema.sql → crea las tablas y datos de ejemplo
.env              → DATABASE_URL=postgres://usuario:contraseña@host:puerto/bd
main.ts           → process.loadEnvFile() mete el .env en process.env
DatabaseModule    → crea el Pool con process.env.DATABASE_URL y lo exporta
BooksService      → pide el Pool y hace this.pool.query(...)
```

---

## 6.2 PostgreSQL con Docker Compose

### Ejemplo REAL (`api-books-nest/compose.yaml`) — contraseña sustituida por un marcador

```yaml
services:
  postgres:
    image: postgres:17
    container_name: practica-sql
    environment:
      POSTGRES_DB: academia
      POSTGRES_USER: alumno
      POSTGRES_PASSWORD: CAMBIAME
      DATABASE_URL: postgres://alumno:CAMBIAME@postgres:5432/academia
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

| Clave | Significado |
|-------|-------------|
| `image: postgres:17` | Imagen oficial de PostgreSQL 17 |
| `POSTGRES_DB/USER/PASSWORD` | Crea la base `academia` y el usuario `alumno` |
| `ports: "5432:5432"` | Puerto de tu PC : puerto del contenedor |
| `volumes` | Los datos sobreviven aunque borres el contenedor |
| `restart: unless-stopped` | Se reinicia solo salvo que lo pares tú |

```bash
docker compose up -d        # arrancar en segundo plano
docker compose ps           # ver si está corriendo
docker compose down         # parar (los datos se quedan en el volumen)
```

> ⚠️ Fíjate en la diferencia de **host**:
> - Desde **tu PC** (donde corre `npm run start:dev`) → `localhost:5432`.
> - Desde **otro contenedor** de la misma red de Compose → `postgres:5432` (el nombre
>   del servicio). Por eso tu `.env.example` usa `localhost`.

---

## 6.3 El esquema: `schema.sql`

### Ejemplo REAL copiado (`api-books-nest/database/schema.sql`)

```sql
CREATE TABLE books (
  id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- el id lo pone PostgreSQL
  title VARCHAR(200) NOT NULL,
  author VARCHAR(100) NOT NULL,
  pages INTEGER NOT NULL CHECK (pages > 0),            -- la base de datos también valida
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()        -- fecha de creación automática
);

INSERT INTO books (title, author, pages)
VALUES
  ('Cien años de soledad', 'Gabriel García Márquez', 471),
  ('El principito', 'Antoine de Saint-Exupéry', 96),
  ('Crónica de una muerte anunciada', 'Gabriel García Márquez', 122);

CREATE TABLE users (
  id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email VARCHAR(255) NOT NULL UNIQUE,      -- no puede haber dos iguales
  password_hash TEXT NOT NULL,             -- el hash, nunca la contraseña
  role VARCHAR(20) NOT NULL DEFAULT 'customer'
    CHECK (role IN ('customer', 'employee'))  -- solo estos dos roles
);
```

Para ejecutarlo dentro del contenedor:

```bash
docker exec -i practica-sql psql -U alumno -d academia < database/schema.sql
```

> 📌 `GENERATED ALWAYS AS IDENTITY` → el `id` lo pone PostgreSQL. Por eso el
> `CreateBookDto` **no tiene** `id`: el cliente no lo envía (y si lo envía,
> `forbidNonWhitelisted` lo rechaza).

---

## 6.4 Variables de entorno y `.env`

### Ejemplo REAL copiado (`api-books-nest/.env.example`)

```bash
DATABASE_URL=postgres://alumno:CAMBIAME@localhost:5432/academia
```

**Formato de la URL de conexión:**

```
postgres://alumno:CAMBIAME@localhost:5432/academia
└──┬───┘   └─┬──┘ └──┬───┘ └───┬───┘ └┬─┘ └──┬───┘
protocolo  usuario contraseña  host  puerto  base de datos
```

| Archivo | ¿Se sube a Git? | Contenido |
|---------|-----------------|-----------|
| `.env` | ❌ **NUNCA** (está en `.gitignore`) | Los valores **reales** |
| `.env.example` | ✅ Sí | Plantilla con valores de ejemplo |

### Ejemplo REAL copiado (`api-books-nest/.gitignore`, fragmento)

```bash
# dotenv environment variable files
.env
.env.development.local
.env.test.local
.env.production.local
.env.local
```

**Cómo se carga** (`api-books-nest/src/main.ts`):

```ts
process.loadEnvFile(); // lee el .env y guarda sus valores en process.env
```

- Es **nativo de Node** (desde v20.12 / v21.7): no hace falta instalar `dotenv`.
- Si el archivo `.env` **no existe**, lanza un error al arrancar.
- Después, en cualquier parte: `process.env.DATABASE_URL`, `process.env.PORT`.

> 📌 Pasos al clonar el proyecto: `cp .env.example .env` → pon tu contraseña → `npm i`
> → `docker compose up -d` → ejecutar `schema.sql` → `npm run start:dev`.

### Alternativa: `@nestjs/config` (ejemplo propio, no está en tu proyecto)

```bash
npm i @nestjs/config
```

```ts
// app.module.ts
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true }), /* ... */],
})
export class AppModule {}

// en cualquier servicio
import { ConfigService } from '@nestjs/config';

constructor(private readonly config: ConfigService) {}

const url = this.config.getOrThrow<string>('DATABASE_URL'); // error si falta
```

| | `process.loadEnvFile()` | `@nestjs/config` |
|--|------------------------|------------------|
| Instalación | Nada | Paquete extra |
| Inyectable / testeable | ❌ | ✅ `ConfigService` |
| Validar variables | A mano | Con esquema (Joi / función `validate`) |
| Varios archivos (`.env.test`) | A mano | `envFilePath` |

---

## 6.5 El `Pool` de `pg`

Un **Pool** mantiene varias conexiones abiertas y las **reutiliza**. Abrir una conexión
nueva en cada petición sería lentísimo.

### Ejemplo REAL copiado (`api-books-nest/src/database/database.module.ts`)

```ts
import { Module } from '@nestjs/common';
import { Pool } from 'pg';

@Module({
  providers: [
    {
      provide: Pool, // cuando alguien pida un Pool...
      // ...NestJS ejecuta esta función una vez y le da el resultado
      useFactory: () => new Pool({ connectionString: process.env.DATABASE_URL }),
    },
  ],
  exports: [Pool], // otros módulos pueden pedir el Pool
})
export class DatabaseModule {}
```

> 📌 `new Pool()` **no conecta todavía**: conecta en la primera `query`. Por eso, si
> `DATABASE_URL` está mal, la app arranca igual y el error aparece en la primera
> petición (un 500). La versión `DatabaseService` (ver `04`) evita eso comprobando la URL
> al arrancar.

---

## 6.6 Consultas: `pool.query` y parámetros `$1, $2...`

```ts
const result = await this.pool.query<TipoDeFila>(sql, [valor1, valor2]);
result.rows       // array de filas (objetos)
result.rowCount   // nº de filas afectadas
```

### Ejemplo REAL copiado (`api-books-nest/src/books/books.service.ts`)

```ts
async create(dto: CreateBookDto): Promise<Book> {
  const result = await this.pool.query<BookRow>(
    `INSERT INTO books (title, author, pages)
     VALUES ($1, $2, $3)
     RETURNING id, title, author, pages, created_at`,
    [dto.title, dto.author, dto.pages],
  );
  return toBook(result.rows[0]);
}
```

- `$1, $2, $3` son **huecos**; los valores van en el **array** en el mismo orden.
- `RETURNING ...` hace que el `INSERT` devuelva la fila creada (con su `id` y fecha) sin
  otra consulta.
- `query<BookRow>` le dice a TypeScript qué forma tienen las filas.

### ⚠️ Inyección SQL: NUNCA concatenes

```ts
// ❌ PELIGRO: si author = "x'; DROP TABLE books; --" ... adiós tabla
this.pool.query(`SELECT * FROM books WHERE author = '${author}'`);

// ✅ Consulta parametrizada: pg envía el valor APARTE del SQL
this.pool.query('SELECT * FROM books WHERE author = $1', [author]);
```

---

## 6.7 Filtros opcionales en SQL

### Ejemplo REAL copiado (`api-books-nest/src/books/books.service.ts`)

```ts
async findAll(author?: string, maxPages?: number): Promise<Book[]> {
  const result = await this.pool.query<BookRow>(
    `SELECT id, title, author, pages, created_at
     FROM books
     WHERE ($1::text IS NULL OR author ILIKE '%' || $1 || '%')
       AND ($2::int IS NULL OR pages <= $2)
     ORDER BY id`,
    [author || null, maxPages ?? null],
  );
  return result.rows.map(toBook);
}
```

**Desglose del truco:**

```
($1::text IS NULL OR author ILIKE '%' || $1 || '%')
 └── si no hay filtro → TRUE (no filtra)
                         └── si lo hay → contiene el texto (ILIKE = sin mayúsculas)

author || null      → "" o undefined se convierten en null (sin filtro)
maxPages ?? null    → undefined → null; pero 0 se queda como 0
::text / ::int      → le dice a PostgreSQL el tipo del parámetro cuando es null
```

| Petición | Resultado |
|----------|-----------|
| `GET /api/books` | Los 3 libros |
| `GET /api/books?author=garcía` | Los 2 de García Márquez |
| `GET /api/books?maxPages=150` | El principito y Crónica... |
| `GET /api/books?author=garcía&maxPages=150` | Solo Crónica de una muerte anunciada |

> 📌 Una sola consulta sirve para **todas** las combinaciones de filtros, sin construir
> el SQL con `if`s.

---

## 6.8 `UPDATE` parcial con `COALESCE`

### Ejemplo REAL copiado (`api-books-nest/src/books/books.service.ts`)

```ts
const result = await this.pool.query<BookRow>(
  `UPDATE books
   SET title = COALESCE($1, title),
       author = COALESCE($2, author),
       pages = COALESCE($3, pages)
   WHERE id = $4
   RETURNING id, title, author, pages, created_at`,
  [dto.title ?? null, dto.author ?? null, dto.pages ?? null, id],
);
return toBook(result.rows[0]);
```

- `COALESCE(a, b)` devuelve el primero que **no sea `NULL`**.
- Si `dto.title` no llega → `null` → `COALESCE(null, title)` = el título **actual**.
- Así un `PATCH { "pages": 500 }` solo cambia las páginas.

> ⚠️ Antes del `UPDATE`, el servicio hace `const book = await this.findOne(id);` solo
> para lanzar el 404 si no existe. La variable `book` no se usa después (bastaría con
> `await this.findOne(id);`).

---

## 6.9 `COUNT` y tipos que devuelve `pg`

### Ejemplo REAL copiado (`api-books-nest/src/books/books.service.ts`)

```ts
async count(): Promise<{ total: number }> {
  const result = await this.pool.query<{ total: string }>(
    'SELECT COUNT(*)::text AS total FROM books',
  );
  return { total: Number(result.rows[0].total) };
}
```

> 📌 `COUNT(*)` en PostgreSQL es `bigint`, y `pg` devuelve los `bigint` como **string**
> (porque pueden superar el máximo seguro de JS). Por eso se tipa como `string` y se
> convierte con `Number(...)`.

---

## 6.10 De fila SQL a objeto de la API

En SQL se usa `snake_case` (`created_at`); en JS, `camelCase` (`createdAt`).

### Ejemplo REAL copiado (`api-books-nest/src/books/book.ts`)

```ts
// Así llega un libro desde PostgreSQL (una fila de la tabla)
export interface BookRow {
  id: number;
  title: string;
  author: string;
  pages: number;
  created_at: Date; // en SQL, con guion bajo
}

// Así lo devuelve nuestra API
export interface Book {
  id: number;
  title: string;
  author: string;
  pages: number;
  createdAt: Date; // en JavaScript, en camelCase
}

// Convierte una fila de la base de datos en un libro de la API
export function toBook(row: BookRow): Book {
  return {
    id: row.id,
    title: row.title,
    author: row.author,
    pages: row.pages,
    createdAt: row.created_at,
  };
}
```

```ts
return result.rows.map(toBook);   // varias filas
return toBook(result.rows[0]);    // una fila
```

> 📌 Separar `BookRow` (BD) de `Book` (API) te permite **no exponer** columnas internas.
> Con usuarios es crucial: **jamás** devuelvas `password_hash` en una respuesta.

---

## 6.11 Usuarios y contraseñas (preparado para auth)

### Ejemplo REAL copiado (`api-books-nest/src/users/user.ts`)

```ts
// Un usuario tal y como está en la tabla users
export interface User {
  id: number;
  email: string;
  password_hash: string;
  role: string; // 'customer' o 'employee'
}
```

Tu `package.json` incluye `argon2`, que se usará para guardar **hashes** y no
contraseñas. Así se usaría (ejemplo propio):

```ts
import * as argon2 from 'argon2';

const hash = await argon2.hash(dto.password);          // al registrar
await this.usersService.create(dto.email, hash);

const ok = await argon2.verify(user.password_hash, dto.password); // al hacer login
if (!ok) throw new UnauthorizedException('Credenciales incorrectas');
```

---

## 6.12 ORMs (para que te suenen)

Tu proyecto usa **SQL a mano** con `pg`. En muchos proyectos Nest verás un ORM:

| Herramienta | Paquete Nest | Estilo |
|-------------|--------------|--------|
| TypeORM | `@nestjs/typeorm` | Entidades con decoradores (`@Entity`, `@Column`) |
| Prisma | (cliente propio) | Esquema `schema.prisma` + cliente generado |
| Mongoose (MongoDB) | `@nestjs/mongoose` | Schemas con `@Schema`, `@Prop` |

> 📌 Aprender primero con SQL puro (como haces) te hace entender lo que el ORM hace por
> debajo.

---

## ✅ Resumen rápido

1. PostgreSQL en Docker con `compose.yaml`; tablas en `schema.sql`.
2. `.env` (secreto, en `.gitignore`) + `.env.example` (plantilla, en Git).
3. `process.loadEnvFile()` carga el `.env`; o `@nestjs/config` como alternativa.
4. `Pool` de `pg` como provider (`useFactory`) exportado por `DatabaseModule`.
5. Siempre consultas **parametrizadas** (`$1`, `$2`) → nada de inyección SQL.
6. `RETURNING`, `COALESCE`, `ILIKE`, `IS NULL OR ...` para CRUD y filtros.
7. Convierte filas (`snake_case`) a objetos de API (`camelCase`) con `toBook`.

👉 **Siguiente:** `07-guards-interceptors-middleware-y-testing.md`
