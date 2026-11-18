# 10 — EJEMPLOS COMPLETOS DE PROYECTOS

> Repaso con código entero. Primero **tu proyecto `academia` tal cual**, archivo
> por archivo y comentado. Después, la **versión full stack**: el mismo catálogo
> pero leyendo y escribiendo en tu API `express-mongo-atlas-render`.

---

## PARTE A — Tu proyecto `Vite/academia` (REAL)

### A.1 Qué hace

```
┌──────────────────────── Catálogo de cursos ────────────────────────┐
│ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐     │
│ │ [Últimas plazas] │ │ [Últimas plazas] │ │                  │     │
│ │ HTML y CSS desde │ │ Excel avanzado   │ │ React para       │     │
│ │ cero             │ │ 25 horas         │ │ principiantes    │     │
│ │ 40 horas         │ │ Productividad    │ │ 30 horas         │     │
│ │ Programación     │ │ 90 €             │ │ Programación     │     │
│ │ 120 €            │ │ 2 plazas libres  │ │ 150 €            │     │
│ │ 3 plazas libres  │ │ [Me interesa]    │ │ Completo         │     │
│ │ [Me interesa]    │ │                  │ │ [Me interesa]    │     │
│ └──────────────────┘ └──────────────────┘ └──────────────────┘     │
│                         Contador                                   │
│                       [−]  0  [+]                                  │
└────────────────────────────────────────────────────────────────────┘
```

### A.2 Árbol de componentes

```
main.tsx
└── <StrictMode>
    └── <App>                         datos: const courses (constante)
        ├── <CourseCard> × 3          props: title, hours, price, category, seats
        │                             estado: isInterested (boolean)
        └── <Counter>                 estado: value (0..10)

    (<Footer> existe pero no se usa todavía)
```

### A.3 `src/main.tsx`

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css"; // estilos globales
import App from "./App.tsx";

// Busca el <div id="root"> y pinta App dentro
createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

> Monta la app (`01`). `!` = "sé que existe `#root`". `StrictMode` = comprobaciones extra en desarrollo.

### A.4 `src/App.tsx`

```tsx
import { CourseCard } from './components/CourseCard'
import { Counter } from './components/Counter'
import './App.css'

const courses = [
  {
    title: 'HTML y CSS desde cero',
    hours: 40,
    price: 120,
    category: 'Programación',
    seats: 3,
  },
  {
    title: 'Excel avanzado',
    hours: 25,
    price: 90,
    category: 'Productividad',
    seats: 2,
  },
  {
    title: 'React para principiantes',
    hours: 30,
    price: 150,
    category: 'Programación',
    seats: 0,
  },
]

function App() {
  return (
    <main className="catalog">
      <h1>Catálogo de cursos</h1>
      <div className="course-list">
        {courses.map((course) => (
          <CourseCard key={course.title} {...course} />
        ))}
      </div>
      <section className="counter-section">
        <h2>Contador</h2>
        <Counter />
      </section>
    </main>
  )
}

export default App
```

| Concepto | Dónde | Doc |
|----------|-------|-----|
| Datos fuera del componente (no se recrean en cada render, no son estado porque no cambian) | `const courses` | `04` |
| Lista con `.map()` y `key` única | `courses.map(...)` | `05` |
| Spread de props | `{...course}` | `03` |
| Composición | `App` contiene `CourseCard` y `Counter` | `02` |
| HTML semántico | `main`, `section`, `h1`/`h2` | `02` |

### A.5 `src/components/CourseCard.tsx`

```tsx
import { useState } from 'react';

// La lista de props que acepta el componente, con su tipo
type CourseCardProps = {
  title: string;
  hours: number;
  price: number;
  category: string;
  seats: number;
};

// Sacamos cada prop del objeto para usarla directamente
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

| Concepto | Dónde | Doc |
|----------|-------|-----|
| Props tipadas + desestructuración | `CourseCardProps` | `03` |
| Estado local booleano (toggle funcional) | `isInterested` | `04` |
| `&&` con condición booleana (sin trampa del `0`) | badge | `05` |
| Ternario | plazas / Completo, texto del botón | `05` |
| Valores derivados de props (no estado) | badge y "Completo" salen de `seats` | `04` |

### A.6 `src/components/Counter.tsx`

```tsx
import { useState } from 'react'

