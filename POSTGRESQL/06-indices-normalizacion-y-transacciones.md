# 06 — ÍNDICES, NORMALIZACIÓN Y TRANSACCIONES

> Tres temas que separan "sé escribir SQL" de "sé usar una base de datos":
> - **Índices** → que las consultas sean **rápidas**.
> - **Normalización** → que el diseño **no repita** datos ni se vuelva incoherente.
> - **Transacciones** → que varias operaciones se hagan **todas o ninguna**.

---

## 6.1 ¿Qué es un índice?

Piensa en el **índice alfabético** al final de un libro: en vez de leer las 500
páginas buscando "Postgres", vas al índice y te dice "página 312".

```
SIN índice (Seq Scan)                     CON índice (Index Scan)
┌─────────────────────────┐               ┌──────────────┐
│ fila 1   ¿email = x? no │               │  árbol B     │
│ fila 2   ¿email = x? no │               │   ana@ ──┐   │
│ fila 3   ¿email = x? no │               │   luis@  │   │
│ ...                     │               │   ...    ▼   │
│ fila 1M  ¿email = x? SÍ │               │  → fila 812  │
└─────────────────────────┘               └──────────────┘
 lee TODA la tabla                         unos pocos saltos
```

- Un índice es una **estructura aparte** (normalmente un **árbol B**) que guarda
  los valores de una columna **ordenados**, con un puntero a su fila.
- Acelera `WHERE`, `JOIN`, `ORDER BY` sobre esa columna.
- **Coste:** ocupa espacio y hace un poco más lentos los `INSERT`/`UPDATE`/`DELETE`
  (hay que actualizar también el índice).

---

## 6.2 Índices que ya tienes (sin saberlo)

Postgres crea un índice **automáticamente** para:
- Cada **`PRIMARY KEY`**.
- Cada **`UNIQUE`** (para poder comprobar rápido si un valor ya existe).

### Ejemplo REAL (`Proyecto-SQL/Proyecto-SQL/gimnasio.sql`)

```sql
CREATE TABLE socios (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,   -- índice socios_pkey
    dni VARCHAR(9) UNIQUE NOT NULL,                        -- índice socios_dni_key
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,                    -- índice socios_email_key
    fecha_alta DATE NOT NULL,
    plan_id INTEGER NOT NULL REFERENCES planes(id)         -- ⚠️ SIN índice automático
);
```

Lo puedes ver en `psql` con `\d socios` (sección `Indexes:`) o con `\di`.

> 📌 **Las FK NO llevan índice automático en Postgres.** Y las FK son justo las
> columnas que usas en los `JOIN` (`ON s.plan_id = p.id`). En tablas grandes,
> conviene crearlo tú.

> 📌 Esto conecta con tu respuesta de `respuestas.md`: *"El id numérico
> autogenerado es más eficiente: [...] los índices y JOINs rinden mejor."* Un
> índice sobre un entero es más pequeño y rápido que sobre un `VARCHAR`.

---

## 6.3 Crear y borrar índices

No aparece en tus proyectos. Ejemplos propios sobre el gimnasio:

```sql
-- Índice simple
CREATE INDEX idx_socios_plan_id ON socios (plan_id);

-- Índice compuesto (varias columnas, el orden importa)
CREATE INDEX idx_clases_sala_fecha ON clases (sala, fecha_hora);

-- Índice único (equivale a una restricción UNIQUE)
CREATE UNIQUE INDEX idx_tarjetas_codigo ON tarjetas (codigo);

-- Borrar
DROP INDEX idx_socios_plan_id;
```

### ¿Dónde poner índices?

| ✅ Sí | ❌ No |
|------|------|
| Columnas en `WHERE` muy frecuentes (`email` para el login) | Tablas pequeñas (unos cientos de filas: leerlas enteras es igual de rápido) |
| Claves ajenas usadas en `JOIN` (`plan_id`, `socio_id`) | Columnas que casi nunca se consultan |
| Columnas de `ORDER BY` en listados grandes (`created_at`) | Columnas con pocos valores distintos (`activa TRUE/FALSE`) |
| | "Por si acaso" en todas las columnas |

> 📌 En `reservas` ya tienes `UNIQUE (socio_id, clase_id)`, que crea un índice
> compuesto. Ese índice **sirve también** para buscar solo por `socio_id` (la
> **primera** columna), pero **no** para buscar solo por `clase_id`.

---

## 6.4 `EXPLAIN`: ver cómo ejecuta Postgres una consulta

```sql
EXPLAIN SELECT * FROM socios WHERE email = 'ana@gym.com';
```
```
Index Scan using socios_email_key on socios  (cost=0.15..8.17 rows=1 width=...)
  Index Cond: ((email)::text = 'ana@gym.com'::text)
```

```sql
EXPLAIN ANALYZE SELECT * FROM socios WHERE nombre = 'Ana Pérez';
```
```
Seq Scan on socios  (cost=0.00..1.10 rows=1 width=...) (actual time=... rows=1 loops=1)
  Filter: ((nombre)::text = 'Ana Pérez'::text)
```

