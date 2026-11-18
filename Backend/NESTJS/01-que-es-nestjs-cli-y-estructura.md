# 01 — QUÉ ES NESTJS, EL CLI Y LA ESTRUCTURA DEL PROYECTO

> **NestJS** es un **framework de Node.js** para construir **APIs y aplicaciones de
> servidor** con **TypeScript**. Te da una **arquitectura ya decidida** (módulos,
> controladores, servicios) para que todos los proyectos se organicen igual.

---

## 1.1 ¿Qué es NestJS?

Piensa en las capas:

```
Node.js        → ejecuta JavaScript fuera del navegador
  └─ Express   → librería mínima para crear un servidor HTTP (rutas, req, res)
      └─ NestJS → framework que ORGANIZA tu código encima de Express
```

**Puntos clave:**
- Está escrito en **TypeScript** y está pensado para usarse con él.
- Usa **clases** y **decoradores** (`@Controller`, `@Get`, `@Injectable`...).
- Trae **inyección de dependencias**: tú pides lo que necesitas en el constructor y
  Nest te lo da.
- Por debajo usa **Express** (o Fastify, si lo cambias). En tu proyecto se ve en
  la dependencia `@nestjs/platform-express`.

### Ejemplo REAL copiado (`api-books-nest/package.json`, dependencias)

```json
"dependencies": {
  "@nestjs/common": "^11.0.1",
  "@nestjs/core": "^11.0.1",
  "@nestjs/platform-express": "^11.0.1",
  "argon2": "^0.45.2",
  "class-transformer": "^0.5.1",
  "class-validator": "^0.15.1",
  "pg": "^8.23.1",
  "reflect-metadata": "^0.2.2",
  "rxjs": "^7.8.1"
}
```

| Paquete | Para qué sirve |
|---------|----------------|
| `@nestjs/common` | Decoradores y utilidades: `@Controller`, `@Get`, `@Injectable`, `ValidationPipe`, excepciones... |
| `@nestjs/core` | El "motor": `NestFactory`, el contenedor de inyección |
| `@nestjs/platform-express` | El adaptador que hace que Nest funcione **sobre Express** |
| `class-validator` | Decoradores de validación para los DTOs (`@IsString`, `@Min`...) |
| `class-transformer` | Convierte el JSON recibido en instancias de clase (`@Transform`) |
| `pg` | Cliente de PostgreSQL (`Pool`) |
| `argon2` | Para hashear contraseñas (preparado para la autenticación) |
| `reflect-metadata` | Permite que los decoradores guarden información (metadatos) |
| `rxjs` | Programación reactiva; Nest lo usa internamente (interceptors) |

---

## 1.2 ¿Por qué NestJS y no solo Express?

| | Express | NestJS |
|--|---------|--------|
| Estructura | Libre: cada uno la monta a su manera | **Opinada**: módulos, controladores, servicios |
| Lenguaje | JS (TS opcional) | **TypeScript** de serie |
| Validación | La haces a mano o con librerías sueltas | DTOs + `ValidationPipe` integrados |
| Errores | `res.status(404).json(...)` a mano | `throw new NotFoundException()` |
| Dependencias | `require`/`import` manual | **Inyección de dependencias** automática |
| Tests | Configuración propia | `@nestjs/testing` ya preparado |
| Curva de aprendizaje | Baja | Media (más conceptos) |

> 📌 **"Opinado" (opinionated)** significa que el framework **ya ha decidido** cómo se
> organizan las cosas. Pierdes algo de libertad, pero ganas orden: si abres un proyecto
> Nest de otra persona, sabrás dónde está todo.

```js
// ❌ Express "a lo loco": todo en un archivo, SQL mezclado con la ruta
app.post('/books', async (req, res) => {
  if (typeof req.body.title !== 'string') return res.status(400).send('title mal');
  const r = await pool.query('INSERT INTO books ...');
  res.json(r.rows[0]);
});
```

```ts
// ✅ NestJS: cada responsabilidad en su capa
// controlador → recibe la petición
@Post()
create(@Body() dto: CreateBookDto) {   // el DTO ya está validado
  return this.booksService.create(dto); // el servicio habla con la BD
}
```

---

## 1.3 El Nest CLI

El **CLI** (Command Line Interface) es la herramienta de terminal de Nest. En tu proyecto
está como dependencia de desarrollo: `"@nestjs/cli": "^11.0.0"`.

