# 05 — RENDERIZADO CONDICIONAL Y LISTAS

> Dos cosas que harás en **todos** los componentes: mostrar algo **solo si** se
> cumple una condición, y pintar **un elemento por cada dato** de un array.

---

## 5.1 Renderizado condicional

### Las 4 formas

| Forma | Cuándo | Ejemplo |
|-------|--------|---------|
| `cond && <X />` | Mostrar **o nada** | `{seats <= 3 && <span>Últimas plazas</span>}` |
| `cond ? <A /> : <B />` | Mostrar **una u otra** | `{seats > 0 ? <p>Libres</p> : <p>Completo</p>}` |
| `if` + `return` temprano | Cambia **todo** el componente | Loading, error, lista vacía |
| Variable antes del `return` | Lógica larga | `let badge = null; if (...) badge = <span/>` |

### Ejemplo REAL (`academia/src/components/CourseCard.tsx`)

```tsx
export function CourseCard({ title, hours, price, category, seats }: CourseCardProps) {
  const [isInterested, setIsInterested] = useState(false);

  return (
    <article className="card">
      {seats > 0 && seats <= 3 && <span className="badge">Últimas plazas</span>}
      <h2>{title}</h2>
      <p>{hours} horas</p>
      <p>{category}</p>
      <p>{price} €</p>
      {seats > 0 ? <p>{seats} plazas libres</p> : <p>Completo</p>}
      <button type="button" onClick={() => setIsInterested((current) => !current)}>
        {isInterested ? 'Te interesa' : 'Me interesa'}
      </button>
    </article>
  );
}
```

| Curso | `seats` | Badge "Últimas plazas" | Texto plazas |
|-------|:------:|:----------------------:|--------------|
| HTML y CSS desde cero | 3 | ✅ | "3 plazas libres" |
| Excel avanzado | 2 | ✅ | "2 plazas libres" |
| React para principiantes | 0 | ❌ (`seats > 0` es `false`) | "Completo" |

### Cómo funciona `&&`

```
a && b   →  si a es falsy, devuelve a;  si no, devuelve b
true  && <span/>  →  <span/>   (se pinta)
false && <span/>  →  false     (React no pinta nada)
```

### ⚠️ La trampa del `0`

```tsx
{seats && <p>{seats} plazas</p>}
// seats = 0 → 0 && ... → devuelve 0 → ¡React pinta un "0"!

{seats > 0 && <p>{seats} plazas</p>}   // ✅ condición booleana
{!!seats && <p>...</p>}                // ✅ también vale
```

> 📌 Tu `CourseCard` lo hace bien: `seats > 0 && ...` siempre es booleano.

### Return temprano

```tsx
function CourseList({ courses, loading, error }: Props) {
  if (loading) return <p>Cargando cursos…</p>
  if (error) return <p role="alert">Error: {error}</p>
  if (courses.length === 0) return <p>No hay cursos todavía.</p>

  return (
    <div className="course-list">
      {courses.map(c => <CourseCard key={c.id} {...c} />)}
    </div>
  )
}
```

> 💡 Patrón de **los 4 estados** de cualquier dato remoto: *cargando*, *error*,
> *vacío* y *con datos*. Lo usarás en `06`.

### Clases CSS condicionales

```tsx
<article className={seats === 0 ? 'card card--full' : 'card'}>
<article className={`card ${seats === 0 ? 'card--full' : ''}`}>
<button className={isInterested ? 'btn active' : 'btn'}>
```

---

## 5.2 Listas con `.map()`

`.map()` transforma un array de **datos** en un array de **JSX**, y React pinta
cada elemento.

### Ejemplo REAL (`academia/src/App.tsx`)

```tsx
<div className="course-list">
  {courses.map((course) => (
    <CourseCard key={course.title} {...course} />
  ))}
</div>
```

```
courses = [ {HTML...}, {Excel...}, {React...} ]
            │            │            │
          .map(course => <CourseCard ... />)
            ▼            ▼            ▼
        [ <CourseCard/>, <CourseCard/>, <CourseCard/> ]   → React pinta las 3
```

> ⚠️ Con llaves en la flecha hay que poner `return`:
> ```tsx
> courses.map(course => { <CourseCard {...course} /> })          // ❌ devuelve undefined
> courses.map(course => { return <CourseCard {...course} /> })   // ✅
> courses.map(course => (<CourseCard {...course} />))            // ✅ return implícito
> ```

