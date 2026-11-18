# 📋 CHEATSHEET — NESTJS

> Chuleta rápida de comandos y código. Teoría en los documentos 01–08 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal

### Nest CLI

| Comando | Qué hace |
|---------|----------|
| `npm i -g @nestjs/cli` | Instala el CLI global |
| `nest new api-books-nest` | Crea un proyecto nuevo |
| `nest g resource books` | Módulo + controlador + servicio + DTOs de golpe |
| `nest g mo books` | Módulo (`generate module`) |
| `nest g co books` | Controlador (`generate controller`) |
| `nest g s books` | Servicio (`generate service`) |
| `nest g gu auth` / `nest g in timing` | Guard / interceptor |
| `nest g co books --no-spec` | Sin archivo de test |

### Scripts (`package.json`)

| Comando | Qué ejecuta |
|---------|-------------|
| `npm run start:dev` | `nest start --watch` → desarrollo con recarga |
| `npm run start:debug` | `nest start --debug --watch` |
| `npm run build` | `nest build` → `dist/` |
| `npm run start:prod` | `node dist/main` → producción |
| `npm run lint` / `npm run format` | ESLint / Prettier |
| `npm test` / `npm run test:e2e` / `npm run test:cov` | Unitarios / e2e / cobertura |

### Dependencias habituales

```bash
npm i class-validator class-transformer     # validación de DTOs
npm i pg && npm i -D @types/pg              # PostgreSQL
npm i @nestjs/mapped-types                  # PartialType, OmitType...
npm i @nestjs/config                        # ConfigModule (alternativa a loadEnvFile)
```

### Docker (Postgres local)

```bash
docker compose up -d                                                    # levantar
docker compose ps                                                       # estado
docker exec -i practica-sql psql -U alumno -d academia < database/schema.sql   # crear tablas
docker compose down                                                     # parar (datos en volumen)
```

---

## 2. Estructura

```
src/
├── main.ts                    ← arranque (bootstrap)
├── app.module.ts              ← módulo raíz
├── books/
│   ├── books.module.ts
│   ├── books.controller.ts    ← HTTP
│   ├── books.service.ts       ← lógica + SQL
│   ├── book.ts                ← tipos Book / BookRow + toBook()
│   └── dto/
│       ├── create-book.dto.ts
│       └── update-book.dto.ts
└── database/database.module.ts ← Pool de pg
test/app.e2e-spec.ts
```

---

## 3. `main.ts` (basado en `api-books-nest`)

```ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

process.loadEnvFile();                       // .env → process.env

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.setGlobalPrefix('api');                // todas las rutas: /api/...
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,                       // quita campos que no estén en el DTO
      forbidNonWhitelisted: true,            // ...o 400 si llegan
      transform: true,                       // aplica @Transform / convierte tipos
    }),
  );
  app.enableCors({ origin: process.env.CORS_ORIGIN ?? 'http://localhost:5173' });   // CORS (añadido; ver 02)
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

> 📌 En Render: `if (existsSync('.env')) process.loadEnvFile();` (allí no hay `.env`).

---

## 4. Módulos

```ts
@Module({
  imports: [DatabaseModule],          // módulos cuyos exports necesito
  controllers: [BooksController],     // rutas
  providers: [BooksService],          // servicios inyectables
  exports: [BooksService],            // lo que pueden usar otros módulos
})
export class BooksModule {}
```

```ts
@Module({ imports: [BooksModule, UsersModule, DatabaseModule] })
export class AppModule {}
```

---

## 5. Controladores y decoradores (REAL)

```ts
@Controller('books')                                     // /api/books
export class BooksController {
  constructor(private readonly booksService: BooksService) {}   // inyección

  @Get()                                                 // GET /api/books?author=x&maxPages=150
  findAll(@Query('author') author?: string, @Query('maxPages') maxPages?: string) {
    return this.booksService.findAll(author, maxPages ? Number(maxPages) : undefined);
  }

