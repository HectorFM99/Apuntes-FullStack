# 07 — FORMULARIOS Y COMUNICACIÓN ENTRE COMPONENTES

> Un formulario en React es **estado + eventos**: cada input refleja un valor del
> estado y lo actualiza al escribir. Y cuando dos componentes necesitan el mismo
> dato, ese dato **sube** al padre común.

---

## 7.1 Inputs controlados ⭐

Un input **controlado** es aquel cuyo valor **lo decide el estado de React**:

```tsx
const [name, setName] = useState('')

<input
  value={name}                                 // el estado → el input
  onChange={(e) => setName(e.target.value)}    // el input → el estado
/>
```

```
Usuario teclea "S"
  → onChange(e)  → setName("S")
  → re-render    → <input value="S" />
```

> 📌 Así el estado es la **única fuente de verdad**: puedes validar, transformar
> (`e.target.value.toUpperCase()`), vaciar el formulario (`setName('')`) o
> desactivar el botón según lo escrito.

### Cada tipo de input

| Elemento | Prop de valor | Leer en `onChange` |
|----------|---------------|--------------------|
| `<input type="text">`, `email`, `password` | `value` | `e.target.value` (string) |
| `<input type="number">` | `value` | `e.target.value` es **string** → `Number(...)` o `e.target.valueAsNumber` |
| `<input type="checkbox">` | `checked` | `e.target.checked` (boolean) |
| `<textarea>` | `value` (no hijos) | `e.target.value` |
| `<select>` | `value` en el **select** | `e.target.value` |

> ⚠️ Si pones `value` sin `onChange`, el input queda **bloqueado** y React avisa.
> Si quieres un valor inicial sin controlarlo: `defaultValue`.

---

## 7.2 Formulario completo: crear un producto en tu API

Tu API valida en el servidor (`validateProduct` en `products.ts`):

```ts
// REAL (express-mongo-atlas-render/src/routes/products.ts)
if (typeof name !== "string" || name.trim() === "") return "El nombre debe ser un texto no vacío";
if (typeof price !== "number" || !Number.isFinite(price) || price < 0)
  return "El precio debe ser un número igual o mayor que cero";
// ...
res.status(201).json({ _id: result.insertedId, ...newProduct });
```

El formulario que la usa:

```tsx
import { useState } from 'react'
import type { ChangeEvent, FormEvent } from 'react'

type NewProduct = { name: string; price: string; category: string }  // price como string mientras se escribe
const empty: NewProduct = { name: '', price: '', category: '' }

type Props = { onCreated: (p: Product) => void }

export function ProductForm({ onCreated }: Props) {
  const [form, setForm] = useState<NewProduct>(empty)
  const [error, setError] = useState<string | null>(null)
  const [sending, setSending] = useState(false)

  // Un solo handler para todos los campos gracias al atributo "name"
  function handleChange(e: ChangeEvent<HTMLInputElement | HTMLSelectElement>) {
    const { name, value } = e.target
    setForm(prev => ({ ...prev, [name]: value }))
  }

  async function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault()                       // ⚠️ evita que el navegador recargue la página
    setError(null)

    // Validación en el cliente: rápida y amable (la del servidor sigue siendo obligatoria)
    const price = Number(form.price)
    if (form.name.trim() === '') return setError('El nombre es obligatorio')
    if (form.price === '' || Number.isNaN(price) || price < 0) return setError('Precio no válido')

    setSending(true)
    try {
      const res = await fetch(`${API_URL}/api/products`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          name: form.name.trim(),
          price,                                         // número, como exige la API
          category: form.category || undefined,          // opcional en la API
        }),
      })
      const data = await res.json()
      if (!res.ok) throw new Error(data.error ?? `Error ${res.status}`)  // muestra el 400 de la API

      onCreated(data)          // avisa al padre con el producto creado (con _id)
      setForm(empty)           // limpia el formulario
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Error desconocido')
    } finally {
      setSending(false)
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="name">Nombre</label>
      <input id="name" name="name" value={form.name} onChange={handleChange} required />

      <label htmlFor="price">Precio (€)</label>
      <input id="price" name="price" type="number" min="0" step="0.01"
             value={form.price} onChange={handleChange} required />

      <label htmlFor="category">Categoría</label>
      <select id="category" name="category" value={form.category} onChange={handleChange}>
        <option value="">Sin categoría</option>
        <option value="muebles">Muebles</option>
        <option value="iluminacion">Iluminación</option>
      </select>

      {error && <p role="alert">{error}</p>}

      <button type="submit" disabled={sending}>
        {sending ? 'Guardando…' : 'Crear producto'}
      </button>
    </form>
  )
}
```

