# 📋 CHEATSHEET — POSTGRESQL

> Chuleta rápida de comandos y código. Teoría en los documentos 01–08 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal (Docker + psql)

```bash
docker compose up -d          # arrancar Postgres en segundo plano
docker compose ps             # ¿está corriendo?
docker compose logs -f        # ver logs (si no arranca)
docker compose stop           # parar (conserva el contenedor)
docker compose down           # parar y borrar el contenedor (el volumen se queda)
docker compose down -v        # ⚠️ borra también el VOLUMEN = pierdes los datos

docker exec -it postgres-sql psql -U alumno -d academia          # entrar a psql
docker exec -i postgres-sql psql -U alumno -d academia < gimnasio.sql   # ejecutar un .sql
```

```yaml
# compose.yaml mínimo
services:
  postgres:
    image: postgres:17
    container_name: postgres-sql
    environment:
      POSTGRES_DB: academia
      POSTGRES_USER: alumno
      POSTGRES_PASSWORD: <PASSWORD>
    ports:
      - "5432:5432"            # "5433:5432" si el 5432 ya está ocupado
    volumes:
      - postgres_data:/var/lib/postgresql/data
volumes:
  postgres_data:
```

```
postgres://alumno:<PASSWORD>@localhost:5432/academia
protocolo  usuario  contraseña   host  puerto  base de datos
```

### Meta-comandos de psql

| Comando | Qué hace |
|---------|----------|
| `\l` | Lista las bases de datos |
| `\c library` | Cambia a la base `library` |
| `\dt` | Lista las tablas |
| `\d clases` | Describe la tabla (columnas, tipos, restricciones) |
| `\di` / `\du` | Lista índices / usuarios |
| `\x` | Vista expandida on/off |
| `\i archivo.sql` | Ejecuta un archivo |
| `\h CREATE TABLE` / `\?` | Ayuda SQL / ayuda de meta-comandos |
| `\q` | Salir |

---

## 2. Tipos de datos

| Tipo | Uso |
|------|-----|
| `INTEGER` / `BIGINT` | Enteros / ids |
| `NUMERIC(6,2)` | Dinero exacto (máx. 9999.99) |
| `VARCHAR(n)` / `TEXT` | Texto con / sin límite |
| `BOOLEAN` | `TRUE` / `FALSE` |
| `DATE` / `TIMESTAMP` / `TIMESTAMPTZ` | Fecha / fecha+hora / con zona horaria |
| `UUID` | `DEFAULT gen_random_uuid()` |
| `JSONB` | JSON indexable |

```sql
SELECT '42'::int;              -- cast propio de Postgres
SELECT CAST('42' AS INTEGER);  -- cast estándar
SELECT NOW()::date;            -- timestamp → fecha
-- Comentario de una línea      /* de varias */
```

---

## 3. DDL: crear y cambiar tablas

```sql
CREATE TABLE planes (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,   -- id autogenerado
    nombre VARCHAR(50) UNIQUE NOT NULL,                     -- único y obligatorio
    precio_mensual NUMERIC(6,2) NOT NULL CHECK (precio_mensual >= 0)
);

CREATE TABLE socios (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(150) UNIQUE NOT NULL,
    fecha_alta DATE NOT NULL DEFAULT CURRENT_DATE,          -- valor por defecto
    plan_id INTEGER NOT NULL REFERENCES planes(id)          -- FK (1:N)
);

CREATE TABLE reservas (                                     -- tabla intermedia (N:M)
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    socio_id INTEGER NOT NULL REFERENCES socios(id) ON DELETE CASCADE,
    clase_id INTEGER NOT NULL REFERENCES clases(id),
    UNIQUE (socio_id, clase_id)                             -- la PAREJA no se repite
);
```

| Restricción | Ejemplo |
|-------------|---------|
| `PRIMARY KEY` | `id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY` (antiguo: `SERIAL`) |
| `NOT NULL` | `nombre VARCHAR(100) NOT NULL` |
| `UNIQUE` | `email ... UNIQUE` · `UNIQUE (a, b)` |
| `CHECK` | `CHECK (plazas > 0)` · `CHECK (role IN ('customer','employee'))` |
| `DEFAULT` | `DEFAULT NOW()` · `DEFAULT TRUE` · `DEFAULT 'customer'` |
| `REFERENCES` | `REFERENCES planes(id)` · con nombre: `CONSTRAINT fk_x FOREIGN KEY (col) REFERENCES t(id)` |
| `ON DELETE` | `CASCADE` (borra hijos) · `SET NULL` · por defecto: impide borrar el padre |
| 1:1 | FK con `UNIQUE`: `socio_id INTEGER UNIQUE NOT NULL REFERENCES socios(id)` |

