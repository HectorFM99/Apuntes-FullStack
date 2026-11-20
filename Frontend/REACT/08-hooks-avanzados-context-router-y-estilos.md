# 08 — HOOKS AVANZADOS, CONTEXT, ROUTER Y ESTILOS

> Con `useState` y `useEffect` haces el 80 % de las apps. Este documento cubre
> lo que aparece **cuando la app crece**: más hooks, estado global, varias
> "páginas" y cómo organizar los estilos.

---

## 8.1 Las reglas de los hooks ⭐

Un **hook** es una función de React que empieza por `use` (`useState`,
`useEffect`…). Dos reglas:

1. **Solo en el nivel superior** del componente: nunca dentro de `if`, bucles ni funciones anidadas.
2. **Solo en componentes de React o en otros hooks** (custom hooks), no en funciones normales.

```tsx
function CourseCard({ seats }: Props) {
  if (seats === 0) return <p>Completo</p>
  const [isInterested, setIsInterested] = useState(false)   // ❌ hook después de un return condicional
}
```

**¿Por qué?** React identifica cada hook por su **orden de llamada**. Si un render
llama a 2 hooks y el siguiente a 1, React mezcla sus estados.

### En tu proyecto (REAL, `academia/.oxlintrc.json`)

```json
{
  "plugins": ["react", "typescript", "oxc"],
  "rules": {
    "react/rules-of-hooks": "error",
    "react/only-export-components": ["warn", { "allowConstantExport": true }]
  }
}
```

> 📌 `rules-of-hooks` como **error**: el linter no te deja romper esta regla.

---

## 8.2 `useRef`: una "caja" que no provoca render

```tsx
const ref = useRef(valorInicial)   // ref.current
```

| Uso | Ejemplo |
|-----|---------|
| Acceder a un elemento del DOM | Hacer `focus()` en un input, medir un elemento, hacer scroll |
| Guardar un valor que **no** se pinta | El id de un `setInterval`, el valor anterior, un contador de clics interno |

```tsx
function SearchBox() {
  const inputRef = useRef<HTMLInputElement>(null)

  return (
    <>
      <input ref={inputRef} />
      <button type="button" onClick={() => inputRef.current?.focus()}>Buscar</button>
    </>
  )
}
```

| | `useState` | `useRef` |
|---|---|---|
| Cambiarlo provoca render | ✅ Sí | ❌ No |
| Se conserva entre renders | ✅ | ✅ |
| Se lee | Directamente | `ref.current` |
| Para | Lo que se ve | Lo que no se ve / el DOM |

> 💡 En React 19, `ref` se puede pasar como **prop normal** a tus componentes
> (antes hacía falta `forwardRef`).

---

## 8.3 `useMemo` y `useCallback`: memorizar

```tsx
// useMemo: guarda el RESULTADO de un cálculo caro
const visible = useMemo(
  () => courses.filter(c => c.title.includes(query)).toSorted((a, b) => a.price - b.price),
  [courses, query]          // solo se recalcula si cambian
)

// useCallback: guarda la misma FUNCIÓN entre renders
const handleDelete = useCallback((id: string) => {
  setProducts(prev => prev.filter(p => p._id !== id))
}, [])
```

| Hook | Memoriza | Útil cuando |
|------|----------|-------------|
| `useMemo` | Un **valor** | El cálculo es caro (miles de elementos) |
| `useCallback` | Una **función** | Se la pasas a un hijo envuelto en `memo()` o a las deps de un efecto |
| `memo(Componente)` | El **render** de un componente | Recibe las mismas props a menudo y es caro |

> 📌 **No los pongas por defecto**: añaden complejidad. Primero haz que funcione;
> optimiza si notas lentitud (React DevTools → *Profiler*). Además, el **React
> Compiler** (mencionado en el `README.md` de tu `academia`) puede añadir esta
> memorización **automáticamente**.

---

## 8.4 Custom hooks: reutilizar lógica

Si copias el mismo `useState` + `useEffect` en varios componentes, extráelo a un
hook propio (función que empieza por `use` y usa otros hooks):

```tsx
// src/hooks/useFetch.ts
import { useEffect, useState } from 'react'

export function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    const controller = new AbortController()
    setLoading(true)
    setError(null)

    fetch(url, { signal: controller.signal })
      .then(res => {
        if (!res.ok) throw new Error(`Error ${res.status}`)
        return res.json() as Promise<T>
      })
      .then(setData)
      .catch(err => { if (err.name !== 'AbortError') setError(err.message) })
      .finally(() => setLoading(false))

    return () => controller.abort()
  }, [url])

  return { data, loading, error }
}
```

