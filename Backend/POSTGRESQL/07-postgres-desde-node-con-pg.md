# 07 — POSTGRES DESDE NODE CON `pg`

> Tu API (Express o NestJS) no "habla" SQL por sí sola: usa una **librería
> cliente** que abre conexiones con el servidor de Postgres, le envía el SQL **como
> texto** y te devuelve las filas como **objetos de JavaScript**. La más usada es
> **`pg`** (también llamada *node-postgres*), la que usas en `api-books-nest`.

---

## 7.1 Instalación

```bash
npm install pg
npm install -D @types/pg      # tipos para TypeScript
```

### Ejemplo REAL (`puta/api-books-nest/package.json`, fragmento)

```json
"dependencies": {
    ...
    "argon2": "^0.45.2",
    ...
    "pg": "^8.23.1",
    ...
},
"devDependencies": {
    ...
    "@types/pg": "^8.23.1",
    ...
}
```

---

## 7.2 La cadena de conexión en `.env`

La API lee los datos de conexión de una **variable de entorno**, nunca escritos en
el código.

### Ejemplo REAL (`puta/api-books-nest/.env.example`)

```
DATABASE_URL=postgres://alumno:CAMBIAME@localhost:5432/academia
```

> 🔒 El `.env` **real** (con la contraseña de verdad) **no** se copia aquí y **no**
> se sube a Git (está en `.gitignore`). El `.env.example` sí se sube, con un valor
> de mentira (`CAMBIAME`), para que quien clone el proyecto sepa qué variables necesita.

> 📌 Fíjate en que aquí el host es `localhost` (la API corre en tu PC y Postgres en
> Docker con el puerto publicado). Ver archivo `02`.

---

## 7.3 `Client` vs `Pool`

| | `Client` | `Pool` |
|---|----------|--------|
| Qué es | **Una** conexión | Un **grupo** de conexiones reutilizables |
| Uso | Scripts sueltos | **Servidores / APIs** ✅ |
| Peticiones a la vez | Una cola | Varias en paralelo (por defecto hasta 10 conexiones) |

```
           peticiones HTTP
   GET /books   POST /books   GET /books/3
        │            │             │
        ▼            ▼             ▼
   ┌──────────────── Pool ────────────────┐
   │  conexión 1   conexión 2   conexión 3 │ ... (se reutilizan)
   └──────┬────────────┬────────────┬──────┘
          └────────────┴────────────┘
                       ▼
                  PostgreSQL
```

> 📌 Abrir una conexión con Postgres es **caro** (autenticación, memoria...). El
> `Pool` las mantiene abiertas y las **presta** a cada consulta. Por eso en una API
> siempre se usa `Pool`, y se crea **una sola vez** para toda la aplicación.

### Ejemplo REAL copiado (`puta/api-books-nest/src/database/database.service.ts`)

```ts
import { Injectable, OnModuleDestroy } from '@nestjs/common';
import { Pool } from 'pg';

@Injectable()
export class DatabaseService implements OnModuleDestroy {
  readonly pool = new Pool({
    connectionString: DatabaseService.databaseUrl(),
  });

  private static databaseUrl(): string {
    const url = process.env.DATABASE_URL;
    if (!url) {
      throw new Error('Falta DATABASE_URL en el entorno');
    }
    return url;
  }

  async onModuleDestroy() {
    await this.pool.end();
  }
}
```

**Puntos clave:**
- `new Pool({ connectionString })` → crea el pool a partir de la `DATABASE_URL`.
- Si falta la variable, **falla al arrancar** con un mensaje claro (mejor que un error raro en la primera consulta).
- `onModuleDestroy` → al apagar la app, `pool.end()` **cierra** las conexiones.

### Ejemplo REAL copiado (`puta/api-books-nest/src/database/database.module.ts`)

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

> 📌 En tu proyecto conviven **dos formas** de crear el pool. La que realmente
> usan los servicios es la del **módulo** (`provide: Pool`): por eso
> `BooksService` y `UsersService` reciben `private readonly pool: Pool` en el
> constructor. NestJS crea **un único** `Pool` y se lo inyecta a todos.

### Lo mismo sin NestJS (Express / Node puro) — ejemplo propio

```ts
// db.ts
import { Pool } from 'pg';

export const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// en cualquier ruta
import { pool } from './db';
const result = await pool.query('SELECT id, title FROM books ORDER BY id');
```

