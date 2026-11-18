# 08 — EJEMPLOS COMPLETOS COPIADOS DE TUS PROYECTOS

> 📁 **Los archivos de los proyectos NO se han modificado.** Todo lo de aquí abajo son
> **copias** de su contenido original (`api-books-nest/`), comentadas para estudiar.
> 🔒 Las contraseñas reales se han sustituido por `CAMBIAME` y el `.env` no se copia.

---

## EJEMPLO 1 — El arranque: `main.ts`

**Origen:** `api-books-nest/src/main.ts` *(copiado íntegro)*

```ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

process.loadEnvFile(); // lee el .env y guarda sus valores en process.env

async function bootstrap() {
  const app = await NestFactory.create(AppModule); // crea la aplicación

  app.setGlobalPrefix('api'); // todas las rutas empiezan por /api

  // Comprueba los DTO en todas las rutas
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true, // quita los campos que no estén en el DTO...
      forbidNonWhitelisted: true, // ...o mejor: responde 400 si llegan
      transform: true, // aplica las transformaciones del DTO (como el trim de abajo)
    }),
  );

  await app.listen(process.env.PORT ?? 3000); // escucha en el puerto 3000
}
bootstrap();
```

**Qué enseña:**
- Cargar el `.env` **antes** de crear la app.
- `NestFactory.create(AppModule)` construye toda la app desde el módulo raíz.
- Configuración **global**: prefijo `/api` y `ValidationPipe`.
- Puerto configurable con `process.env.PORT ?? 3000`.

---

## EJEMPLO 2 — El módulo raíz + controlador + servicio básicos

**Origen:** `api-books-nest/src/app.module.ts` *(copiado íntegro)*

```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { BooksModule } from './books/books.module';
import { DatabaseModule } from './database/database.module';
import { UsersModule } from './users/users.module';

@Module({
  imports: [DatabaseModule, BooksModule, UsersModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

**Origen:** `api-books-nest/src/app.controller.ts` *(copiado íntegro)*

```ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }

  @Get('ping')
  ping(): { pong: boolean } {
    return { pong: true };
  }
}
```

**Origen:** `api-books-nest/src/app.service.ts` *(copiado íntegro)*

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hola desde NestJS';
  }
}
```

**Qué enseña:**
- La tríada **módulo → controlador → servicio** en su forma más simple.
- `AppController` pide `AppService` por constructor (inyección de dependencias).
- `GET /api` → texto; `GET /api/ping` → JSON `{ "pong": true }` (útil como "health check").

---

## EJEMPLO 3 — El módulo de libros completo

**Origen:** `api-books-nest/src/books/books.module.ts` *(copiado íntegro)*

```ts
import { Module } from '@nestjs/common';
import { DatabaseModule } from '../database/database.module';
import { BooksController } from './books.controller';
import { BooksService } from './books.service';

@Module({
  imports: [DatabaseModule], // ← nuevo: ahora el servicio puede pedir el Pool
  controllers: [BooksController],
  providers: [BooksService],
})
export class BooksModule {}
```

**Origen:** `api-books-nest/src/books/books.controller.ts` *(copiado íntegro)*

```ts
import { Body, Controller, Get, Param, ParseIntPipe, Patch, Post, Query } from '@nestjs/common';
import { BooksService } from './books.service';
import { CreateBookDto } from './dto/create-book.dto';
import { UpdateBookDto } from './dto/update-book.dto';

@Controller('books') // todas las rutas de esta clase: /api/books...
export class BooksController {
  constructor(private readonly booksService: BooksService) {} // pide el servicio

  @Get() // GET /api/books?author=gabriel&maxPages=150
  findAll(
    @Query('author') author?: string,
    @Query('maxPages') maxPages?: string, // llega como texto: "150"
  ) {
    const max = maxPages ? Number(maxPages) : undefined; // lo pasamos a número
    return this.booksService.findAll(author, max);
  }

  @Get('count') // GET /api/books/count
  count() {
    return this.booksService.count(); // el cálculo lo hace el servicio
  }

  @Get(':id') // GET /api/books/1
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.booksService.findOne(id);
  }

  @Post() // POST /api/books
  create(@Body() dto: CreateBookDto) {
    return this.booksService.create(dto); // el ValidationPipe ya comprobó el DTO
  }

  @Patch(':id') // PATCH /api/books/1
  update(@Param('id', ParseIntPipe) id: number, @Body() dto: UpdateBookDto) {
    return this.booksService.update(id, dto);
  }
}
```

