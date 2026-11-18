# 03 — CONTROLADORES Y DECORADORES DE RUTAS

> Un **controlador** es la clase que **recibe las peticiones HTTP** y devuelve la
> respuesta. Su único trabajo es: leer lo que llega (params, query, body), llamar al
> **servicio** y devolver el resultado. **Nada de lógica de negocio ni SQL aquí.**

---

## 3.1 Anatomía de un controlador

```ts
import { Controller, Get } from '@nestjs/common';

@Controller('ruta-base')            // 1. prefijo de todas las rutas de la clase
export class EjemploController {
  constructor(private readonly servicio: EjemploService) {} // 2. pide el servicio

  @Get()                            // 3. método HTTP + subruta
  listar() {
    return this.servicio.listar();  // 4. lo que devuelves se envía como JSON
  }
}
```

**Puntos clave:**
- `@Controller('books')` → todas las rutas de la clase empiezan por `/books`.
- Cada **método** con `@Get`, `@Post`... es un **handler** (manejador de ruta).
- **Lo que devuelves** es la respuesta:
  - un objeto o array → se envía como **JSON**;
  - un `string` → se envía como texto;
  - una **Promise** → Nest espera (`await`) y envía el resultado.
- Código de estado por defecto: **200**, salvo `POST` que es **201**.

### Ejemplo REAL copiado (`api-books-nest/src/app.controller.ts`)

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

| Petición | Handler | Respuesta |
|----------|---------|-----------|
| `GET /api` | `getHello()` | `Hola desde NestJS` (texto) |
| `GET /api/ping` | `ping()` | `{ "pong": true }` (JSON) |

> 📌 `@Controller()` sin argumento = ruta base vacía. Con el prefijo global, cuelga de
> `/api`.

---

## 3.2 Comparación con Express

```js
// EXPRESS
app.get('/api/ping', (req, res) => {
  res.json({ pong: true });
});
```

```ts
// NESTJS
@Get('ping')
ping() {
  return { pong: true };   // sin res.json(): Nest lo hace por ti
}
```

> 📌 En Nest **no tocas `req` ni `res`** normalmente. Pides solo lo que necesitas con
> decoradores (`@Param`, `@Body`...) y **devuelves** el resultado.

---

## 3.3 Decoradores de método HTTP

| Decorador | Método | Uso típico |
|-----------|--------|-----------|
| `@Get()` | GET | Leer (listar / obtener uno) |
| `@Post()` | POST | Crear |
| `@Put()` | PUT | Reemplazar entero |
| `@Patch()` | PATCH | Modificar parcialmente |
| `@Delete()` | DELETE | Borrar |

### Ejemplo REAL copiado (`api-books-nest/src/books/books.controller.ts`)

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

**Tabla de rutas de tu API de libros:**

| Petición | Handler | Servicio |
|----------|---------|----------|
| `GET /api/books` | `findAll` | `booksService.findAll(author, max)` |
| `GET /api/books?author=garcía&maxPages=150` | `findAll` | con filtros |
| `GET /api/books/count` | `count` | `booksService.count()` |
| `GET /api/books/1` | `findOne` | `booksService.findOne(1)` |
| `POST /api/books` | `create` | `booksService.create(dto)` |
| `PATCH /api/books/1` | `update` | `booksService.update(1, dto)` |

---

## 3.4 ⚠️ El ORDEN de las rutas importa

Fíjate en que `@Get('count')` está **antes** que `@Get(':id')`. ¿Por qué?

```ts
// ❌ Si :id va primero...
@Get(':id')    // "count" encaja con :id → ParseIntPipe("count") → 400 Bad Request
findOne(...) {}

@Get('count')  // nunca se llega aquí
count() {}
```

```ts
// ✅ Rutas fijas ANTES que rutas con parámetros
@Get('count')
count() {}

@Get(':id')
findOne(...) {}
```

> 📌 Nest registra las rutas **en el orden en que las escribes**. Las rutas concretas
> (`count`, `search`, `me`...) siempre **arriba** de las genéricas (`:id`).

---

## 3.5 `@Param` — parámetros de ruta

```
GET /api/books/7
               └── :id = "7"
```

```ts
@Get(':id')
findOne(@Param('id') id: string) {          // llega SIEMPRE como string
  return this.booksService.findOne(Number(id));
}
```

Mejor, con un **pipe** que convierte y valida (ver `05`):

### Ejemplo REAL copiado (`api-books-nest/src/books/books.controller.ts`)

```ts
@Get(':id') // GET /api/books/1
findOne(@Param('id', ParseIntPipe) id: number) {
  return this.booksService.findOne(id);
}
```

| Petición | Resultado |
|----------|-----------|
| `GET /api/books/1` | `id = 1` (número) → busca el libro |
| `GET /api/books/abc` | **400** automático: `Validation failed (numeric string is expected)` |

```ts
// Varios parámetros (ejemplo propio)
@Get(':bookId/reviews/:reviewId')
findReview(
  @Param('bookId', ParseIntPipe) bookId: number,
  @Param('reviewId', ParseIntPipe) reviewId: number,
) {}

// Todos de golpe como objeto (menos recomendable: sin tipos ni pipes por campo)
@Get(':id')
findOne(@Param() params: { id: string }) {}
```

---

## 3.6 `@Query` — parámetros de consulta (`?clave=valor`)

```
GET /api/books?author=gabriel&maxPages=150
              └──────────── query ─────────┘
```

