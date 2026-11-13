# 05 — JOINS, AGREGADOS (GROUP BY) Y SUBCONSULTAS

> En el archivo `01` separaste los datos en varias tablas para no repetir
> información. Ahora toca **volver a juntarlos** para responder preguntas como
> "¿qué plan tiene cada socio?" → eso es un **`JOIN`**. Y para preguntas como
> "¿cuántos socios hay en cada plan?" → **funciones de agregado** + **`GROUP BY`**.

---

## 5.1 La idea del JOIN

Un `JOIN` combina filas de dos tablas **emparejándolas** por una condición
(casi siempre `FK = PK`):

```
socios                              planes
┌────┬─────────────┬─────────┐      ┌────┬────────────┬───────┐
│ id │ nombre      │ plan_id │      │ id │ nombre     │ precio│
├────┼─────────────┼─────────┤      ├────┼────────────┼───────┤
│ 1  │ Ana Pérez   │ 2 ──────┼──┐   │ 1  │ Básico     │ 29.90 │
│ 2  │ Luis García │ 1 ──────┼──┼──►│ 2  │ Completo   │ 44.90 │
│ 3  │ Marta López │ 3 ──────┼──┼─┐ │ 3  │ Estudiante │ 24.90 │
└────┴─────────────┴─────────┘  │ └►│ 4  │ Mañanas    │ 19.90 │
                                └──►└────┴────────────┴───────┘
                     ON s.plan_id = p.id
                              │
                              ▼
          ┌─────────────┬────────────┬───────┐
          │ s.nombre    │ plan       │ precio│
          ├─────────────┼────────────┼───────┤
          │ Ana Pérez   │ Completo   │ 44.90 │
          │ Luis García │ Básico     │ 29.90 │
          │ Marta López │ Estudiante │ 24.90 │
          └─────────────┴────────────┴───────┘
```

### Ejemplo REAL copiado (`Proyecto-SQL/Proyecto-SQL/gimnasio.sql`, B4.1)

```sql
SELECT s.nombre, p.nombre AS plan, p.precio_mensual
FROM socios AS s
JOIN planes AS p ON s.plan_id = p.id
ORDER BY s.nombre;
```

**Desglose:**

```
FROM socios AS s               → tabla de partida, con alias "s"
JOIN planes AS p               → tabla que se une, con alias "p"
ON s.plan_id = p.id            → CÓMO se emparejan (FK = PK)
s.nombre, p.nombre AS plan     → como las dos tienen "nombre", se usa
                                 el alias delante para no confundirlas
```

> 📌 `JOIN` a secas = `INNER JOIN`. Son lo mismo.
> ⚠️ Si dos tablas tienen una columna con el mismo nombre (`nombre`, `id`) y no
> pones el alias delante → `ERROR: column reference "nombre" is ambiguous`.

---

## 5.2 Los tipos de JOIN (con diagramas)

Usaremos `socios` (A) y `tarjetas` (B). En tus datos, **Diego** (socio 8) **no tiene tarjeta**.

```
     INNER JOIN               LEFT JOIN               RIGHT JOIN              FULL JOIN
   ┌─────┬─────┐           ┌─────┬─────┐           ┌─────┬─────┐           ┌─────┬─────┐
   │  A  │█████│  B        │█████│█████│  B        │  A  │█████│████       │█████│█████│████
   │     │█████│           │█████│█████│           │     │█████│████       │█████│█████│████
   └─────┴─────┘           └─────┴─────┘           └─────┴─────┘           └─────┴─────┘
   solo los que            TODOS los de A          TODOS los de B          TODOS de A y de B
   tienen pareja           + su pareja o NULL      + su pareja o NULL      + NULL donde falte
```

