# 06 — `useEffect` Y CONSUMIR TUS APIS

> Hasta ahora los datos estaban **escritos en el código** (`const courses = [...]`).
> En una app real vienen de **tu backend**. Para pedirlos se usa `fetch`
> (`API/04`) dentro de un **efecto**: código que se ejecuta **después** de pintar.

---

## 6.1 Qué es un efecto secundario

El render debe ser **puro** (`02`). Pero hay cosas que **no son pintar**:

- Pedir datos a una API.
- Suscribirse a algo (un `setInterval`, un `addEventListener` en `window`, un WebSocket).
- Tocar algo fuera de React (`document.title`, `localStorage`).

Eso son **efectos secundarios** y van en `useEffect`.

---

## 6.2 `useEffect`

```tsx
import { useEffect } from 'react'

useEffect(() => {
  // 1. Código del efecto: se ejecuta DESPUÉS de que React pinte

  return () => {
    // 2. (Opcional) Limpieza: antes de repetir el efecto y al desmontar
  }
}, [dep1, dep2])   // 3. Dependencias: CUÁNDO repetirlo
```

### El array de dependencias

| Dependencias | Se ejecuta… | Uso típico |
|--------------|-------------|------------|
| `[]` | **Una vez**, al montar (aparecer) el componente | Cargar datos iniciales |
| `[query]` | Al montar **y** cada vez que `query` cambie | Buscar cuando cambia el filtro |
| *(sin array)* | Después de **cada** render | Casi nunca: suele ser un bug |

```tsx
// Ejemplo sencillo: cambiar el título de la pestaña
useEffect(() => {
  document.title = `${value} en el contador`
}, [value])
```

> 📌 Regla: **todo lo que el efecto use** de props o estado tiene que estar en las
> dependencias. El linter (`react-hooks/exhaustive-deps`) te avisa si falta.

### Ciclo de vida de un componente

```
MONTAR            ACTUALIZAR (cambia una dep)             DESMONTAR
render → pinta    render → pinta                          desaparece del árbol
  → efecto          → limpieza del efecto anterior           → limpieza
                    → efecto nuevo
```

### Limpieza: ejemplo con un intervalo

```tsx
function Clock() {
  const [now, setNow] = useState(() => new Date())

  useEffect(() => {
    const id = setInterval(() => setNow(new Date()), 1000)
    return () => clearInterval(id)    // sin esto, el intervalo sigue vivo al desmontar
  }, [])

  return <p>{now.toLocaleTimeString()}</p>
}
```

---

## 6.3 Cargar datos de tu API

### El backend: tu API REAL (`express-mongo-atlas-render/src/routes/products.ts`)

```ts
export type Product = {
  name: string;
  price: number;
  category: string;
};

// GET /api/products?category=muebles&search=sil
router.get("/", async (req, res) => {
  // ...filtros...
  const products = await productsCollection().find(filter).toArray();
  res.json(products);     // → [{ _id: "665f...", name: "Silla", price: 45, category: "muebles" }]
});
```

### El frontend: componente que la consume

```tsx
import { useEffect, useState } from 'react'

type Product = {
  _id: string        // Mongo lo devuelve como string en el JSON
  name: string
  price: number
  category: string
}

const API_URL = import.meta.env.VITE_API_URL   // ver 6.6

export function ProductList() {
  const [products, setProducts] = useState<Product[]>([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    async function load() {
      try {
        const res = await fetch(`${API_URL}/api/products`)
        if (!res.ok) throw new Error(`Error ${res.status}`)   // fetch NO lanza en 404/500
        const data: Product[] = await res.json()
        setProducts(data)
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Error desconocido')
      } finally {
        setLoading(false)
      }
    }
    load()
  }, [])

  if (loading) return <p>Cargando productos…</p>
  if (error) return <p role="alert">No se pudieron cargar: {error}</p>
  if (products.length === 0) return <p>No hay productos.</p>

  return (
    <ul>
      {products.map(p => (
        <li key={p._id}>{p.name} — {p.price} € ({p.category})</li>
      ))}
    </ul>
  )
}
```