| Detalle | Por qué |
|---------|---------|
| `onSubmit` en el `<form>`, no `onClick` en el botón | Funciona también al pulsar **Enter** y es accesible |
| `e.preventDefault()` | Sin él, el navegador envía el form a la vieja usanza y **recarga** |
| `[name]: value` | *Computed property name*: un solo handler para N campos |
| `price` como string en el estado | El input siempre da strings; se convierte al enviar |
| Mostrar `data.error` | Tu API ya devuelve `{ error: "..." }` con 400: aprovéchalo |
| `disabled={sending}` | Evita el doble clic = doble POST |
| `label htmlFor` + `id` | Accesibilidad (`HTML/05`) |

> 📌 **Valida en los dos lados**: en el cliente por experiencia de usuario, en el
> servidor por **seguridad** (cualquiera puede saltarse tu frontend con `curl`).

---

## 7.3 Flujo de datos unidireccional

```
              ┌────────────── App ──────────────┐
              │ products (estado)               │
              │ handleCreated(p)                │
              └───┬─────────────────────┬───────┘
     props ↓      │                     │      ↓ props
   products       ▼                     ▼      onCreated={handleCreated}
         ┌────────────────┐     ┌──────────────────┐
         │ ProductList    │     │ ProductForm      │
         │ (solo pinta)   │     │ llama onCreated ─┼──▶ ↑ "evento" hacia arriba
         └────────────────┘     └──────────────────┘
```

- **Datos** → bajan por **props**.
- **Eventos** → suben por **callbacks** (funciones pasadas como props).

> 📌 React es **unidireccional**: un hijo nunca cambia directamente el estado del
> padre; le pide que lo haga llamando a una función.

---

## 7.4 *Lifting state up* (elevar el estado) ⭐

**Problema**: `ProductForm` crea un producto y `ProductList` debe mostrarlo. Si
cada uno tuviera su propio estado, no se enterarían el uno del otro.

**Solución**: subir el estado al **padre común más cercano** (`App`) y bajarlo
por props.

```tsx
function App() {
  const [products, setProducts] = useState<Product[]>([])

  useEffect(() => { /* cargar productos (06) */ }, [])

  function handleCreated(p: Product) {
    setProducts(prev => [...prev, p])                     // inmutable (04)
  }

  async function handleDelete(id: string) {
    const res = await fetch(`${API_URL}/api/products/${id}`, { method: 'DELETE' })
    if (res.status === 204) {                             // tu API responde 204 sin cuerpo
      setProducts(prev => prev.filter(p => p._id !== id))
    }
  }

  return (
    <main>
      <ProductForm onCreated={handleCreated} />
      <ProductList products={products} onDelete={handleDelete} />
    </main>
  )
}

function ProductList({ products, onDelete }: { products: Product[]; onDelete: (id: string) => void }) {
  return (
    <ul>
      {products.map(p => (
        <li key={p._id}>
          {p.name} — {p.price} €
          <button type="button" onClick={() => onDelete(p._id)}>Borrar</button>
        </li>
      ))}
    </ul>
  )
}
```

> 💡 Patrón **contenedor / presentacional**: `App` tiene el estado y la lógica;
> `ProductList` y `CourseCard` solo reciben props y pintan (fáciles de reutilizar
> y de testear).

---

## 7.5 Actualización optimista (opcional)

```tsx
async function handleDelete(id: string) {
  const backup = products
  setProducts(prev => prev.filter(p => p._id !== id))     // 1. quita YA de la pantalla
  const res = await fetch(`${API_URL}/api/products/${id}`, { method: 'DELETE' })
  if (!res.ok) setProducts(backup)                        // 2. si falla, deshaz
}
```

La interfaz parece instantánea; si el servidor falla, se revierte.

---

## 7.6 Prop drilling (y cuándo se convierte en problema)

Si un dato tiene que bajar **muchos niveles** pasando por componentes que no lo
usan (`App → Layout → Sidebar → Menu → UserAvatar`), aparece el *prop drilling*.

| Solución | Cuándo |
|----------|--------|
| Composición con `children` | Muchas veces basta con reorganizar (`03`) |
| **Context** (`useContext`) | Datos "globales": usuario logueado, tema, idioma (`08`) |
| Librería de estado (Zustand, Redux Toolkit) | Apps grandes con mucho estado compartido (`11`, `12`) |

---

## 7.7 Formularios con librerías

En proyectos grandes se usan **React Hook Form** + **Zod** (validación con
esquema, parecido a los DTO con `class-validator` de `NESTJS/05`). React 19 trae
además *Actions* (`<form action={fn}>` y `useActionState`) para formularios que
envían datos de forma asíncrona. Primero domina el patrón controlado de este
documento: es lo que te van a pedir en una prueba técnica.

---

## 🎯 Resumen

| Concepto | Clave |
|----------|-------|
| Input controlado | `value` del estado + `onChange` que lo actualiza |
| `onSubmit` + `preventDefault()` | Enviar sin recargar |
| Un handler para todo | `name` en el input + `[name]: value` |
| Validar | En el cliente (UX) **y** en el servidor (seguridad) |
| Unidireccional | Datos bajan (props), eventos suben (callbacks) |
| Lifting state up | El estado compartido vive en el padre común |
