# 07 — EJEMPLOS COMPLETOS DE PROYECTOS

> Tu proyecto `postgresql-render-atlas/nestjs-postgresql-supabase-render/` entero
> (la parte que importa para Supabase), copiado tal cual. Después, la **misma API**
> reescrita con `supabase-js` + RLS para que compares las dos formas de usar
> Supabase (`01`, apartado 1.4).
>
> 🔒 Contraseñas de desarrollo sustituidas por `TU_PASSWORD`. El `.env` real no se copia;
> su **forma** es:
>
> ```
> DATABASE_URL=postgresql://postgres.<REF>:<PASSWORD>@aws-0-eu-central-1.pooler.supabase.com:5432/postgres
> ```
>
> (y en Render, terminada en `?sslmode=no-verify`, como dice el README).

---

## A) Forma A — Tu API NestJS usando Supabase como PostgreSQL

### Recorrido de una petición

```
GET https://<tu-api>.onrender.com/api/books?author=garcia
   │
   ├─ main.ts            prefijo /api, ValidationPipe, CORS
   ├─ BooksController    @Get() findAll(@Query('author'))
   ├─ BooksService       pool.query('... WHERE author ILIKE ... $1', ['garcia'])
   ├─ DatabaseModule     Pool(DATABASE_URL)  ──TLS──▶  Supavisor (session, :5432)  ──▶  Postgres de Supabase
   └─ toBook()           created_at → createdAt  ──▶  JSON al cliente
```

### `database/schema.sql`

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

### `compose.yaml`

```yaml
services:
  postgres:
    image: postgres:17
    container_name: postgres-bootcamp-api
    environment:
      POSTGRES_USER: alumno
      POSTGRES_PASSWORD: TU_PASSWORD
      POSTGRES_DB: bootcamp_api
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      # Carga el esquema la primera vez que se crea la base
      - ./database/schema.sql:/docker-entrypoint-initdb.d/schema.sql:ro

volumes:
  postgres_data:
```

### `.env.example`

```
DATABASE_URL=postgres://alumno:TU_PASSWORD@localhost:5432/bootcamp_api

```

### `src/main.ts`

```ts
import { existsSync } from 'node:fs';
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

// En tu ordenador, lee el .env. En Render no hay .env: las variables las pone Render.
if (existsSync('.env')) process.loadEnvFile();

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Todas las rutas empiezan por /api
  app.setGlobalPrefix('api');

  // Comprueba los DTO en todas las rutas
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true, // quita los campos que no estén en el DTO...
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  app.enableCors({ origin: 'http://localhost:5173' });

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();

```

### `src/app.module.ts`

```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { BooksModule } from './books/books.module';

@Module({
  imports: [BooksModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

```

### `src/database/database.module.ts`

```ts
import { Module } from '@nestjs/common';
import { Pool } from 'pg';

// Crea el Pool y avisa (sin caerse) si PostgreSQL corta una conexión
function createPool() {
  const pool = new Pool({ connectionString: process.env.DATABASE_URL });

  // Si PostgreSQL se para, cierra las conexiones que no se estaban usando.
  // Sin esta línea, ese aviso tumbaría la API entera.
  pool.on('error', (error) => console.error('PostgreSQL:', error.message));

  return pool;
}

@Module({
  providers: [
    {
      provide: Pool, // cuando alguien pida un Pool...
      useFactory: createPool, // ...NestJS ejecuta esta función una vez
    },
  ],
  exports: [Pool], // otros módulos pueden pedir el Pool
})
export class DatabaseModule {}

```

### `src/books/books.module.ts`

```ts
import { Module } from '@nestjs/common';
import { DatabaseModule } from '../database/database.module';
import { BooksController } from './books.controller';
import { BooksService } from './books.service';

@Module({
  imports: [DatabaseModule], // para poder pedir el Pool
  controllers: [BooksController],
  providers: [BooksService],
})
export class BooksModule {}

```

### `src/books/book.ts`