**Origen:** `api-books-nest/src/books/books.service.ts` *(copiado íntegro)*

```ts
import { BadRequestException, Injectable, NotFoundException } from '@nestjs/common';
import { Pool } from 'pg';
import { Book, BookRow, toBook } from './book';
import { CreateBookDto } from './dto/create-book.dto';
import { UpdateBookDto } from './dto/update-book.dto';
import { BooksContract } from './books.contract';

@Injectable()
export class BooksService implements BooksContract  {
  constructor(private readonly pool: Pool) {}

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

  async count(): Promise<{ total: number }> {
    const result = await this.pool.query<{ total: string }>(
      'SELECT COUNT(*)::text AS total FROM books',
    );
    return { total: Number(result.rows[0].total) };
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
}
```

**Qué enseña:**
- Controlador **fino** (solo traduce HTTP → servicio) y servicio con **toda la lógica**.
- `ParseIntPipe`, `@Query` opcionales, `@Body` con DTO validado.
- Ruta fija `count` **antes** de `:id`.
- SQL **parametrizado** (`$1...`), filtros opcionales, `RETURNING`, `COALESCE`.
- `NotFoundException` / `BadRequestException` para errores HTTP.
- `remove` existe en el servicio pero **falta la ruta `@Delete`** en el controlador
  (ejercicio en `03`, apartado 3.9).

---

## EJEMPLO 4 — Tipos, contrato y DTOs de libros

**Origen:** `api-books-nest/src/books/book.ts` *(copiado íntegro)*

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

**Origen:** `api-books-nest/src/books/books.contract.ts` *(copiado íntegro)*

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

**Origen:** `api-books-nest/src/books/dto/create-book.dto.ts` *(copiado íntegro)*

```ts
import { Transform } from 'class-transformer';
import { IsInt, IsString, Max, MaxLength, Min, MinLength } from 'class-validator';

// Lo que tiene que enviar el cliente para crear un libro
export class CreateBookDto {
  // Si llega un texto, le quita los espacios del principio y del final
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
  @Min(1, { message: 'El libro tiene que tener al menos 1 página' }) // al menos 1 página
  @Max(2000) // como mucho 2000 páginas
  pages: number;
}
```

**Origen:** `api-books-nest/src/books/dto/update-book.dto.ts` *(copiado íntegro)*