export function Counter() {
  const [value, setValue] = useState(0)

  return (
    <div className="counter-widget">
      <button
        type="button"
        onClick={() => setValue((current) => Math.max(0, current - 1))}
        disabled={value === 0}
      >
        −
      </button>
      <span>{value}</span>
      <button
        type="button"
        onClick={() => setValue((current) => Math.min(10, current + 1))}
        disabled={value === 10}
      >
        +
      </button>
    </div>
  )
}
```

> Actualización funcional + límites con `Math.max/min` + `disabled` derivado del estado (`04`).

### A.7 `src/components/Footer.tsx`

```tsx
export function Footer() {
  return (
    <footer>
      <p>© 2026 Academia · Valencia</p>
    </footer>
  )
}
```

### A.8 `src/App.css` (lo esencial)

```css
.course-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 20px;
  text-align: left;
}

.card {
  padding: 24px;
  border: 1px solid var(--border);
  border-radius: 12px;
  box-shadow: var(--shadow);
}

.card p + p { margin-top: 8px; }        /* selector de hermano adyacente */

.badge {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 999px;
  color: var(--accent);
  background: var(--accent-bg);
}

.counter-widget {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 16px;
}

.counter-widget span {
  min-width: 24px;
  font-variant-numeric: tabular-nums;   /* los números no "bailan" al cambiar */
}
```

### A.9 Lo que ya está bien y lo que mejoraría

| ✅ Bien hecho | 🔧 Mejora posible |
|--------------|-------------------|
| Componentes pequeños, uno por archivo, exports nombrados | Usar `<Footer />` en `App` (y darle `className="footer"`) |
| Props tipadas con TypeScript | Extraer `type Course` a `src/types.ts` y tipar `courses: Course[]` |
| Actualizaciones de estado funcionales | Añadir un `id` a cada curso y usarlo como `key` |
| `type="button"` en todos los botones | `<html lang="es">` y `<title>Academia</title>` en `index.html` |
| `&&` con condición booleana (sin el bug del `0`) | `aria-pressed={isInterested}` en el botón "Me interesa" (es un *toggle*) |
| Lint con `rules-of-hooks` como error | Limpiar CSS sobrante de la plantilla (`#social`, `.counter`) |
| Grid responsive sin media queries | Cargar los cursos desde una API (Parte B) |

---

## PARTE B — Versión full stack: catálogo de productos con tu API

Mismo patrón que `academia`, pero los datos vienen de
`express-mongo-atlas-render` (Express + MongoDB Atlas, desplegada en Render).

### B.1 Arquitectura

```
React (Vite, :5173)                         Express (:3000 / Render)        Atlas
┌───────────────────────────────┐           ┌─────────────────────────┐     ┌──────────┐
│ App  products[], loading, err │ GET ────▶ │ GET    /api/products    │ ──▶ │ products │
│ ├─ ProductForm  (onCreated)   │ POST ───▶ │ POST   /api/products    │     │          │
│ ├─ SearchBar    (search)      │ ?search=  │ PATCH  /api/products/:id│     │          │
│ └─ ProductList  (onDelete)    │ DELETE ─▶ │ DELETE /api/products/:id│     │          │
│     └─ ProductCard × N        │           └─────────────────────────┘     └──────────┘
└───────────────────────────────┘
```

### B.2 Estructura

```
productos-front/
├── .env.development        VITE_API_URL=http://localhost:3000
├── .env.production         VITE_API_URL=https://<tu-api>.onrender.com
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── types.ts
    ├── api.ts              ← todas las llamadas fetch en un sitio (capa de servicios)
    └── components/
        ├── ProductForm.tsx
        ├── ProductList.tsx
        └── ProductCard.tsx
```

> 📌 `api.ts` es la misma idea que la **arquitectura por capas** de tu backend
> (`EXPRESS/06`): los componentes no saben de URLs ni de `fetch`, solo llaman a
> funciones.

### B.3 `src/types.ts`

