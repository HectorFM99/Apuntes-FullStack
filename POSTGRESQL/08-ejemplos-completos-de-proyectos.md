# 08 — EJEMPLOS COMPLETOS DE PROYECTOS

> Archivos **enteros** copiados de tus proyectos, para repasar todo junto.
> Lee cada uno intentando reconocer los conceptos de los archivos `01`–`07`.
>
> ⚠️ Los proyectos **NO** se han modificado.
> 🔒 En los `compose.yaml` la contraseña real se ha sustituido por `TU_PASSWORD`.
> Los archivos `.env` **no** se han copiado.

---

## 8.1 Proyecto SQL del gimnasio — esquema, datos y consultas

**Archivo:** `Proyecto-SQL/Proyecto-SQL/gimnasio.sql` *(copiado tal cual)*

**Qué repasar aquí:**
- **B1** → `CREATE TABLE` con `IDENTITY`, `NOT NULL`, `UNIQUE`, `CHECK`, `REFERENCES` y `UNIQUE` compuesto (archivo `03`).
- **B2** → `INSERT` de varias filas (archivo `04`).
- **B3** → inserts y deletes que **fallan a propósito** por las restricciones (archivo `03`).
- **B4** → `JOIN`, `LEFT JOIN ... IS NULL`, `GROUP BY` + `COUNT` (archivo `05`).
- **B5** → `UPDATE`, `INSERT ... SELECT`, `DELETE` con subconsultas (archivos `04` y `05`).