| Tipo | Devuelve | Si no hay pareja |
|------|----------|------------------|
| `INNER JOIN` | Solo filas **con** pareja en las dos tablas | La fila **desaparece** |
| `LEFT JOIN` | **Todas** las de la izquierda (la del `FROM`) | Columnas de la derecha a `NULL` |
| `RIGHT JOIN` | **Todas** las de la derecha | Columnas de la izquierda a `NULL` |
| `FULL JOIN` | **Todas** las de las dos | `NULL` en el lado que falte |
| `CROSS JOIN` | **Todas** las combinaciones posibles (A × B) | — (no lleva `ON`) |

### INNER JOIN: solo los emparejados

```sql
SELECT s.nombre, t.codigo
FROM socios AS s
JOIN tarjetas AS t ON t.socio_id = s.id;
```
```
 nombre       │ codigo
──────────────┼─────────
 Ana Pérez    │ TJ-0001
 ...          │ ...
 Sara Gil     │ TJ-0007
                          ← Diego NO aparece (no tiene tarjeta)
```

### LEFT JOIN: todos los de la izquierda

**Ejemplo REAL (`gimnasio.sql`, B4.3):**

```sql
SELECT s.nombre, t.codigo
FROM socios AS s
LEFT JOIN tarjetas AS t ON t.socio_id = s.id;
```
```
 nombre       │ codigo
──────────────┼─────────
 Ana Pérez    │ TJ-0001
 ...          │ ...
 Sara Gil     │ TJ-0007
 Diego Vidal  │ NULL      ← aparece, con NULL en lo de la derecha
```

### El patrón "los que NO tienen": `LEFT JOIN` + `IS NULL`

**Ejemplo REAL (`gimnasio.sql`, B4.4):** socios sin tarjeta

```sql
SELECT s.nombre
FROM socios AS s
LEFT JOIN tarjetas AS t ON t.socio_id = s.id
WHERE t.id IS NULL;
```
→ `Diego Vidal`

**Ejemplo REAL (`gimnasio.sql`, B4.9):** monitores que no dan ninguna clase

```sql
SELECT m.nombre
FROM monitores AS m
LEFT JOIN clases AS c ON c.monitor_id = m.id
WHERE c.id IS NULL;
```
→ `Rubén Castro` (las clases solo usan los monitores 1, 2 y 3)

> 📌 Este patrón es **pregunta fija de examen**: "¿qué X no tienen ningún Y?" →
> `LEFT JOIN Y` + `WHERE Y.id IS NULL`.

### RIGHT JOIN (ejemplo propio)

```sql
-- Equivalente al B4.3, escrito al revés
SELECT s.nombre, t.codigo
FROM tarjetas AS t
RIGHT JOIN socios AS s ON t.socio_id = s.id;
```

> 📌 Todo `RIGHT JOIN` se puede reescribir como `LEFT JOIN` cambiando el orden de
> las tablas. Casi todo el mundo usa **solo `LEFT`**, porque se lee mejor
> ("parto de los socios y les pego su tarjeta si la tienen").

### FULL JOIN (ejemplo propio)

```sql
-- Todos los monitores y todas las especialidades del campo "especialidad",
-- estén o no emparejados
SELECT m.nombre AS monitor, e.nombre AS especialidad
FROM monitores AS m
FULL JOIN especialidades AS e ON e.nombre = m.especialidad;
```

Útil para **comparar** dos listas y ver qué sobra de cada lado. Se usa poco.

---

## 5.3 Varios JOINs encadenados (tablas N:M)

Para recorrer una relación **N:M** pasas **por la tabla intermedia**:

```
socios ──< reservas >── clases
  s.id = r.socio_id    r.clase_id = c.id
```

### Ejemplo REAL copiado (`gimnasio.sql`, B4.6)

```sql
SELECT s.nombre AS socio, c.nombre AS actividad, c.fecha_hora
FROM reservas AS r
JOIN socios AS s ON r.socio_id = s.id
JOIN clases AS c ON r.clase_id = c.id;
```

Esta misma consulta la presentaste en `Proyecto-SQL/consulta-reservas.html`
(con colores para cada parte: palabras clave, alias, columnas y tablas).