```ts
import { IsInt, IsString, MaxLength, Min, MinLength, ValidateIf } from 'class-validator';

// PATCH: los campos pueden faltar, pero si llegan tienen que ser válidos
export class UpdateBookDto {
  @ValidateIf((_, value) => value !== undefined) // valida solo si ha llegado
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

**Qué enseña:**
- `interface` para datos internos (`BookRow`, `Book`, `BooksContract`) y `class` para
  DTOs (porque necesitan decoradores en tiempo de ejecución).
- Mapeo `snake_case` → `camelCase` con una función pura (`toBook`).
- Validación de creación (todo obligatorio) vs actualización (todo opcional, pero sin
  `null`).
- Ojo: `UpdateBookDto.pages` no tiene `@Max(2000)` como el de creación → con un PATCH
  se podría poner un libro de 5000 páginas. ¿Lo añadirías?

---

## EJEMPLO 5 — Base de datos

**Origen:** `api-books-nest/src/database/database.module.ts` *(copiado íntegro)*

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

**Origen:** `api-books-nest/src/database/database.service.ts` *(copiado íntegro)*

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

**Origen:** `api-books-nest/database/schema.sql` *(copiado íntegro)*

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

**Origen:** `api-books-nest/compose.yaml` *(copiado; contraseña sustituida por `CAMBIAME`)*

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

**Origen:** `api-books-nest/.env.example` *(copiado íntegro)*

```bash
DATABASE_URL=postgres://alumno:CAMBIAME@localhost:5432/academia
```

**Qué enseña:**
- Provider con `useFactory` y token `Pool`, exportado para otros módulos.
- Alternativa con servicio propio + hook de ciclo de vida `onModuleDestroy`
  (preparado, pero **no registrado** en ningún módulo).
- La BD **también valida** (`CHECK`, `UNIQUE`, `NOT NULL`).
- `.env.example` como plantilla segura; el `.env` real nunca se sube.

---

## EJEMPLO 6 — Módulo de usuarios (base para autenticación)

**Origen:** `api-books-nest/src/users/user.ts` *(copiado íntegro)*

```ts
// Un usuario tal y como está en la tabla users
export interface User {
  id: number;
  email: string;
  password_hash: string;
  role: string; // 'customer' o 'employee'
}
```

**Origen:** `api-books-nest/src/users/users.module.ts` *(copiado íntegro)*

```ts
import { Module } from '@nestjs/common';
import { DatabaseModule } from '../database/database.module';
import { UsersService } from './users.service';

@Module({
  imports: [DatabaseModule], // para poder pedir el Pool
  providers: [UsersService],
  exports: [UsersService], // lo usará el módulo de autenticación
})
export class UsersModule {}
```

**Origen:** `api-books-nest/src/users/users.service.ts` *(copiado íntegro)*

```ts
import { Injectable } from '@nestjs/common';
import { Pool } from 'pg';
import { User } from './user';

@Injectable()
export class UsersService {
  constructor(private readonly pool: Pool) {} // el Pool de la 03b

  // Busca un usuario por su correo. undefined si no existe
  async findByEmail(email: string): Promise<User | undefined> {
    const result = await this.pool.query<User>(
      'SELECT id, email, password_hash, role FROM users WHERE email = $1',
      [email],
    );
    return result.rows[0];
  }

  // Busca un usuario por su id. undefined si no existe (lo usará el AuthGuard)
  async findById(id: number): Promise<User | undefined> {
    const result = await this.pool.query<User>(
      'SELECT id, email, password_hash, role FROM users WHERE id = $1',
      [id],
    );
    return result.rows[0];
  }

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
}
```

**Qué enseña:**
- Un módulo **sin controladores** que solo ofrece (exporta) un servicio.
- Devolver `undefined` en vez de lanzar → el que llama decide (401, 404...).
- Se guarda el **hash** (con `argon2`, ya en las dependencias), nunca la contraseña.
- `User` incluye `password_hash`: si algún día lo devuelves en una ruta, crea un tipo
  "público" sin ese campo (igual que `Book` vs `BookRow`).

---

## EJEMPLO 7 — Los tests

**Origen:** `api-books-nest/src/app.controller.spec.ts` *(copiado íntegro)*

```ts
import { Test, TestingModule } from '@nestjs/testing';
import { AppController } from './app.controller';
import { AppService } from './app.service';

describe('AppController', () => {
  let appController: AppController;

  beforeEach(async () => {
    const app: TestingModule = await Test.createTestingModule({
      controllers: [AppController],
      providers: [AppService],
    }).compile();

    appController = app.get<AppController>(AppController);
  });

  describe('root', () => {
    it('should return "Hola desde NestJS"', () => {
      expect(appController.getHello()).toBe('Hola desde NestJS');
    });
  });
});
```

**Origen:** `api-books-nest/test/app.e2e-spec.ts` *(copiado íntegro)*

```ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import request from 'supertest';
import { App } from 'supertest/types';
import { AppModule } from './../src/app.module';