```ts
// Así llega un libro desde PostgreSQL (una fila de la tabla)
export interface BookRow {
  id: number;
  title: string;
  author: string;
  pages: number;
  created_at: Date; // en SQL se escribe con guion bajo
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

### `src/books/books.contract.ts`

```ts
import { Book } from './book';
import { CreateBookDto } from './dto/create-book.dto';
import { UpdateBookDto } from './dto/update-book.dto';

// El contrato: qué tiene que saber hacer un servicio de libros
export interface BooksContract {
  findAll(author?: string): Promise<Book[]>;
  findOne(id: number): Promise<Book>;
  create(dto: CreateBookDto): Promise<Book>;
  update(id: number, dto: UpdateBookDto): Promise<Book>;
  remove(id: number): Promise<Book>;
}

```

### `src/books/books.controller.ts`

```ts
import {
  Body,
  Controller,
  Delete,
  Get,
  Param,
  ParseIntPipe,
  Patch,
  Post,
  Query,
} from '@nestjs/common';
import { BooksService } from './books.service';
import { CreateBookDto } from './dto/create-book.dto';
import { UpdateBookDto } from './dto/update-book.dto';

@Controller('books') // todas las rutas de esta clase empiezan por /api/books
export class BooksController {
  // Inyección de dependencias: NestJS crea el servicio y nos lo pasa
  constructor(private readonly booksService: BooksService) {}

  @Get() // GET /api/books  (y  /api/books?author=garcia)
  findAll(@Query('author') author?: string) {
    return this.booksService.findAll(author);
  }

  @Get(':id') // GET /api/books/1
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.booksService.findOne(id);
  }

  @Post() // POST /api/books  → 201
  create(@Body() dto: CreateBookDto) {
    return this.booksService.create(dto);
  }

  @Patch(':id') // PATCH /api/books/1
  update(@Param('id', ParseIntPipe) id: number, @Body() dto: UpdateBookDto) {
    return this.booksService.update(id, dto);
  }

  @Delete(':id') // DELETE /api/books/1  → 200 con el libro borrado
  remove(@Param('id', ParseIntPipe) id: number) {
    return this.booksService.remove(id);
  }
}

```

### `src/books/books.service.ts`

```ts
import {
  BadRequestException,
  Injectable,
  NotFoundException,
} from '@nestjs/common';
import { Pool } from 'pg';
import { Book, BookRow, toBook } from './book';
import { BooksContract } from './books.contract';
import { CreateBookDto } from './dto/create-book.dto';
import { UpdateBookDto } from './dto/update-book.dto';

@Injectable()
export class BooksService implements BooksContract {
  constructor(private readonly pool: Pool) {} // NestJS nos da el Pool

  async findAll(author?: string): Promise<Book[]> {
    if (!author) {
      const result = await this.pool.query<BookRow>(
        'SELECT id, title, author, pages, created_at FROM books ORDER BY id',
      );
      return result.rows.map(toBook); // cada fila → un libro de la API
    }

    const result = await this.pool.query<BookRow>(
      `SELECT id, title, author, pages, created_at
       FROM books
       WHERE author ILIKE '%' || $1 || '%'
       ORDER BY id`,
      [author],
    );
    return result.rows.map(toBook);
  }

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

  async create(dto: CreateBookDto): Promise<Book> {
    const result = await this.pool.query<BookRow>(
      `INSERT INTO books (title, author, pages)
       VALUES ($1, $2, $3)
       RETURNING id, title, author, pages, created_at`,
      [dto.title, dto.author, dto.pages],
    );
    return toBook(result.rows[0]);
  }

  async update(id: number, dto: UpdateBookDto): Promise<Book> {
    if (
      dto.title === undefined &&
      dto.author === undefined &&
      dto.pages === undefined
    ) {
      throw new BadRequestException('Envía al menos un campo');
    }
    const book = await this.findOne(id); // 404 si no existe

    const result = await this.pool.query<BookRow>(
      `UPDATE books
       SET title = $1, author = $2, pages = $3
       WHERE id = $4
       RETURNING id, title, author, pages, created_at`,
      [
        dto.title ?? book.title,
        dto.author ?? book.author,
        dto.pages ?? book.pages,
        id,
      ],
    );
    return toBook(result.rows[0]);
  }