> **B4.11** (tu respuesta en `respuestas.md`): *"No es un error. La consulta une
> reservas con socios y clases; Ana aparece una vez por cada reserva que tiene, así
> que sale tres veces porque tiene tres reservas."*

> 📌 Un `JOIN` produce **una fila por cada pareja**. Si Ana tiene 3 reservas,
> sale 3 veces. Es lo correcto.

### Ejemplo REAL copiado (`gimnasio.sql`, B4.10) — monitores y sus especialidades

```sql
SELECT m.nombre AS monitor, e.nombre AS especialidad
FROM monitores AS m
JOIN monitor_especialidades AS me ON me.monitor_id = m.id
JOIN especialidades AS e ON e.id = me.especialidad_id
ORDER BY m.nombre, e.nombre;
```

### JOIN + WHERE (`gimnasio.sql`, B4.7) — quién va a la clase 1

```sql
SELECT s.nombre
FROM reservas AS r
JOIN socios AS s ON r.socio_id = s.id
WHERE r.clase_id = 1;
```
→ Ana Pérez, Luis García, Hugo Sanz

### ❌ / ✅ JOIN sin condición

```sql
-- ❌ Te olvidas del ON (estilo antiguo con coma) → producto cartesiano:
--    8 socios × 4 planes = 32 filas sin sentido
SELECT s.nombre, p.nombre FROM socios s, planes p;

-- ✅
SELECT s.nombre, p.nombre FROM socios s JOIN planes p ON s.plan_id = p.id;
```

---

## 5.4 Funciones de agregado

Reducen **muchas filas a un solo valor**:

| Función | Qué calcula |
|---------|-------------|
| `COUNT(*)` | Número de filas |
| `COUNT(columna)` | Número de filas donde esa columna **no es NULL** |
| `COUNT(DISTINCT col)` | Número de valores distintos |
| `SUM(col)` | Suma |
| `AVG(col)` | Media |
| `MIN(col)` / `MAX(col)` | Mínimo / máximo |

```sql
SELECT COUNT(*) FROM socios;                    -- 8
SELECT AVG(precio_mensual) FROM planes;         -- 29.90
SELECT MIN(fecha_alta), MAX(fecha_alta) FROM socios;
SELECT COUNT(DISTINCT sala) FROM clases;        -- 3
```

**Ejemplo REAL (`puta/api-books-nest/src/books/books.service.ts`):**

```sql
SELECT COUNT(*)::text AS total FROM books
```

---

## 5.5 `GROUP BY`: un resultado por grupo

```
socios agrupados por plan_id:

  plan_id = 1 → [Luis, Pablo, Diego]   → COUNT = 3
  plan_id = 2 → [Ana, Hugo, Sara]      → COUNT = 3
  plan_id = 3 → [Marta, Lucía]         → COUNT = 2
```

### Ejemplo REAL copiado (`gimnasio.sql`, B4.5) — socios por plan

```sql
SELECT p.nombre AS plan, COUNT(s.id) AS cantidad_socios
FROM planes AS p
LEFT JOIN socios AS s ON s.plan_id = p.id
GROUP BY p.id, p.nombre
ORDER BY p.nombre;
```
```
 plan       │ cantidad_socios
────────────┼─────────────────
 Básico     │ 3
 Completo   │ 3
 Estudiante │ 2
 Mañanas    │ 0      ← sale gracias al LEFT JOIN (si no lo borraste en el B3.7)
```

**Dos detalles finos de esta consulta:**

1. **`LEFT JOIN`** → para que salgan también los planes con **0** socios. Con un
   `JOIN` normal, "Mañanas" desaparecería.
2. **`COUNT(s.id)` y no `COUNT(*)`** → en el plan sin socios, el `LEFT JOIN` genera
   una fila con `s.id = NULL`. `COUNT(*)` la contaría (daría **1**), `COUNT(s.id)`
   ignora los NULL (da **0**). ✅