### Ejemplo REAL copiado (`api-books-nest/src/books/books.controller.ts`)

```ts
@Get() // GET /api/books?author=gabriel&maxPages=150
findAll(
  @Query('author') author?: string,
  @Query('maxPages') maxPages?: string, // llega como texto: "150"
) {
  const max = maxPages ? Number(maxPages) : undefined; // lo pasamos a número
  return this.booksService.findAll(author, max);
}
```

**Puntos clave:**
- Los query params son **opcionales** → por eso `author?: string`.
- Si no llegan, valen `undefined`.
- Todo llega como **texto** → `Number(maxPages)` para convertir.

> 📌 Alternativa más "Nest": un pipe opcional que convierte y valida.
>
> ```ts
> @Query('maxPages', new ParseIntPipe({ optional: true })) maxPages?: number
> ```
> Si llega `?maxPages=abc` → 400 automático, en vez de `NaN`.

---

## 3.7 `@Body` — el cuerpo de la petición

```ts
@Post() // POST /api/books
create(@Body() dto: CreateBookDto) {
  return this.booksService.create(dto); // el ValidationPipe ya comprobó el DTO
}
```

Petición:

```http
POST /api/books
Content-Type: application/json

{ "title": "  Rayuela ", "author": "Julio Cortázar", "pages": 600 }
```

- `@Body()` sin argumento → **todo** el JSON, tipado como `CreateBookDto`.
- `@Body('title')` → solo ese campo (poco usado).
- Gracias al `ValidationPipe` global, si el body no cumple el DTO, el handler **ni se
  ejecuta** (400). Y el `@Transform` del DTO le quita los espacios a `title`.

> ⚠️ No hace falta `app.use(express.json())`: Nest ya parsea el JSON por ti.

---

## 3.8 `@HttpCode` y otros decoradores útiles (no están en tu proyecto)

```ts
import { Delete, HttpCode, HttpStatus, Header, Headers, Redirect, Req, Res } from '@nestjs/common';

@Delete(':id')
@HttpCode(HttpStatus.NO_CONTENT)   // 204 en vez de 200
remove(@Param('id', ParseIntPipe) id: number) {
  return this.booksService.remove(id);
}

@Post('login')
@HttpCode(200)                     // un login no "crea" nada → 200 en vez de 201
login(@Body() dto: LoginDto) {}

@Get('privado')
leerCabecera(@Headers('authorization') auth?: string) {} // leer una cabecera

@Get('csv')
@Header('Content-Type', 'text/csv') // poner una cabecera en la respuesta
exportar() {}
```

| Decorador | Para qué |
|-----------|---------|
| `@HttpCode(204)` | Cambiar el código de estado de éxito |
| `@Headers('x')` | Leer una cabecera de la petición |
| `@Header('k', 'v')` | Poner una cabecera en la respuesta |
| `@Req()` / `@Res()` | Acceder al `req`/`res` de Express (evítalo salvo necesidad) |

> ⚠️ Si usas `@Res()`, Nest **deja de enviar la respuesta por ti**: tendrás que llamar a
> `res.json()` tú. Por eso casi nunca se usa.

---

## 3.9 💡 Ejercicio: la ruta DELETE que falta

Tu `BooksService` ya tiene un método `remove(id)` (y el contrato `BooksContract` lo
exige), pero `BooksController` **no tiene** ruta `@Delete`. Así quedaría:

```ts
// Ejemplo propio: NO está en tu proyecto
import { Delete } from '@nestjs/common';

@Delete(':id') // DELETE /api/books/1
remove(@Param('id', ParseIntPipe) id: number) {
  return this.booksService.remove(id); // devuelve el libro borrado (200)
}
```

> 📌 Tu servicio devuelve el libro borrado, así que tiene sentido responder **200** con
> él. Si no devolvieras nada, lo correcto sería `@HttpCode(204)`.

---

## 3.10 ❌/✅ Responsabilidades del controlador

```ts
// ❌ Controlador "gordo": SQL, validación y errores a mano
@Get(':id')
async findOne(@Param('id') id: string) {
  const n = Number(id);
  if (isNaN(n)) throw new BadRequestException('id mal');
  const r = await this.pool.query('SELECT * FROM books WHERE id = $1', [n]);
  if (!r.rows[0]) throw new NotFoundException();
  return r.rows[0];
}
```

```ts
// ✅ Controlador "fino": delega (así está en tu proyecto)
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  return this.booksService.findOne(id);
}
```

| Capa | Hace |
|------|------|
| **Pipe** | Convertir/validar la entrada (`ParseIntPipe`, `ValidationPipe`) |
| **Controlador** | Mapear la petición HTTP → llamada al servicio |
| **Servicio** | Lógica, SQL, lanzar `NotFoundException` |

---

## ✅ Resumen rápido

1. `@Controller('books')` + `@Get/@Post/@Patch/@Delete('subruta')`.
2. Lo que **devuelves** es la respuesta (JSON automático; las Promises se esperan).
3. `@Param('id')` → ruta; `@Query('x')` → `?x=`; `@Body()` → JSON del cuerpo.
4. Todo lo de la URL llega como **string** → usa `ParseIntPipe` o convierte.
5. Rutas fijas (`count`) **antes** que rutas con parámetros (`:id`).
6. `@HttpCode()` cambia el estado por defecto (200, o 201 en POST).
7. Controladores **finos**: la lógica va en el servicio.

👉 **Siguiente:** `04-providers-servicios-e-inyeccion.md`
