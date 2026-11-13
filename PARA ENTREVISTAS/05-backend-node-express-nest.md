# 05 — 🖥️ BACKEND: NODE, EXPRESS Y NESTJS

> Aquí es donde más puedes **lucirte con tus proyectos**: has hecho APIs con Express +
> MongoDB (`api-rastro`, `mongo-api-libros`), con NestJS + PostgreSQL
> (`api-books-nest`) y una app completa con login (`ProyectoReproductor`).

---

## 5.1 ¿Qué es Node.js?

| Pregunta | Respuesta |
|----------|-----------|
| ¿Qué es? | Un **entorno de ejecución** de JavaScript fuera del navegador, basado en el motor **V8** de Chrome |
| ¿Es un lenguaje? ¿Un framework? | ❌ Ninguno de los dos. Es un *runtime* |
| ¿Qué tiene de especial? | **I/O no bloqueante** y orientado a eventos |
| ¿Para qué es bueno? | APIs, tiempo real, muchas conexiones con poca CPU |
| ¿Para qué es peor? | Cálculo intensivo de CPU (bloquea el hilo principal) |
| ¿Qué es npm? | El gestor de paquetes. `package.json` lista dependencias y scripts |

### Event loop de Node ⭐

Node ejecuta tu JS en **un solo hilo**, pero las operaciones lentas (BD, ficheros, red)
las delega a **libuv** (que usa el sistema operativo y un pool de hilos). Cuando
terminan, su callback vuelve a la cola y el event loop lo ejecuta.

```js
// ❌ Bloquea: mientras lee, el servidor no atiende a NADIE
const data = fs.readFileSync("enorme.txt");

// ✅ No bloquea: Node sigue atendiendo peticiones mientras lee
const data = await fs.promises.readFile("enorme.txt");
```

> 📌 Fases del event loop de Node (por si preguntan más a fondo): **timers** →
> pending callbacks → **poll** (I/O) → **check** (`setImmediate`) → close callbacks.
> Entre fase y fase se vacían las microtareas (`process.nextTick` y promesas).

| | `dependencies` | `devDependencies` |
|---|---|---|
| Qué | Lo que la app necesita **en producción** | Solo para **desarrollar** |
| Ejemplo (`api-rastro`) | `express`, `mongodb` | `typescript`, `eslint`, `tsx`, `@types/*` |

> 📌 `npm ci` instala **exactamente** lo del `package-lock.json` (ideal en CI).

---

## 5.2 Express y middleware ⭐

**Express** es un framework minimalista para crear servidores HTTP en Node.

Un **middleware** es una función `(req, res, next)` que se ejecuta **entre** que llega
la petición y se envía la respuesta. Puede:
- Modificar `req`/`res` (p. ej. parsear JSON, añadir el usuario).
- **Cortar** la cadena respondiendo (p. ej. `401`).
- Pasar al siguiente con `next()`.

```
Petición → [express.json()] → [cors] → [auth] → [ruta] → Respuesta
                                           │
                                           └─ si no hay sesión → 401 y fin
```

```ts
import express from "express";
const app = express();

app.use(express.json());                 // middleware global: parsea el body

function auth(req, res, next) {          // middleware propio
  if (!req.headers.authorization) {
    return res.status(401).json({ error: "No autenticado" });
  }
  next();                                // ✅ sigue a la ruta
}

app.get("/api/listings", auth, async (req, res) => {   // middleware en una ruta
  res.json(await listings.find().toArray());
});

// Middleware de errores: 4 parámetros, va al FINAL
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: "Error interno" });
});
```

**Ejemplo real (`ProyectoReproductor/server/index.ts`, simplificado):** un middleware
`session` que busca el usuario por la cookie; si no existe responde `401`, y si existe
lo guarda en `res.locals.user` y llama a `next()`. Luego se usa así:

```ts
app.get('/api/tracks/search', session, async (req, res) => { ... });
```

> ⚠️ Si un middleware no llama a `next()` **ni** responde, la petición se queda
> **colgada** para siempre.

---

## 5.3 REST ⭐

**REST** es un estilo de arquitectura para APIs basado en **recursos** (sustantivos)
manipulados con **métodos HTTP** (verbos).

| Acción | Método + ruta | Status típico |
|--------|---------------|---------------|
| Listar | `GET /api/books` | `200` |
| Ver uno | `GET /api/books/7` | `200` / `404` |
| Crear | `POST /api/books` | `201` |
| Actualizar parcial | `PATCH /api/books/7` | `200` |
| Reemplazar | `PUT /api/books/7` | `200` |
| Borrar | `DELETE /api/books/7` | `204` |
| Filtrar | `GET /api/books?author=gabriel&maxPages=150` | `200` |
| Recurso anidado | `GET /api/listings/3/offers` | `200` |

```
❌ GET  /api/getAllBooks         ❌ POST /api/deleteBook/7
✅ GET  /api/books               ✅ DELETE /api/books/7
```

### Principios de REST