  @Get('count')                                          // GET /api/books/count (antes de :id)
  count() { return this.booksService.count(); }

  @Get(':id')                                            // GET /api/books/1
  findOne(@Param('id', ParseIntPipe) id: number) { return this.booksService.findOne(id); }

  @Post()                                                // POST → 201 por defecto
  create(@Body() dto: CreateBookDto) { return this.booksService.create(dto); }

  @Patch(':id')
  update(@Param('id', ParseIntPipe) id: number, @Body() dto: UpdateBookDto) {
    return this.booksService.update(id, dto);
  }

  @Delete(':id')
  @HttpCode(204)                                         // cambiar el código
  remove(@Param('id', ParseIntPipe) id: number) { return this.booksService.remove(id); }
}
```

| Decorador | Equivale en Express |
|-----------|---------------------|
| `@Controller('books')` | `app.use('/books', router)` |
| `@Get()`, `@Post()`, `@Put()`, `@Patch()`, `@Delete()` | `router.get(...)`… |
| `@Param('id')` | `req.params.id` |
| `@Query('author')` | `req.query.author` |
| `@Body()` | `req.body` |
| `@Headers('authorization')` | `req.headers.authorization` |
| `@HttpCode(204)` | `res.status(204)` |
| `return valor` | `res.json(valor)` |

---

## 6. Servicios e inyección de dependencias

```ts
@Injectable()
export class BooksService implements BooksContract {
  constructor(private readonly pool: Pool) {}            // Nest crea y pasa el Pool

  async findOne(id: number): Promise<Book> {
    const result = await this.pool.query<BookRow>(
      'SELECT id, title, author, pages, created_at FROM books WHERE id = $1',   // $1 = sin SQL injection
      [id],
    );
    if (result.rows.length === 0) throw new NotFoundException('Libro no encontrado');   // → 404
    return toBook(result.rows[0]);                       // snake_case → camelCase
  }
}
```

### Tipos de provider

```ts
providers: [
  BooksService,                                              // atajo de useClass
  { provide: BOOKS_SERVICE, useClass: BooksService },        // token propio
  { provide: 'API_URL', useValue: 'https://...' },           // valor fijo
  { provide: Pool, useFactory: () => new Pool({ connectionString: process.env.DATABASE_URL }) },
]
// Inyectar un token que no es clase:
constructor(@Inject(BOOKS_SERVICE) private readonly books: BooksContract) {}
```

---

## 7. Base de datos (`database.module.ts`, REAL)

```ts
import { Module } from '@nestjs/common';
import { Pool } from 'pg';

@Module({
  providers: [
    { provide: Pool, useFactory: () => new Pool({ connectionString: process.env.DATABASE_URL }) },
  ],
  exports: [Pool],
})
export class DatabaseModule {}
```

```ts
// Filtros opcionales en una sola consulta
const result = await this.pool.query<BookRow>(
  `SELECT id, title, author, pages, created_at FROM books
   WHERE ($1::text IS NULL OR author ILIKE '%' || $1 || '%')
     AND ($2::int IS NULL OR pages <= $2)
   ORDER BY id`,
  [author || null, maxPages ?? null],
);
```

```bash
# .env.example
DATABASE_URL=postgres://alumno:TU_PASSWORD@localhost:5432/academia
PORT=3000
```

---

## 8. DTOs y validación (REAL)

```ts
import { Transform } from 'class-transformer';
import { IsInt, IsString, Max, MaxLength, Min, MinLength } from 'class-validator';

export class CreateBookDto {
  @Transform(({ value }: { value: unknown }) => (typeof value === 'string' ? value.trim() : value))
  @IsString() @MinLength(1) @MaxLength(200)
  title: string;

  @IsString() @MinLength(1) @MaxLength(100)
  author: string;

  @IsInt()
  @Min(1, { message: 'El libro tiene que tener al menos 1 página' })
  @Max(2000)
  pages: number;
}

