# 📋 CHEATSHEET — REACT

> Chuleta rápida de comandos y código. Teoría en los documentos 01–10 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal

| Comando | Qué hace |
|---------|----------|
| `npm create vite@latest academia -- --template react-ts` | Crea proyecto React + TS con Vite |
| `npm install` | Instala dependencias |
| `npm run dev` | Servidor de desarrollo con HMR → `http://localhost:5173` |
| `npm run build` | `tsc -b` (tipos) + `vite build` → `dist/` |
| `npm run preview` | Sirve `dist/` en `http://localhost:4173` |
| `npm run lint` | Linter (oxlint en `academia`) |
| `npm install react-router` | Rutas |
| `npm install -D vitest @testing-library/react @testing-library/user-event` | Tests |

```
index.html → <div id="root"> → src/main.tsx (createRoot) → App.tsx → components/*.tsx
```

---

## 2. Montar la app (`main.tsx`)

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css";                 // CSS global
import App from "./App.tsx";

createRoot(document.getElementById("root")!).render(
  <StrictMode>                        {/* comprobaciones extra en desarrollo */}
    <App />
  </StrictMode>,
);
```

---

## 3. JSX y componentes

```tsx
export function Footer() {            // Mayúscula, devuelve JSX
  return (
    <>                                {/* fragment: varias raíces sin div */}
      <footer className="footer">     {/* className, no class */}
        <label htmlFor="email">Email</label>   {/* htmlFor, no for */}
        <img src={logo} alt="Logo" />          {/* etiquetas cerradas */}
        <p style={{ fontSize: 14 }}>{2026}</p> {/* style = objeto */}
        {/* comentario */}
      </footer>
    </>
  )
}

import { Footer } from './components/Footer'   // export con nombre
import App from './App.tsx'                     // export default
```

| `{}` acepta | No acepta |
|-------------|-----------|
| Variables, operaciones, ternarios, `.map()`, llamadas | `if`, `for`, declarar variables |

---

## 4. Props

```tsx
type CourseCardProps = {
  title: string
  price: number
  category?: string                       // opcional
  onEnroll: (title: string) => void       // callback
  children?: ReactNode                    // import type { ReactNode } from 'react'
}

export function CourseCard({ title, price, category = 'General', onEnroll }: CourseCardProps) {
  return <button onClick={() => onEnroll(title)}>{title} · {price} €</button>
}

<CourseCard title="React" price={150} onEnroll={handleEnroll} />  {/* números con {} */}
<CourseCard key={course.title} {...course} />                      {/* spread de un objeto */}
<Card title="Contador"><Counter /></Card>                          {/* children */}
```

---

## 5. Estado y eventos

```tsx
const [value, setValue] = useState(0)                    // number inferido
const [courses, setCourses] = useState<Course[]>([])     // array vacío: indica tipo
const [user, setUser] = useState<User | null>(null)
const [fav] = useState(() => leerLocalStorage())         // inicial perezoso

setValue(v => v + 1)                                     // depende del anterior
setIsOpen(o => !o)                                       // toggle

// Inmutable
setCourses(prev => [...prev, nuevo])                                   // añadir
setCourses(prev => prev.filter(c => c.id !== id))                      // borrar
setCourses(prev => prev.map(c => c.id === id ? { ...c, seats: 0 } : c)) // editar
setForm(prev => ({ ...prev, name: 'x' }))                              // objeto

<button type="button" onClick={handleClick}>          {/* ✅ referencia */}
<button onClick={() => handleDelete(id)}>             {/* ✅ con argumentos */}
<button onClick={handleClick()}>                      {/* ❌ se ejecuta al renderizar */}
```

---

## 6. Condicionales y listas

```tsx
{seats > 0 && seats <= 3 && <span className="badge">Últimas plazas</span>}  // o nada
{seats > 0 ? <p>{seats} plazas libres</p> : <p>Completo</p>}            // una u otra
{count && <p>…</p>}       // ❌ pinta "0" si count = 0 → usa count > 0 &&

if (loading) return <p>Cargando…</p>          // return temprano
if (error) return <p role="alert">{error}</p>

{courses.map(course => (
  <CourseCard key={course.id} {...course} />  // key única y estable (id de la BD)
))}

const visible = courses.filter(c => c.title.includes(query))   // estado derivado
<article className={seats === 0 ? 'card card--full' : 'card'}>  // clase condicional
```

---

## 7. `useEffect` y fetch

```tsx
useEffect(() => { /* … */ }, [])        // al montar
useEffect(() => { /* … */ }, [query])   // al montar y cuando cambie query
useEffect(() => {
  const id = setInterval(tick, 1000)
  return () => clearInterval(id)        // limpieza
}, [])