  async remove(id: number): Promise<Book> {
    const book = await this.findOne(id); // 404 si no existe
    await this.pool.query('DELETE FROM books WHERE id = $1', [id]);
    return book;
  }
}

```

### `src/books/dto/create-book.dto.ts`

```ts
import { Transform } from 'class-transformer';
import { IsInt, IsString, MaxLength, Min, MinLength } from 'class-validator';

// Lo que tiene que enviar el cliente para crear un libro
export class CreateBookDto {
  @Transform(({ value }: { value: unknown }) =>
    typeof value === 'string' ? value.trim() : value,
  )
  @IsString() // tiene que ser texto
  @MinLength(1) // no puede estar vacío
  @MaxLength(200) // como mucho 200 caracteres
  title: string;

  @IsString()
  @MinLength(1)
  @MaxLength(100)
  author: string;

  @IsInt() // número entero
  @Min(1) // al menos 1 página
  pages: number;
}

```

### `src/books/dto/update-book.dto.ts`

```ts
import {
  IsInt,
  IsString,
  MaxLength,
  Min,
  MinLength,
  ValidateIf,
} from 'class-validator';

// PATCH: todos los campos son opcionales, pero si llegan tienen que ser válidos
export class UpdateBookDto {
  // "Valida este campo solo si ha llegado" (si llega null, también se valida y falla)
  @ValidateIf((_, value) => value !== undefined)
  @IsString()
  @MinLength(1)
  @MaxLength(200)
  title?: string;

  @ValidateIf((_, value) => value !== undefined)
  @IsString()
  @MinLength(1)
  @MaxLength(100)
  author?: string;

  @ValidateIf((_, value) => value !== undefined)
  @IsInt()
  @Min(1)
  pages?: number;
}

```

### `README.md`

```markdown
# 03b · NestJS + PostgreSQL

Estado al terminar los apuntes `03b_nestjs_postgresql.md`: la API de la 03a, con el servicio guardando los libros en PostgreSQL (`pg`, SQL a mano con `$1`, `$2`...). Incluye `toBook` (fila → libro de la API) y el contrato opcional `BooksContract`.

```bash
npm install
cp .env.example .env
docker exec -i postgres-sql psql -U alumno -d academia -c "CREATE DATABASE bootcamp_api;"
docker exec -i postgres-sql psql -U alumno -d bootcamp_api < database/schema.sql
npm run start:dev
```

Necesita el contenedor `postgres-sql` de la clase de SQL. El `.env` se lee con `process.loadEnvFile()` (Node 20.12 o superior), solo si existe: en Render no hay `.env` y las variables las pone Render.

## Desplegar en Render

Paso a paso en los apuntes `06_despliegue_render.md`. Resumen:

| Campo de Render | Valor |
|---|---|
| Root Directory | `03b-nestjs-postgresql` (si el proyecto está en una subcarpeta) |
| Build Command | `npm install && npm run build` |
| Start Command | `npm run start:prod` |
| Variables | `DATABASE_URL` (la del *Session pooler* de Supabase, terminada en `?sslmode=no-verify`) |

Para `04` y `05`, lo mismo con su carpeta y una variable más: `JWT_SECRET`.