### Ejemplo REAL copiado (`gimnasio.sql`, B4.8) — reservas por clase

```sql
SELECT c.nombre AS actividad, c.fecha_hora, c.plazas, COUNT(r.id) AS numero_reservas
FROM clases AS c
LEFT JOIN reservas AS r ON r.clase_id = c.id
GROUP BY c.id, c.nombre, c.fecha_hora, c.plazas
ORDER BY c.fecha_hora;
```

### La regla de oro del `GROUP BY`

> Toda columna del `SELECT` debe estar **en el `GROUP BY`** o **dentro de una
> función de agregado**.

```sql
-- ❌ ERROR: column "s.nombre" must appear in the GROUP BY clause
--          or be used in an aggregate function
SELECT p.nombre, s.nombre, COUNT(*)
FROM planes p JOIN socios s ON s.plan_id = p.id
GROUP BY p.nombre;
-- ¿Qué s.nombre pongo, si en el grupo "Básico" hay 3 socios?

-- ✅
SELECT p.nombre, COUNT(*)
FROM planes p JOIN socios s ON s.plan_id = p.id
GROUP BY p.nombre;
```

> 📌 En tus consultas agrupas por `p.id, p.nombre`. Agrupar por el **id** es buena
> práctica: si hubiera dos planes con el mismo nombre, no se mezclarían. (En
> Postgres, si agrupas por la PK, podrías incluso omitir el resto de columnas de
> esa tabla, pero ponerlas es más claro.)

### Comparación con Mongo (`api-rastro/api-rastro/src/routes/listings.ts`)

Lo que en tu API de MongoDB hacías con una *aggregation pipeline*:

```ts
.aggregate([
  { $group: { _id: "$category", count: { $sum: 1 }, averagePrice: { $avg: "$price" } } },
  { $project: { _id: 0, category: "$_id", count: 1, averagePrice: { $round: ["$averagePrice", 2] } } },
  { $sort: { category: 1 } },
])
```

en SQL sería (equivalente propio, suponiendo una tabla `listings`):

```sql
SELECT category, COUNT(*) AS count, ROUND(AVG(price), 2) AS "averagePrice"
FROM listings
GROUP BY category
ORDER BY category;
```

---

## 5.6 `HAVING`: filtrar **grupos**

- `WHERE` filtra **filas** (antes de agrupar).
- `HAVING` filtra **grupos** (después de agrupar) → puede usar `COUNT`, `SUM`...

No aparece en tus proyectos. Ejemplos propios con el gimnasio:

```sql
-- Planes con más de 2 socios
SELECT p.nombre, COUNT(s.id) AS socios
FROM planes p
JOIN socios s ON s.plan_id = p.id
GROUP BY p.id, p.nombre
HAVING COUNT(s.id) > 2;
```

```sql
-- Clases LLENAS (reservas >= plazas) → sale el Crossfit (3 plazas, 3 reservas)
SELECT c.nombre, c.fecha_hora, c.plazas, COUNT(r.id) AS reservas
FROM clases c
JOIN reservas r ON r.clase_id = c.id
GROUP BY c.id, c.nombre, c.fecha_hora, c.plazas
HAVING COUNT(r.id) >= c.plazas;
```

```sql
-- ❌ WHERE no puede usar agregados
SELECT plan_id, COUNT(*) FROM socios WHERE COUNT(*) > 2 GROUP BY plan_id;
-- ERROR: aggregate functions are not allowed in WHERE

-- ✅
SELECT plan_id, COUNT(*) FROM socios GROUP BY plan_id HAVING COUNT(*) > 2;
```

**Todo junto:**

```sql
SELECT   s.plan_id, COUNT(*) AS altas_2026
FROM     socios s
WHERE    s.fecha_alta >= '2026-01-01'   -- 1º filtra filas
GROUP BY s.plan_id                      -- 2º agrupa
HAVING   COUNT(*) >= 2                  -- 3º filtra grupos
ORDER BY altas_2026 DESC;               -- 4º ordena
```

