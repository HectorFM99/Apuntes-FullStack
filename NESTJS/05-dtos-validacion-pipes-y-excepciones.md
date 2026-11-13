# 05 — DTOs, VALIDACIÓN, PIPES Y EXCEPCIONES

> Nunca te fíes de lo que envía el cliente. Un **DTO** describe **qué forma** deben
> tener los datos; un **pipe** los **valida y transforma** antes de que lleguen al
> controlador; y las **excepciones** cortan la ejecución y devuelven un **error HTTP**
> con el código correcto.

---

## 5.1 ¿Qué es un DTO?

**DTO** = *Data Transfer Object* (objeto de transferencia de datos). Es una **clase**
que describe el cuerpo que debe enviar el cliente.

```ts
export class CreateBookDto {
  title: string;
  author: string;
  pages: number;
}
```

**¿Por qué una `class` y no una `interface`?**

```
interface → desaparece al compilar → en tiempo de ejecución no hay nada que validar
class     → existe en JS → los decoradores de class-validator se guardan en ella
```

> 📌 Por eso los DTOs **siempre son clases**. Las `interface` (como `Book`, `BookRow`)
> se usan para tipar datos que **tú** controlas (lo que sale de la BD o de la API).

---

## 5.2 `class-validator`: decoradores de validación

### Ejemplo REAL copiado (`api-books-nest/src/books/dto/create-book.dto.ts`)

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

**Decoradores más usados:**

| Decorador | Comprueba |
|-----------|-----------|
| `@IsString()` | Es texto |
| `@IsInt()` / `@IsNumber()` | Entero / número |
| `@IsBoolean()` | Booleano |
| `@IsEmail()` | Email válido |
| `@MinLength(n)` / `@MaxLength(n)` | Longitud de un texto |
| `@Min(n)` / `@Max(n)` | Valor de un número |
| `@IsOptional()` | Si falta (`undefined`/`null`), no valida el resto |
| `@ValidateIf(fn)` | Valida solo si `fn` devuelve `true` |
| `@IsIn(['a','b'])` | Uno de esos valores |
| `@IsNotEmpty()` | No vacío |

> 📌 Todos aceptan `{ message: '...' }` para personalizar el error, como el `@Min` de
> `pages`.

**¿Por qué `@MaxLength(200)` y `@MaxLength(100)`?** Porque coinciden con la tabla:

```sql
title VARCHAR(200) NOT NULL,
author VARCHAR(100) NOT NULL,
pages INTEGER NOT NULL CHECK (pages > 0),
```

> 📌 Validas **dos veces**: el DTO da un error **bonito (400)** al cliente; la BD es la
> **última línea de defensa** si algo se cuela.

---

## 5.3 `class-transformer`: `@Transform`

```ts
@Transform(({ value }: { value: unknown }) =>
  typeof value === 'string' ? value.trim() : value,
)
```

- Se ejecuta **antes** de validar (solo si el `ValidationPipe` tiene `transform: true`).
- `"  Rayuela  "` → `"Rayuela"`.
- Combinado con `@MinLength(1)`: un título `"   "` se convierte en `""` y **falla** la
  validación. ✅ ¡Sin el trim, un título de solo espacios sería válido!

> 📌 El `typeof value === 'string'` es para no romper si llega otra cosa (un número):
> en ese caso se deja igual y `@IsString()` se encarga de rechazarlo.

---

## 5.4 DTO para actualizar (PATCH)

### Ejemplo REAL copiado (`api-books-nest/src/books/dto/update-book.dto.ts`)

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

**`@ValidateIf` vs `@IsOptional`:**

| | `{ }` (no llega) | `{ "title": null }` | `{ "title": "" }` |
|--|-----------------|---------------------|-------------------|
| `@IsOptional()` | ✅ se salta | ✅ se salta (¡`null` pasa!) | ❌ `MinLength` falla |
| `@ValidateIf(v !== undefined)` | ✅ se salta | ❌ `IsString` falla | ❌ `MinLength` falla |

> 📌 Tu proyecto usa `@ValidateIf` a propósito: así **no se puede enviar `null`**. Y es
> importante, porque el `UPDATE` usa `COALESCE($1, title)`: un `null` significa
> "no cambies este campo".

### Alternativa con `PartialType` (ejemplo propio)

```ts
import { PartialType } from '@nestjs/mapped-types'; // paquete aparte
import { CreateBookDto } from './create-book.dto';

// Todos los campos de CreateBookDto, pero opcionales (con @IsOptional)
export class UpdateBookDto extends PartialType(CreateBookDto) {}
```