| Detalle | Por qué |
|---------|---------|
| `async function load()` **dentro** del efecto | La función de `useEffect` **no puede** ser `async` (devolvería una promesa, y React espera una función de limpieza) |
| `if (!res.ok) throw` | `fetch` solo falla si **no hay red**; un 404 o 500 llega como respuesta "normal" (`API/04`) |
| 3 estados: `products`, `loading`, `error` | Los 4 casos de la UI: cargando / error / vacío / datos |
| `key={p._id}` | El id de Mongo: único y estable (`05`) |
| `err instanceof Error` | En TS el `catch` recibe `unknown` (`TYPESCRIPT/06`) |

---

## 6.4 Peticiones que dependen de un valor + cancelación

Tu API acepta `?search=` y `?category=`. Si el usuario escribe rápido, salen
varias peticiones y **pueden llegar desordenadas** (*race condition*): la
respuesta de "si" podría llegar después de la de "silla" y pisarla.

```tsx
export function ProductSearch() {
  const [search, setSearch] = useState('')
  const [products, setProducts] = useState<Product[]>([])

  useEffect(() => {
    const controller = new AbortController()

    const params = new URLSearchParams()
    if (search) params.set('search', search)

    fetch(`${API_URL}/api/products?${params}`, { signal: controller.signal })
      .then(res => res.json())
      .then((data: Product[]) => setProducts(data))
      .catch(err => {
        if (err.name !== 'AbortError') console.error(err)
      })

    return () => controller.abort()   // cancela la petición anterior
  }, [search])                         // ← se repite cada vez que cambia "search"

  return (
    <>
      <input value={search} onChange={e => setSearch(e.target.value)} placeholder="Buscar…" />
      <ul>{products.map(p => <li key={p._id}>{p.name}</li>)}</ul>
    </>
  )
}
```

> 💡 `URLSearchParams` codifica bien los caracteres especiales (`API/04`).
> 💡 Para no lanzar una petición por tecla se usa *debounce* (esperar ~300 ms a
> que el usuario deje de escribir).

---

## 6.5 `StrictMode` y "mi petición sale dos veces"

En desarrollo, `<StrictMode>` (lo tienes en `main.tsx`) **monta, desmonta y
vuelve a montar** cada componente para comprobar que tus efectos tienen una
limpieza correcta. Por eso en la pestaña *Network* verás **dos** `GET`.

- ✅ Es normal y **solo pasa en desarrollo**; en el build de producción, una.
- ✅ Con el `AbortController` de 6.4, la primera se cancela.
- ❌ No quites `StrictMode` para "arreglarlo".

---

## 6.6 Variables de entorno en Vite: `VITE_API_URL`

La URL de la API cambia entre local y producción (igual que `DATABASE_URL` en el
backend, `RENDER/04`).

```bash
# .env.development  (npm run dev)
VITE_API_URL=http://localhost:3000

# .env.production   (npm run build)
VITE_API_URL=https://<tu-api>.onrender.com
```

```tsx
const API_URL = import.meta.env.VITE_API_URL
```

| Backend (Node) | Frontend (Vite) |
|----------------|-----------------|
| `process.env.X` | `import.meta.env.VITE_X` |
| Se lee al **arrancar** el servidor | Se **incrusta en el JS** al hacer el **build** |
| Puede tener secretos | ⚠️ **Todo es público**: solo se exponen las que empiezan por `VITE_` y cualquiera puede verlas en el bundle |

> ⚠️ **Nunca** pongas en un `.env` de frontend contraseñas, la `MONGODB_URI`, la
> `service_role` de Supabase ni claves de pago. Eso va **solo** en el backend.

Para tiparlas (opcional), en `src/vite-env.d.ts`:

```ts
interface ImportMetaEnv {
  readonly VITE_API_URL: string
}
```

---