describe('AppController (e2e)', () => {
  let app: INestApplication<App>;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('/ (GET)', () => {
    return request(app.getHttpServer())
      .get('/')
      .expect(200)
      .expect('Hello World!');
  });

  afterEach(async () => {
    await app.close();
  });
});
```

**Qué enseña:**
- Unitario: módulo de pruebas con solo lo necesario, y llamada directa al método.
- E2E: app completa + `supertest` haciendo peticiones HTTP reales.
- ⚠️ El e2e sigue esperando `'Hello World!'` (texto de la plantilla original), pero el
  servicio ya devuelve `'Hola desde NestJS'` → **fallará** hasta actualizarlo.

---

## EJEMPLO 8 — Configuración del proyecto

**Origen:** `api-books-nest/nest-cli.json` *(copiado íntegro)*

```json
{
  "$schema": "https://json.schemastore.org/nest-cli",
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "deleteOutDir": true
  }
}
```

**Origen:** `api-books-nest/tsconfig.json` *(copiado íntegro)*

```json
{
  "compilerOptions": {
    "module": "nodenext",
    "moduleResolution": "nodenext",
    "resolvePackageJsonExports": true,
    "esModuleInterop": true,
    "isolatedModules": true,
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "ES2023",
    "sourceMap": true,
    "outDir": "./dist",
    "incremental": true,
    "skipLibCheck": true,
    "strictNullChecks": true,
    "forceConsistentCasingInFileNames": true,
    "noImplicitAny": false,
    "strictBindCallApply": false,
    "noFallthroughCasesInSwitch": false
  }
}
```

---

## 🧪 Prueba completa de la API (con `curl` o Thunder Client / Postman)

```bash
# 0. Arrancar
docker compose up -d
npm run start:dev

# 1. Comprobar que vive
curl http://localhost:3000/api/ping
# → {"pong":true}

# 2. Listar y filtrar
curl http://localhost:3000/api/books
curl "http://localhost:3000/api/books?author=garcia&maxPages=150"
curl http://localhost:3000/api/books/count
# → {"total":3}

# 3. Uno por id (y errores)
curl http://localhost:3000/api/books/1
curl http://localhost:3000/api/books/999   # → 404 "Libro no encontrado"
curl http://localhost:3000/api/books/abc   # → 400 (ParseIntPipe)

# 4. Crear (y validación)
curl -X POST http://localhost:3000/api/books \
  -H "Content-Type: application/json" \
  -d '{"title":"  Rayuela  ","author":"Julio Cortázar","pages":600}'
# → 201 con "title":"Rayuela" (sin espacios gracias a @Transform)

curl -X POST http://localhost:3000/api/books \
  -H "Content-Type: application/json" \
  -d '{"title":"X","author":"Y","pages":0,"precio":10}'
# → 400: "property precio should not exist", "El libro tiene que tener al menos 1 página"

# 5. Actualizar parcialmente
curl -X PATCH http://localhost:3000/api/books/2 \
  -H "Content-Type: application/json" \
  -d '{"pages":100}'

curl -X PATCH http://localhost:3000/api/books/2 \
  -H "Content-Type: application/json" -d '{}'
# → 400 "Envía al menos un campo"
```

> ⚠️ `?author=garcia` (sin tilde) **no** encuentra "García": `ILIKE` ignora mayúsculas,
> pero **no** tildes. Prueba con `?author=garcía` o `?author=gabriel`.

**Ejercicios propuestos:**
1. Añade la ruta `@Delete(':id')` que llame a `booksService.remove(id)`.
2. Añade `@Max(2000)` a `pages` en `UpdateBookDto`.
3. Actualiza el test e2e para que espere `'Hola desde NestJS'`.
4. Escribe `books.service.spec.ts` con un `Pool` mockeado (modelo en `07`, apartado 7.7).
5. Crea un `AuthModule` que importe `UsersModule` y un `AuthGuard` (boceto en `07`).

👉 **Vuelve al índice:** `00-INDICE.md`