```sql
--Ejercicio B1

--B1.1
CREATE TABLE planes (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre VARCHAR(50) UNIQUE NOT NULL,
    precio_mensual NUMERIC(6,2) NOT NULL CHECK (precio_mensual >= 0)
);

--B1.2
CREATE TABLE socios (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    dni VARCHAR(9) UNIQUE NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    fecha_alta DATE NOT NULL,
    plan_id INTEGER NOT NULL REFERENCES planes(id)
);

--B1.3
CREATE TABLE tarjetas (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    codigo VARCHAR(20) UNIQUE NOT NULL,
    activa BOOLEAN NOT NULL,
    socio_id INTEGER UNIQUE NOT NULL REFERENCES socios(id)
);

--B1.4
CREATE TABLE monitores (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    dni VARCHAR(9) UNIQUE NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    telefono VARCHAR(15),
    especialidad VARCHAR(50)
);

--B1.5
CREATE TABLE clases (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    fecha_hora TIMESTAMP NOT NULL,
    sala VARCHAR(50) NOT NULL,
    plazas INTEGER NOT NULL CHECK (plazas > 0),
    monitor_id INTEGER NOT NULL REFERENCES monitores(id)
);

--B1.6
CREATE TABLE reservas (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    socio_id INTEGER NOT NULL REFERENCES socios(id),
    clase_id INTEGER NOT NULL REFERENCES clases(id),
    fecha_reserva DATE NOT NULL,
    UNIQUE (socio_id, clase_id)
);

--B1.7
CREATE TABLE especialidades (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre VARCHAR(100) UNIQUE NOT NULL,
    descripcion TEXT
);

--B1.8
CREATE TABLE monitor_especialidades (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    monitor_id INTEGER NOT NULL REFERENCES monitores(id),
    especialidad_id INTEGER NOT NULL REFERENCES especialidades(id),
    CONSTRAINT uk_monitor_especialidad UNIQUE (monitor_id, especialidad_id)
);

--\dt y \d clases en psql, o Tables → Refresh en pgAdmin.

--Ejercicio B2

--B2.1
INSERT INTO planes (nombre, precio_mensual)
VALUES
    ('Básico', 29.90),
    ('Completo', 44.90),
    ('Estudiante', 24.90),
    ('Mañanas', 19.90);

--B2.2
INSERT INTO socios (dni, nombre, email, fecha_alta, plan_id)
VALUES
    ('11111111A', 'Ana Pérez', 'ana@gym.com', '2026-01-10', 2),
    ('22222222B', 'Luis García', 'luis@gym.com', '2026-02-03', 1),
    ('33333333C', 'Marta López', 'marta@gym.com', '2026-03-15', 3),
    ('44444444D', 'Hugo Sanz', 'hugo@gym.com', '2026-03-20', 2),
    ('55555555E', 'Lucía Ruiz', 'lucia@gym.com', '2026-05-02', 3),
    ('66666666F', 'Pablo Martín', 'pablo@gym.com', '2026-06-11', 1),
    ('77777777G', 'Sara Gil', 'sara@gym.com', '2026-09-01', 2),
    ('88888888H', 'Diego Vidal', 'diego@gym.com', '2026-10-01', 1);

--B2.3
INSERT INTO tarjetas (codigo, activa, socio_id)
VALUES
    ('TJ-0001', TRUE, 1),
    ('TJ-0002', TRUE, 2),
    ('TJ-0003', TRUE, 3),
    ('TJ-0004', FALSE, 4),
    ('TJ-0005', TRUE, 5),
    ('TJ-0006', TRUE, 6),
    ('TJ-0007', TRUE, 7);

--B2.4
INSERT INTO monitores (dni, nombre, email)
VALUES
    ('90000001J', 'Carla Navarro', 'carla@gym.com'),
    ('90000002K', 'Javier Ortega', 'javier@gym.com'),
    ('90000003L', 'Elena Romero', 'elena@gym.com'),
    ('90000004M', 'Rubén Castro', 'ruben@gym.com');

--B2.5
INSERT INTO clases (nombre, sala, fecha_hora, plazas, monitor_id)
VALUES
    ('Spinning', 'Sala 1', '2026-11-02 09:00', 12, 1),
    ('Yoga', 'Sala 2', '2026-11-02 18:00', 10, 2),
    ('Pilates', 'Sala 2', '2026-11-03 10:00', 8, 3),
    ('Spinning', 'Sala 1', '2026-11-03 19:00', 12, 1),
    ('Zumba', 'Sala 3', '2026-11-04 18:30', 20, 3),
    ('Yoga', 'Sala 2', '2026-11-05 09:30', 10, 2),
    ('Crossfit', 'Sala 3', '2026-11-05 20:00', 3, 1);

--B2.6
INSERT INTO reservas (socio_id, clase_id, fecha_reserva)
VALUES
    (1, 1, '2026-10-28'),
    (1, 3, '2026-10-28'),
    (1, 7, '2026-10-29'),
    (2, 1, '2026-10-29'),
    (2, 2, '2026-10-29'),
    (3, 2, '2026-10-30'),
    (3, 5, '2026-10-30'),
    (4, 1, '2026-10-30'),
    (4, 7, '2026-10-31'),
    (5, 3, '2026-10-31'),
    (5, 5, '2026-10-31'),
    (6, 4, '2026-11-01'),
    (7, 5, '2026-11-01'),
    (7, 7, '2026-11-01');

--B2.7
INSERT INTO especialidades (nombre)
VALUES
    ('Ciclo indoor'),
    ('Yoga'),
    ('Pilates'),
    ('Baile'),
    ('Fuerza');

--B2.8
INSERT INTO monitor_especialidades (monitor_id, especialidad_id)
VALUES
    (1, 1),
    (1, 5),
    (2, 2),
    (2, 3),
    (3, 3),
    (3, 4),
    (4, 5);

--Ejercicio B3

-- B3.1
INSERT INTO socios (dni, nombre, email, fecha_alta, plan_id)
VALUES ('99999999Z', 'Nora Prueba', 'nora@gym.com', '2026-10-27', 99);

-- B3.2
INSERT INTO socios (dni, nombre, email, fecha_alta, plan_id)
VALUES ('99999999Z', 'Nora Prueba', 'ana@gym.com', '2026-10-27', 1);

-- B3.3
INSERT INTO tarjetas (codigo, activa, socio_id)
VALUES ('TJ-0099', TRUE, 1);

-- B3.4
INSERT INTO reservas (socio_id, clase_id, fecha_reserva)
VALUES (1, 1, '2026-10-30');

-- B3.5
INSERT INTO reservas (socio_id, clase_id, fecha_reserva)
VALUES (2, 50, '2026-10-30');

-- B3.6
DELETE FROM planes WHERE nombre = 'Básico';

-- B3.7
DELETE FROM planes WHERE nombre = 'Mañanas';

--Ejercicio B4

--B4.1
SELECT s.nombre, p.nombre AS plan, p.precio_mensual
FROM socios AS s
JOIN planes AS p ON s.plan_id = p.id
ORDER BY s.nombre;

--B4.2
SELECT c.nombre AS actividad, c.fecha_hora, c.sala, m.nombre AS monitor
FROM clases AS c
JOIN monitores AS m ON c.monitor_id = m.id
ORDER BY c.fecha_hora;

--B4.3
SELECT s.nombre, t.codigo
FROM socios AS s
LEFT JOIN tarjetas AS t ON t.socio_id = s.id;

--B4.4
SELECT s.nombre
FROM socios AS s
LEFT JOIN tarjetas AS t ON t.socio_id = s.id
WHERE t.id IS NULL;

--B4.5
SELECT p.nombre AS plan, COUNT(s.id) AS cantidad_socios
FROM planes AS p
LEFT JOIN socios AS s ON s.plan_id = p.id
GROUP BY p.id, p.nombre
ORDER BY p.nombre;

--B4.6
SELECT s.nombre AS socio, c.nombre AS actividad, c.fecha_hora
FROM reservas AS r
JOIN socios AS s ON r.socio_id = s.id
JOIN clases AS c ON r.clase_id = c.id;

--B4.7
SELECT s.nombre
FROM reservas AS r
JOIN socios AS s ON r.socio_id = s.id
WHERE r.clase_id = 1;

--B4.8
SELECT c.nombre AS actividad, c.fecha_hora, c.plazas, COUNT(r.id) AS numero_reservas
FROM clases AS c
LEFT JOIN reservas AS r ON r.clase_id = c.id
GROUP BY c.id, c.nombre, c.fecha_hora, c.plazas
ORDER BY c.fecha_hora;

--B4.9
SELECT m.nombre
FROM monitores AS m
LEFT JOIN clases AS c ON c.monitor_id = m.id
WHERE c.id IS NULL;

--B4.10
SELECT m.nombre AS monitor, e.nombre AS especialidad
FROM monitores AS m
JOIN monitor_especialidades AS me ON me.monitor_id = m.id
JOIN especialidades AS e ON e.id = me.especialidad_id
ORDER BY m.nombre, e.nombre;

--Ejercicios B5

--B5.1
UPDATE socios
SET plan_id = (
    SELECT id
    FROM planes
    WHERE nombre = 'Completo'
)
WHERE email = 'luis@gym.com';

--B5.2
UPDATE tarjetas
SET activa = TRUE
WHERE codigo = 'TJ-0004';

--B5.3
INSERT INTO tarjetas (codigo, activa, socio_id)
SELECT 'TJ-0008', TRUE, id
FROM socios
WHERE email = 'diego@gym.com';

--B5.4
INSERT INTO reservas (socio_id, clase_id, fecha_reserva)
SELECT id, 6, '2026-11-02'
FROM socios
WHERE email = 'diego@gym.com';

--B5.5
DELETE FROM reservas
WHERE socio_id = (SELECT id FROM socios WHERE email = 'luis@gym.com')
  AND clase_id = 2;

--B5.6
DELETE FROM socios
WHERE email = 'pablo@gym.com';

--B5.7
DELETE FROM reservas
WHERE socio_id = (SELECT id FROM socios WHERE email = 'pablo@gym.com');

--B5.8
DELETE FROM socios
WHERE email = 'pablo@gym.com';
```