```bash
# Instalar el CLI globalmente (opcional)
npm i -g @nestjs/cli

# Crear un proyecto nuevo (así se creó api-books-nest)
nest new api-books-nest

# Generar piezas (crea el archivo Y lo registra en el módulo)
nest generate module books        # o: nest g mo books
nest generate controller books    # o: nest g co books
nest generate service books       # o: nest g s books
nest g resource books             # módulo + controlador + servicio + DTOs de golpe
```

| Comando corto | Genera |
|---------------|--------|
| `nest g mo nombre` | `nombre.module.ts` |
| `nest g co nombre` | `nombre.controller.ts` + `.spec.ts` |
| `nest g s nombre` | `nombre.service.ts` + `.spec.ts` |
| `nest g pi nombre` | un pipe |
| `nest g gu nombre` | un guard |
| `nest g in nombre` | un interceptor |
| `nest g res nombre` | un recurso CRUD completo |

> 📌 Añade `--no-spec` si no quieres que te cree el archivo de test.

### Ejemplo REAL copiado (`api-books-nest/nest-cli.json`)

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

- `collection`: de dónde saca las plantillas (`@nestjs/schematics`) para `nest g`.
- `sourceRoot`: tu código vive en `src/`.
- `deleteOutDir`: antes de compilar **borra `dist/`** para no dejar archivos viejos.

---

## 1.4 Scripts de `package.json`

### Ejemplo REAL copiado (`api-books-nest/package.json`)

```json
"scripts": {
  "build": "nest build",
  "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
  "start": "nest start",
  "start:dev": "nest start --watch",
  "start:debug": "nest start --debug --watch",
  "start:prod": "node dist/main",
  "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
  "test": "jest",
  "test:watch": "jest --watch",
  "test:cov": "jest --coverage",
  "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
  "test:e2e": "jest --config ./test/jest-e2e.json"
}
```

| Script | Qué hace | Cuándo usarlo |
|--------|----------|---------------|
| `npm run start:dev` | Compila y **reinicia al guardar** | Mientras programas (el 90% del tiempo) |
| `npm run build` | Compila TS → JS en `dist/` | Antes de desplegar |
| `npm run start:prod` | Ejecuta `dist/main.js` con Node | En producción |
| `npm run test` | Tests unitarios (`*.spec.ts` en `src/`) | Para comprobar la lógica |
| `npm run test:e2e` | Tests de extremo a extremo (`test/`) | Para probar la API entera |
| `npm run lint` / `format` | ESLint / Prettier | Código limpio y uniforme |

> 📌 `start:prod` ejecuta `node dist/main` → **dist** es el JavaScript compilado. Nunca
> edites nada dentro de `dist/`: se borra y regenera en cada build.

---

## 1.5 Estructura de carpetas

### Estructura REAL de `api-books-nest/`

```
api-books-nest/
├── database/
│   └── schema.sql              ← tablas y datos de ejemplo (PostgreSQL)
├── src/
│   ├── main.ts                 ← PUNTO DE ENTRADA: arranca la app
│   ├── app.module.ts           ← módulo RAÍZ: une todos los módulos
│   ├── app.controller.ts       ← rutas GET / y GET /ping
│   ├── app.controller.spec.ts  ← test unitario del AppController
│   ├── app.service.ts          ← servicio con getHello()
│   ├── books/                  ← MÓDULO de libros (una "feature")
│   │   ├── book.ts             ← interfaces Book / BookRow + toBook()
│   │   ├── books.contract.ts   ← interface BooksContract (qué debe saber hacer)
│   │   ├── books.controller.ts ← rutas /api/books
│   │   ├── books.module.ts
│   │   ├── books.service.ts    ← lógica + SQL
│   │   └── dto/
│   │       ├── create-book.dto.ts
│   │       └── update-book.dto.ts
│   ├── database/
│   │   ├── database.module.ts  ← provee y exporta el Pool de pg
│   │   └── database.service.ts
│   └── users/
│       ├── user.ts
│       ├── users.module.ts
│       └── users.service.ts
├── test/
│   ├── app.e2e-spec.ts         ← test e2e
│   └── jest-e2e.json
├── .env.example                ← plantilla de variables de entorno
├── compose.yaml                ← PostgreSQL en Docker
├── nest-cli.json
├── package.json
├── tsconfig.json
└── tsconfig.build.json
```

**Convención de nombres:** `nombre.tipo.ts`

| Sufijo | Qué contiene |
|--------|--------------|
| `.module.ts` | Una clase con `@Module` |
| `.controller.ts` | Una clase con `@Controller` (rutas) |
| `.service.ts` | Una clase con `@Injectable` (lógica) |
| `.dto.ts` | Una clase DTO (forma de los datos que llegan) |
| `.spec.ts` | Test unitario |
| `.e2e-spec.ts` | Test end-to-end |

