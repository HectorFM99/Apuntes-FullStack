# 04 — DML: INSERT, SELECT, UPDATE, DELETE

> **DML** (*Data Manipulation Language*) son las sentencias que trabajan con los
> **datos**. Son las 4 operaciones del famoso **CRUD**:
>
> | CRUD | SQL | HTTP (tu API) |
> |------|-----|---------------|
> | **C**reate | `INSERT` | `POST` |
> | **R**ead | `SELECT` | `GET` |
> | **U**pdate | `UPDATE` | `PUT` / `PATCH` |
> | **D**elete | `DELETE` | `DELETE` |

---

## 4.1 `INSERT`: meter filas

```sql
INSERT INTO tabla (columna1, columna2, ...)
VALUES (valor1, valor2, ...);
```

**Puntos clave:**
- Las columnas y los valores van en el **mismo orden**.
- Las columnas con `IDENTITY` o `DEFAULT` se pueden **omitir**.
- Texto y fechas entre **comillas simples**; números y `TRUE/FALSE` sin comillas.

### Ejemplo REAL copiado (`Proyecto-SQL/Proyecto-SQL/gimnasio.sql`)

```sql
--B2.1
INSERT INTO planes (nombre, precio_mensual)
VALUES
    ('Básico', 29.90),
    ('Completo', 44.90),
    ('Estudiante', 24.90),
    ('Mañanas', 19.90);
```

> 📌 **Varias filas en un solo `INSERT`**: separa cada paréntesis con comas. Es
> mucho más rápido que 4 `INSERT` sueltos.

```sql
--B2.2
INSERT INTO socios (dni, nombre, email, fecha_alta, plan_id)
VALUES
    ('11111111A', 'Ana Pérez', 'ana@gym.com', '2026-01-10', 2),
    ('22222222B', 'Luis García', 'luis@gym.com', '2026-02-03', 1),
    ('33333333C', 'Marta López', 'marta@gym.com', '2026-03-15', 3),
    ...
```

```sql
--B2.3
INSERT INTO tarjetas (codigo, activa, socio_id)
VALUES
    ('TJ-0001', TRUE, 1),
    ('TJ-0002', TRUE, 2),
    ...
    ('TJ-0004', FALSE, 4),
```

> 📌 El orden de las columnas lo eliges **tú**, no tiene que ser el del `CREATE`.
> En el B2.5 pusiste `(nombre, sala, fecha_hora, ...)` aunque en la tabla
> `fecha_hora` va antes que `sala`. Lo importante es que **cuadren** con `VALUES`.

### Omitir columnas opcionales

**Ejemplo REAL (`gimnasio.sql`, B2.4):** no se dan `telefono` ni `especialidad` → quedan `NULL`.

```sql
INSERT INTO monitores (dni, nombre, email)
VALUES
    ('90000001J', 'Carla Navarro', 'carla@gym.com'),
    ...
```

### `INSERT ... SELECT`: insertar a partir de una consulta

**Ejemplo REAL (`gimnasio.sql`, B5.3 y B5.4):**

```sql
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
```

> 📌 En vez de **adivinar** que Diego tiene el `id = 8`, se **busca** por su email.
> Si cambian los ids, el script sigue funcionando. ¡Muy buena práctica!

### `RETURNING`: que el INSERT te devuelva la fila

**Ejemplo REAL (`puta/api-books-nest/src/books/books.service.ts`):**

```sql
INSERT INTO books (title, author, pages)
VALUES ($1, $2, $3)
RETURNING id, title, author, pages, created_at
```

> 📌 `RETURNING` es propio de Postgres y es **oro** para una API: te devuelve el
> `id` y el `created_at` que ha generado la BD, **sin** hacer un segundo `SELECT`.
> Funciona también con `UPDATE` y `DELETE`.

---

## 4.2 `SELECT`: leer datos

```sql
SELECT columnas
FROM tabla
WHERE condición
ORDER BY columna
LIMIT n;
```

```sql
SELECT * FROM socios;                    -- todas las columnas
SELECT nombre, email FROM socios;        -- solo algunas
```

> ⚠️ `SELECT *` es cómodo en `psql` para mirar, pero en el código de tu API pide
> **las columnas que necesitas**, como haces en `books.service.ts`:
> `SELECT id, title, author, pages, created_at FROM books`. Así no envías datos que
> no tocan (¡imagina un `SELECT *` de `users` devolviendo el `password_hash`!).

### Alias con `AS`