export class UpdateBookDto extends PartialType(CreateBookDto) {}   // todo opcional
```

| Decorador | Valida |
|-----------|--------|
| `@IsString()`, `@IsInt()`, `@IsNumber()`, `@IsBoolean()` | Tipo |
| `@IsEmail()`, `@IsUrl()`, `@IsUUID()` | Formato |
| `@MinLength(n)`, `@MaxLength(n)` | Longitud de texto |
| `@Min(n)`, `@Max(n)` | Rango numérico |
| `@IsOptional()` | Puede faltar |
| `@IsIn(['a','b'])`, `@IsEnum(E)` | Valores permitidos |

### Pipes

| Pipe | Qué hace |
|------|----------|
| `ValidationPipe` | Valida DTOs (global en `main.ts`) |
| `ParseIntPipe` | `"1"` → `1`; si no → 400 |
| `ParseUUIDPipe`, `ParseBoolPipe` | Igual para UUID / boolean |
| `new DefaultValuePipe(10)` | Valor por defecto en `@Query` |

---

## 9. Excepciones

| Excepción | Código |
|-----------|--------|
| `BadRequestException` | 400 |
| `UnauthorizedException` | 401 |
| `ForbiddenException` | 403 |
| `NotFoundException` | 404 |
| `ConflictException` | 409 |
| `InternalServerErrorException` | 500 |

```ts
throw new NotFoundException('Libro no encontrado');
// → { "statusCode": 404, "message": "Libro no encontrado", "error": "Not Found" }

if (err.code === '23505') throw new ConflictException('Ya existe');   // unique de Postgres
```

---

## 10. Guards e interceptors

```ts
@Injectable()
export class ApiKeyGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const req = context.switchToHttp().getRequest<Request>();
    return req.headers['x-api-key'] === process.env.API_KEY;   // false → 403
  }
}
@UseGuards(ApiKeyGuard)        // en un controlador o en un método
```

```ts
@Injectable()
export class TimingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const start = Date.now();
    return next.handle().pipe(tap(() => console.log(`${Date.now() - start} ms`)));   // después
  }
}
@UseInterceptors(TimingInterceptor)
```

| Pieza | Pregunta que responde |
|-------|------------------------|
| Middleware | ¿Algo antes de todo? (logs) |
| Guard | ¿Puede pasar? (auth, roles) |
| Pipe | ¿Datos válidos/transformados? |
| Interceptor | ¿Envuelvo antes/después? (tiempos, formato) |
| Filtro de excepciones | ¿Cómo respondo al error? |

---

## 11. Testing

```ts
// Unitario
const moduleRef = await Test.createTestingModule({
  controllers: [BooksController],
  providers: [{ provide: BooksService, useValue: { findOne: jest.fn().mockResolvedValue(book) } }],  // mock
}).compile();
const controller = moduleRef.get(BooksController);
await expect(controller.findOne(1)).resolves.toEqual(book);
```

```ts
// e2e con supertest
const app = moduleFixture.createNestApplication();
await app.init();
await request(app.getHttpServer()).get('/').expect(200);
```

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `Nest can't resolve dependencies of the BooksService (?)` | El provider no está en `providers` ni importado su módulo con `exports` |
| El DTO no valida nada | Falta `ValidationPipe` global o los paquetes `class-validator`/`class-transformer` |
| `property X should not exist` (400) | `forbidNonWhitelisted`: el cliente envía campos que no están en el DTO |
| `Validation failed (numeric string is expected)` | `ParseIntPipe` recibió un id no numérico (correcto: es un 400) |
| `GET /books/count` entra en `:id` | Declara `@Get('count')` antes de `@Get(':id')` |
| Rutas sin `/api` | Falta `app.setGlobalPrefix('api')` |
| `ECONNREFUSED 127.0.0.1:5432` | Postgres no levantado (`docker compose up -d`) o `DATABASE_URL` sin cargar |
| `nest: not found` en Render | `@nestjs/cli` es devDependency: instala sin `NODE_ENV=production` en el build |
