# 07 — GUARDS, INTERCEPTORS, MIDDLEWARE Y TESTING

> **Guards**, **interceptors** y **middleware** son piezas que se ejecutan
> **alrededor** de tus controladores (antes o después) para cosas transversales:
> autenticación, logs, formato de respuestas... Tu proyecto **aún no los usa** (aunque
> `UsersService` ya menciona un futuro `AuthGuard`), así que aquí van con **ejemplos
> propios**. La segunda parte trata los **tests**, que sí están en tu proyecto.

---

## 7.1 Orden de ejecución

```
Petición
  │
  ▼
Middleware        → estilo Express (req, res, next). Logs, cabeceras...
  │
  ▼
Guards            → ¿PUEDE pasar? (true/false). Autenticación, roles
  │
  ▼
Interceptors (antes)
  │
  ▼
Pipes             → validar/transformar (ValidationPipe, ParseIntPipe)
  │
  ▼
Controlador → Servicio
  │
  ▼
Interceptors (después)  → transformar la respuesta, medir tiempos
  │
  ▼
Exception filters (si algo lanzó una excepción)
  │
  ▼
Respuesta
```

| Pieza | Pregunta que responde | Se crea con |
|-------|----------------------|-------------|
| Middleware | ¿Hago algo con la petición cruda? | `NestMiddleware` |
| Guard | ¿Dejo pasar esta petición? | `CanActivate` |
| Interceptor | ¿Envuelvo la ejecución (antes/después)? | `NestInterceptor` |
| Pipe | ¿Los datos de entrada son válidos? | `PipeTransform` (ver `05`) |
| Filter | ¿Cómo respondo a un error? | `ExceptionFilter` (ver `05`) |

---

## 7.2 Middleware (ejemplo propio)

Es **exactamente** el middleware de Express: recibe `req`, `res`, `next`.

```ts
// logger.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { NextFunction, Request, Response } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`${req.method} ${req.originalUrl}`);
    next(); // ⚠️ sin next() la petición se queda colgada
  }
}
```

Se registra en un módulo implementando `NestModule`:

```ts
import { MiddlewareConsumer, Module, NestModule } from '@nestjs/common';

@Module({ /* imports, controllers, providers */ })
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('*'); // o forRoutes(BooksController)
  }
}
```

> 📌 También puedes usar middleware de Express directamente en `main.ts`:
> `app.use(helmet())`, `app.use(cookieParser())`...

---

## 7.3 Guards (ejemplo propio)

Un guard devuelve `true` (pasa) o `false`/lanza excepción (no pasa). Es el sitio para la
**autenticación** y la **autorización**.

```ts
// auth.guard.ts — boceto de lo que podría usar tu UsersService.findById
import { CanActivate, ExecutionContext, Injectable, UnauthorizedException } from '@nestjs/common';
import { Request } from 'express';
import { UsersService } from '../users/users.service';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private readonly usersService: UsersService) {} // ¡los guards también reciben DI!

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const req = context.switchToHttp().getRequest<Request & { user?: unknown }>();
    const userId = obtenerIdDeLaSesionOToken(req); // cookie de sesión, JWT...

    if (!userId) throw new UnauthorizedException(); // 401
    const user = await this.usersService.findById(userId);
    if (!user) throw new UnauthorizedException();

    req.user = user; // lo dejamos a mano para el controlador
    return true;
  }
}
```

**Uso:**

```ts
import { UseGuards } from '@nestjs/common';

@UseGuards(AuthGuard)            // en todo el controlador...
@Controller('books')
export class BooksController {
  @UseGuards(AuthGuard)          // ...o solo en un handler
  @Post()
  create(@Body() dto: CreateBookDto) {}
}
```

### Guard de roles (ejemplo propio)

Tu tabla `users` tiene `role IN ('customer', 'employee')`. Un guard podría permitir
crear libros solo a empleados:

```ts
import { CanActivate, ExecutionContext, ForbiddenException, Injectable } from '@nestjs/common';

@Injectable()
export class EmployeeGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const req = context.switchToHttp().getRequest<{ user?: { role: string } }>();
    if (req.user?.role !== 'employee') {
      throw new ForbiddenException('Solo empleados'); // 403
    }
    return true;
  }
}

// Se ejecutan en orden: primero autentica, luego comprueba el rol
@UseGuards(AuthGuard, EmployeeGuard)
@Post()
create(@Body() dto: CreateBookDto) {}
```