> 📌 **Recuerda (B5.6–B5.8):** Pablo también tiene tarjeta (`TJ-0006`), así que para
> que el último `DELETE` funcione hay que borrar antes su fila de `tarjetas` (ver
> archivo `04`, apartado 4.8). Y si ejecutaste el B3.7, el plan "Mañanas" ya no
> existe cuando llegas al B4.5.

---

## 8.2 La consulta de reservas presentada en HTML

**Archivo:** `Proyecto-SQL/consulta-reservas.html` *(copiado tal cual)*

Es la consulta B4.6 (doble `JOIN` a través de la tabla intermedia `reservas`),
coloreada a mano con `<span>`: `.kw` palabras clave, `.alias` alias, `.col`
columnas, `.table` tablas.

```html
<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Consulta SQL — Reservas</title>
<style>
*{box-sizing:border-box}body{margin:0;min-height:100vh;display:grid;place-items:center;background:#0b1020;color:#e5e7eb;font:15px/1.65 ui-monospace,SFMono-Regular,Consolas,monospace}.card{width:min(900px,92vw);padding:28px 32px;background:#111827;border:1px solid #273449;border-radius:14px;box-shadow:0 24px 70px #0008}.title{margin:0 0 22px;color:#94a3b8;font:600 13px/1.2 system-ui,sans-serif;letter-spacing:.08em;text-transform:uppercase}pre{margin:0;white-space:pre-wrap}.kw{color:#c084fc}.alias{color:#7dd3fc}.col{color:#fbbf24}.table{color:#86efac}.str{color:#fb7185}</style>
</head>
<body><main class="card"><h1 class="title">Consulta SQL · Lista de reservas</h1><pre><span class="kw">SELECT</span> <span class="alias">s</span>.<span class="col">nombre</span> <span class="kw">AS</span> socio, <span class="alias">c</span>.<span class="col">nombre</span> <span class="kw">AS</span> actividad, <span class="alias">c</span>.<span class="col">fecha_hora</span>
<span class="kw">FROM</span> <span class="table">reservas</span> <span class="kw">AS</span> <span class="alias">r</span>
<span class="kw">JOIN</span> <span class="table">socios</span> <span class="kw">AS</span> <span class="alias">s</span> <span class="kw">ON</span> <span class="alias">r</span>.<span class="col">socio_id</span> = <span class="alias">s</span>.<span class="col">id</span>
<span class="kw">JOIN</span> <span class="table">clases</span> <span class="kw">AS</span> <span class="alias">c</span> <span class="kw">ON</span> <span class="alias">r</span>.<span class="col">clase_id</span> = <span class="alias">c</span>.<span class="col">id</span>;</pre></main></body>
</html>
```

