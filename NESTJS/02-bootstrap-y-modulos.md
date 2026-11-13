# 02 — ARRANQUE (`main.ts`) Y MÓDULOS

> Toda app Nest empieza en **`main.ts`**, que crea la aplicación a partir de un
> **módulo raíz** (`AppModule`). Un **módulo** es una caja que agrupa controladores y
> servicios de una misma funcionalidad.

---

## 2.1 `main.ts` — el punto de entrada

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule); // 1. crea la app
  await app.listen(3000);                          // 2. escucha en un puerto
}
bootstrap();
```

**Puntos clave:**
- `bootstrap` ("arrancar") es solo un nombre de convención para la función de inicio.
- Es `async` porque crear la app y escuchar son operaciones asíncronas.
- `NestFactory.create(AppModule)` lee el módulo raíz, y a partir de él **todos** los
  módulos importados, crea los servicios y registra las rutas.
- `app` es una `INestApplication`: con ella configuras cosas **globales**.

### Ejemplo REAL copiado (`api-books-nest/src/main.ts`)

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

**Desglose línea a línea:**

```
process.loadEnvFile()            → función NATIVA de Node (v20.12+/v21.7+): lee .env
                                   y rellena process.env. Sin instalar dotenv.
NestFactory.create(AppModule)    → construye la app a partir del módulo raíz
app.setGlobalPrefix('api')       → /books pasa a ser /api/books
app.useGlobalPipes(...)          → el ValidationPipe se aplica a TODAS las rutas
process.env.PORT ?? 3000         → usa PORT si existe; si no, 3000
```

> 📌 `process.loadEnvFile()` se llama **antes** de `bootstrap()` a propósito: los
> módulos (como `DatabaseModule`) leen `process.env.DATABASE_URL` al crearse, así que
> el `.env` tiene que estar cargado antes.

---

## 2.2 Prefijo global

```ts
app.setGlobalPrefix('api');
```

| Controlador | Decorador del método | Ruta final |
|-------------|----------------------|------------|
| `@Controller()` | `@Get()` | `GET /api` |
| `@Controller()` | `@Get('ping')` | `GET /api/ping` |
| `@Controller('books')` | `@Get()` | `GET /api/books` |
| `@Controller('books')` | `@Get(':id')` | `GET /api/books/1` |

> ⚠️ El prefijo se configura en `main.ts`. Los **tests e2e** no ejecutan `main.ts`, así
> que allí las rutas **no llevan** `/api` (lo verás en `07`).

---

## 2.3 Otras opciones útiles de `main.ts` (no están en tu proyecto)

```ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.enableCors({ origin: 'http://localhost:5173' }); // permitir un frontend (React/Vite)
  app.setGlobalPrefix('api');
  app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
  // app.useGlobalFilters(new MiFiltro());       → filtro de excepciones global
  // app.useGlobalInterceptors(new MiInterceptor()); → interceptor global

  const port = Number(process.env.PORT ?? 3000);
  await app.listen(port);
  console.log(`🚀 API en http://localhost:${port}/api`);
}
```

> 📌 `enableCors` es imprescindible cuando un frontend en **otro puerto** llama a tu
> API. Sin él, el navegador bloquea la petición.

---

## 2.4 ¿Qué es un módulo?

Un módulo es una **clase vacía** decorada con `@Module({...})`. Lo importante es el
**objeto** que le pasas:

```ts
@Module({
  imports: [],      // otros MÓDULOS cuyos providers exportados quiero usar
  controllers: [],  // controladores (rutas) de este módulo
  providers: [],    // servicios y demás "inyectables" de este módulo
  exports: [],      // providers que dejo usar a QUIEN me importe
})
export class MiModulo {}
```

| Propiedad | Pregunta que responde |
|-----------|----------------------|
| `imports` | ¿Qué necesito **de fuera**? |
| `controllers` | ¿Qué **rutas** tengo? |
| `providers` | ¿Qué **servicios** creo yo? |
| `exports` | ¿Qué **comparto** con los demás? |

> 📌 Por defecto, los providers de un módulo son **privados**. Si otro módulo quiere
> usarlos, tienen que estar en `exports` **y** ese otro módulo tiene que **importar** el
> módulo.

---

## 2.5 El módulo raíz

### Ejemplo REAL copiado (`api-books-nest/src/app.module.ts`)

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

**Árbol de módulos de tu proyecto:**

```
AppModule  (raíz: AppController + AppService)
├── DatabaseModule   → provee y EXPORTA Pool
├── BooksModule      → importa DatabaseModule; BooksController + BooksService
└── UsersModule      → importa DatabaseModule; UsersService (EXPORTADO)
```

> 📌 Si un módulo **no** está en el árbol (no lo importa nadie a partir de `AppModule`),
> Nest **no lo carga**: sus rutas no existen. Es el fallo nº1 al crear un módulo a mano.

```ts
// ❌ Creas BooksModule pero no lo importas → GET /api/books da 404
@Module({ imports: [DatabaseModule] })
export class AppModule {}