```tsx
// Uso: una línea en cualquier componente
const { data: products, loading, error } = useFetch<Product[]>(`${API_URL}/api/products`)
```

> 💡 Usa **genéricos** (`<T>`, `TYPESCRIPT/04`) para que el hook sirva para
> productos, categorías o libros.

Otro clásico: estado que se guarda en `localStorage`.

```tsx
export function useLocalStorage<T>(key: string, initial: T) {
  const [value, setValue] = useState<T>(() => {
    const saved = localStorage.getItem(key)
    return saved ? (JSON.parse(saved) as T) : initial
  })
  useEffect(() => { localStorage.setItem(key, JSON.stringify(value)) }, [key, value])
  return [value, setValue] as const
}

// En CourseCard: que "Me interesa" sobreviva a recargar la página
const [isInterested, setIsInterested] = useLocalStorage(`interest:${title}`, false)
```

---

## 8.5 Context: datos "globales" sin prop drilling

```tsx
// src/context/AuthContext.tsx
import { createContext, useContext, useState } from 'react'
import type { ReactNode } from 'react'

type User = { id: string; email: string }
type AuthValue = {
  user: User | null
  login: (u: User) => void
  logout: () => void
}

const AuthContext = createContext<AuthValue | null>(null)

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null)
  return (
    <AuthContext value={{ user, login: setUser, logout: () => setUser(null) }}>
      {children}
    </AuthContext>
  )
}

export function useAuth() {
  const ctx = useContext(AuthContext)
  if (!ctx) throw new Error('useAuth debe usarse dentro de <AuthProvider>')
  return ctx
}
```

```tsx
// main.tsx
<StrictMode>
  <AuthProvider>
    <App />
  </AuthProvider>
</StrictMode>

// Cualquier componente, a cualquier profundidad
function Header() {
  const { user, logout } = useAuth()
  return user ? <button onClick={logout}>Salir ({user.email})</button> : <a href="/login">Entrar</a>
}
```

> 📌 En React 19 el *provider* se escribe `<AuthContext value={...}>`; en versiones
> anteriores `<AuthContext.Provider value={...}>` (lo verás en muchos tutoriales).

| Context sirve para | No sirve para |
|--------------------|---------------|
| Usuario logueado, tema claro/oscuro, idioma | Estado que cambia muy a menudo y lo leen muchos (cada cambio re-renderiza a **todos** los consumidores) |
| Datos que leen muchos componentes y cambian poco | Sustituir **todas** las props |

> 💡 Cuando Context se queda corto (mucho estado compartido que cambia a menudo)
> entra **Redux Toolkit** → `11` y `12`.

### Conectado con tu autenticación (`autenticacion`, `PARA ENTREVISTAS/07`)

| Dónde guardar el JWT en el frontend | Pros | Contras |
|-------------------------------------|------|---------|
| Cookie `HttpOnly` (la pone el servidor) | JS no puede leerla → protegida de **XSS** | Hay que pensar en CSRF y en `credentials: 'include'` |
| `localStorage` | Muy sencillo | Cualquier script inyectado (XSS) puede robarlo |

```tsx
fetch(`${API_URL}/api/me`, { headers: { Authorization: `Bearer ${token}` } })
```

---

## 8.6 Varias "páginas": React Router

Una SPA tiene un solo HTML, pero el usuario espera URLs (`/cursos/react`) que se
puedan compartir y el botón "atrás". Eso lo hace **React Router**:

```bash
npm install react-router
```

```tsx
// main.tsx
import { BrowserRouter, Routes, Route } from 'react-router'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<Catalog />} />
          <Route path="cursos/:slug" element={<CourseDetail />} />
          <Route path="*" element={<NotFound />} />
        </Route>
      </Routes>
    </BrowserRouter>
  </StrictMode>,
)
```

```tsx
import { Link, NavLink, Outlet, useParams, useNavigate } from 'react-router'

function Layout() {
  return (
    <>
      <nav>
        <NavLink to="/">Catálogo</NavLink>     {/* añade la clase "active" sola */}
      </nav>
      <Outlet />                                {/* aquí se pinta la ruta hija */}
      <Footer />
    </>
  )
}

function CourseDetail() {
  const { slug } = useParams()                 // /cursos/react → slug = "react"
  const navigate = useNavigate()
  return (
    <>
      <h1>Curso: {slug}</h1>
      <button onClick={() => navigate(-1)}>Volver</button>
    </>
  )
}
```

