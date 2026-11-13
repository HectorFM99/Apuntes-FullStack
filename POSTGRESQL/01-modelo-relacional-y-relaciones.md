# 01 — EL MODELO RELACIONAL Y LAS RELACIONES

> Una **base de datos relacional** guarda la información en **tablas** (como hojas
> de Excel muy estrictas) que se **relacionan** entre sí mediante **claves**.
> PostgreSQL es uno de los gestores (SGBD) relacionales más usados: gratuito,
> potente y muy respetuoso con el estándar SQL.

---

## 1.1 Tabla, fila y columna

```
                    TABLA: socios
         ┌────┬───────────┬─────────────┬───────────────┬─────────┐
columnas │ id │ dni       │ nombre      │ email         │ plan_id │
         ├────┼───────────┼─────────────┼───────────────┼─────────┤
  fila → │ 1  │ 11111111A │ Ana Pérez   │ ana@gym.com   │ 2       │
  fila → │ 2  │ 22222222B │ Luis García │ luis@gym.com  │ 1       │
  fila → │ 3  │ 33333333C │ Marta López │ marta@gym.com │ 3       │
         └────┴───────────┴─────────────┴───────────────┴─────────┘
```

| Concepto | Otros nombres | Qué es |
|----------|---------------|--------|
| **Tabla** | relación, entidad | Un conjunto de cosas del **mismo tipo** (socios, clases, libros...) |
| **Fila** | registro, tupla | **Una** cosa concreta (Ana Pérez) |
| **Columna** | campo, atributo | Una **característica** que tienen todas (nombre, email...) |
| **Celda** | valor | El dato de una fila en una columna |

**Reglas de oro:**
- Cada columna tiene **un tipo** fijo (texto, número, fecha...). No puedes meter
  texto en una columna numérica.
- Cada celda guarda **un solo valor** (nada de listas `"1, 2, 4"` dentro de una celda).
- Cada fila debe poder **identificarse** de forma única → **clave primaria**.

> 📌 Comparación con lo que ya conoces: en **MongoDB** (tu proyecto `api-rastro`)
> guardabas **documentos** JSON flexibles dentro de **colecciones**. En Postgres
> la estructura es **fija** y se define **antes** de meter datos. Más rígido, pero
> la base de datos te **protege** de datos incoherentes.

---

## 1.2 Clave primaria (PRIMARY KEY, PK)

La **clave primaria** es la columna (o columnas) que **identifica cada fila**.

- **No se puede repetir** nunca.
- **No puede ser NULL** (vacía).
- Solo hay **una** por tabla.

```sql
id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY
--         └──────────────┬─────────────┘
--   Postgres genera 1, 2, 3... automáticamente
```

### Ejemplo REAL copiado (`Proyecto-SQL/Proyecto-SQL/respuestas.md`)

> **¿Qué columnas no se pueden repetir nunca?**
>
> id, dni y email. Las tres tienen restricciones que impiden valores duplicados (id es PRIMARY KEY; dni y email son UNIQUE)
>
> **¿Cuál es la clave primaria? ¿Cuáles son claves alternativas?**
>
> - Clave primaria: id.
> - Claves alternativas: dni y email
>
> **El DNI ya identifica a cada socio. ¿Por qué crees que usamos un id numérico como clave primaria en vez del DNI? Da al menos una razón.**
>
> El id numérico autogenerado es más eficiente: menos espacio en disco, las comparaciones numéricas son más rápidas y los índices y JOINs rinden mejor.

### Clave natural vs clave artificial (surrogate)

| Tipo | Ejemplo | Ventaja | Inconveniente |
|------|---------|---------|---------------|
| **Natural** | `dni`, `email` | Ya existe en el mundo real | Puede **cambiar** (alguien cambia de email), es texto largo |
| **Artificial** | `id` autogenerado | Nunca cambia, es pequeño y rápido | No significa nada fuera de la BD |

> 📌 Además de la razón de tu respuesta (rendimiento), hay otra muy importante:
> **la clave primaria no debería cambiar nunca**. Si usas el email como PK y el
> socio cambia de email, tendrías que actualizarlo en **todas** las tablas que lo
> referencian. Con un `id` eso no pasa.

---

## 1.3 Clave ajena / foránea (FOREIGN KEY, FK)

Una **clave ajena** es una columna que **apunta a la clave primaria de otra tabla**.
Es el "pegamento" entre tablas.