---

## 7.4 `pool.query()` y el objeto `result`

```ts
const result = await pool.query('SELECT id, title FROM books WHERE id = $1', [3]);
```

| Propiedad | Qué contiene |
|-----------|--------------|
| `result.rows` | **Array** de objetos, uno por fila: `[{ id: 3, title: '...' }]` |
| `result.rowCount` | Nº de filas devueltas / afectadas (útil en `UPDATE`/`DELETE`) |
| `result.fields` | Info de las columnas |

```ts
result.rows            // [] si no hay ninguna
result.rows[0]         // la primera fila (o undefined)
result.rows.map(...)   // transformar todas
```

### Ejemplo REAL copiado (`puta/api-books-nest/src/books/books.service.ts`)

```ts
  async findOne(id: number): Promise<Book> {
    const result = await this.pool.query<BookRow>(
      'SELECT id, title, author, pages, created_at FROM books WHERE id = $1',
      [id],
    );
    if (result.rows.length === 0) {
      throw new NotFoundException('Libro no encontrado');
    }
    return toBook(result.rows[0]);
  }
```

> 📌 `pool.query<BookRow>(...)` → el genérico le dice a TypeScript cómo son las
> filas (`result.rows` será `BookRow[]`). **Ojo:** es solo una "promesa" de tipos;
> TypeScript **no** comprueba que el SQL devuelva de verdad esas columnas.

---

## 7.5 Consultas parametrizadas: `$1`, `$2`... 🔐

En vez de **pegar** los valores dentro del texto SQL, pones **marcadores** `$1`,
`$2`... y pasas los valores en un **array** aparte:

```ts
pool.query(
  'SELECT ... FROM books WHERE id = $1',   // ← el SQL con huecos
  [id],                                    // ← los valores: $1 = id
);
```

### Ejemplo REAL copiado (`books.service.ts`, método `create`)

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

```
$1 ← dto.title
$2 ← dto.author
$3 ← dto.pages
```

> 📌 El número del `$` es la **posición en el array** (empezando en 1). Un mismo
> `$1` se puede usar varias veces en la consulta (como en el `findAll`).

---

## 7.6 Inyección SQL: por qué NUNCA concatenar

### ❌ El código vulnerable (ejemplo propio, NO está en tus proyectos)

```ts
// ❌❌❌ NUNCA hagas esto
const email = req.body.email;
const result = await pool.query(
  `SELECT * FROM users WHERE email = '${email}'`
);
```

Si un atacante envía como email:

```
' OR '1'='1
```

el SQL que se ejecuta es:

```sql
SELECT * FROM users WHERE email = '' OR '1'='1'
--                                   └─────┬──┘
--                           siempre TRUE → devuelve TODOS los usuarios
```

Y con algo como `'; DROP TABLE users; --` podría intentar **borrarte la tabla**.

### ✅ El código seguro: el de tu proyecto

**Ejemplo REAL copiado (`puta/api-books-nest/src/users/users.service.ts`):**

```ts
  // Busca un usuario por su correo. undefined si no existe
  async findByEmail(email: string): Promise<User | undefined> {
    const result = await this.pool.query<User>(
      'SELECT id, email, password_hash, role FROM users WHERE email = $1',
      [email],
    );
    return result.rows[0];
  }
```

Con `$1`, el texto y los datos viajan **por separado** al servidor. Postgres trata
`' OR '1'='1` como **un email literal** (raro, pero inofensivo) → no encuentra a nadie.

| | Concatenar `${x}` | Parametrizar `$1` |
|---|---|---|
| Inyección SQL | ❌ Vulnerable | ✅ Imposible |
| Comillas en el dato (`O'Brien`) | ❌ Rompe el SQL | ✅ Funciona |
| Tipos (fechas, null...) | A mano | Los convierte `pg` |

> ⚠️ Los `$1` solo sirven para **valores**, no para nombres de tablas/columnas ni
> palabras clave. Si necesitas un `ORDER BY` dinámico, usa una **lista blanca**:
> ```ts
> const columnas = { titulo: 'title', paginas: 'pages' } as const;
> const orden = columnas[req.query.sort as keyof typeof columnas] ?? 'id';
> pool.query(`SELECT ... FROM books ORDER BY ${orden}`);   // orden solo puede ser uno de esos
> ```