> ⚠️ Es más corto, pero usa `@IsOptional` (deja pasar `null`) y requiere instalar
> `@nestjs/mapped-types`.

---

## 5.5 `ValidationPipe`: activar la validación

Los decoradores del DTO **no hacen nada solos**. Alguien tiene que leerlos: el
`ValidationPipe`.

### Ejemplo REAL copiado (`api-books-nest/src/main.ts`)

```ts
// Comprueba los DTO en todas las rutas
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true, // quita los campos que no estén en el DTO...
    forbidNonWhitelisted: true, // ...o mejor: responde 400 si llegan
    transform: true, // aplica las transformaciones del DTO (como el trim de abajo)
  }),
);
```

| Opción | Efecto |
|--------|--------|
| `whitelist: true` | Quita del body las propiedades **sin decoradores** en el DTO |
| `forbidNonWhitelisted: true` | En vez de quitarlas, **responde 400** |
| `transform: true` | Convierte el JSON en una **instancia** del DTO y aplica `@Transform`; también convierte `@Param`/`@Query` al tipo primitivo declarado |

**Ejemplo de petición rechazada:**

```http
POST /api/books
Content-Type: application/json

{ "title": "", "author": "Borges", "pages": 0, "precio": 10 }
```

Respuesta **400**:

```json
{
  "message": [
    "property precio should not exist",
    "title must be longer than or equal to 1 characters",
    "El libro tiene que tener al menos 1 página"
  ],
  "error": "Bad Request",
  "statusCode": 400
}
```

> 📌 Gracias a `forbidNonWhitelisted`, nadie puede colar campos como
> `"role": "employee"` en un body. Es una medida de **seguridad**, no solo de orden.

```ts
// ❌ Sin ValidationPipe: el DTO es solo un "tipo", no se comprueba nada
create(@Body() dto: CreateBookDto) {}   // dto.pages podría ser "hola"

// ✅ Con ValidationPipe global: si llega mal, el handler ni se ejecuta
```

---

## 5.6 Pipes: qué son y cuáles trae Nest

Un **pipe** recibe un valor de entrada y:
1. lo **transforma** (string → número), y/o
2. lo **valida** (si no vale, lanza una excepción → 400).

### Ejemplo REAL copiado (`api-books-nest/src/books/books.controller.ts`)

```ts
@Patch(':id') // PATCH /api/books/1
update(@Param('id', ParseIntPipe) id: number, @Body() dto: UpdateBookDto) {
  return this.booksService.update(id, dto);
}
```

| Pipe | Hace |
|------|------|
| `ValidationPipe` | Valida DTOs con `class-validator` |
| `ParseIntPipe` | `"7"` → `7`; `"abc"` → 400 |
| `ParseFloatPipe` | `"3.5"` → `3.5` |
| `ParseBoolPipe` | `"true"` → `true` |
| `ParseUUIDPipe` | Comprueba que es un UUID |
| `ParseEnumPipe` | Valor dentro de un enum |
| `DefaultValuePipe` | Pone un valor por defecto si falta |

**Dónde se pueden poner:**

```ts
@Param('id', ParseIntPipe) id: number                // en un parámetro
@UsePipes(new ValidationPipe()) @Post() create() {}  // en un handler
app.useGlobalPipes(new ValidationPipe())             // global (tu proyecto)
```

```ts
// Ejemplo propio: paginación con valores por defecto
@Get()
findAll(
  @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
  @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit: number,
) {}
```

### Pipe personalizado (ejemplo propio)

```ts
import { BadRequestException, Injectable, PipeTransform } from '@nestjs/common';

@Injectable()
export class PositiveIntPipe implements PipeTransform<string, number> {
  transform(value: string): number {
    const n = Number(value);
    if (!Number.isInteger(n) || n <= 0) {
      throw new BadRequestException('El id debe ser un entero positivo');
    }
    return n;
  }
}

// Uso
@Get(':id')
findOne(@Param('id', PositiveIntPipe) id: number) {}
```

---

## 5.7 Excepciones HTTP

En Nest, para devolver un error **lanzas** (`throw`) una excepción. Nest la captura y
responde con el código y un JSON.

### Ejemplo REAL copiado (`api-books-nest/src/books/books.service.ts`)

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

Respuesta de `GET /api/books/999`:

```json
{
  "message": "Libro no encontrado",
  "error": "Not Found",
  "statusCode": 404
}
```

### Ejemplo REAL copiado (`api-books-nest/src/books/books.service.ts`)

```ts
async update(id: number, dto: UpdateBookDto): Promise<Book> {
  if (dto.title === undefined && dto.author === undefined && dto.pages === undefined) {
    throw new BadRequestException('Envía al menos un campo');
  }
  const book = await this.findOne(id);
  // ...
}
```

- `PATCH /api/books/1` con `{}` → **400** "Envía al menos un campo".
- `PATCH /api/books/999` con `{ "pages": 10 }` → `findOne` lanza **404**.

> 📌 La excepción **atraviesa** todas las funciones: `update` llama a `findOne`, que
> lanza; no hace falta `try/catch` ni comprobar nada en el controlador.

**Excepciones incluidas en `@nestjs/common`:**

| Clase | Código | Cuándo |
|-------|--------|--------|
| `BadRequestException` | 400 | Datos incorrectos |
| `UnauthorizedException` | 401 | No autenticado (sin login / token malo) |
| `ForbiddenException` | 403 | Autenticado pero sin permiso |
| `NotFoundException` | 404 | No existe |
| `ConflictException` | 409 | Choca con algo existente (email repetido) |
| `UnprocessableEntityException` | 422 | Semánticamente inválido |
| `InternalServerErrorException` | 500 | Error del servidor |

```ts
// ❌ Estilo Express dentro de Nest
if (!book) return { error: 'no existe' };       // ¡responde 200!

// ✅
if (!book) throw new NotFoundException('Libro no encontrado'); // 404
```

### `HttpException` genérica (ejemplo propio)

Todas las anteriores heredan de `HttpException`. Puedes usarla directamente:

```ts
import { HttpException, HttpStatus } from '@nestjs/common';

throw new HttpException('Demasiadas peticiones', HttpStatus.TOO_MANY_REQUESTS); // 429

throw new HttpException(
  { statusCode: 418, message: 'Soy una tetera', extra: 'lo que quieras' },
  418,
);
```

---

## 5.8 Errores de la base de datos → errores HTTP (ejemplo propio)

Si en `users` insertas un email repetido, PostgreSQL lanza un error con código `23505`
(violación de `UNIQUE`). Sin tratarlo, el cliente recibe un **500**. Lo correcto:

```ts
async create(email: string, passwordHash: string): Promise<User> {
  try {
    const result = await this.pool.query<User>(/* INSERT ... */);
    return result.rows[0];
  } catch (error) {
    if ((error as { code?: string }).code === '23505') {
      throw new ConflictException('Ese email ya está registrado'); // 409
    }
    throw error; // otros errores: que sigan siendo 500
  }
}
```

---

## 5.9 Exception filters (concepto; no están en tu proyecto)

Por defecto Nest tiene un **filtro global** que convierte cualquier `HttpException` en
JSON, y cualquier otro error en un **500** "Internal server error".

Si quieres **cambiar el formato** de los errores, creas tu propio filtro:

```ts
import { ArgumentsHost, Catch, ExceptionFilter, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)                       // qué excepciones captura
export class HttpErrorFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const res = ctx.getResponse<Response>();
    const req = ctx.getRequest<Request>();
    const status = exception.getStatus();

    res.status(status).json({
      ok: false,
      status,
      path: req.url,
      fecha: new Date().toISOString(),
      error: exception.getResponse(),
    });
  }
}

// main.ts
app.useGlobalFilters(new HttpErrorFilter());
// o en un controlador: @UseFilters(HttpErrorFilter)
```

> 📌 Es el equivalente al **middleware de errores** de Express
> (`app.use((err, req, res, next) => ...)`).

---

## ✅ Resumen rápido

1. **DTO** = `class` con decoradores de `class-validator` (`@IsString`, `@Min`...).
2. `@Transform` (de `class-transformer`) limpia datos antes de validar (`trim`).
3. Para PATCH: campos `?` + `@ValidateIf(v !== undefined)` (o `@IsOptional`).
4. `ValidationPipe` global con `whitelist`, `forbidNonWhitelisted`, `transform`.
5. `ParseIntPipe` convierte `:id` a número o responde 400.
6. Errores = `throw new NotFoundException(...)`, `BadRequestException(...)`...
7. Filtros de excepciones para personalizar el formato de error.

👉 **Siguiente:** `06-base-de-datos-y-configuracion.md`