```sql
ALTER TABLE socios ADD COLUMN telefono VARCHAR(15);                   -- añadir columna
ALTER TABLE tarjetas ADD COLUMN creada_en TIMESTAMPTZ NOT NULL DEFAULT NOW();
ALTER TABLE monitores DROP COLUMN especialidad;                       -- borrar columna
ALTER TABLE socios RENAME COLUMN nombre TO nombre_completo;           -- renombrar columna
ALTER TABLE socios RENAME TO clientes;                                -- renombrar tabla
ALTER TABLE socios ALTER COLUMN email TYPE VARCHAR(255);              -- cambiar tipo
ALTER TABLE monitores ALTER COLUMN telefono SET NOT NULL;             -- / DROP NOT NULL
ALTER TABLE tarjetas ALTER COLUMN activa SET DEFAULT TRUE;
ALTER TABLE clases ADD CONSTRAINT ck_plazas_max CHECK (plazas <= 50);
ALTER TABLE monitor_especialidades DROP CONSTRAINT uk_monitor_especialidad;

DROP TABLE IF EXISTS reservas;              -- borra estructura + datos
DROP TABLE socios CASCADE;                  -- ⚠️ y las FK que apuntan a ella
TRUNCATE TABLE reservas RESTART IDENTITY;   -- vacía y reinicia el id
```

> 📌 Orden de creación: primero los **padres** (`planes`), luego los hijos (`socios`). Al borrar, al revés.

---

## 4. DML: INSERT, SELECT, UPDATE, DELETE

```sql
INSERT INTO books (title, author, pages)
VALUES ('Cien años de soledad', 'Gabriel García Márquez', 471),
       ('Rayuela', 'Julio Cortázar', 600)
RETURNING id, title;                              -- devuelve las filas creadas

SELECT id, title FROM books;                      -- columnas concretas (mejor que *)
SELECT * FROM socios WHERE plan_id = 1 AND fecha_alta >= '2026-01-01';
SELECT * FROM planes WHERE precio_mensual BETWEEN 20 AND 30;   -- ambos incluidos
SELECT * FROM socios WHERE plan_id IN (1, 3);
SELECT * FROM socios WHERE email LIKE '%@gym.com';             -- % = cualquier cosa, _ = 1 carácter
SELECT * FROM books  WHERE author ILIKE '%garcía%';            -- sin distinguir mayúsculas
SELECT * FROM monitores WHERE telefono IS NULL;                -- ❌ nunca "= NULL"
SELECT COALESCE(telefono, 'sin teléfono') FROM monitores;      -- valor si es NULL
SELECT DISTINCT sala FROM clases;
SELECT id, title FROM books ORDER BY pages DESC, title ASC;
SELECT id, title FROM books ORDER BY id LIMIT 10 OFFSET 10;    -- página 2: OFFSET = (pág-1)*tamaño

UPDATE books SET pages = 210 WHERE id = 1 RETURNING *;         -- ⚠️ sin WHERE cambia TODAS
DELETE FROM reservas WHERE id = 5 RETURNING *;                 -- ⚠️ sin WHERE borra TODAS
```

---

## 5. JOINs

```sql
SELECT s.nombre, p.nombre AS plan, p.precio_mensual
FROM socios AS s
JOIN planes AS p ON s.plan_id = p.id;                 -- INNER: solo con pareja

SELECT s.nombre, t.codigo
FROM socios AS s
LEFT JOIN tarjetas AS t ON t.socio_id = s.id;         -- todos los socios, NULL si no hay tarjeta

SELECT s.nombre FROM socios s
LEFT JOIN tarjetas t ON t.socio_id = s.id
WHERE t.id IS NULL;                                   -- socios SIN tarjeta

SELECT s.nombre AS socio, c.nombre AS actividad       -- N:M a través de la intermedia
FROM reservas AS r
JOIN socios AS s ON r.socio_id = s.id
JOIN clases AS c ON r.clase_id = c.id;
```

| JOIN | Devuelve |
|------|----------|
| `INNER JOIN` (`JOIN`) | Solo filas con pareja en ambas |
| `LEFT JOIN` | Todas las de la izquierda + pareja o NULL |
| `RIGHT JOIN` | Todas las de la derecha + pareja o NULL |
| `FULL JOIN` | Todas de ambas, NULL donde falte |

---

## 6. Agregados, GROUP BY, HAVING y subconsultas

```sql
SELECT COUNT(*) FROM socios;
SELECT AVG(precio_mensual), MIN(precio_mensual), MAX(precio_mensual), SUM(precio_mensual) FROM planes;
SELECT COUNT(DISTINCT sala) FROM clases;

SELECT p.nombre AS plan, COUNT(s.id) AS socios        -- COUNT(col) no cuenta NULL → 0 con LEFT JOIN
FROM planes p
LEFT JOIN socios s ON s.plan_id = p.id
GROUP BY p.id, p.nombre                                -- toda columna no agregada va en GROUP BY
HAVING COUNT(s.id) > 2                                 -- filtra GRUPOS (WHERE filtra filas)
ORDER BY socios DESC;

SELECT category, COUNT(*) AS count, ROUND(AVG(price), 2) AS "averagePrice"
FROM listings GROUP BY category ORDER BY category;
```