// ✅
@Module({ imports: [DatabaseModule, BooksModule] })
export class AppModule {}
```

> 💡 Si generas con `nest g mo books`, el CLI **lo añade solo** a `AppModule`.

---

## 2.6 Módulo de funcionalidad (feature module)

### Ejemplo REAL copiado (`api-books-nest/src/books/books.module.ts`)

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

- `imports: [DatabaseModule]` → como `DatabaseModule` **exporta** `Pool`, el
  `BooksService` puede pedirlo en su constructor.
- `controllers: [BooksController]` → registra las rutas `/api/books...`.
- `providers: [BooksService]` → Nest crea **una** instancia de `BooksService` y la
  inyecta donde se pida (en `BooksController`).
- No tiene `exports`: nadie de fuera necesita `BooksService`.

---

## 2.7 Módulo que exporta (compartir servicios)

### Ejemplo REAL copiado (`api-books-nest/src/users/users.module.ts`)

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

- Este módulo **no tiene controladores**: no expone rutas, solo un servicio.
- `exports: [UsersService]` prepara el terreno para un futuro `AuthModule`:

```ts
// Ejemplo propio (no está en tu proyecto): cómo lo usaría AuthModule
@Module({
  imports: [UsersModule],          // ← importa el módulo que exporta UsersService
  controllers: [AuthController],
  providers: [AuthService],        // AuthService puede pedir UsersService
})
export class AuthModule {}
```

---

## 2.8 Módulo con un provider personalizado

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

Aquí `providers` no recibe una clase propia sino un **objeto** `{ provide, useFactory }`.
Esto se explica a fondo en `04` (tokens y factorías). La idea:

```
provide: Pool        → la "etiqueta" (token) con la que otros lo piden
useFactory: () => …  → cómo se fabrica (una sola vez: es un singleton)
exports: [Pool]      → los módulos que importen DatabaseModule pueden pedir Pool
```

> 📌 Como `BooksModule` y `UsersModule` importan el **mismo** `DatabaseModule`, ambos
> reciben **el mismo `Pool`** (una sola conexión compartida). Los módulos en Nest son
> **singletons**.

---

## 2.9 Módulos globales (no está en tu proyecto)

Si un módulo lo necesita *todo el mundo* (como la base de datos), puedes marcarlo con
`@Global()` y así no tienes que importarlo en cada módulo:

```ts
import { Global, Module } from '@nestjs/common';

@Global()                     // ← disponible en TODA la app
@Module({
  providers: [{ provide: Pool, useFactory: () => new Pool({ /* ... */ }) }],
  exports: [Pool],
})
export class DatabaseModule {}

// Basta con importarlo UNA vez en AppModule.
// BooksModule ya no necesita imports: [DatabaseModule]
```

> ⚠️ Úsalo con moderación. Importar explícitamente (como hace tu proyecto) deja más
> claro **de dónde viene** cada dependencia.

---

## 2.10 Errores comunes con módulos

| Error | Causa | Solución |
|-------|-------|----------|
| Ruta da 404 | El módulo no está importado en `AppModule` | Añádelo a `imports` |
| `Nest can't resolve dependencies of the BooksService (?)` | El `Pool` no está disponible en `BooksModule` | Importa `DatabaseModule` y comprueba que **exporta** `Pool` |
| Pones un **módulo** en `providers` | Confusión de propiedades | Los módulos van en `imports`; los servicios en `providers` |
| Pones un **servicio** en `imports` | Igual | Al revés |
| Exportas algo que no es provider tuyo | `exports` solo acepta tus providers o módulos importados | Regístralo primero en `providers` |

---

## ✅ Resumen rápido

1. `main.ts` → `NestFactory.create(AppModule)` + configuración global + `listen`.
2. `setGlobalPrefix('api')` → todas las rutas cuelgan de `/api`.
3. `process.loadEnvFile()` carga el `.env` **antes** de crear la app.
4. `@Module({ imports, controllers, providers, exports })`.
5. Providers **privados** por defecto → compártelos con `exports` + `imports`.
6. Todo módulo tiene que estar en el árbol que sale de `AppModule`.

👉 **Siguiente:** `03-controladores-y-decoradores.md`