---

## 5.7 Subconsultas

Una **subconsulta** es un `SELECT` **dentro** de otra sentencia, entre paréntesis.

### Subconsulta que devuelve **un valor** (escalar)

**Ejemplo REAL (`gimnasio.sql`, B5.1):**

```sql
UPDATE socios
SET plan_id = (
    SELECT id
    FROM planes
    WHERE nombre = 'Completo'
)
WHERE email = 'luis@gym.com';
```

**Ejemplo REAL (`gimnasio.sql`, B5.5 y B5.7):**

```sql
DELETE FROM reservas
WHERE socio_id = (SELECT id FROM socios WHERE email = 'luis@gym.com')
  AND clase_id = 2;
```

> ⚠️ Con `=` la subconsulta debe devolver **como mucho una fila**. Aquí es seguro
> porque `email` es `UNIQUE`. Si pudiera devolver varias:
> `ERROR: more than one row returned by a subquery used as an expression` → usa `IN`.

### Subconsulta con `IN` (lista de valores) — ejemplo propio

```sql
-- Socios que han reservado alguna clase de Yoga
SELECT nombre
FROM socios
WHERE id IN (
    SELECT r.socio_id
    FROM reservas r
    JOIN clases c ON c.id = r.clase_id
    WHERE c.nombre = 'Yoga'
);
```

### Subconsulta con `EXISTS` / `NOT EXISTS` — ejemplo propio

```sql
-- Socios sin ninguna reserva (alternativa al LEFT JOIN + IS NULL)
SELECT s.nombre
FROM socios s
WHERE NOT EXISTS (
    SELECT 1 FROM reservas r WHERE r.socio_id = s.id
);
```

### Subconsulta comparando con un agregado — ejemplo propio

```sql
-- Planes más caros que la media
SELECT nombre, precio_mensual
FROM planes
WHERE precio_mensual > (SELECT AVG(precio_mensual) FROM planes);
```

### Subconsulta en el `FROM` y `WITH` (CTE) — ejemplo propio

```sql
-- Con WITH le das NOMBRE a una subconsulta: se lee de arriba abajo
WITH ocupacion AS (
    SELECT c.id, c.nombre, c.plazas, COUNT(r.id) AS reservas
    FROM clases c
    LEFT JOIN reservas r ON r.clase_id = c.id
    GROUP BY c.id, c.nombre, c.plazas
)
SELECT nombre, reservas, plazas,
       ROUND(100.0 * reservas / plazas, 1) AS porcentaje
FROM ocupacion
ORDER BY porcentaje DESC;
```

> 📌 ¿JOIN o subconsulta? Muchas veces se puede hacer de las dos formas. Usa la
> que **se lea mejor**. Para "los que no tienen" valen igual `LEFT JOIN ... IS NULL`
> y `NOT EXISTS`.

---

## ✅ Resumen rápido

1. `JOIN t2 ON t1.fk = t2.id` → une filas emparejadas (= `INNER JOIN`).
2. `LEFT JOIN` → todas las de la izquierda, `NULL` si no hay pareja.
3. "Los X sin Y" → `LEFT JOIN Y ... WHERE Y.id IS NULL`.
4. N:M → dos `JOIN` pasando por la tabla intermedia.
5. Agregados: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`. `COUNT(col)` ignora `NULL`.
6. `GROUP BY` → un resultado por grupo; lo que no está agregado va al `GROUP BY`.
7. `WHERE` filtra filas; `HAVING` filtra grupos.
8. Subconsultas: `= (SELECT ...)` un valor · `IN (SELECT ...)` lista · `EXISTS` · `WITH` para nombrarlas.

👉 **Siguiente:** `06-indices-normalizacion-y-transacciones.md`