```sql
SELECT nombre AS socio, fecha_alta AS "Fecha de alta"
FROM socios AS s;
```

- Alias de **columna**: cambia el nombre en el resultado.
- Alias de **tabla** (`socios AS s`): para escribir menos en los `JOIN` (`s.nombre`).
- Comillas **dobles** si el alias tiene espacios o mayúsculas.

### Orden **lógico** de ejecución (no es el orden en que se escribe)

```
 Escribes:                   Postgres ejecuta:
 1. SELECT                   1. FROM / JOIN    ← de qué tablas
 2. FROM                     2. WHERE          ← filtra filas
 3. WHERE                    3. GROUP BY       ← agrupa
 4. GROUP BY                 4. HAVING         ← filtra grupos
 5. HAVING                   5. SELECT         ← elige columnas / calcula
 6. ORDER BY                 6. ORDER BY       ← ordena
 7. LIMIT                    7. LIMIT          ← recorta
```

> 📌 Por eso **no** puedes usar en el `WHERE` un alias definido en el `SELECT`
> (aún no existe), pero **sí** en el `ORDER BY`.

---

## 4.3 `WHERE`: filtrar filas

### Operadores de comparación

| Operador | Significado | Ejemplo |
|----------|-------------|---------|
| `=` | igual (¡uno solo, no `==`!) | `WHERE clase_id = 1` |
| `<>` o `!=` | distinto | `WHERE sala <> 'Sala 1'` |
| `>` `<` `>=` `<=` | mayor / menor | `WHERE plazas > 10` |
| `IS NULL` / `IS NOT NULL` | vacío / no vacío | `WHERE t.id IS NULL` |

### Operadores lógicos

```sql
-- AND: se cumplen las dos
SELECT * FROM clases WHERE sala = 'Sala 2' AND plazas >= 10;

-- OR: se cumple alguna
SELECT * FROM clases WHERE nombre = 'Yoga' OR nombre = 'Pilates';

-- NOT
SELECT * FROM tarjetas WHERE NOT activa;
```

**Ejemplo REAL (`gimnasio.sql`, B5.5) — `AND` en un `DELETE`:**

```sql
DELETE FROM reservas
WHERE socio_id = (SELECT id FROM socios WHERE email = 'luis@gym.com')
  AND clase_id = 2;
```

> ⚠️ Cuidado al mezclar `AND` y `OR`: `AND` va **primero** (como `*` antes que `+`).
> Usa **paréntesis** para no liarte:
> ```sql
> WHERE sala = 'Sala 2' AND (nombre = 'Yoga' OR nombre = 'Pilates')
> ```

### `IN`: está en una lista

```sql
-- ❌ Largo
WHERE nombre = 'Yoga' OR nombre = 'Pilates' OR nombre = 'Zumba'

-- ✅ Corto
WHERE nombre IN ('Yoga', 'Pilates', 'Zumba')

WHERE nombre NOT IN ('Crossfit')
```

**Ejemplo REAL (`api-books-nest/database/schema.sql`)** — `IN` dentro de un `CHECK`:

```sql
CHECK (role IN ('customer', 'employee'))
```

### `BETWEEN`: entre dos valores (ambos incluidos)

```sql
SELECT * FROM planes WHERE precio_mensual BETWEEN 20 AND 30;
-- equivale a: precio_mensual >= 20 AND precio_mensual <= 30

SELECT * FROM socios WHERE fecha_alta BETWEEN '2026-03-01' AND '2026-06-30';
```

> ⚠️ Con `TIMESTAMP`, `BETWEEN '2026-11-02' AND '2026-11-03'` corta en
> `2026-11-03 00:00`, **no** incluye el resto del día 3. Para rangos de fecha-hora
> es más seguro: `fecha_hora >= '2026-11-02' AND fecha_hora < '2026-11-04'`.

### `LIKE` e `ILIKE`: buscar texto con patrones

| Comodín | Significado |
|---------|-------------|
| `%` | cualquier cantidad de caracteres (incluido ninguno) |
| `_` | exactamente un carácter |

```sql
SELECT * FROM socios WHERE email LIKE '%@gym.com';   -- termina en @gym.com
SELECT * FROM socios WHERE nombre LIKE 'L%';         -- empieza por L
SELECT * FROM clases WHERE nombre LIKE '%ga%';       -- contiene "ga"
SELECT * FROM tarjetas WHERE codigo LIKE 'TJ-000_';  -- TJ-0001 ... TJ-0009
```

