# 🐘 CARPETA POSTGRESQL — ÍNDICE DE ESTUDIO

Teoría de **bases de datos relacionales con PostgreSQL** explicada con ejemplos
reales **copiados y pegados de tus proyectos** del Bootcamp.

> ⚠️ Los proyectos **NO** se han modificado. Solo se han copiado fragmentos de su código.
> 🔒 Las **contraseñas reales** de los `compose.yaml` se han sustituido por
> `TU_PASSWORD` y **no** se ha copiado ningún archivo `.env`.

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a aprender |
|---|---------|--------------------|
| 1 | `01-modelo-relacional-y-relaciones.md` | Tablas, filas, columnas, PK, FK, relaciones 1:1 / 1:N / N:M, tabla intermedia |
| 2 | `02-docker-psql-y-tipos-de-datos.md` | Levantar Postgres con Docker Compose, conectarte con `psql`, comandos `\dt` `\d`, tipos de datos |
| 3 | `03-ddl-crear-tablas-y-restricciones.md` | `CREATE TABLE`, `PRIMARY KEY`, `REFERENCES`, `NOT NULL`, `UNIQUE`, `CHECK`, `DEFAULT`, `ALTER`, `DROP` |
| 4 | `04-dml-insert-select-update-delete.md` | `INSERT`, `SELECT`, `WHERE`, `ORDER BY`, `LIMIT`, `LIKE`, `IN`, `BETWEEN`, `UPDATE`, `DELETE`, `RETURNING` |
| 5 | `05-joins-agregados-y-subconsultas.md` | `JOIN` (INNER/LEFT/RIGHT/FULL) con diagramas, `COUNT`/`SUM`/`AVG`, `GROUP BY`, `HAVING`, subconsultas |
| 6 | `06-indices-normalizacion-y-transacciones.md` | Índices, formas normales, transacciones `BEGIN`/`COMMIT`/`ROLLBACK`, ACID |
| 7 | `07-postgres-desde-node-con-pg.md` | Librería `pg`, `Pool`, consultas parametrizadas `$1`, inyección SQL, ORMs |
| 8 | `08-ejemplos-completos-de-proyectos.md` | Archivos SQL y TS enteros copiados de tus proyectos |

---

## 🗂️ Proyectos de los que se han extraído los ejemplos

| Proyecto | Archivo | Temas que cubre |
|----------|---------|-----------------|
| `Proyecto-SQL/Proyecto-SQL/` | `gimnasio.sql` | **Todo el SQL**: `CREATE TABLE`, restricciones, `INSERT`, errores de FK/UNIQUE, `JOIN`, `GROUP BY`, `UPDATE`, `DELETE`, subconsultas |
| `Proyecto-SQL/Proyecto-SQL/` | `respuestas.md` | Teoría del modelo relacional: PK vs UNIQUE, dónde va la FK, 1:1, 1:N, N:M |
| `Proyecto-SQL/` | `consulta-reservas.html` | Una consulta con doble `JOIN` presentada en HTML |
| `practica-sql/` | `compose.yaml` | Postgres 17 con Docker (base de datos `academia`) |
| `pruebas/` | `compose.yaml` | Postgres 17 con Docker (base de datos `library`) |
| `puta/api-books-nest/` | `compose.yaml` | Postgres con Docker + variable `DATABASE_URL` |
| `puta/api-books-nest/` | `database/schema.sql` | Tablas `books` y `users` con `DEFAULT NOW()`, `CHECK ... IN (...)` |
| `puta/api-books-nest/` | `src/database/*.ts` | `Pool` de la librería `pg` |
| `puta/api-books-nest/` | `src/books/books.service.ts` | CRUD completo con SQL parametrizado (`$1`, `$2`...), `RETURNING`, `COALESCE`, `ILIKE` |
| `puta/api-books-nest/` | `src/users/users.service.ts` | Buscar/crear usuarios guardando el **hash** de la contraseña |
| `api-rastro/api-rastro/` | `src/db.ts` | ⚠️ Usa **MongoDB**, no Postgres. Se usa solo como **comparación** SQL vs NoSQL |

---

## 🚀 Orden de estudio recomendado

1. `01` → entender **cómo se piensa** en tablas y relaciones (sin esto el SQL no tiene sentido).
2. `02` → tener Postgres **funcionando** en tu máquina y saber qué tipos de datos hay.
3. `03` → **crear** tablas con sus reglas (DDL).
4. `04` → **meter, leer, cambiar y borrar** datos (DML).
5. `05` → **cruzar tablas** (`JOIN`) y **resumir** datos (`GROUP BY`): lo que más se pregunta en exámenes.
6. `06` → conceptos de "nivel siguiente": rendimiento, diseño y seguridad de los datos.
7. `07` → usar todo lo anterior **desde tu API** en Node/NestJS.
8. `08` → código real completo de tus proyectos para repasar de un tirón.

---

## 💡 Las dos "familias" de SQL

```sql
-- DDL (Data Definition Language): define la ESTRUCTURA
CREATE TABLE planes (id INTEGER PRIMARY KEY, nombre VARCHAR(50));
ALTER TABLE planes ADD COLUMN activo BOOLEAN;
DROP TABLE planes;
```
```sql
-- DML (Data Manipulation Language): trabaja con los DATOS
INSERT INTO planes (id, nombre) VALUES (1, 'Básico');
SELECT * FROM planes;
UPDATE planes SET nombre = 'Básico+' WHERE id = 1;
DELETE FROM planes WHERE id = 1;
```
```ts
// Y desde tu API (Node + pg): el mismo SQL, enviado como texto
const result = await pool.query('SELECT * FROM planes WHERE id = $1', [1]);
```

> 📌 SQL **no distingue mayúsculas** en las palabras clave (`select` = `SELECT`),
> pero por convención se escriben en **MAYÚSCULAS** y los nombres de tablas y
> columnas en **minúsculas_con_guiones_bajos** (`snake_case`).