Sin el HTML, la consulta es:

```sql
SELECT s.nombre AS socio, c.nombre AS actividad, c.fecha_hora
FROM reservas AS r
JOIN socios AS s ON r.socio_id = s.id
JOIN clases AS c ON r.clase_id = c.id;
```

---

## 8.3 Docker Compose para Postgres

**Archivo:** `practica-sql/compose.yaml` *(copiado, contraseña sustituida)*

```yaml
services:
  postgres:
    image: postgres:17
    container_name: postgres-sql
    environment:
      POSTGRES_DB: academia
      POSTGRES_USER: alumno
      POSTGRES_PASSWORD: TU_PASSWORD
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

**Archivo:** `puta/api-books-nest/compose.yaml` *(copiado, contraseña sustituida)*

```yaml
services:
  postgres:
    image: postgres:17
    container_name: practica-sql
    environment:
      POSTGRES_DB: academia
      POSTGRES_USER: alumno
      POSTGRES_PASSWORD: TU_PASSWORD
      DATABASE_URL: postgres://alumno:TU_PASSWORD@postgres:5432/academia
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

> 📌 `pruebas/compose.yaml` es igual que el de `practica-sql`, pero con
> `POSTGRES_DB: library`. Los tres usan el puerto `5432`: arranca solo uno a la vez.

---

## 8.4 API de libros (NestJS + `pg`) — esquema de la base de datos

**Archivo:** `puta/api-books-nest/database/schema.sql` *(copiado tal cual)*

**Qué repasar aquí:** `IDENTITY`, `CHECK (pages > 0)`, `DEFAULT NOW()`,
`TIMESTAMPTZ`, `UNIQUE` en email, `DEFAULT 'customer'` + `CHECK (role IN (...))`,
y que se guarda el **hash** de la contraseña.

```sql
CREATE TABLE books (
  id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- el id lo pone PostgreSQL
  title VARCHAR(200) NOT NULL,
  author VARCHAR(100) NOT NULL,
  pages INTEGER NOT NULL CHECK (pages > 0),            -- la base de datos también valida
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()        -- fecha de creación automática
);

INSERT INTO books (title, author, pages)
VALUES
  ('Cien años de soledad', 'Gabriel García Márquez', 471),
  ('El principito', 'Antoine de Saint-Exupéry', 96),
  ('Crónica de una muerte anunciada', 'Gabriel García Márquez', 122);

CREATE TABLE users (
  id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email VARCHAR(255) NOT NULL UNIQUE,      -- no puede haber dos iguales
  password_hash TEXT NOT NULL,             -- el hash, nunca la contraseña
  role VARCHAR(20) NOT NULL DEFAULT 'customer'
    CHECK (role IN ('customer', 'employee'))  -- solo estos dos roles
);
```

---

## 8.5 API de libros — conexión con el `Pool`