```ts
// Igual que el type Product de tu API + el _id que añade Mongo
export type Product = {
  _id: string
  name: string
  price: number
  category: string
}

export type NewProduct = Omit<Product, '_id'>          // utility type (TYPESCRIPT/05)
export type ProductChanges = Partial<NewProduct>        // para PATCH
```

### B.4 `src/api.ts`

```ts
import type { NewProduct, Product, ProductChanges } from './types'

const API_URL = import.meta.env.VITE_API_URL

// Helper: hace la petición, y si la API responde con error lanza su mensaje
async function request<T>(path: string, options: RequestInit = {}): Promise<T> {
  const res = await fetch(`${API_URL}${path}`, {
    ...options,
    headers: { 'Content-Type': 'application/json', ...options.headers },
  })

  if (res.status === 204) return undefined as T          // DELETE: sin cuerpo

  const data = await res.json()
  if (!res.ok) throw new Error(data.error ?? `Error ${res.status}`)   // { error } de tu API
  return data as T
}

export const productsApi = {
  list: (search = '') =>
    request<Product[]>(`/api/products${search ? `?search=${encodeURIComponent(search)}` : ''}`),

  create: (product: NewProduct) =>
    request<Product>('/api/products', { method: 'POST', body: JSON.stringify(product) }),

  update: (id: string, changes: ProductChanges) =>
    request<Product>(`/api/products/${id}`, { method: 'PATCH', body: JSON.stringify(changes) }),

  remove: (id: string) =>
    request<void>(`/api/products/${id}`, { method: 'DELETE' }),
}
```

### B.5 `src/App.tsx`

```tsx
import { useEffect, useState } from 'react'
import { productsApi } from './api'
import type { Product } from './types'
import { ProductForm } from './components/ProductForm'
import { ProductList } from './components/ProductList'
import './App.css'

function App() {
  const [products, setProducts] = useState<Product[]>([])
  const [search, setSearch] = useState('')
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  // Carga inicial y cada vez que cambia la búsqueda
  useEffect(() => {
    let ignore = false                     // evita pisar con respuestas viejas (race condition)
    setLoading(true)

    productsApi
      .list(search)
      .then(data => { if (!ignore) { setProducts(data); setError(null) } })
      .catch(err => { if (!ignore) setError(err.message) })
      .finally(() => { if (!ignore) setLoading(false) })

    return () => { ignore = true }
  }, [search])

  function handleCreated(p: Product) {
    setProducts(prev => [...prev, p])
  }

  async function handleDelete(id: string) {
    try {
      await productsApi.remove(id)
      setProducts(prev => prev.filter(p => p._id !== id))
    } catch (err) {
      alert(err instanceof Error ? err.message : 'No se pudo borrar')
    }
  }

  async function handlePriceChange(id: string, price: number) {
    const updated = await productsApi.update(id, { price })
    setProducts(prev => prev.map(p => (p._id === id ? updated : p)))
  }

  const total = products.reduce((sum, p) => sum + p.price, 0)   // estado derivado

  return (
    <main className="catalog">
      <h1>Productos</h1>

      <ProductForm onCreated={handleCreated} />

      <input
        type="search"
        aria-label="Buscar productos"
        placeholder="Buscar por nombre…"
        value={search}
        onChange={e => setSearch(e.target.value)}
      />

      {loading && <p>Cargando…</p>}
      {error && <p role="alert">Error: {error}</p>}
      {!loading && !error && (
        <>
          <p>{products.length} productos · {total.toFixed(2)} € en total</p>
          <ProductList products={products} onDelete={handleDelete} onPriceChange={handlePriceChange} />
        </>
      )}
    </main>
  )
}

export default App
```

> 💡 Aquí se usa la bandera `ignore` en vez de `AbortController` (`06`): son las
> dos formas habituales de evitar que una respuesta antigua pise a la nueva.

### B.6 `src/components/ProductList.tsx` y `ProductCard.tsx`

```tsx
import type { Product } from '../types'
import { ProductCard } from './ProductCard'

type Props = {
  products: Product[]
  onDelete: (id: string) => void
  onPriceChange: (id: string, price: number) => void
}

export function ProductList({ products, onDelete, onPriceChange }: Props) {
  if (products.length === 0) return <p>No hay productos.</p>

  return (
    <div className="course-list">
      {products.map(p => (
        <ProductCard key={p._id} product={p} onDelete={onDelete} onPriceChange={onPriceChange} />
      ))}
    </div>
  )
}
```