```
Orden lógico: FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

```sql
SELECT nombre FROM planes
WHERE precio_mensual > (SELECT AVG(precio_mensual) FROM planes);   -- subconsulta escalar

SELECT nombre FROM socios WHERE id IN (SELECT socio_id FROM reservas WHERE clase_id = 1);

SELECT s.nombre FROM socios s
WHERE NOT EXISTS (SELECT 1 FROM reservas r WHERE r.socio_id = s.id);   -- sin reservas

WITH ocupacion AS (                                    -- CTE: subconsulta con nombre
  SELECT c.id, c.nombre, c.plazas, COUNT(r.id) AS reservas
  FROM clases c LEFT JOIN reservas r ON r.clase_id = c.id
  GROUP BY c.id, c.nombre, c.plazas
)
SELECT * FROM ocupacion WHERE reservas >= plazas;
```

---

## 7. Índices y transacciones

```sql
CREATE INDEX idx_socios_plan_id ON socios (plan_id);              -- las FK NO llevan índice automático
CREATE INDEX idx_clases_sala_fecha ON clases (sala, fecha_hora);  -- compuesto (el orden importa)
CREATE UNIQUE INDEX idx_tarjetas_codigo ON tarjetas (codigo);
DROP INDEX idx_socios_plan_id;
EXPLAIN SELECT * FROM socios WHERE email = 'ana@gym.com';          -- Index Scan vs Seq Scan
EXPLAIN ANALYZE SELECT * FROM socios WHERE nombre = 'Ana Pérez';   -- y lo ejecuta

BEGIN;                                                 -- todo o nada
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;
COMMIT;                                                -- o ROLLBACK; para deshacer
```

> PK y UNIQUE crean índice solos (`socios_pkey`, `socios_email_key`).

---

## 8. Postgres desde Node (`pg`)

```bash
npm install pg
npm install -D @types/pg
```

```
# .env
DATABASE_URL=postgres://alumno:<PASSWORD>@localhost:5432/academia
```

```ts
// db.ts — UN Pool para toda la app
import { Pool } from 'pg';
export const pool = new Pool({ connectionString: process.env.DATABASE_URL });

const result = await pool.query('SELECT id, title FROM books WHERE id = $1', [3]);  // $1 ← 3
result.rows        // array de filas ([] si no hay)
result.rows[0]     // primera fila o undefined
result.rowCount    // filas afectadas (UPDATE/DELETE)
```

```ts
// INSERT parametrizado + RETURNING
const { rows } = await pool.query<BookRow>(
  `INSERT INTO books (title, author, pages) VALUES ($1, $2, $3)
   RETURNING id, title, author, pages, created_at`,
  [dto.title, dto.author, dto.pages],
);

// ❌ NUNCA concatenar: inyección SQL (' OR '1'='1)
pool.query(`SELECT * FROM users WHERE email = '${email}'`);
```

```sql
-- Filtros opcionales con un solo SQL
WHERE ($1::text IS NULL OR author ILIKE '%' || $1 || '%')
  AND ($2::int IS NULL OR pages <= $2)
```

```ts
// NestJS: Pool inyectable
@Module({
  providers: [{ provide: Pool, useFactory: () => new Pool({ connectionString: process.env.DATABASE_URL }) }],
  exports: [Pool],
})
export class DatabaseModule {}
```

```ts
// Errores de restricciones → código HTTP
catch (error: any) {
  if (error.code === '23505') throw new ConflictException('Ese email ya está registrado');  // unique → 409
  if (error.code === '23503') throw new BadRequestException('La referencia no existe');     // FK → 400
  throw error;                                                                              // → 500
}
```

```ts
// Transacción: misma conexión para todo
const client = await pool.connect();
try {
  await client.query('BEGIN');
  await client.query('UPDATE offers SET status = $1 WHERE id = $2', ['ACCEPTED', offerId]);
  await client.query('UPDATE listings SET status = $1 WHERE id = $2', ['RESERVED', listingId]);
  await client.query('COMMIT');
} catch (e) {
  await client.query('ROLLBACK');
  throw e;
} finally {
  client.release();                                    // ⚠️ siempre
}
```

```ts
// snake_case (BD) → camelCase (API)
export function toBook(row: BookRow): Book {
  return { id: row.id, title: row.title, author: row.author, pages: row.pages, createdAt: row.created_at };
}
```

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `port is already allocated` (5432) | `docker compose down` del otro proyecto o mapear `"5433:5432"` |
| `violates foreign key constraint` / código `23503` | El id referenciado no existe, o borras un padre con hijos |
| `duplicate key value violates unique constraint` / `23505` | Valor repetido → responder 409 |
| `violates check constraint` | El valor no cumple el `CHECK` |
| `must appear in the GROUP BY clause` | Añadir la columna al `GROUP BY` o agregarla |
| `aggregate functions are not allowed in WHERE` | Usar `HAVING` |
| `WHERE col = NULL` no devuelve nada | `IS NULL` / `IS NOT NULL` |
| JOIN con muchísimas filas | Faltaba el `ON` (producto cartesiano) |
