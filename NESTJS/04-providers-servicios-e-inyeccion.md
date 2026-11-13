# 04 — PROVIDERS, SERVICIOS E INYECCIÓN DE DEPENDENCIAS

> Un **provider** es cualquier cosa que Nest puede **crear e inyectar**: servicios,
> repositorios, conexiones a la BD... Un **servicio** es el provider más típico: una
> clase con `@Injectable()` que contiene la **lógica de negocio**.
> La **inyección de dependencias (DI)** es el mecanismo por el que Nest te da esas
> instancias **sin que tú hagas `new`**.

---

## 4.1 Un servicio mínimo

### Ejemplo REAL copiado (`api-books-nest/src/app.service.ts`)

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hola desde NestJS';
  }
}
```

**Puntos clave:**
- `@Injectable()` marca la clase como **inyectable**: Nest puede crearla y dársela a
  quien la pida.
- Para que exista, tiene que estar en `providers` de algún módulo:

```ts
@Module({
  controllers: [AppController],
  providers: [AppService],   // ← registrado
})
export class AppModule {}
```

---

## 4.2 Inyección por constructor

```ts
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}
  //          └──────┬───────┘ └───┬────┘  └───┬────┘
  //     atajo de TS: crea      nombre    TIPO = qué quiero
  //     this.appService
}
```

**¿Qué pasa por dentro?**

```
1. Nest ve que AppController necesita un AppService (lo sabe por el TIPO,
   gracias a emitDecoratorMetadata).
2. Busca AppService en los providers del módulo.
3. Si no existe aún, lo crea (new AppService()) — UNA sola vez.
4. Se lo pasa al constructor de AppController.
```

```ts
// ❌ Sin DI: tú creas las dependencias → acoplado y difícil de testear
export class AppController {
  private appService = new AppService();
}

// ✅ Con DI: las pides → Nest decide qué instancia darte (en tests, una falsa)
export class AppController {
  constructor(private readonly appService: AppService) {}
}
```

> 📌 `private readonly` en el parámetro del constructor es un **atajo de TypeScript**:
> declara la propiedad `this.appService` y la asigna, todo en una línea.
> `readonly` evita que la reasignes por error.

---

## 4.3 Singletons: una instancia para todos

Por defecto, cada provider es un **singleton** dentro de la app: se crea **una vez** y
todos los que lo piden reciben **la misma** instancia.

```
BooksController ──┐
                  ├──▶  misma instancia de BooksService
(otro consumidor)─┘

BooksService  ──┐
                ├──▶  mismo Pool (una sola conexión compartida)
UsersService ───┘
```

> 📌 Por eso **no guardes datos de una petición** en propiedades del servicio
> (`this.usuarioActual = ...`): ¡lo compartirían todas las peticiones!

---

## 4.4 Un servicio real: `BooksService`

### Ejemplo REAL copiado (`api-books-nest/src/books/books.service.ts`, fragmento)

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

  async remove(id: number): Promise<Book> {
    const book = await this.findOne(id);
    await this.pool.query('DELETE FROM books WHERE id = $1', [id]);
    return book;
  }
}
```

**Cadena de inyección completa:**

```
BooksController  pide  BooksService   (está en providers de BooksModule)
BooksService     pide  Pool           (viene de DatabaseModule, que lo exporta)
Pool             se fabrica con useFactory → new Pool({ connectionString })
```

> 📌 Un servicio puede **reutilizar sus propios métodos**: `remove` llama a
> `this.findOne(id)`, que ya lanza el 404 si no existe. Así no repites la comprobación.

---

## 4.5 Contratos con `interface`

### Ejemplo REAL copiado (`api-books-nest/src/books/books.contract.ts`)

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

Y el servicio lo **implementa**:

```ts
@Injectable()
export class BooksService implements BooksContract  {
```

**¿Qué ganas?**
- Si te olvidas de un método, o lo escribes con otro tipo, **TypeScript da error**.
- Documenta de un vistazo **qué ofrece** el servicio.
- Permite tener **otra implementación** (por ejemplo, en memoria para tests).

> 📌 Fíjate: el contrato dice `findAll(author?: string)` y el servicio tiene
> `findAll(author?: string, maxPages?: number)`. Es válido: un parámetro opcional **de
> más** sigue cumpliendo el contrato. Y `count()` está en el servicio pero no en el
> contrato: una clase puede tener **más** métodos que su interfaz.

### ⚠️ Una interface NO se puede inyectar directamente

Las `interface` de TypeScript **desaparecen al compilar** (no existen en JS), así que
Nest no puede usarlas como "etiqueta":

```ts
// ❌ No funciona: BooksContract no existe en tiempo de ejecución
constructor(private readonly books: BooksContract) {}
```

Si quisieras inyectar "cualquier cosa que cumpla `BooksContract`", necesitas un
**token** (siguiente apartado). En tu proyecto, el contrato se usa solo con
`implements` (comprobación en compilación), y se inyecta la clase `BooksService`.

---

## 4.6 Tokens y providers personalizados

Cuando escribes `providers: [BooksService]`, en realidad es un atajo de:

```ts
providers: [
  { provide: BooksService, useClass: BooksService },
]
// provide  = el TOKEN (la etiqueta con la que se pide)
// useClass = qué clase instanciar
```

Hay cuatro formas de decir **cómo se crea** el provider:

| Forma | Ejemplo | Cuándo |
|-------|---------|--------|
| `useClass` | `{ provide: BooksService, useClass: BooksService }` | Lo normal (atajo: `BooksService`) |
| `useValue` | `{ provide: 'CONFIG', useValue: { port: 3000 } }` | Un valor fijo, o un mock en tests |
| `useFactory` | `{ provide: Pool, useFactory: () => new Pool(...) }` | Hace falta código para crearlo |
| `useExisting` | `{ provide: 'ALIAS', useExisting: BooksService }` | Un alias de otro provider |

### Ejemplo REAL copiado (`api-books-nest/src/database/database.module.ts`)

```ts
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

- El **token** es la propia clase `Pool` de `pg` (una clase sí existe en JS).
- `Pool` no tiene `@Injectable()` (es de una librería), por eso hace falta `useFactory`
  para decirle a Nest **cómo** crearlo.
- Como el token es una clase, en los servicios basta con tipar:
  `constructor(private readonly pool: Pool) {}` ✨

### Token de texto o símbolo + `@Inject` (ejemplo propio)

```ts
// books.tokens.ts
export const BOOKS_SERVICE = Symbol('BOOKS_SERVICE');

// books.module.ts
providers: [{ provide: BOOKS_SERVICE, useClass: BooksService }],

// books.controller.ts → pides por token y tipas con la interface
import { Inject } from '@nestjs/common';

constructor(@Inject(BOOKS_SERVICE) private readonly books: BooksContract) {}
```

> 📌 Así el controlador depende del **contrato**, no de la clase concreta. Podrías
> cambiar `useClass: BooksService` por `useClass: BooksMemoryService` sin tocar el
> controlador.

---

## 4.7 Ciclo de vida: `OnModuleInit` / `OnModuleDestroy`

Nest avisa a los providers en ciertos momentos si implementan estos "hooks":

| Hook | Cuándo se llama |
|------|-----------------|
| `onModuleInit()` | Cuando el módulo se ha creado |
| `onApplicationBootstrap()` | Cuando toda la app está lista |
| `onModuleDestroy()` | Al cerrar la app (`app.close()`) |

### Ejemplo REAL copiado (`api-books-nest/src/database/database.service.ts`)

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

**Qué hace:**
- Crea su propio `Pool`, pero **comprobando** que `DATABASE_URL` existe (si no, error
  claro al arrancar en vez de un fallo raro en la primera consulta).
- Al apagar la app, `onModuleDestroy` cierra las conexiones con `pool.end()`.

> ⚠️ **Ojo:** en el estado actual de tu proyecto, `DatabaseService` **no está
> registrado** en ningún `providers` (el `DatabaseModule` usa el `useFactory`). Es decir,
> es código preparado pero que Nest no está usando. Son dos enfoques alternativos:
>
> | Enfoque | Se inyecta | Valida la URL | Cierra el pool |
> |---------|-----------|---------------|----------------|
> | `useFactory` (el que se usa) | `Pool` | ❌ | ❌ |
> | `DatabaseService` | `DatabaseService` (`this.db.pool.query`) | ✅ | ✅ |

Para usarlo habría que registrarlo y exportarlo (ejemplo propio):

```ts
@Module({
  providers: [DatabaseService],
  exports: [DatabaseService],
})
export class DatabaseModule {}

// y en los servicios:
constructor(private readonly db: DatabaseService) {}
// this.db.pool.query(...)
```

---

## 4.8 Un servicio exportado: `UsersService`

### Ejemplo REAL copiado (`api-books-nest/src/users/users.service.ts`, fragmento)

```ts
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
}
```

- Devuelve `User | undefined` en vez de lanzar 404: así **quien lo use** (un futuro
  `AuthService`) decide qué error dar (p. ej. 401 "credenciales incorrectas").
- `UsersModule` lo **exporta** para que otro módulo lo inyecte.

> 📌 Regla práctica: los servicios que responden directamente a una ruta lanzan
> excepciones HTTP (`NotFoundException`); los servicios "internos" que usan otros
> servicios suelen devolver `undefined`/`null` y dejan decidir al que llama.

---

## 4.9 Errores comunes de inyección

```
Nest can't resolve dependencies of the BooksService (?).
Please make sure that the argument Pool at index [0] is available in the BooksModule context.
```

| Causa | Solución |
|-------|----------|
| El servicio no está en `providers` | Añádelo al módulo |
| La dependencia viene de otro módulo que **no está importado** | `imports: [DatabaseModule]` |
| El otro módulo **no la exporta** | `exports: [Pool]` |
| Olvidaste `@Injectable()` en la clase que tiene dependencias | Añádelo |
| Intentas inyectar una `interface` | Usa un token + `@Inject(TOKEN)` |
| Dependencia circular (A pide B y B pide A) | Rediseña, o `forwardRef(() => B)` |

---

## ✅ Resumen rápido

1. `@Injectable()` + registrarlo en `providers` = Nest puede inyectarlo.
2. Se pide en el **constructor**: `constructor(private readonly x: Tipo) {}`.
3. Los providers son **singletons** (una instancia compartida).
4. `providers: [X]` es atajo de `{ provide: X, useClass: X }`.
5. `useValue`, `useFactory`, `useExisting` para casos especiales (como el `Pool`).
6. Una `interface` sirve para `implements` (contrato), pero para **inyectarla** hace
   falta un token y `@Inject`.
7. `onModuleDestroy()` → liberar recursos al apagar.

👉 **Siguiente:** `05-dtos-validacion-pipes-y-excepciones.md`
