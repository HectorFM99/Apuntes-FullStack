# 🐈 CARPETA NESTJS — ÍNDICE DE ESTUDIO

Teoría de **NestJS** explicada con ejemplos reales **copiados y pegados de tus
proyectos** del Bootcamp.

> ⚠️ Los proyectos **NO** se han modificado. Solo se han copiado fragmentos de su código.
> 🔒 Los secretos (contraseñas, `.env`) **no** se copian: se usan marcadores como `CAMBIAME`.

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a aprender |
|---|---------|--------------------|
| 1 | `01-que-es-nestjs-cli-y-estructura.md` | Qué es NestJS, por qué usarlo (vs Express), Nest CLI, estructura de carpetas, `package.json`, `tsconfig` |
| 2 | `02-bootstrap-y-modulos.md` | `main.ts`, `NestFactory`, prefijo global, `@Module` (`imports`, `controllers`, `providers`, `exports`) |
| 3 | `03-controladores-y-decoradores.md` | `@Controller`, `@Get`, `@Post`, `@Patch`, `@Delete`, `@Param`, `@Query`, `@Body`, `@HttpCode` |
| 4 | `04-providers-servicios-e-inyeccion.md` | `@Injectable`, inyección de dependencias, contratos (`interface`), tokens, `useFactory`, ciclo de vida |
| 5 | `05-dtos-validacion-pipes-y-excepciones.md` | DTOs, `class-validator`, `class-transformer`, `ValidationPipe`, `ParseIntPipe`, `NotFoundException`, filtros |
| 6 | `06-base-de-datos-y-configuracion.md` | PostgreSQL con `pg`, `Pool`, consultas parametrizadas, `schema.sql`, Docker Compose, `.env` |
| 7 | `07-guards-interceptors-middleware-y-testing.md` | Guards, interceptors, middleware (concepto), tests unitarios y e2e con Jest |
| 8 | `08-ejemplos-completos-de-proyectos.md` | Archivos enteros copiados de tu proyecto, comentados |

---

## 🗂️ Proyecto del que se han extraído los ejemplos

| Proyecto | Archivo(s) | Temas que cubre |
|----------|-----------|-----------------|
| `api-books-nest/` | `src/main.ts` | Arranque, prefijo `/api`, `ValidationPipe` global, `process.loadEnvFile()` |
| `api-books-nest/` | `src/app.module.ts`, `app.controller.ts`, `app.service.ts` | Módulo raíz, controlador y servicio básicos |
| `api-books-nest/` | `src/books/*` | CRUD de libros: controlador, servicio, contrato, DTOs, mapeo de filas |
| `api-books-nest/` | `src/database/*` | Proveedor del `Pool` de `pg` con `useFactory`, `OnModuleDestroy` |
| `api-books-nest/` | `src/users/*` | Servicio de usuarios exportado para un futuro módulo de autenticación |
| `api-books-nest/` | `src/app.controller.spec.ts`, `test/app.e2e-spec.ts` | Tests unitarios y e2e con `@nestjs/testing` y `supertest` |
| `api-books-nest/` | `database/schema.sql`, `compose.yaml`, `.env.example` | Tablas, PostgreSQL en Docker, variables de entorno |
| `api-books-nest/` | `package.json`, `nest-cli.json`, `tsconfig.json` | Scripts, dependencias, configuración del CLI y de TypeScript |

---

## 🚀 Orden de estudio recomendado

1. `01` → entender **qué es** Nest y cómo está organizado un proyecto.
2. `02` → cómo **arranca** la app y qué es un **módulo** (la pieza básica).
3. `03` → **controladores**: las rutas HTTP.
4. `04` → **servicios** e **inyección de dependencias** (el corazón de Nest).
5. `05` → **DTOs, validación y errores**: lo que hace robusta a tu API.
6. `06` → conectar con **PostgreSQL** y configurar el entorno.
7. `07` → piezas avanzadas (guards, interceptors) y **tests**.
8. `08` → código real completo de tu proyecto.

---

## 💡 Diferencia clave con Express

```js
// EXPRESS: tú decides cómo organizar todo (libertad total, pero cada proyecto es distinto)
app.get('/api/books/:id', async (req, res) => {
  const id = Number(req.params.id);
  const book = await buscarLibro(id);
  if (!book) return res.status(404).json({ message: 'Libro no encontrado' });
  res.json(book);
});
```

```ts
// NESTJS: clases + decoradores, y cada cosa en su sitio
@Controller('books')
export class BooksController {
  constructor(private readonly booksService: BooksService) {}

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.booksService.findOne(id); // el 404 lo lanza el servicio
  }
}
```

> 📌 NestJS **usa Express por debajo** (`@nestjs/platform-express`). No lo sustituye:
> lo **organiza** con una arquitectura fija (módulos → controladores → servicios).