```
  socios                              planes
┌────┬─────────────┬─────────┐       ┌────┬────────────┬────────────────┐
│ id │ nombre      │ plan_id │──┐    │ id │ nombre     │ precio_mensual │
├────┼─────────────┼─────────┤  │    ├────┼────────────┼────────────────┤
│ 1  │ Ana Pérez   │ 2       │──┼──► │ 1  │ Básico     │ 29.90          │
│ 2  │ Luis García │ 1       │──┘  ┌►│ 2  │ Completo   │ 44.90          │
│ 4  │ Hugo Sanz   │ 2       │─────┘ │ 3  │ Estudiante │ 24.90          │
└────┴─────────────┴─────────┘       └────┴────────────┴────────────────┘
          plan_id = FK  ──────────────────►  planes.id = PK
```

```sql
plan_id INTEGER NOT NULL REFERENCES planes(id)
```

**Qué garantiza la FK (integridad referencial):**
1. No puedes poner un `plan_id` que **no exista** en `planes`.
2. No puedes **borrar** un plan si hay socios que lo usan.

### Ejemplo REAL copiado (`Proyecto-SQL/Proyecto-SQL/respuestas.md`)

> **¿Qué columna apunta a otra tabla? ¿A cuál?**
>
> plan_id es una clave ajena (FK) que apunta a la tabla planes, exactamente a la columna planes(id)

> **B6.2 ¿Por qué una clave ajena sí puede repetirse?**
> Porque varias filas del lado muchos pueden apuntar a la misma fila referenciada.

> 📌 Fíjate en el dibujo: `plan_id = 2` aparece **dos veces** (Ana y Hugo). La FK
> **sí** se repite; la PK (`planes.id`) **no**.

### PK vs UNIQUE vs FK (resumen)

| | ¿Se repite? | ¿Admite NULL? | ¿Cuántas por tabla? |
|---|---|---|---|
| **PRIMARY KEY** | ❌ No | ❌ No | **1** |
| **UNIQUE** | ❌ No | ✅ Sí (salvo que pongas `NOT NULL`) | Varias |
| **FOREIGN KEY** | ✅ Sí | ✅ Sí (salvo que pongas `NOT NULL`) | Varias |

**Tu respuesta REAL (`respuestas.md`):**

> **B6.1 ¿Qué diferencia hay entre una clave primaria y una clave UNIQUE?**
> La clave primaria identifica cada fila, no admite NULL y solo puede haber una por tabla. UNIQUE impide duplicados y puede haber varias restricciones de este tipo por tabla.

---

## 1.4 Cómo descubrir el tipo de relación: las DOS preguntas

Para cada pareja de tablas A–B te haces **dos preguntas**:

1. Un **A**, ¿cuántos **B** puede tener?
2. Un **B**, ¿cuántos **A** puede tener?

| Respuesta 1 | Respuesta 2 | Relación |
|-------------|-------------|----------|
| uno | uno | **1:1** |
| muchos | uno | **1:N** |
| muchos | muchos | **N:M** |

### Ejemplo REAL copiado (`Proyecto-SQL/Proyecto-SQL/respuestas.md` — A2)

| Pareja | Un A, ¿cuántos B? | Un B, ¿cuántos A? | Relación |
|:---|:---|:---|:---|
| planes – socios | muchos | uno | 1:N |
| socios – tarjetas | uno | uno | 1:1 |
| monitores – clases | muchos | uno | 1:N |
| socios – clases | muchos | muchos | N:M |
| monitores – especialidades | muchos | muchos | N:M |

---

## 1.5 Relación 1:N (uno a muchos) — la más común

> Un plan tiene **muchos** socios; un socio tiene **un** plan.

**Regla:** la FK va en el lado **N** (el de "muchos").

```
planes (1) ───────< socios (N)
                    └─ plan_id  (FK)
```

### Ejemplo REAL copiado (`respuestas.md` — A3)

> - Un monitor, ¿cuántos socios? → muchos
> - Una clase, ¿cuántos monitores? → uno
> - 1:N. Lado muchos: clases. La clave ajena va en clases.
>
> **¿En qué tabla va la clave ajena? ¿Cómo la llamarías?**
> En la tabla clases, con el nombre monitor_id.
>
> Dibuja (en una tabla Markdown) cómo quedaría si la pusieras en el lado equivocado y explica en una frase por qué falla.
>
> | id | dni | nombre | clase_id |
> |:---|:----|:-------|:---------|
> | 1 | 90000001J | Carla Navarro | 1, 2, 4, 6|
> | 2 | 90000002K | Javier Ortega | 3, 5|