```

### Qué se lleva Supabase y qué se queda en tu código

| Responsabilidad | Dónde está |
|-----------------|------------|
| Guardar los datos, `CHECK (pages > 0)`, ids, fechas | **Supabase** (Postgres) |
| Conexión, pool, SSL | `database.module.ts` + `DATABASE_URL` |
| Validar el body (`@IsInt`, `@MinLength`...) | `create-book.dto.ts` + `ValidationPipe` |
| Construir el SQL de forma segura (`$1`) | `books.service.ts` |
| 404 / 400 con mensajes claros | `books.service.ts` (`NotFoundException`, `BadRequestException`) |
| Forma de la respuesta (`createdAt`) | `book.ts` → `toBook()` |
| Quién puede llamar (CORS) | `main.ts` |

---

## B) Forma B — La misma API sin backend: `supabase-js` + RLS (NO está en tus proyectos)

### B.1 SQL a ejecutar en Supabase

```sql
-- La misma tabla de schema.sql
CREATE TABLE books (
  id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  title VARCHAR(200) NOT NULL CHECK (length(trim(title)) > 0),   -- sustituye a @MinLength(1)
  author VARCHAR(100) NOT NULL CHECK (length(trim(author)) > 0),
  pages INTEGER NOT NULL CHECK (pages > 0),                       -- sustituye a @Min(1)
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE books ENABLE ROW LEVEL SECURITY;

-- Igual que tu API actual: lectura y escritura abiertas a todos (no hay login)
CREATE POLICY "leer libros"   ON books FOR SELECT TO anon, authenticated USING (true);
CREATE POLICY "crear libros"  ON books FOR INSERT TO anon, authenticated WITH CHECK (true);
CREATE POLICY "editar libros" ON books FOR UPDATE TO anon, authenticated USING (true) WITH CHECK (true);
CREATE POLICY "borrar libros" ON books FOR DELETE TO anon, authenticated USING (true);
```

> ⚠️ Estas políticas reproducen **exactamente** tu API actual, que no tiene login:
> cualquiera puede borrar libros. En un proyecto real, `INSERT/UPDATE/DELETE`
> irían `TO authenticated` con `user_id = auth.uid()` (ver `05`, apartado 5.4).

### B.2 El "service" en el frontend

```ts
// src/books.ts (frontend React/Vite)
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY,
);

export interface Book { id: number; title: string; author: string; pages: number; createdAt: string }

type BookRow = { id: number; title: string; author: string; pages: number; created_at: string };
const toBook = (r: BookRow): Book => ({ id: r.id, title: r.title, author: r.author, pages: r.pages, createdAt: r.created_at });
const COLUMNS = 'id, title, author, pages, created_at';

export async function findAll(author?: string): Promise<Book[]> {
  let q = supabase.from('books').select(COLUMNS).order('id');
  if (author) q = q.ilike('author', `%${author}%`);
  const { data, error } = await q;
  if (error) throw error;
  return data.map(toBook);
}

export async function findOne(id: number): Promise<Book> {
  const { data, error } = await supabase.from('books').select(COLUMNS).eq('id', id).single();
  if (error) throw new Error(error.code === 'PGRST116' ? 'Libro no encontrado' : error.message);
  return toBook(data);
}

export async function create(input: { title: string; author: string; pages: number }): Promise<Book> {
  const { data, error } = await supabase.from('books').insert(input).select(COLUMNS).single();
  if (error) throw error;          // ej. 23514 = violación de CHECK
  return toBook(data);
}

export async function update(id: number, changes: Partial<{ title: string; author: string; pages: number }>) {
  if (Object.keys(changes).length === 0) throw new Error('Envía al menos un campo');
  const { data, error } = await supabase.from('books').update(changes).eq('id', id).select(COLUMNS).single();
  if (error) throw error;
  return toBook(data);
}

export async function remove(id: number): Promise<Book> {
  const { data, error } = await supabase.from('books').delete().eq('id', id).select(COLUMNS).single();
  if (error) throw error;
  return toBook(data);
}
```

### B.3 Comparativa final

| | Forma A (tu proyecto) | Forma B |
|---|---|---|
| Archivos de backend | ~10 (módulos, controller, service, DTOs, main) | 0 (solo SQL) |
| Despliegue | Render (Web Service) + Supabase | Supabase + frontend estático |
| *Cold start* | Sí (Render Free se duerme) | No (la API de Supabase está siempre activa) |
| Validación | DTO detallado con mensajes | Restricciones SQL (mensajes menos amigables) |
| Seguridad | En tu código | En políticas RLS |
| Cambiar de proveedor | Cambiar `DATABASE_URL` | Reescribir el acceso a datos |
| Lo que aprendes | Arquitectura backend completa | Modelado SQL y seguridad en BD |