useEffect(() => {
  const controller = new AbortController()
  async function load() {
    try {
      const res = await fetch(`${import.meta.env.VITE_API_URL}/api/products`, { signal: controller.signal })
      if (!res.ok) throw new Error(`Error ${res.status}`)
      setProducts(await res.json())
    } catch (err) {
      if (err instanceof Error && err.name !== 'AbortError') setError(err.message)
    } finally {
      setLoading(false)
    }
  }
  load()
  return () => controller.abort()
}, [])
```

```bash
# .env.development / .env.production   (solo las VITE_* llegan al front, y son PÚBLICAS)
VITE_API_URL=http://localhost:3000
```

```ts
// vite.config.ts — proxy en desarrollo (evita CORS)
export default defineConfig({ plugins: [react()], server: { proxy: { '/api': 'http://localhost:3000' } } })
```

---

## 8. Formularios

```tsx
const [form, setForm] = useState({ name: '', price: '' })

function handleChange(e: ChangeEvent<HTMLInputElement>) {
  const { name, value } = e.target
  setForm(prev => ({ ...prev, [name]: value }))      // un handler para todos
}

async function handleSubmit(e: FormEvent<HTMLFormElement>) {
  e.preventDefault()                                  // no recargar
  await fetch(`${API_URL}/api/products`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name: form.name, price: Number(form.price) }),
  })
}

<form onSubmit={handleSubmit}>
  <input name="name" value={form.name} onChange={handleChange} />
  <input type="checkbox" checked={ok} onChange={e => setOk(e.target.checked)} />
  <button type="submit" disabled={sending}>Guardar</button>
</form>
```

---

## 9. Otros hooks y Context

```tsx
const inputRef = useRef<HTMLInputElement>(null)        // <input ref={inputRef} />
inputRef.current?.focus()

const visible = useMemo(() => filtrar(courses, q), [courses, q])   // valor memorizado
const handle = useCallback((id: string) => borrar(id), [])         // función memorizada

// Custom hook
function useFetch<T>(url: string) { /* useState + useEffect */ return { data, loading, error } }

// Context
const AuthContext = createContext<AuthValue | null>(null)
<AuthContext value={{ user, login, logout }}>{children}</AuthContext>   // React 19
const ctx = useContext(AuthContext)
```

Reglas de los hooks: solo en el nivel superior y solo en componentes o custom hooks.

---

## 10. React Router

```tsx
import { BrowserRouter, Routes, Route, Link, NavLink, Outlet, useParams, useNavigate } from 'react-router'

<BrowserRouter>
  <Routes>
    <Route path="/" element={<Layout />}>
      <Route index element={<Catalog />} />
      <Route path="cursos/:slug" element={<CourseDetail />} />
      <Route path="*" element={<NotFound />} />
    </Route>
  </Routes>
</BrowserRouter>

<Link to="/cursos/react">React</Link>    <Outlet />
const { slug } = useParams()             const navigate = useNavigate(); navigate(-1)
```

---

## 11. Tipos de React útiles

| Tipo | Para |
|------|------|
| `ReactNode` | `children` |
| `ChangeEvent<HTMLInputElement>` | `onChange` |
| `FormEvent<HTMLFormElement>` | `onSubmit` |
| `MouseEvent<HTMLButtonElement>` | `onClick` |
| `ComponentProps<'button'>` | Props de un elemento nativo |

```tsx
import type { ChangeEvent, FormEvent, ReactNode } from 'react'   // import type (verbatimModuleSyntax)
```

---

## 12. Despliegue (Render Static Site)

| Campo | Valor |
|-------|-------|
| Build Command | `npm install && npm run build` |
| Publish Directory | `dist` |
| Env | `VITE_API_URL=https://<tu-api>.onrender.com` (requiere rebuild si cambia) |
| Rewrite (con Router) | `/*` → `/index.html` |
| En la API | `CORS_ORIGIN=https://<tu-front>.onrender.com` |

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `Each child in a list should have a unique "key"` | `key={item.id}` en el elemento del `.map()` |
| `Objects are not valid as a React child` | Pinta un campo: `{p.name}`, no `{p}` |
| `Too many re-renders` | `onClick={() => setX(1)}`, no `onClick={setX(1)}` |
| La pantalla no se actualiza | Mutaste el estado (`push`): crea un array/objeto nuevo |
| Petición duplicada en desarrollo | `StrictMode`: normal; añade limpieza con `AbortController` |
| `blocked by CORS policy` | `CORS_ORIGIN` en la API o proxy de Vite |
| `VITE_API_URL` es `undefined` | Prefijo `VITE_` y reiniciar `npm run dev` |
| `dev` funciona, `build` falla | Errores de tipos o mayúsculas en imports: `npm run build` antes de subir |