| Ves... | Significa |
|--------|-----------|
| `Seq Scan` | Lee **toda** la tabla (normal en tablas pequeñas) |
| `Index Scan` / `Index Only Scan` | Usa un índice ✅ |
| `Bitmap Heap Scan` | Usa un índice para muchas filas |

> 📌 `EXPLAIN ANALYZE` **ejecuta** de verdad la consulta (¡cuidado con un `DELETE`!).

---

## 6.5 Normalización: no repetir datos

**Normalizar** = organizar las tablas para que **cada dato esté en un solo sitio**.
Así evitas las **anomalías**:

### El problema: una tabla "Excel" sin normalizar (ejemplo propio)

```
reservas_mal
┌──────────┬───────────────┬───────────────┬──────────┬────────────┬──────────────┐
│ socio    │ email         │ plan          │ precio   │ clase      │ monitor      │
├──────────┼───────────────┼───────────────┼──────────┼────────────┼──────────────┤
│ Ana      │ ana@gym.com   │ Completo      │ 44.90    │ Spinning   │ Carla        │
│ Ana      │ ana@gym.com   │ Completo      │ 44.90    │ Pilates    │ Elena        │
│ Luis     │ luis@gym.com  │ Básico        │ 29.90    │ Spinning   │ Carla        │
└──────────┴───────────────┴───────────────┴──────────┴────────────┴──────────────┘
```

| Anomalía | Ejemplo |
|----------|---------|
| **De actualización** | Ana cambia de email → hay que cambiarlo en **todas** sus filas; si se te olvida una, hay **dos** emails de Ana |
| **De inserción** | No puedes dar de alta un plan nuevo hasta que alguien reserve una clase |
| **De borrado** | Si borras la única reserva de Luis, **pierdes** que Luis existe y su plan |
| **Redundancia** | El precio 44.90 está repetido en cada fila de cada socio Completo |

### La solución: tu diseño del gimnasio

Separar en `socios`, `planes`, `clases`, `monitores` y `reservas`, unidas por
claves. **Es exactamente lo que hiciste en `gimnasio.sql`.** ✅

---

## 6.6 Las formas normales (versión práctica)

### 1FN — Primera Forma Normal: valores **atómicos**

> Cada celda, **un solo valor**. Nada de listas ni grupos repetidos.

```
❌ monitores                              ✅ monitor_especialidades
┌────┬────────┬─────────────────┐         ┌────────────┬─────────────────┐
│ id │ nombre │ especialidades  │         │ monitor_id │ especialidad_id │
├────┼────────┼─────────────────┤         ├────────────┼─────────────────┤
│ 1  │ Carla  │ Ciclo, Fuerza   │  ──►    │ 1          │ 1               │
└────┴────────┴─────────────────┘         │ 1          │ 5               │
                                          └────────────┴─────────────────┘
```

Es lo mismo que dibujaste en tu A3 (`clase_id = "1, 2, 4, 6"`) para explicar por
qué fallaba poner la FK en el lado equivocado.

### 2FN — Segunda Forma Normal: depender de **toda** la clave

> Cumple 1FN y, si la clave es **compuesta**, ninguna columna depende **solo de
> una parte** de ella.

```
❌ reservas con clave (socio_id, clase_id) y una columna "nombre_socio"
   → nombre_socio depende solo de socio_id, no de la pareja
✅ nombre_socio va en la tabla socios
```

> 📌 Tu respuesta **B6.6** es justo esta idea al revés: `fecha_reserva` depende de
> la **pareja** (socio + clase), por eso va en `reservas` y no en `socios` ni en `clases`.

### 3FN — Tercera Forma Normal: nada depende de algo que **no sea la clave**

> Cumple 2FN y ninguna columna depende de **otra columna no clave**.

```
❌ socios (id, nombre, plan_id, precio_plan)
   → precio_plan depende de plan_id, no del socio
✅ el precio va en planes(precio_mensual) y se consulta con un JOIN
```

**Regla mnemotécnica clásica:** cada columna depende de *"la clave, toda la clave
y nada más que la clave"*.

### ¿Se puede **desnormalizar**?

Sí, a veces **a propósito**, por rendimiento o para guardar una **foto histórica**.

**Ejemplo REAL de tu `api-rastro` (MongoDB) — `respuestas.md`, Pregunta 4:**

> La copia `listing.price: 380` de la oferta es una **foto del momento** en que Sara
> ofertó: indica el precio que tenía el anuncio cuando hizo su oferta de 340 €, y ese
> dato histórico debe conservarse.

En SQL pasa igual: en una tabla `pedido_producto` guardas el `precio_unitario`
**del momento de la compra**, aunque el producto cambie de precio después (lo
pusiste en tu A6 de Furniro). Eso **no** es redundancia: es un dato distinto.

---

## 6.7 Transacciones: todo o nada

Una **transacción** agrupa varias sentencias para que se ejecuten como **una sola
unidad**: o se aplican **todas**, o **ninguna**.

### El ejemplo clásico: una transferencia (ejemplo propio)

```sql
BEGIN;                                                   -- empieza la transacción
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;     -- quita 100 a la cuenta 1
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;     -- suma 100 a la cuenta 2
COMMIT;                                                  -- confirma: ahora sí es definitivo
```