**Archivo:** `puta/api-books-nest/src/database/database.module.ts` *(copiado tal cual)*

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

**Archivo:** `puta/api-books-nest/src/database/database.service.ts` *(copiado tal cual)*

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

---

## 8.6 API de libros — tipos de fila y conversión

**Archivo:** `puta/api-books-nest/src/books/book.ts` *(copiado tal cual)*

```ts
// Así llega un libro desde PostgreSQL (una fila de la tabla)
export interface BookRow {
  id: number;
  title: string;
  author: string;
  pages: number;
  created_at: Date; // en SQL, con guion bajo
}

// Así lo devuelve nuestra API
export interface Book {
  id: number;
  title: string;
  author: string;
  pages: number;
  createdAt: Date; // en JavaScript, en camelCase
}

// Convierte una fila de la base de datos en un libro de la API
export function toBook(row: BookRow): Book {
  return {
    id: row.id,
    title: row.title,
    author: row.author,
    pages: row.pages,
    createdAt: row.created_at,
  };
}
```

---

## 8.7 API de libros — CRUD completo con SQL parametrizado

**Archivo:** `puta/api-books-nest/src/books/books.service.ts` *(copiado tal cual)*

**Qué repasar aquí:**
- `findAll` → filtros opcionales con `($1::text IS NULL OR ...)`, `ILIKE`, `||`, `ORDER BY`.
- `count` → `COUNT(*)::text` + `Number(...)` (el `bigint` llega como string).
- `findOne` → `WHERE id = $1` + 404 si `rows.length === 0`.
- `create` → `INSERT ... RETURNING`.
- `update` → `COALESCE($1, title)` para actualizar solo lo que llega.
- `remove` → `DELETE ... WHERE id = $1`.

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

  async findAll(author?: string, maxPages?: number): Promise<Book[]> {
    const result = await this.pool.query<BookRow>(
      `SELECT id, title, author, pages, created_at
       FROM books
       WHERE ($1::text IS NULL OR author ILIKE '%' || $1 || '%')
         AND ($2::int IS NULL OR pages <= $2)
       ORDER BY id`,
      [author || null, maxPages ?? null],
    );
    return result.rows.map(toBook);
  }

  async count(): Promise<{ total: number }> {
    const result = await this.pool.query<{ total: string }>(
      'SELECT COUNT(*)::text AS total FROM books',
    );
    return { total: Number(result.rows[0].total) };
  }

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

  async create(dto: CreateBookDto): Promise<Book> {
    const result = await this.pool.query<BookRow>(
      `INSERT INTO books (title, author, pages)
       VALUES ($1, $2, $3)
       RETURNING id, title, author, pages, created_at`,
      [dto.title, dto.author, dto.pages],
    );
    return toBook(result.rows[0]);
  }

  async update(id: number, dto: UpdateBookDto): Promise<Book> {
    if (dto.title === undefined && dto.author === undefined && dto.pages === undefined) {
      throw new BadRequestException('Envía al menos un campo');
    }
    const book = await this.findOne(id);

    const result = await this.pool.query<BookRow>(
      `UPDATE books
       SET title = COALESCE($1, title),
           author = COALESCE($2, author),
           pages = COALESCE($3, pages)
       WHERE id = $4
       RETURNING id, title, author, pages, created_at`,
      [dto.title ?? null, dto.author ?? null, dto.pages ?? null, id],
    );
    return toBook(result.rows[0]);
  }

  async remove(id: number): Promise<Book> {
    const book = await this.findOne(id);
    await this.pool.query('DELETE FROM books WHERE id = $1', [id]);
    return book;
  }
}
```

---

## 8.8 API de libros — usuarios

**Archivo:** `puta/api-books-nest/src/users/user.ts` *(copiado tal cual)*

```ts
// Un usuario tal y como está en la tabla users
export interface User {
  id: number;
  email: string;
  password_hash: string;
  role: string; // 'customer' o 'employee'
}
```

**Archivo:** `puta/api-books-nest/src/users/users.service.ts` *(copiado tal cual)*

```ts
import { Injectable } from '@nestjs/common';
import { Pool } from 'pg';
import { User } from './user';

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

  // Busca un usuario por su id. undefined si no existe (lo usará el AuthGuard)
  async findById(id: number): Promise<User | undefined> {
    const result = await this.pool.query<User>(
      'SELECT id, email, password_hash, role FROM users WHERE id = $1',
      [id],
    );
    return result.rows[0];
  }

  // Guarda un usuario nuevo (con el hash, no con la contraseña)
  async create(email: string, passwordHash: string): Promise<User> {
    const result = await this.pool.query<User>(
      `INSERT INTO users (email, password_hash)
       VALUES ($1, $2)
       RETURNING id, email, password_hash, role`,
      [email, passwordHash],
    );
    return result.rows[0];
  }
}
```

---

## 8.9 Comparación: la conexión de `api-rastro` (MongoDB, NO Postgres)

**Archivo:** `api-rastro/api-rastro/src/db.ts` *(copiado tal cual)*

Este proyecto usa **MongoDB**. Se incluye solo para **comparar** con la conexión
de Postgres del apartado 8.5.

```ts
import { Db, MongoClient } from "mongodb";
import type { Listing, Offer, User } from "./types";