- `LIKE` distingue **mayúsculas** (`'yoga'` ≠ `'Yoga'`).
- `ILIKE` (propio de Postgres) **no** las distingue.

**Ejemplo REAL (`puta/api-books-nest/src/books/books.service.ts`):**

```sql
WHERE ($1::text IS NULL OR author ILIKE '%' || $1 || '%')
```

```
'%' || $1 || '%'      → el operador || concatena texto en SQL
si $1 = 'garcía'      → '%garcía%'  → encuentra "Gabriel García Márquez"
```

---

## 4.4 `ORDER BY`: ordenar

```sql
ORDER BY columna            -- ascendente (ASC, por defecto)
ORDER BY columna DESC       -- descendente
ORDER BY col1, col2         -- por col1; si empatan, por col2
```

**Ejemplo REAL (`gimnasio.sql`, B4.1, B4.2 y B4.10):**

```sql
ORDER BY s.nombre;
ORDER BY c.fecha_hora;
ORDER BY m.nombre, e.nombre;
```

> 📌 **Sin `ORDER BY`, el orden NO está garantizado.** Aunque "parezca" que salen
> por `id`, Postgres puede devolverlas en cualquier orden. Por eso tu
> `books.service.ts` pone `ORDER BY id` en el `findAll`.

---

## 4.5 `LIMIT` y `OFFSET`: paginar

No aparece en tus proyectos. Ejemplo propio:

```sql
SELECT id, title FROM books ORDER BY id LIMIT 10;              -- página 1
SELECT id, title FROM books ORDER BY id LIMIT 10 OFFSET 10;    -- página 2
SELECT id, title FROM books ORDER BY id LIMIT 10 OFFSET 20;    -- página 3
-- OFFSET = (página - 1) * tamaño
```

```sql
-- Top 3 planes más caros
SELECT nombre, precio_mensual
FROM planes
ORDER BY precio_mensual DESC
LIMIT 3;
```

> ⚠️ `LIMIT` **sin** `ORDER BY` = "dame 10 cualquiera". Siempre van juntos.

### `DISTINCT`: quitar repetidos

```sql
SELECT DISTINCT sala FROM clases;       -- Sala 1, Sala 2, Sala 3 (sin repetir)
```

---

## 4.6 Expresiones y funciones útiles en el `SELECT`

```sql
SELECT nombre, precio_mensual * 12 AS precio_anual FROM planes;
SELECT UPPER(nombre), LOWER(email), LENGTH(dni) FROM socios;
SELECT nombre || ' <' || email || '>' AS contacto FROM socios;
SELECT NOW(), CURRENT_DATE;
SELECT EXTRACT(MONTH FROM fecha_alta) AS mes FROM socios;
SELECT ROUND(AVG(precio_mensual), 2) FROM planes;
SELECT COALESCE(telefono, 'sin teléfono') FROM monitores;
```

### `COALESCE`: el primer valor que no sea NULL

**Ejemplo REAL (`books.service.ts`, método `update`):**

```sql
UPDATE books
SET title = COALESCE($1, title),
    author = COALESCE($2, author),
    pages = COALESCE($3, pages)
WHERE id = $4
RETURNING id, title, author, pages, created_at
```

```
COALESCE($1, title)
  si $1 = 'Nuevo título'  → 'Nuevo título'
  si $1 = NULL            → title (se queda el que había)
```

> 📌 Truco perfecto para un `PATCH`: el cliente manda **solo** los campos que quiere
> cambiar, y los que no manda (`null`) se quedan como estaban.

### `CASE`: un "if" dentro de SQL (ejemplo propio)

```sql
SELECT nombre,
       CASE
         WHEN precio_mensual < 25 THEN 'barato'
         WHEN precio_mensual < 40 THEN 'medio'
         ELSE 'caro'
       END AS categoria
FROM planes;
```

---

## 4.7 `UPDATE`: modificar filas

```sql
UPDATE tabla
SET columna1 = valor1, columna2 = valor2
WHERE condición;
```

**Ejemplo REAL (`gimnasio.sql`, B5.2):**

```sql
UPDATE tarjetas
SET activa = TRUE
WHERE codigo = 'TJ-0004';
```

**Ejemplo REAL (`gimnasio.sql`, B5.1)** — el nuevo valor sale de una **subconsulta**:

```sql
UPDATE socios
SET plan_id = (
    SELECT id
    FROM planes
    WHERE nombre = 'Completo'
)
WHERE email = 'luis@gym.com';
```