---

## 7.7 Filtros opcionales con un solo SQL

### Ejemplo REAL copiado (`books.service.ts`, método `findAll`)

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

**Cómo funciona el truco `($1 IS NULL OR ...)`:**

```
GET /books                     → $1 = null, $2 = null
   (TRUE OR ...) AND (TRUE OR ...)            → todos los libros

GET /books?author=garcía       → $1 = 'garcía', $2 = null
   (FALSE OR author ILIKE '%garcía%') AND TRUE → solo los de García

GET /books?maxPages=150        → $1 = null, $2 = 150
   TRUE AND (FALSE OR pages <= 150)            → los cortos
```

- `author || null` → si llega `''` (cadena vacía) también se trata como "sin filtro".
- `maxPages ?? null` → `undefined` se convierte en `null` (`pg` necesita `null`).
- `$1::text` / `$2::int` → le dicen a Postgres el tipo, porque con `null` no lo puede adivinar.

---

## 7.8 De fila SQL a objeto de la API (snake_case → camelCase)

### Ejemplo REAL copiado (`puta/api-books-nest/src/books/book.ts`)

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

> 📌 Convención: **SQL en `snake_case`**, **JS en `camelCase`**. La función `toBook`
> es la "frontera" entre los dos mundos. Alternativa en SQL: `created_at AS "createdAt"`
> (con comillas dobles para conservar la mayúscula).

### Tipos que `pg` convierte (y uno que no)

| Postgres | JavaScript |
|----------|-----------|
| `INTEGER`, `SMALLINT`, `REAL` | `number` |
| `TEXT`, `VARCHAR` | `string` |
| `BOOLEAN` | `boolean` |
| `TIMESTAMPTZ`, `TIMESTAMP`, `DATE` | `Date` |
| `JSONB` | objeto |
| `NULL` | `null` |
| ⚠️ `BIGINT` (y `COUNT(*)`) | **`string`** |
| ⚠️ `NUMERIC` | **`string`** |

**Ejemplo REAL copiado (`books.service.ts`, método `count`):**

```ts
  async count(): Promise<{ total: number }> {
    const result = await this.pool.query<{ total: string }>(
      'SELECT COUNT(*)::text AS total FROM books',
    );
    return { total: Number(result.rows[0].total) };
  }
```

> 📌 `COUNT(*)` devuelve un `bigint`, que puede ser más grande que el máximo seguro
> de un `number` de JS. Por eso `pg` lo entrega como **texto**, y en tu código se
> convierte con `Number(...)`. Lo mismo pasa con `NUMERIC` (precios): llegan como
> `"29.90"`.

---

## 7.9 `UPDATE` y `DELETE` desde la API

### Ejemplo REAL copiado (`books.service.ts`)

```ts
  async update(id: number, dto: UpdateBookDto): Promise<Book> {
    if (dto.title === undefined && dto.author === undefined && dto.pages === undefined) {
      throw new BadRequestException('Envía al menos un campo');
    }
    const book = await this.findOne(id);

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
  }

  async remove(id: number): Promise<Book> {
    const book = await this.findOne(id);
    await this.pool.query('DELETE FROM books WHERE id = $1', [id]);
    return book;
  }
```

> 📌 En `update`, el `findOne(id)` sirve para lanzar el **404** si el libro no
> existe (la variable `book` no se usa después). Alternativa con una sola consulta:
> hacer el `UPDATE ... RETURNING` y comprobar `result.rowCount === 0` → 404.

### Guardar contraseñas: siempre el **hash**

**Ejemplo REAL copiado (`users.service.ts` y `database/schema.sql`):**

```ts
  // Guarda un usuario nuevo (con el hash, no con la contraseña)
  async create(email: string, passwordHash: string): Promise<User> {
    const result = await this.pool.query<User>(
      `INSERT INTO users (email, password_hash)
       VALUES ($1, $2)
       RETURNING id, email, password_hash, role`,
      [email, passwordHash],
    );
    return result.rows[0];
  }
```

```sql
  password_hash TEXT NOT NULL,             -- el hash, nunca la contraseña
```

> 📌 El hash se calcula **antes**, en la API, con `argon2` (está en tu
> `package.json`). La BD nunca ve la contraseña en claro.

---

## 7.10 Manejar errores de restricciones (ejemplo propio)