```
❌ FK en el lado 1 (monitores.clase_id) → "1, 2, 4, 6" en UNA celda → rompe la regla
   de "un valor por celda": no puedes hacer JOIN, ni FK, ni buscar bien.

✅ FK en el lado N (clases.monitor_id) → cada clase guarda UN número: el de su monitor.
```

> **B6.3 En una relación 1:N, ¿en qué lado va la clave ajena? ¿Por qué no en el otro?**
> Va en el lado N. En el lado 1 habría que guardar varios valores en una celda o repetir filas.

> 📌 **Orden de creación:** primero se crea la tabla del lado **1** (la referenciada).
> Tu respuesta: *"monitores, porque clases tiene una FK que apunta a monitores(id)
> y la tabla referenciada debe existir primero."*

---

## 1.6 Relación 1:1 (uno a uno)

> Un socio tiene **una** tarjeta; una tarjeta es de **un** socio.

**Truco:** es una 1:N en la que a la FK le añades **`UNIQUE`**. Así no se puede repetir.

```sql
socio_id INTEGER UNIQUE NOT NULL REFERENCES socios(id)
--               ▲
--   sin UNIQUE sería 1:N (un socio con varias tarjetas)
```

### Ejemplo REAL copiado (`respuestas.md` — A4 y B6.4)

> **¿Qué pasaría si quitásemos UNIQUE de socio_id? Ejemplo de datos:**
>
> Un mismo socio tendría dos tarjetas en la tabla tarjetas, dejaria de ser 1:1
>
> **Hay dos columnas UNIQUE en tarjetas. ¿Qué evita cada una?**
> - codigo UNIQUE: evita que existan dos tarjetas con el mismo código
> - socio_id UNIQUE: evita que un socio tenga asignadas múltiples tarjetas
>
> Relación: Persona 1----1 Pasaporte.
> FK: en la tabla pasaportes, apuntando a personas(id) con UNIQUE.

> **B6.4 ¿Qué convierte una relación 1:N en una 1:1?**
> Añadir UNIQUE a la clave ajena para impedir que se repita.

> 📌 ¿En qué tabla va la FK en una 1:1? Normalmente en la tabla "dependiente", la
> que **no puede existir sin la otra** (no hay tarjeta sin socio, pero sí socio sin
> tarjeta).

---

## 1.7 Relación N:M (muchos a muchos) → tabla intermedia

> Un socio reserva **muchas** clases; una clase tiene **muchos** socios.

No cabe la FK en ninguna de las dos tablas (sería una lista en una celda). Se crea
una **tercera tabla** (intermedia, de unión, *pivot*) con **dos FK**:

```
socios (1) ───< reservas >─── (1) clases
                ├ socio_id  (FK → socios.id)
                ├ clase_id  (FK → clases.id)
                └ fecha_reserva  (dato propio de la relación)

   N:M  =  dos relaciones 1:N que se encuentran en la tabla intermedia
```

```
reservas
┌────┬──────────┬──────────┬───────────────┐
│ id │ socio_id │ clase_id │ fecha_reserva │
├────┼──────────┼──────────┼───────────────┤
│ 1  │ 1        │ 1        │ 2026-10-28    │  ← Ana en Spinning
│ 2  │ 1        │ 3        │ 2026-10-28    │  ← Ana en Pilates
│ 4  │ 2        │ 1        │ 2026-10-29    │  ← Luis en Spinning
└────┴──────────┴──────────┴───────────────┘
```

### Ejemplo REAL copiado (`respuestas.md` — A5)

> *Tabla monitor_especialidades:*
> - id (PK)
> - monitor_id (FK → monitores(id))
> - especialidad_id (FK → especialidades(id))
> - UNIQUE( monitor_id, especialidad_id)
>
> **¿Qué evita el UNIQUE de la tabla intermedia?**
> Evita que un mismo monitor tenga asignada la misma especialidad dos veces.
>
> **¿Qué tienen en común reservas y monitor_especialidades? ¿En qué se diferencian?**
>
> *En común:* Ambas son tablas intermedias que resuelven relaciones N:M con dos claves ajenas.
>
> *Diferencian:* monitor_especialidades solo vincula (es puramente relacional). reservas además almacena datos propios como fecha_reserva.