## 6.7 CORS: el error que verás seguro

Tu frontend vive en `http://localhost:5173` y tu API en `http://localhost:3000`:
**orígenes distintos** → el navegador aplica CORS (`API/06`).

### Lo que tiene hoy tu API REAL (`express-mongo-atlas-render/src/index.ts`)

```ts
app.use((req, res, next) => {
  res.setHeader("Access-Control-Allow-Origin", "http://127.0.0.1:5500");   // Live Server
  res.setHeader("Access-Control-Allow-Methods", "GET, POST, PATCH, DELETE");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type");
  if (req.method === "OPTIONS") { res.sendStatus(204); return; }
  next();
});
```

Estaba pensada para una web con **Live Server** (puerto 5500). Desde React verías:

```
Access to fetch at 'http://localhost:3000/api/products' from origin
'http://localhost:5173' has been blocked by CORS policy
```

**Dos soluciones:**

| Solución | Cómo | Cuándo |
|----------|------|--------|
| A. Permitir el origen en la API | `Access-Control-Allow-Origin` = `process.env.CORS_ORIGIN ?? "http://localhost:5173"` | Siempre que front y API estén en dominios distintos (producción) |
| B. **Proxy** de Vite | El navegador pide a `5173/api/...` y Vite lo reenvía a `3000` (mismo origen → sin CORS) | Solo en desarrollo |

```ts
// vite.config.ts  (solución B)
export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': 'http://localhost:3000',
    },
  },
})
```
```tsx
fetch('/api/products')    // con proxy, URL relativa
```

> 📌 CORS lo **aplica el navegador** y se **arregla en el servidor**. Por eso con
> `curl` o Thunder Client funciona y desde React no.

---

## 6.8 Errores comunes con `useEffect`

| Síntoma | Causa | Arreglo |
|---------|-------|---------|
| Bucle infinito de peticiones | `setX` dentro de un efecto **sin** array de dependencias, o con un objeto/array nuevo en las deps | Pon `[]` o deps primitivas |
| *Too many re-renders* | Llamas a `setX` **durante el render** (o `onClick={setX(1)}`) | Muévelo a un evento o efecto |
| El efecto usa un valor viejo | Falta esa variable en las dependencias | Añádela (hazle caso al linter) |
| `useEffect(async () => ...)` | Devuelve una promesa | Función `async` **dentro** |
| `products.map is not a function` | La API devolvió un **objeto** (`{ error: ... }`) y no un array | Comprueba `res.ok` antes de `setProducts` |

### ¿Necesito realmente un efecto?

```tsx
// ❌ Efecto para calcular algo derivado (render extra + más código)
const [total, setTotal] = useState(0)
useEffect(() => { setTotal(products.reduce((s, p) => s + p.price, 0)) }, [products])

// ✅ Calcúlalo en el render
const total = products.reduce((s, p) => s + p.price, 0)
```

> 📌 `useEffect` es para **sincronizarse con algo de fuera** de React. Si no hay
> "algo de fuera", probablemente no lo necesitas.

---

## 6.9 Más allá: librerías de datos

En proyectos reales se suele usar **TanStack Query** (React Query) o SWR, que te
dan caché, reintentos, recarga en segundo plano y estados `isLoading` / `error`
sin escribir el `useEffect` a mano. Merece la pena saber que existen, pero
**primero** entiende el patrón manual de este documento.

---

## 🎯 Resumen

| Concepto | Clave |
|----------|-------|
| `useEffect(fn, deps)` | Código **después** de pintar, para sincronizarse con el exterior |
| `[]` | Una vez al montar |
| Limpieza | `return () => ...` (intervalos, suscripciones, `abort()`) |
| `fetch` | `async` dentro del efecto, comprobar `res.ok`, estados loading/error |
| `VITE_API_URL` | Config por entorno; **todo** lo del frontend es público |
| CORS | Se arregla en la API (`CORS_ORIGIN`) o con el proxy de Vite en desarrollo |
