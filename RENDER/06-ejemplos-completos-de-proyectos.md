# 06 — EJEMPLOS COMPLETOS DE PROYECTOS

> Archivos **enteros** (copiados tal cual) de tus dos proyectos desplegables en
> Render. Solo se han sustituido las contraseñas de desarrollo por `TU_PASSWORD`.
> Los `.env` reales **no** se copian.

---

## A) `postgresql-render-atlas/nestjs-postgresql-supabase-render/` — NestJS + PostgreSQL (Supabase) en Render

### Qué mirar en cada archivo

| Archivo | Lo importante para el despliegue |
|---------|----------------------------------|
| `package.json` | `build` = `nest build`, `start:prod` = `node dist/main` |
| `src/main.ts` | `.env` opcional, `PORT` de Render, prefijo `/api`, CORS |
| `src/database/database.module.ts` | Un único `Pool` con `DATABASE_URL` + `pool.on('error')` |
| `src/books/books.module.ts` | Importa `DatabaseModule` para poder inyectar el `Pool` |
| `database/schema.sql` | Lo que hay que ejecutar **en Supabase** antes del primer despliegue |
| `.gitignore` | `.env` y `dist/` **no** se suben |

### `package.json`

```json
{
  "name": "api-books-nest",
  "version": "0.0.1",
  "description": "",
  "author": "",
  "private": true,
  "license": "UNLICENSED",
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
  },
  "dependencies": {
    "@nestjs/common": "^11.0.1",
    "@nestjs/core": "^11.0.1",
    "@nestjs/platform-express": "^11.0.1",
    "class-transformer": "^0.5.1",
    "class-validator": "^0.15.1",
    "pg": "^8.23.1",
    "reflect-metadata": "^0.2.2",
    "rxjs": "^7.8.1"
  },
  "devDependencies": {
    "@eslint/eslintrc": "^3.2.0",
    "@eslint/js": "^9.18.0",
    "@nestjs/cli": "^11.0.0",
    "@nestjs/schematics": "^11.0.0",
    "@nestjs/testing": "^11.0.1",
    "@types/express": "^5.0.0",
    "@types/jest": "^30.0.0",
    "@types/node": "^24.0.0",
    "@types/pg": "^8.23.1",
    "@types/supertest": "^7.0.0",
    "eslint": "^9.18.0",
    "eslint-config-prettier": "^10.0.1",
    "eslint-plugin-prettier": "^5.2.2",
    "globals": "^17.0.0",
    "jest": "^30.0.0",
    "prettier": "^3.4.2",
    "source-map-support": "^0.5.21",
    "supertest": "^7.0.0",
    "ts-jest": "^29.2.5",
    "ts-loader": "^9.5.2",
    "ts-node": "^10.9.2",
    "tsconfig-paths": "^4.2.0",
    "typescript": "^5.7.3",
    "typescript-eslint": "^8.20.0"
  },
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
}

```

### `src/main.ts`

```ts
import { existsSync } from 'node:fs';
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

// En tu ordenador, lee el .env. En Render no hay .env: las variables las pone Render.
if (existsSync('.env')) process.loadEnvFile();

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Todas las rutas empiezan por /api
  app.setGlobalPrefix('api');

  // Comprueba los DTO en todas las rutas
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true, // quita los campos que no estén en el DTO...
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  app.enableCors({ origin: 'http://localhost:5173' });

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();

```

### `src/database/database.module.ts`

```ts
import { Module } from '@nestjs/common';
import { Pool } from 'pg';

// Crea el Pool y avisa (sin caerse) si PostgreSQL corta una conexión
function createPool() {
  const pool = new Pool({ connectionString: process.env.DATABASE_URL });

  // Si PostgreSQL se para, cierra las conexiones que no se estaban usando.
  // Sin esta línea, ese aviso tumbaría la API entera.
  pool.on('error', (error) => console.error('PostgreSQL:', error.message));

  return pool;
}

@Module({
  providers: [
    {
      provide: Pool, // cuando alguien pida un Pool...
      useFactory: createPool, // ...NestJS ejecuta esta función una vez
    },
  ],
  exports: [Pool], // otros módulos pueden pedir el Pool
})
export class DatabaseModule {}

```