---

## 5.3 La prop `key` ⭐

Cada elemento de una lista necesita una `key` **única entre sus hermanos** y
**estable** (la misma para el mismo dato en cada render).

### ¿Para qué sirve?

Cuando la lista cambia (añades, borras, reordenas), React usa la `key` para saber
**qué elemento es cuál** y conservar su estado y su DOM.

```
Render 1:  key=HTML [isInterested=true]   key=Excel   key=React
Borras "HTML"
Render 2:  key=Excel   key=React
           → React sabe que Excel sigue siendo Excel y conserva su estado ✅
```

### Qué usar como `key`

| Opción | ¿Vale? |
|--------|--------|
| `id` de la base de datos (`_id` de Mongo, `id` de Postgres) | ✅ **La mejor** |
| Un campo único y que no cambia (`course.title` en tu `academia`) | ✅ Si de verdad es único |
| `crypto.randomUUID()` generado **al crear** el dato | ✅ |
| Índice del array (`index`) | ⚠️ Solo si la lista **nunca** se reordena, filtra ni borra |
| `Math.random()` en el render | ❌ Cambia en cada render → React destruye y recrea todo |

```tsx
// ⚠️ Con index: si borras el primero, el estado "isInterested" salta al siguiente
{courses.map((course, index) => <CourseCard key={index} {...course} />)}
```

> 📌 En tu proyecto `key={course.title}` funciona porque no hay títulos repetidos.
> En cuanto los cursos vengan de tu API, usa el `id` / `_id`.

- La `key` **no** llega como prop al componente (`props.key` es `undefined`).
- Va en el elemento que devuelve el `.map()`, **no** dentro del componente hijo.
- Si falta, React avisa en consola: *Each child in a list should have a unique "key" prop*.

---

## 5.4 Filtrar, ordenar y buscar (estado derivado)

```tsx
function Catalog({ courses }: { courses: Course[] }) {
  const [query, setQuery] = useState('')
  const [onlyAvailable, setOnlyAvailable] = useState(false)

  // Se calcula en cada render a partir del estado: NO es estado
  const visible = courses
    .filter(c => c.title.toLowerCase().includes(query.toLowerCase()))
    .filter(c => !onlyAvailable || c.seats > 0)
    .toSorted((a, b) => a.price - b.price)

  return (
    <>
      <input value={query} onChange={e => setQuery(e.target.value)} placeholder="Buscar…" />
      <label>
        <input type="checkbox" checked={onlyAvailable}
               onChange={e => setOnlyAvailable(e.target.checked)} />
        Solo con plazas
      </label>

      <p>{visible.length} de {courses.length} cursos</p>
      {visible.length === 0 && <p>Ningún curso coincide.</p>}
      <div className="course-list">
        {visible.map(c => <CourseCard key={c.title} {...c} />)}
      </div>
    </>
  )
}
```

> 💡 `toSorted` (ES2023, en tu `target`) devuelve un array **nuevo**; `sort` muta
> el original (y si es una prop o estado, es un bug).

---

## 5.5 Agrupar: listas anidadas

```tsx
const categories = [...new Set(courses.map(c => c.category))]   // ['Programación', 'Productividad']

{categories.map(cat => (
  <section key={cat}>
    <h2>{cat}</h2>
    {courses
      .filter(c => c.category === cat)
      .map(c => <CourseCard key={c.title} {...c} />)}
  </section>
))}
```

---

## 5.6 CSS para la lista (REAL, `academia/src/App.css`)

```css
.course-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 20px;
}
```

> 📌 `repeat(auto-fit, minmax(240px, 1fr))` hace la rejilla **responsive sin media
> queries**: caben tantas columnas de mínimo 240 px como permita el ancho
> (ver `CSS/06-grid.md`). Ideal para listas de tarjetas.

---

## 🎯 Resumen

| Concepto | Clave |
|----------|-------|
| `&&` | Mostrar o nada; cuidado con el `0` |
| Ternario | Una cosa u otra |
| Return temprano | Cargando / error / vacío |
| `.map()` | Datos → JSX |
| `key` | Única y estable; mejor el `id` de la BD; evitar el índice |
| Filtrar/ordenar | Estado derivado, calculado en el render, sin mutar |