| Principio | Significa |
|-----------|-----------|
| **Cliente-servidor** | Frontend y backend separados |
| **Stateless** | Cada petición lleva todo lo necesario (el servidor no "recuerda" entre peticiones) |
| **Interfaz uniforme** | Recursos con URLs y métodos HTTP estándar |
| **Cacheable** | Las respuestas indican si se pueden cachear |
| **Sistema en capas** | Puede haber proxies, balanceadores… en medio sin que el cliente lo note |

> 📌 Otras alternativas que pueden nombrarte: **GraphQL** (el cliente pide exactamente
> los campos que quiere, un solo endpoint) y **WebSockets** (tiempo real).

---

## 5.4 Arquitectura en capas ⭐

Separar el código por **responsabilidades**:

```
Petición HTTP
    │
┌───▼──────────┐  Controller / Router: recibe la petición, lee params/body,
│  Controller  │  llama al servicio y devuelve la respuesta. NADA de lógica de negocio.
└───┬──────────┘
┌───▼──────────┐  Service: la lógica de negocio (reglas, cálculos, validaciones
│   Service    │  de dominio, lanzar NotFound...).
└───┬──────────┘
┌───▼──────────┐  Repository / acceso a datos: las consultas SQL o Mongo.
│  Repository  │
└───┬──────────┘
    ▼
 Base de datos
```

**Ejemplo real (`api-books-nest`):**

```ts
// books.controller.ts → solo HTTP
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  return this.booksService.findOne(id);
}

// books.service.ts → lógica + consulta
async findOne(id: number): Promise<Book> {
  const result = await this.pool.query<BookRow>(
    'SELECT id, title, author, pages, created_at FROM books WHERE id = $1', [id],
  );
  if (result.rows.length === 0) throw new NotFoundException('Libro no encontrado');
  return toBook(result.rows[0]);
}
```

| Ventaja | Por qué |
|---------|---------|
| 🧪 Testeable | Puedes probar el servicio sin levantar el servidor HTTP |
| 🔁 Reutilizable | El mismo servicio lo usan varias rutas |
| 🔧 Mantenible | Cambiar de BD solo toca la capa de datos |

### MVC

**Model – View – Controller**: patrón clásico.

| Pieza | Qué es | En una API REST |
|-------|--------|-----------------|
| **Model** | Los datos y sus reglas | Entidades / esquemas / acceso a BD |
| **View** | Lo que ve el usuario | El **JSON** de respuesta (o el frontend) |
| **Controller** | Recibe la entrada y coordina | Los controllers / routers |

---

## 5.5 NestJS

Framework de Node **con TypeScript**, inspirado en Angular, que impone una
**estructura** (módulos, controllers, providers) y usa **decoradores** e
**inyección de dependencias**. Por debajo usa Express (o Fastify).

| Pieza Nest | Qué es | Ejemplo en `api-books-nest` |
|------------|--------|-----------------------------|
| **Module** `@Module` | Agrupa una funcionalidad | `BooksModule`, `DatabaseModule`, `UsersModule` |
| **Controller** `@Controller` | Rutas | `@Controller('books')`, `@Get()`, `@Post()`, `@Patch(':id')` |
| **Provider / Service** `@Injectable` | Lógica, inyectable | `BooksService` |
| **DTO** | Clase que define y valida lo que llega | `CreateBookDto`, `UpdateBookDto` |
| **Pipe** | Transforma/valida entradas | `ParseIntPipe`, `ValidationPipe` |
| **Guard** | Decide si la petición puede pasar (auth, roles) | — |
| **Interceptor** | Antes/después del handler (logs, transformar respuesta) | — |
| **Exception filter** | Convierte errores en respuestas HTTP | Nest ya trae uno: `NotFoundException` → `404` |

### Express vs NestJS

| | Express | NestJS |
|---|---|---|
| Estructura | Libre (la decides tú) | Impuesta (módulos, capas) |
| TypeScript | Opcional | Nativo |
| DI | No trae | ✅ Integrada |
| Curva de aprendizaje | Baja | Media |
| Ideal para | APIs pequeñas, prototipos | Proyectos grandes y equipos |

---

## 5.6 Inyección de dependencias (DI) ⭐

En vez de que una clase **cree** sus dependencias, **se las pasan** desde fuera.

```ts
// ❌ Acoplado: el servicio crea su propia conexión → difícil de testear y cambiar
class BooksService {
  private pool = new Pool({ connectionString: "..." });
}

// ✅ Inyección: la recibe en el constructor; Nest se encarga de crearla y pasarla
@Injectable()
export class BooksService {
  constructor(private readonly pool: Pool) {}
}

@Controller('books')
export class BooksController {
  constructor(private readonly booksService: BooksService) {} // Nest lo inyecta
}
```

| Ventaja | Por qué |
|---------|---------|
| 🧪 Tests | En el test le pasas un **mock** en lugar de la BD real |
| 🔌 Bajo acoplamiento | Cambias la implementación sin tocar quien la usa |
| ♻️ Una sola instancia | Nest crea los providers como *singletons* por defecto |

> 📌 Es la **D** de SOLID (*Dependency Inversion*): depende de abstracciones, no de
> implementaciones concretas. Tu `BooksService implements BooksContract` va en esa línea.

---