### `src/books/books.module.ts`

```ts
import { Module } from '@nestjs/common';
import { DatabaseModule } from '../database/database.module';
import { BooksController } from './books.controller';
import { BooksService } from './books.service';

@Module({
  imports: [DatabaseModule], // para poder pedir el Pool
  controllers: [BooksController],
  providers: [BooksService],
})
export class BooksModule {}

```

### `database/schema.sql`

```sql
CREATE TABLE books (
  id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  author VARCHAR(100) NOT NULL,
  pages INTEGER NOT NULL CHECK (pages > 0),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

INSERT INTO books (title, author, pages)
VALUES
  ('Cien años de soledad', 'Gabriel García Márquez', 471),
  ('El principito', 'Antoine de Saint-Exupéry', 96),
  ('Crónica de una muerte anunciada', 'Gabriel García Márquez', 122);

```

### `.env.example`

```
DATABASE_URL=postgres://alumno:TU_PASSWORD@localhost:5432/bootcamp_api

```

### `.gitignore`

```
node_modules/
dist/
coverage/
.env
*.log

```

### `compose.yaml`

```yaml
services:
  postgres:
    image: postgres:17
    container_name: postgres-bootcamp-api
    environment:
      POSTGRES_USER: alumno
      POSTGRES_PASSWORD: TU_PASSWORD
      POSTGRES_DB: bootcamp_api
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      # Carga el esquema la primera vez que se crea la base
      - ./database/schema.sql:/docker-entrypoint-initdb.d/schema.sql:ro

volumes:
  postgres_data:
```

> 📌 `compose.yaml` es **solo para desarrollo**: en producción Postgres lo pone Supabase. Por eso Render no lo usa.

### `README.md` (sección de despliegue)

```markdown
## Desplegar en Render

Paso a paso en los apuntes `06_despliegue_render.md`. Resumen:

| Campo de Render | Valor |
|---|---|
| Root Directory | `03b-nestjs-postgresql` (si el proyecto está en una subcarpeta) |
| Build Command | `npm install && npm run build` |
| Start Command | `npm run start:prod` |
| Variables | `DATABASE_URL` (la del *Session pooler* de Supabase, terminada en `?sslmode=no-verify`) |

Para `04` y `05`, lo mismo con su carpeta y una variable más: `JWT_SECRET`.
```

---

## B) `mongo-atlas-render/express-mongo-atlas-render/` — Express + MongoDB Atlas en Render

### Qué mirar en cada archivo

| Archivo | Lo importante para el despliegue |
|---------|----------------------------------|
| `package.json` | `dev` usa `--env-file` (solo local); `build` = `tsc`; `start` = `node dist/index.js` |
| `tsconfig.json` | `rootDir: src` → `outDir: dist` (de ahí sale `dist/index.js`) |
| `src/index.ts` | `PORT` de Render, middleware de errores, `connectDb()` **antes** de `listen` |
| `src/db.ts` | Falla rápido si falta `MONGODB_URI`; `DB_NAME` con valor por defecto |

### `package.json`

```json
{
  "name": "02b-express-mongodb",
  "version": "1.0.0",
  "scripts": {
    "dev": "tsx watch --env-file=.env src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "typecheck": "tsc --noEmit",
    "lint": "eslint .",
    "seed": "docker cp mongo/seed.js mongodb-practica:/tmp/seed.js && docker exec mongodb-practica mongosh --quiet -u admin -p TU_PASSWORD --authenticationDatabase admin /tmp/seed.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^5.3.0",
    "mongodb": "^7.8.0"
  },
  "devDependencies": {
    "@eslint/js": "^10.0.1",
    "@types/express": "^5.0.6",
    "@types/node": "^26.6.5",
    "eslint": "^10.14.0",
    "globals": "^17.14.0",
    "tsx": "^4.23.15",
    "typescript": "~6.0",
    "typescript-eslint": "^8.72.1"
  },
  "private": true
}

```

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "rootDir": "src",
    "outDir": "dist"
  },
  "include": ["src/**/*.ts"]
}