> 📌 **Organiza por funcionalidad, no por tipo.** Todo lo de libros va en `books/`,
> todo lo de usuarios en `users/`. ❌ No hagas una carpeta `controllers/` con todos los
> controladores mezclados.

---

## 1.6 TypeScript y los decoradores

Los decoradores (`@Algo`) son funciones que **añaden información** a una clase, método,
propiedad o parámetro. Nest los lee para saber qué es cada cosa.

```ts
@Controller('books')        // decorador de CLASE
export class BooksController {
  @Get(':id')               // decorador de MÉTODO
  findOne(@Param('id') id: string) {}   // decorador de PARÁMETRO
}

export class CreateBookDto {
  @IsString()               // decorador de PROPIEDAD
  title: string;
}
```

Para que funcionen, `tsconfig.json` necesita dos opciones.

### Ejemplo REAL copiado (`api-books-nest/tsconfig.json`, fragmento)

```json
{
  "compilerOptions": {
    "module": "nodenext",
    "moduleResolution": "nodenext",
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "target": "ES2023",
    "outDir": "./dist",
    "strictNullChecks": true,
    "noImplicitAny": false
  }
}
```

- `experimentalDecorators: true` → permite usar `@Decoradores`.
- `emitDecoratorMetadata: true` → guarda los **tipos** del constructor. Gracias a esto
  Nest sabe que `constructor(private readonly pool: Pool)` necesita un `Pool`.
- `outDir: ./dist` → dónde se deja el JavaScript compilado.
- `strictNullChecks: true` → TS te obliga a tener en cuenta `null`/`undefined`.

> ⚠️ Si quitas `emitDecoratorMetadata`, la inyección de dependencias **deja de funcionar**
> (Nest no sabría qué inyectar).

---

## 1.7 Flujo de una petición en Nest (visión general)

```
Cliente  ──HTTP──▶  Middleware  ──▶  Guard  ──▶  Interceptor (antes)
                                                    │
                                                    ▼
                                                  Pipe (valida/transforma)
                                                    │
                                                    ▼
                                             Controlador (@Get, @Post...)
                                                    │
                                                    ▼
                                             Servicio (lógica, BD)
                                                    │
Cliente  ◀──JSON──  Exception filter  ◀──  Interceptor (después)
```

| Pieza | Responsabilidad | ¿En tu proyecto? |
|-------|-----------------|------------------|
| Módulo | Agrupa piezas relacionadas | ✅ `AppModule`, `BooksModule`, `DatabaseModule`, `UsersModule` |
| Controlador | Recibe la petición y devuelve la respuesta | ✅ `AppController`, `BooksController` |
| Servicio | Lógica de negocio y acceso a datos | ✅ `BooksService`, `UsersService`... |
| Pipe | Valida y transforma la entrada | ✅ `ValidationPipe`, `ParseIntPipe` |
| Excepción | Corta y devuelve un error HTTP | ✅ `NotFoundException`, `BadRequestException` |
| Guard | Decide si se puede acceder | ⏳ Aún no (lo verás en `07`) |
| Interceptor | Antes/después del handler | ⏳ Aún no (lo verás en `07`) |
| Middleware | Antes de todo, estilo Express | ⏳ Aún no (lo verás en `07`) |

---

## 1.8 Errores comunes al empezar

| Error | Solución |
|-------|----------|
| `nest: command not found` | Usa `npx nest ...` o instala el CLI global |
| Cambios que no se ven | Estás con `npm run start` → usa `start:dev` (watch) |
| `Nest can't resolve dependencies of X` | Falta registrar el provider o importar su módulo (ver `04`) |
| Editas `dist/` y se pierde | `dist/` se regenera: edita siempre `src/` |
| Decoradores en rojo | Revisa `experimentalDecorators` en `tsconfig.json` |

---

## ✅ Resumen rápido

1. NestJS = framework **TypeScript** que **organiza** un servidor Express.
2. Arquitectura opinada: **módulos → controladores → servicios**.
3. El **CLI** genera piezas: `nest g mo|co|s|res nombre`.
4. `npm run start:dev` para desarrollar; `build` + `start:prod` para producción.
5. Carpetas **por funcionalidad** (`books/`, `users/`) y nombres `nombre.tipo.ts`.
6. Los **decoradores** necesitan `experimentalDecorators` y `emitDecoratorMetadata`.

👉 **Siguiente:** `02-bootstrap-y-modulos.md`