## 5.7 Validación ⭐

**Nunca confíes en lo que manda el cliente.** Valida en el backend siempre (la
validación del frontend es solo para la experiencia de usuario).

**Ejemplo real (`api-books-nest`):**

```ts
// create-book.dto.ts
export class CreateBookDto {
  @IsString() @MinLength(1) @MaxLength(200)
  title: string;

  @IsInt() @Min(1) @Max(2000)
  pages: number;
}

// main.ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,             // quita campos que no están en el DTO
  forbidNonWhitelisted: true,  // ...o responde 400 si llegan
  transform: true,
}));
```

En Express se suele usar **Zod**, **Joi** o `express-validator`, o validar a mano.

---

## 5.8 Manejo de errores

```ts
// ❌ Errores que se tragan o filtran detalles internos
app.get("/api/books/:id", async (req, res) => {
  const libro = await db.find(req.params.id);   // si falla → promesa rechazada sin manejar
  res.json(libro);                              // si no existe → 200 con null 😬
});

// ✅ Códigos correctos + mensaje claro + sin detalles internos
app.get("/api/books/:id", async (req, res, next) => {
  try {
    const libro = await db.find(req.params.id);
    if (!libro) return res.status(404).json({ error: "Libro no encontrado" });
    res.json(libro);
  } catch (err) {
    next(err);   // → middleware de errores centralizado → 500
  }
});
```

| Buena práctica | Por qué |
|----------------|---------|
| Status code correcto (400, 404, 409, 500…) | El cliente sabe qué ha pasado |
| Formato de error **consistente** (`{ error: "..." }`) | El frontend lo maneja igual siempre |
| No enviar el *stack trace* al cliente | Da pistas a un atacante |
| Manejador **centralizado** | No repites try/catch con la misma lógica |
| Loguear el error en el servidor | Para poder depurarlo |

> 📌 En Nest lanzas `throw new NotFoundException()` desde el servicio y el *exception
> filter* global lo convierte en un `404` automáticamente. Express 5 (el que usas en
> `api-rastro`) ya pasa al manejador de errores los errores de handlers `async`.

---

## 5.9 Otras preguntas rápidas

| Pregunta | Respuesta corta |
|----------|-----------------|
| `req.params` vs `req.query` vs `req.body` | `/books/:id` → params · `?author=x` → query · JSON enviado → body |
| ¿Qué son las variables de entorno? | Configuración fuera del código (`process.env.PORT`), ver `07` |
| ¿Cómo paginas? | `?page=2&limit=20` → `LIMIT 20 OFFSET 20` / `.skip(20).limit(20)` |
| ¿Qué es un ORM/ODM? | Librería que mapea tablas/colecciones a objetos (Prisma, TypeORM / Mongoose) |
| ¿CommonJS vs ESM? | `require` vs `import`. Se define con `"type"` en `package.json` |
| ¿Cómo escalarías? | Varias instancias tras un balanceador, caché, índices en BD, colas |

---

## 🎯 Cómo responderlo en la entrevista

**¿Qué es Node.js y por qué es rápido?**
> "Es un entorno para ejecutar JavaScript en el servidor, sobre el motor V8. Ejecuta el
> código en un solo hilo pero con I/O no bloqueante: las operaciones lentas como
> consultas a la base de datos se delegan y, mientras, sigue atendiendo otras
> peticiones. Por eso es muy bueno para APIs con muchas conexiones, y menos para
> cálculos pesados de CPU."

**¿Qué es un middleware?**
> "Una función que se ejecuta entre la petición y la respuesta, con acceso a `req`,
> `res` y `next`. Sirve para parsear el body, autenticar, loguear o manejar errores.
> En mi reproductor de música hice uno que comprueba la cookie de sesión y responde
> 401 si no hay usuario."

**¿Qué es una API REST?**
> "Una API organizada en recursos con URLs y que usa los métodos HTTP para las
> acciones: GET para leer, POST para crear, PATCH/PUT para actualizar y DELETE para
> borrar, con los status codes adecuados. Es stateless: cada petición lleva todo lo
> necesario."

**¿Cómo estructuras un backend?**
> "En capas: el controller solo maneja HTTP, el service tiene la lógica de negocio y
> una capa de datos hace las consultas. En mi API de libros con NestJS el controller
> recibe el id con `ParseIntPipe` y el servicio hace la consulta a PostgreSQL y lanza
> `NotFoundException` si no existe."

**¿Qué es la inyección de dependencias?**
> "Que una clase reciba sus dependencias desde fuera en vez de crearlas. En Nest
> declaro el servicio en el constructor del controller y el framework lo inyecta. Así
> el código está menos acoplado y en los tests puedo pasar un mock."

**¿Cómo validas los datos?**
> "Siempre en el backend. En NestJS uso DTOs con `class-validator` y un `ValidationPipe`
> global con `whitelist` para descartar campos que no esperaba."

**¿Express o NestJS?**
> "Express es minimalista y flexible, ideal para algo pequeño. Nest impone una
> arquitectura con módulos, DI y TypeScript, lo que ayuda en proyectos grandes y en
> equipo. He trabajado con los dos."