```tsx
import { useState } from 'react'
import type { Product } from '../types'

type Props = {
  product: Product
  onDelete: (id: string) => void
  onPriceChange: (id: string, price: number) => void
}

export function ProductCard({ product, onDelete, onPriceChange }: Props) {
  const [editing, setEditing] = useState(false)
  const [price, setPrice] = useState(String(product.price))

  function save() {
    const n = Number(price)
    if (!Number.isNaN(n) && n >= 0) onPriceChange(product._id, n)
    setEditing(false)
  }

  return (
    <article className="card">
      {product.category !== 'sin-categoria' && <span className="badge">{product.category}</span>}
      <h2>{product.name}</h2>

      {editing ? (
        <>
          <input type="number" min="0" value={price} onChange={e => setPrice(e.target.value)} />
          <button type="button" onClick={save}>Guardar</button>
        </>
      ) : (
        <p onDoubleClick={() => setEditing(true)}>{product.price} €</p>
      )}

      <button type="button" onClick={() => setEditing(e => !e)}>
        {editing ? 'Cancelar' : 'Editar precio'}
      </button>
      <button type="button" onClick={() => onDelete(product._id)}>Borrar</button>
    </article>
  )
}
```

> 📌 `'sin-categoria'` es el valor por defecto que pone **tu API** en el POST
> (`body.category?.trim() ?? "sin-categoria"`). El frontend tiene que conocer el
> "contrato" del backend.

### B.7 `src/components/ProductForm.tsx`

Es el formulario controlado del documento `07` (§7.2), usando
`productsApi.create(...)` en lugar del `fetch` directo:

```tsx
const created = await productsApi.create({
  name: form.name.trim(),
  price: Number(form.price),
  category: form.category || 'sin-categoria',
})
onCreated(created)
```

### B.8 Lo que hay que tocar en la API para que funcione

```ts
// express-mongo-atlas-render/src/index.ts — hoy permite solo http://127.0.0.1:5500
res.setHeader("Access-Control-Allow-Origin", process.env.CORS_ORIGIN ?? "http://localhost:5173");
```

| Entorno | `CORS_ORIGIN` en la API | `VITE_API_URL` en el front |
|---------|-------------------------|----------------------------|
| Local | `http://localhost:5173` | `http://localhost:3000` |
| Render | `https://<tu-front>.onrender.com` | `https://<tu-api>.onrender.com` |

> ⚠️ Con el plan gratuito, la **primera** carga tras 15 min sin uso tardará ~50 s
> (la API "despierta"). Tu estado `loading` es justo lo que hace que la app no
> parezca rota mientras tanto.

---

## PARTE C — Ejercicios para practicar sobre `academia`

| # | Ejercicio | Practicas |
|---|-----------|-----------|
| 1 | Usa `<Footer />` en `App` | Importar y componer |
| 2 | Crea `src/types.ts` con `Course` y tipa el array | Tipos compartidos, `import type` |
| 3 | Añade un buscador por título | Input controlado + estado derivado con `filter` |
| 4 | Añade botones de categoría ("Todas", "Programación", "Productividad") | Estado + `Set` + clases condicionales |
| 5 | Muestra "Te interesan N cursos" en `App` | **Lifting state up**: mover `isInterested` al padre |
| 6 | Guarda los intereses en `localStorage` | `useEffect` / custom hook `useLocalStorage` |
| 7 | Botón "Reservar plaza" que reste 1 a `seats` | Estado de un array de objetos, `map` inmutable |
| 8 | Carga los cursos desde una API (o un `cursos.json` en `public/`) | `useEffect` + `fetch` + loading/error |
| 9 | Página de detalle `/cursos/:slug` | React Router, `useParams` |
| 10 | Despliega en Render como Static Site | Build, `dist`, variables `VITE_` |

> 💡 El ejercicio 5 es el más importante: es exactamente lo que te preguntarán
> como "¿cómo compartes estado entre dos componentes?".