Cuando una restricción falla (archivo `03`), `pg` lanza un error con `code`:

```ts
try {
  await this.pool.query(
    'INSERT INTO users (email, password_hash) VALUES ($1, $2)',
    [email, hash],
  );
} catch (error: any) {
  if (error.code === '23505') {                 // unique_violation
    throw new ConflictException('Ese email ya está registrado');   // 409
  }
  if (error.code === '23503') {                 // foreign_key_violation
    throw new BadRequestException('La referencia no existe');      // 400
  }
  throw error;                                  // otro error → 500
}
```

---

## 7.11 Transacciones desde Node (ejemplo propio)

Con `pool.query()` cada consulta puede ir por una **conexión distinta** del pool.
Para una transacción necesitas que **todas** vayan por la **misma** → `pool.connect()`:

```ts
const client = await pool.connect();          // "alquila" UNA conexión
try {
  await client.query('BEGIN');
  await client.query('UPDATE offers SET status = $1 WHERE id = $2', ['ACCEPTED', offerId]);
  await client.query('UPDATE listings SET status = $1 WHERE id = $2', ['RESERVED', listingId]);
  await client.query('COMMIT');
} catch (error) {
  await client.query('ROLLBACK');             // algo falló → deshacer todo
  throw error;
} finally {
  client.release();                           // ⚠️ SIEMPRE devolverla al pool
}
```

> ⚠️ Si olvidas `client.release()`, la conexión no vuelve al pool. Tras 10 olvidos,
> la API se queda **colgada** esperando conexiones libres.

---

## 7.12 ORMs y query builders (mención breve)

Escribir SQL a mano con `pg` (lo que haces tú) es la mejor forma de **aprender**.
En proyectos grandes es común usar una capa por encima:

| Herramienta | Tipo | Cómo se ve |
|-------------|------|-----------|
| **Prisma** | ORM con su propio esquema (`schema.prisma`) y migraciones | `prisma.book.findMany({ where: { pages: { lte: 150 } } })` |
| **TypeORM** | ORM con clases y decoradores (muy usado con NestJS: `@nestjs/typeorm`) | `@Entity() class Book { @PrimaryGeneratedColumn() id: number }` |
| **Drizzle** | ORM ligero, "SQL con tipos" | `db.select().from(books).where(lte(books.pages, 150))` |
| **Sequelize** | ORM clásico de Node | `Book.findAll({ where: { ... } })` |
| **Knex** | *Query builder* (no ORM) | `knex('books').where('pages', '<=', 150)` |

**ORM** (*Object-Relational Mapping*) = traduce entre **tablas** y **objetos/clases**
y genera el SQL por ti.

| ✅ Ventajas | ❌ Inconvenientes |
|-----------|-----------------|
| Tipos automáticos, menos código repetido | Otra cosa más que aprender |
| **Migraciones** (versionar los cambios de esquema) | Consultas complejas a veces generan SQL poco eficiente |
| Parametriza solo (protege de inyección) | Si no sabes SQL, no sabes qué está pasando |

> 📌 Todos los ORMs acaban enviando **SQL** a Postgres. Todo lo de estos apuntes
> (JOIN, índices, transacciones, restricciones) **sigue aplicando**.

---

## ✅ Resumen rápido

1. `npm i pg` + `@types/pg`. Conexión con `DATABASE_URL` desde `.env` (nunca en el código).
2. En una API → **un único `Pool`**, creado al arrancar y cerrado con `pool.end()`.
3. `const { rows } = await pool.query(sql, [valores])` → `rows` es un array de objetos.
4. **Siempre `$1, $2...`**, nunca `${variable}` dentro del SQL → evita **inyección SQL**.
5. `RETURNING` para recibir la fila creada/modificada.
6. `snake_case` en SQL → `camelCase` en JS (función tipo `toBook`).
7. `COUNT`/`BIGINT`/`NUMERIC` llegan como **string** → `Number(...)`.
8. Errores: `error.code` `23505` (UNIQUE) → 409, `23503` (FK) → 400.
9. Transacciones: `pool.connect()` + `BEGIN`/`COMMIT`/`ROLLBACK` + `release()`.
10. ORMs (Prisma, TypeORM, Drizzle...) generan el SQL por ti, pero por debajo es lo mismo.

👉 **Siguiente:** `08-ejemplos-completos-de-proyectos.md`