const uri = process.env.MONGODB_URI;
if (!uri) {
  throw new Error("Falta MONGODB_URI en el archivo .env");
}

const client = new MongoClient(uri, { serverSelectionTimeoutMS: 5000 });

let db: Db;

/**
 * Conecta a la base de datos MongoDB y asigna la instancia de Db.
 */
export async function connectDb() {
  await client.connect();
  db = client.db(process.env.DB_NAME ?? "rastro");
  console.log(`Conectado a MongoDB (base de datos ${db.databaseName})`);
}

export function getDb() {
  return db;
}

export function usersCollection() {
  return getDb().collection<User>("users");
}

export function listingsCollection() {
  return getDb().collection<Listing>("listings");
}

export function offersCollection() {
  return getDb().collection<Offer>("offers");
}
```

| | `api-books-nest` (Postgres) | `api-rastro` (MongoDB) |
|---|---|---|
| Librería | `pg` | `mongodb` |
| Variable de entorno | `DATABASE_URL` | `MONGODB_URI` + `DB_NAME` |
| Objeto de conexión | `Pool` | `MongoClient` → `Db` |
| Dónde viven los datos | **Tablas** con columnas fijas | **Colecciones** de documentos JSON |
| Estructura | Definida antes con `CREATE TABLE` | Flexible (la define tu tipo `Listing`) |
| Relaciones | **FK** comprobadas por la BD | Ids o copias dentro del documento; las comprueba **tu código** |
| Consulta | Texto SQL: `SELECT ... WHERE id = $1` | Objetos: `find({ _id: new ObjectId(id) })` |
| Agrupar | `GROUP BY` | `aggregate([{ $group: ... }])` |
| Transacciones | `BEGIN` / `COMMIT` siempre disponibles | Necesitan replica set |

---

## ✅ Checklist final de repaso

- [ ] Sé explicar PK, FK y UNIQUE, y en qué se diferencian.
- [ ] Sé decidir si una relación es 1:1, 1:N o N:M con **las dos preguntas**, y dónde va la FK.
- [ ] Sé levantar Postgres con `docker compose up -d` y entrar con `psql`.
- [ ] Sé escribir un `CREATE TABLE` con `IDENTITY`, `NOT NULL`, `UNIQUE`, `CHECK`, `DEFAULT` y `REFERENCES`.
- [ ] Sé leer un error de FK / UNIQUE / CHECK y explicar qué lo ha impedido.
- [ ] Sé hacer `INSERT`, `SELECT` con `WHERE`/`ORDER BY`/`LIMIT`, `UPDATE` y `DELETE` **con `WHERE`**.
- [ ] Sé hacer `JOIN` y `LEFT JOIN`, y el patrón "los que no tienen" (`IS NULL`).
- [ ] Sé usar `COUNT` + `GROUP BY`, y cuándo `HAVING` en vez de `WHERE`.
- [ ] Sé usar una subconsulta para buscar un id por otro dato (email).
- [ ] Sé qué es un índice, cuándo crearlo y que las FK no lo tienen automático.
- [ ] Sé explicar 1FN, 2FN y 3FN con un ejemplo.
- [ ] Sé usar `BEGIN` / `COMMIT` / `ROLLBACK` y qué significa ACID.
- [ ] Sé consultar Postgres desde Node con `Pool` y **`$1`**, y por qué evita la inyección SQL.

👉 **Vuelve al índice:** `00-INDICE.md`