> 🚨 **EL ERROR MÁS PELIGROSO DE SQL:**
> ```sql
> -- ❌ Sin WHERE → TODAS las tarjetas pasan a activa
> UPDATE tarjetas SET activa = TRUE;
>
> -- ✅ Con WHERE → solo la que quieres
> UPDATE tarjetas SET activa = TRUE WHERE codigo = 'TJ-0004';
> ```
> 📌 Truco: escribe **primero** el `SELECT ... WHERE ...` para ver qué filas salen,
> y cuando estés seguro, cámbialo por `UPDATE` / `DELETE` con el **mismo** `WHERE`.

```sql
-- Actualizar usando el valor actual (ejemplo propio)
UPDATE planes SET precio_mensual = precio_mensual * 1.05;   -- subida del 5% a todos
```

---

## 4.8 `DELETE`: borrar filas

```sql
DELETE FROM tabla WHERE condición;
```

**Ejemplo REAL (`api-books-nest/src/books/books.service.ts`):**

```sql
DELETE FROM books WHERE id = $1
```

**Ejemplo REAL (`gimnasio.sql`, B5.6 → B5.8)** — el orden importa por las FK:

```sql
--B5.6
DELETE FROM socios
WHERE email = 'pablo@gym.com';
-- ❌ Falla: Pablo tiene reservas que apuntan a él

--B5.7
DELETE FROM reservas
WHERE socio_id = (SELECT id FROM socios WHERE email = 'pablo@gym.com');
-- ✅ Primero borras los "hijos"

--B5.8
DELETE FROM socios
WHERE email = 'pablo@gym.com';
```

> ⚠️ **Ojo con un detalle:** Pablo (socio 6) **también tiene tarjeta** (`TJ-0006`
> en el B2.3). Así que el B5.8 todavía choca con la FK de `tarjetas`. Para borrarlo
> del todo hay que borrar antes su tarjeta:
> ```sql
> DELETE FROM tarjetas
> WHERE socio_id = (SELECT id FROM socios WHERE email = 'pablo@gym.com');
> ```
> (o haber definido las FK con `ON DELETE CASCADE`, ver archivo `03`).

> 🚨 Igual que el `UPDATE`: `DELETE FROM reservas;` **sin `WHERE` borra todo**.

### `DELETE ... RETURNING` (ejemplo propio)

```sql
DELETE FROM books WHERE id = 3 RETURNING *;   -- borra y te enseña lo que ha borrado
```

> 📌 En tu `books.service.ts`, en vez de `RETURNING`, se hace un `findOne` **antes**
> de borrar para devolver el libro (y dar 404 si no existe). Con `RETURNING` lo
> podrías hacer en **una sola** consulta y comprobar `result.rows.length === 0`.

---

## 4.9 Errores comunes con DML

| Error | Causa | Solución |
|-------|-------|----------|
| `column "ana" does not exist` | Usaste comillas **dobles** para un valor: `"ana"` | Comillas **simples**: `'ana'` |
| `INSERT has more expressions than target columns` | Más valores que columnas | Cuenta columnas y valores |
| `invalid input syntax for type integer` | Texto en columna numérica | Revisa el tipo |
| Actualizaste/borraste todo | `UPDATE`/`DELETE` sin `WHERE` | Haz antes el `SELECT` con ese `WHERE` |
| `WHERE x = NULL` no devuelve nada | `NULL` no se compara con `=` | `IS NULL` |
| Resultados en orden "raro" | Falta `ORDER BY` | Añádelo siempre que el orden importe |

---

## ✅ Resumen rápido

1. `INSERT INTO t (cols) VALUES (...), (...);` — omite `id` y columnas con `DEFAULT`.
2. `INSERT ... SELECT` para insertar buscando ids por otro dato (email).
3. `SELECT cols FROM t WHERE ... ORDER BY ... LIMIT ...;`
4. Filtros: `=`, `<>`, `AND`, `OR`, `IN (...)`, `BETWEEN a AND b`, `LIKE '%x%'`, `ILIKE`, `IS NULL`.
5. Sin `ORDER BY` no hay orden garantizado.
6. `UPDATE t SET col = valor WHERE ...;` · `DELETE FROM t WHERE ...;` → **¡siempre con `WHERE`!**
7. `RETURNING` te devuelve las filas afectadas (ideal para APIs).
8. `COALESCE(a, b)` → primer valor no nulo (truco del `PATCH`).

👉 **Siguiente:** `05-joins-agregados-y-subconsultas.md`