```

### `src/index.ts`

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import productsRouter from "./routes/products";
import categoriesRouter from "./routes/categories";
import { connectDb } from "./db";

const app = express();
// Render nos dice el puerto en la variable PORT. En tu ordenador no existe: usamos 3000.
const PORT = process.env.PORT ?? 3000;

// 1. Registrar cada petición en la terminal
app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// 2. CORS: permitir peticiones desde nuestra página web
app.use((req, res, next) => {
  res.setHeader("Access-Control-Allow-Origin", "http://127.0.0.1:5500");
  res.setHeader("Access-Control-Allow-Methods", "GET, POST, PATCH, DELETE");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type");

  if (req.method === "OPTIONS") {
    res.sendStatus(204);
    return;
  }

  next();
});

// 3. Leer bodies JSON
app.use(express.json());

// 4. Rutas
app.get("/", (_req, res) => {
  res.send("Mi primera API con Express");
});

app.get("/api/hello", (req, res) => {
  const name = req.query.name;

  res.json({ message: `Hola, ${name ?? "persona"}` });
});

app.use("/api/products", productsRouter);
app.use("/api/categories", categoriesRouter); // Reto 2

// 5. Ninguna ruta ha respondido: 404
app.use((req, res) => {
  res.status(404).json({ error: `No existe la ruta ${req.method} ${req.url}` });
});

// 6. Ha ocurrido un error: respuesta JSON sin detalles internos
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  if (err instanceof SyntaxError) {
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }

  console.error(err);
  res.status(500).json({ error: "Error interno del servidor" });
});

async function start() {
  await connectDb();

  app.listen(PORT, () => {
    console.log(`Servidor escuchando en http://localhost:${PORT}`);
  });
}

start();

```

### `src/db.ts`

```ts
import { Db, MongoClient } from "mongodb";

const uri = process.env.MONGODB_URI;
if (!uri) {
  throw new Error("Falta MONGODB_URI en el archivo .env");
}

const client = new MongoClient(uri);

let db: Db;

export async function connectDb() {
  await client.connect();
  db = client.db(process.env.DB_NAME ?? "bootcamp_api");

  // Reto 2: índice único para que no se repita el nombre de una categoría.
  // Si ya existe, no hace nada.
  await db.collection("categories").createIndex({ name: 1 }, { unique: true });

  console.log("Conectado a MongoDB");
}

export function getDb() {
  return db;
}

```

### `.env.example`

```
MONGODB_URI=mongodb://admin:TU_PASSWORD@localhost:27017/?authSource=admin
DB_NAME=bootcamp_api

```

### `README.md` (sección de despliegue)

```markdown
## Desplegar en Render

Paso a paso en los apuntes `06_despliegue_render.md`. Resumen:

| Campo de Render | Valor |
|---|---|
| Root Directory | `02b-express-mongodb` (si el proyecto está en una subcarpeta) |
| Build Command | `npm install && npm run build` |
| Start Command | `npm start` |
| Variables | `MONGODB_URI` (la de MongoDB Atlas) y `DB_NAME=bootcamp_api` |

`npm run build` compila TypeScript a `dist/` y `npm start` ejecuta `node dist/index.js`. El puerto sale de `process.env.PORT` (Render lo pone solo; en tu ordenador es 3000).
```

---

## C) Comparativa rápida de los dos despliegues

| | NestJS + Supabase | Express + Atlas |
|---|---|---|
| Build Command | `npm install && npm run build` | `npm install && npm run build` |
| Compilador | `nest build` (usa `tsc` por debajo) | `tsc` |
| Start Command | `npm run start:prod` → `node dist/main` | `npm start` → `node dist/index.js` |
| Cómo carga `.env` en local | `process.loadEnvFile()` si existe | `tsx --env-file=.env` en el script `dev` |
| Variables en Render | `DATABASE_URL` | `MONGODB_URI`, `DB_NAME` |
| Base de datos | PostgreSQL en Supabase (Session pooler) | MongoDB Atlas (`mongodb+srv://`) |
| ¿Arranca si la BD no responde? | Sí (el `Pool` conecta en la 1ª consulta) | No (`await connectDb()` antes de `listen`) |
| Health check sugerido | `/api` | `/` |
| CORS actual | `http://localhost:5173` | `http://127.0.0.1:5500` |