Si algo falla a mitad (o te arrepientes):

```sql
BEGIN;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
-- ¡error! / me he equivocado
ROLLBACK;                                                -- deshace TODO desde el BEGIN
```

```
 BEGIN ──► sentencia 1 ──► sentencia 2 ──► COMMIT   ✅ todo se guarda
                │              │
                └──── error ───┴──► ROLLBACK        ↩️ nada ha pasado
```

> 📌 Si no escribes `BEGIN`, Postgres trabaja en modo **autocommit**: cada sentencia
> es su propia transacción y se confirma sola al terminar.
> ⚠️ Dentro de una transacción en `psql`, si una sentencia da error, las siguientes
> fallan con `current transaction is aborted` hasta que hagas `ROLLBACK`.

### Ejemplo con tu gimnasio (propio): borrar a Pablo del todo

En el archivo `04` vimos que para borrar a Pablo hay que borrar antes reservas y
tarjeta. Con una transacción, o se borra **todo** o no se borra **nada**:

```sql
BEGIN;
DELETE FROM reservas WHERE socio_id = (SELECT id FROM socios WHERE email = 'pablo@gym.com');
DELETE FROM tarjetas WHERE socio_id = (SELECT id FROM socios WHERE email = 'pablo@gym.com');
DELETE FROM socios   WHERE email = 'pablo@gym.com';
COMMIT;
```

### El caso REAL de tu `api-rastro` (MongoDB) — `respuestas.md`, Reto 1

> **¿Qué pasa si el servidor se apaga entre una escritura y otra?** Los datos quedan
> incoherentes: la oferta queda `ACCEPTED` pero el anuncio sigue `AVAILABLE`, y otros
> compradores podrían seguir ofertando por algo que ya está reservado. [...] para que
> las dos se hagan juntas o ninguna haría falta una **transacción** (que necesita
> MongoDB en modo replica set).

En Postgres sería directo (equivalente propio):

```sql
BEGIN;
UPDATE offers   SET status = 'ACCEPTED' WHERE id = 7 AND status = 'PENDING';
UPDATE listings SET status = 'RESERVED' WHERE id = 3;
COMMIT;
```

> 📌 Una de las grandes ventajas de las bases de datos **relacionales**: las
> transacciones con varias tablas vienen "de serie".

---

## 6.8 ACID: las 4 garantías de una transacción

| Letra | Propiedad | Qué significa | En la transferencia |
|-------|-----------|---------------|---------------------|
| **A** | **Atomicidad** | Todo o nada | No puede salir el dinero de una cuenta sin llegar a la otra |
| **C** | **Consistencia** | La BD pasa de un estado válido a otro válido (se cumplen todas las restricciones) | Un `CHECK (saldo >= 0)` hace fallar la transacción entera si alguien queda en negativo |
| **I** | **Aislamiento** (*Isolation*) | Las transacciones a la vez no se pisan; cada una no ve los cambios a medias de otra | Otro usuario no ve el dinero "en el aire" |
| **D** | **Durabilidad** | Tras el `COMMIT`, el dato sobrevive aunque se vaya la luz | Postgres lo escribe en disco (WAL) antes de decir "OK" |

> 📌 La **C** está muy ligada a lo que viste en el archivo `03`: las restricciones
> (`NOT NULL`, `UNIQUE`, `CHECK`, FK) son las que definen qué es un "estado válido".

### Concurrencia: dos reservas a la vez (ejemplo propio)

```
Usuario A: ¿plazas libres en Crossfit? → 1      Usuario B: ¿plazas libres? → 1
Usuario A: INSERT reserva ✅                      Usuario B: INSERT reserva ✅
                     → ¡4 reservas para 3 plazas! 😱
```

Soluciones típicas:
- **Bloquear la fila** mientras decides: `SELECT ... FROM clases WHERE id = 7 FOR UPDATE;`
  dentro de la transacción (el segundo espera a que el primero termine).
- Hacer la comprobación **en la propia sentencia** (como hace tu `api-rastro` con
  `findOneAndUpdate({ _id: offerId, status: "PENDING" }, ...)`; en SQL:
  `UPDATE ... WHERE id = $1 AND status = 'PENDING'` y mirar cuántas filas cambió).

---

## ✅ Resumen rápido

1. **Índice** = búsqueda rápida a cambio de espacio y escrituras algo más lentas.
2. PK y UNIQUE crean índice **automático**; las **FK no** → créalo si la tabla es grande.
3. `CREATE INDEX nombre ON tabla (columna);` · `EXPLAIN` para ver si se usa.
4. **Normalizar** = cada dato en un solo sitio → evita anomalías.
5. 1FN: un valor por celda · 2FN: depende de **toda** la clave · 3FN: solo de la clave.
6. Guardar una "foto" histórica (precio de compra) **no** es mala redundancia.
7. **Transacción**: `BEGIN` ... `COMMIT` (o `ROLLBACK`) → todo o nada.
8. **ACID**: Atomicidad, Consistencia, Aislamiento, Durabilidad.

👉 **Siguiente:** `07-postgres-desde-node-con-pg.md`