| Herramienta | Para qué |
|-------------|----------|
| `<Link to="/x">` | Navegar **sin recargar** (no uses `<a href>` para rutas internas) |
| `useParams()` | Leer `:id` de la URL (como `req.params` en Express) |
| `useSearchParams()` | Leer/escribir `?search=` (como `req.query`) |
| `useNavigate()` | Navegar desde código (tras un login, tras crear algo) |
| `<Outlet />` | Layout compartido con rutas anidadas |

> ⚠️ Al desplegar una SPA con rutas hay que decirle al servidor que **cualquier
> ruta** devuelva `index.html` (si no, recargar en `/cursos/react` da 404). Ver `09`.

---

## 8.7 Formas de dar estilos

| Forma | Ejemplo | Ámbito | Notas |
|-------|---------|--------|-------|
| **CSS global** importado (tu `academia`) | `import './App.css'` + `className="card"` | **Global**: `.card` afecta a toda la app | Simple; riesgo de choques de nombres → usa nombres claros o BEM (`card__title`) |
| **CSS Modules** | `import s from './Card.module.css'` + `className={s.card}` | Local al componente | Vite lo soporta sin configurar; genera nombres únicos |
| **Tailwind CSS** | `className="p-6 rounded-xl shadow"` | Utilidades | Muy usado en empresas; no escribes CSS aparte |
| Estilos en línea | `style={{ color: 'red' }}` | Ese elemento | Para valores **dinámicos** (`width: \`${pct}%\``), no para todo |
| CSS-in-JS (styled-components…) | `` const Card = styled.article`...` `` | Componente | Menos usado hoy |

### Tu `index.css` REAL: variables y modo oscuro

```css
:root {
  --text: #6b6375;
  --bg: #fff;
  --accent: #aa3bff;
  --accent-bg: rgba(170, 59, 255, 0.1);
  color-scheme: light dark;
}

@media (prefers-color-scheme: dark) {
  :root {
    --text: #9ca3af;
    --bg: #16171d;
    --accent: #c084fc;
  }
}
```

Y el `App.css` las usa:

```css
.badge {
  color: var(--accent);
  background: var(--accent-bg);
  border-radius: 999px;     /* "píldora" */
}
```

> 📌 Las **variables CSS** (`CSS/09`) + `prefers-color-scheme` dan modo oscuro
> **sin tocar React**. React solo pone `className`; el CSS sigue siendo CSS.

> 💡 Curiosidad del proyecto: `index.css` aún tiene reglas de la plantilla de
> Vite que no se usan (`#social .button-icon`, `.counter`). No molestan, pero en
> un proyecto real se limpian.

---

## 8.8 `StrictMode`, DevTools y depuración

| Herramienta | Qué te da |
|-------------|-----------|
| `<StrictMode>` (en tu `main.tsx`) | En desarrollo: doble render y doble efecto para destapar impurezas y efectos sin limpieza |
| **React Developer Tools** (extensión) | Pestaña *Components*: ver el árbol, las props y el estado de cada componente y editarlos en vivo. *Profiler*: qué se re-renderiza y cuánto tarda |
| Pestaña *Network* | Ver los `fetch` a tu API, status y respuesta |
| `console.log` en el render | Ver cuándo y cuántas veces se renderiza |

### ¿Cuándo se re-renderiza un componente?

1. Cambia **su estado**.
2. Se re-renderiza **su padre** (aunque las props no cambien).
3. Cambia un **context** que usa.

> 📌 Re-renderizar **no** significa tocar el DOM: React solo aplica las
> diferencias. Normalmente no es un problema de rendimiento.

---

## 🎯 Resumen

| Herramienta | Para qué |
|-------------|----------|
| Reglas de hooks | Siempre arriba, nunca condicionales |
| `useRef` | DOM o valores que no se pintan |
| `useMemo` / `useCallback` | Optimizar, solo si hace falta |
| Custom hook | Reutilizar lógica con estado (`useFetch`) |
| Context | Datos globales que cambian poco (usuario, tema) |
| React Router | URLs, `Link`, `useParams`, layouts |
| Estilos | Global, Modules, Tailwind; variables CSS para temas |