> **B6.6 ¿Por qué fecha_reserva está en reservas y no en socios ni en clases?**
> Porque corresponde a cada reserva concreta entre un socio y una clase, no al socio ni a la clase por separado.

> 📌 **Pregunta trampa típica:** "¿Dónde guardo la `cantidad` de un producto en un
> pedido?" → en la tabla intermedia `pedido_producto`, porque es un dato **de la
> pareja**, no del pedido ni del producto (lo pusiste tú mismo en el ejercicio A6
> de Furniro).

---

## 1.8 Más ejemplos REALES de tu ejercicio A6 (`respuestas.md`)

| Pareja | Relación | Dónde va la FK |
|--------|----------|----------------|
| País – ciudad | 1:N | `pais_id` en `ciudades` |
| Persona – pasaporte | 1:1 | `persona_id` en `pasaportes` + `UNIQUE` |
| Alumno – asignatura | N:M | Tabla intermedia `alumno_asignatura` |
| Cliente – pedido | 1:N | `cliente_id` en `pedidos` |
| Película – actor | N:M | Tabla intermedia `pelicula_actor` con `nombre_personaje` |
| Autor – libro | 1:N **o** N:M | Depende de si un libro puede tener varios autores |
| Furniro: usuario – pedido | 1:N | `usuario_id` en `pedidos` |
| Furniro: pedido – producto | N:M | `pedido_producto` con `cantidad`, `precio_unitario`, `subtotal` |

> 📌 El caso **Autor – libro** es muy buena respuesta: el tipo de relación **no lo
> decide el programador**, lo decide la **regla del negocio**. Si no la sabes, pregunta.

---

## 1.9 Diagrama completo del gimnasio (tu `gimnasio.sql`)

```
                 ┌──────────┐
                 │  planes  │
                 └────┬─────┘
                      │ 1:N
                      ▼
 ┌──────────┐  1:1  ┌──────────┐   1:N   ┌──────────┐   N:1   ┌──────────┐
 │ tarjetas │──────►│  socios  │────────<│ reservas │>────────│  clases  │
 └──────────┘       └──────────┘         └──────────┘         └────┬─────┘
 socio_id UNIQUE                     socio_id + clase_id           │ N:1
                                     UNIQUE (socio_id, clase_id)   ▼
                                                             ┌───────────┐
  ┌────────────────┐   N:1   ┌─────────────────────────┐ N:1 │ monitores │
  │ especialidades │◄────────│ monitor_especialidades  │────►│           │
  └────────────────┘         └─────────────────────────┘     └───────────┘
```

Leyenda: `───<` = "uno a muchos" (la punta `<` es el lado N, donde vive la FK).

---

## 1.10 Errores típicos de diseño

| Error | Por qué está mal | Solución |
|-------|------------------|----------|
| Guardar `"1, 2, 4"` en una celda | No se puede filtrar, ni hacer JOIN, ni poner FK | Tabla intermedia (N:M) o FK en el lado N |
| FK en el lado 1 | Obliga a listas o a duplicar filas | FK **siempre en el lado N** |
| Olvidar `UNIQUE` en 1:1 | Se convierte en 1:N sin querer | `UNIQUE` en la FK |
| Olvidar `UNIQUE(a_id, b_id)` en N:M | La misma pareja se puede repetir | `UNIQUE (socio_id, clase_id)` |
| Usar `email` como PK | Si cambia, hay que actualizarlo en todas partes | `id` autogenerado + `email UNIQUE` |
| Dato de la relación en una de las tablas | `fecha_reserva` en `socios` no tiene sentido | En la tabla intermedia |

---

## ✅ Resumen rápido

1. **Tabla** = tipo de cosa · **fila** = una cosa · **columna** = una característica.
2. **PK**: identifica la fila; única, no NULL, una por tabla.
3. **FK**: apunta a la PK de otra tabla; **sí se repite**.
4. Para saber la relación → **las dos preguntas** ("un A, ¿cuántos B?" y al revés).
5. **1:N** → FK en el lado N.
6. **1:1** → FK + `UNIQUE`.
7. **N:M** → tabla intermedia con dos FK + `UNIQUE` de la pareja (+ datos propios de la relación).

👉 **Siguiente:** `02-docker-psql-y-tipos-de-datos.md`