> 📌 **401 vs 403:** 401 = "no sé quién eres"; 403 = "sé quién eres, pero no puedes".

---

## 7.4 Interceptors (ejemplo propio)

Un interceptor envuelve al handler: código **antes**, y con RxJS (`pipe`, `map`, `tap`)
código **después**.

```ts
import { CallHandler, ExecutionContext, Injectable, NestInterceptor } from '@nestjs/common';
import { Observable, map, tap } from 'rxjs';

// 1) Medir cuánto tarda cada petición
@Injectable()
export class TimingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<unknown> {
    const inicio = Date.now();                       // ANTES
    return next.handle().pipe(
      tap(() => console.log(`Tardó ${Date.now() - inicio} ms`)), // DESPUÉS
    );
  }
}

// 2) Envolver todas las respuestas en { data: ... }
@Injectable()
export class WrapInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<unknown> {
    return next.handle().pipe(map((data) => ({ data })));
  }
}
```

```ts
@UseInterceptors(TimingInterceptor)         // en un controlador o handler
app.useGlobalInterceptors(new WrapInterceptor()); // o global en main.ts
```

> 📌 Por esto `rxjs` está en tus dependencias: Nest lo usa para los interceptors.

---

## 7.5 Testing: herramientas

Tu proyecto viene con **Jest** configurado:

### Ejemplo REAL copiado (`api-books-nest/package.json`, sección `jest`)

```json
"jest": {
  "moduleFileExtensions": [
    "js",
    "json",
    "ts"
  ],
  "rootDir": "src",
  "testRegex": ".*\\.spec\\.ts$",
  "transform": {
    "^.+\\.(t|j)s$": "ts-jest"
  },
  "collectCoverageFrom": [
    "**/*.(t|j)s"
  ],
  "coverageDirectory": "../coverage",
  "testEnvironment": "node"
}
```

| Tipo de test | Archivos | Comando | Qué prueba |
|--------------|----------|---------|-----------|
| **Unitario** | `src/**/*.spec.ts` | `npm run test` | Una clase aislada |
| **E2E** | `test/*.e2e-spec.ts` | `npm run test:e2e` | La app entera vía HTTP |
| Cobertura | — | `npm run test:cov` | % de código probado |

**Vocabulario de Jest:**

```ts
describe('Grupo', () => {          // agrupa tests
  beforeEach(() => { /* ... */ }); // se ejecuta antes de CADA test
  afterEach(() => { /* ... */ });  // después de cada test
  it('hace algo', () => {          // un test (también: test(...))
    expect(valor).toBe(esperado);  // aserción
  });
});
```

| Matcher | Comprueba |
|---------|-----------|
| `toBe(x)` | Igualdad estricta (`===`) |
| `toEqual(obj)` | Igualdad de contenido (objetos/arrays) |
| `toHaveBeenCalledWith(...)` | Un mock fue llamado con esos argumentos |
| `rejects.toThrow(Clase)` | Una promesa falla con esa excepción |

---

## 7.6 Test unitario con `Test.createTestingModule`

### Ejemplo REAL copiado (`api-books-nest/src/app.controller.spec.ts`)

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

**Paso a paso:**

```
Test.createTestingModule({...})  → un módulo "de mentira", como un @Module
.compile()                        → Nest resuelve las dependencias (la DI de verdad)
app.get(AppController)            → saca la instancia ya construida
appController.getHello()          → llamas al método directamente (sin HTTP)
```

> 📌 El test se actualizó al cambiar el texto a `'Hola desde NestJS'`. Si cambias lo que
> devuelve un servicio, **actualiza su test**.

---

## 7.7 Mockear dependencias (ejemplo propio)

Para testear `BooksService` **sin base de datos**, se sustituye el `Pool` por un objeto
falso con `useValue`:

```ts
import { NotFoundException } from '@nestjs/common';
import { Test } from '@nestjs/testing';
import { Pool } from 'pg';
import { BooksService } from './books.service';

describe('BooksService', () => {
  let service: BooksService;
  const poolMock = { query: jest.fn() };   // un Pool falso

  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
      providers: [
        BooksService,
        { provide: Pool, useValue: poolMock }, // ← mismo TOKEN, otro valor
      ],
    }).compile();

    service = moduleRef.get(BooksService);
    poolMock.query.mockReset();
  });

  it('findOne devuelve el libro en camelCase', async () => {
    const fecha = new Date();
    poolMock.query.mockResolvedValue({
      rows: [{ id: 1, title: 'X', author: 'Y', pages: 10, created_at: fecha }],
    });

    await expect(service.findOne(1)).resolves.toEqual({
      id: 1, title: 'X', author: 'Y', pages: 10, createdAt: fecha,
    });
    expect(poolMock.query).toHaveBeenCalledWith(expect.any(String), [1]);
  });

  it('findOne lanza 404 si no hay filas', async () => {
    poolMock.query.mockResolvedValue({ rows: [] });
    await expect(service.findOne(999)).rejects.toThrow(NotFoundException);
  });
});
```

> 📌 Aquí se ve **la gran ventaja de la inyección de dependencias**: como `BooksService`
> *pide* el `Pool` en vez de crearlo, en el test le das uno falso y listo.

Y para testear el controlador, se mockea el servicio:

```ts
const booksServiceMock = { findOne: jest.fn().mockResolvedValue({ id: 1 }) };

await Test.createTestingModule({
  controllers: [BooksController],
  providers: [{ provide: BooksService, useValue: booksServiceMock }],
}).compile();
```

---

## 7.8 Test e2e con `supertest`

### Ejemplo REAL copiado (`api-books-nest/test/app.e2e-spec.ts`)

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

### Ejemplo REAL copiado (`api-books-nest/test/jest-e2e.json`)

```json
{
  "moduleFileExtensions": ["js", "json", "ts"],
  "rootDir": ".",
  "testEnvironment": "node",
  "testRegex": ".e2e-spec.ts$",
  "transform": {
    "^.+\\.(t|j)s$": "ts-jest"
  }
}
```

**Diferencias con el unitario:**
- Importa **todo** `AppModule` (todos los módulos reales).
- `createNestApplication()` + `app.init()` → una app de verdad, sin `listen`.
- `request(app.getHttpServer()).get('/')` → hace una **petición HTTP real**.

### ⚠️ Tres cosas a tener en cuenta en este test

1. **Espera `'Hello World!'`**, pero `AppService.getHello()` ahora devuelve
   `'Hola desde NestJS'` → este test **fallará** hasta que lo actualices
   (el unitario sí se actualizó; el e2e no).
2. **No ejecuta `main.ts`**: no hay prefijo `/api` ni `ValidationPipe`. Por eso la ruta
   es `'/'` y no `'/api'`. Si quieres probar validaciones, configúralo también en el
   test:

   ```ts
   app = moduleFixture.createNestApplication();
   app.setGlobalPrefix('api');
   app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true }));
   await app.init();
   ```
3. **Tampoco carga el `.env`** (`process.loadEnvFile()` está en `main.ts`): las rutas
   de libros necesitarían `DATABASE_URL` y una BD real en marcha.

### Más ejemplos e2e (ejemplo propio)

```ts
it('POST /api/books con datos inválidos → 400', () => {
  return request(app.getHttpServer())
    .post('/api/books')
    .send({ title: '', author: 'X', pages: 0 })
    .expect(400);
});

it('GET /api/books/abc → 400 (ParseIntPipe)', () => {
  return request(app.getHttpServer()).get('/api/books/abc').expect(400);
});
```

---

## ✅ Resumen rápido

1. Orden: middleware → guards → interceptors → pipes → handler → interceptors → filters.
2. **Middleware** = el de Express; se registra con `configure(consumer)`.
3. **Guard** (`CanActivate`) decide si se pasa → autenticación (401) y roles (403).
4. **Interceptor** envuelve el handler (logs, tiempos, formato de respuesta) con RxJS.
5. Tests unitarios: `Test.createTestingModule({...}).compile()` + `get(Clase)`.
6. Mocks con `{ provide: Token, useValue: mock }` → gracias a la DI.
7. Tests e2e: `createNestApplication()` + `supertest`; ¡no ejecutan `main.ts`!

👉 **Siguiente:** `08-ejemplos-completos-de-proyectos.md`
