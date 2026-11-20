# 12 — REDUX ASÍNCRONO: THUNKS Y RTK QUERY

> Los reducers son **puros**: no pueden hacer `fetch`. Entonces, ¿cómo pedimos los
> productos a tu API de Express/NestJS con Redux? Dos caminos:
>
> 1. **`createAsyncThunk`** → tú controlas la petición y guardas el resultado en un slice.
> 2. **RTK Query** → describes los *endpoints* y Redux hace el `fetch`, la caché,
>    el loading y el refresco por ti. ⭐ La opción recomendada hoy para datos de API.
>
> 📌 Backend de ejemplo: tu API REAL `express-mongo-atlas-render`
> (`GET/POST /api/products`, ver `06`).

---

## 12.1 ¿Dónde va el código asíncrono? → Thunks

```
 ❌ reducer: (state, action) => { fetch(...) }      ← impuro, PROHIBIDO

 ✅ thunk: una función que se "despacha" y puede hacer cosas async
           y despachar acciones normales cuando termina

   dispatch(fetchProducts())
        │
        ├─▶ dispatch({ type: 'products/fetchAll/pending' })     → status = 'loading'
        ├─▶ await fetch('/api/products')
        └─▶ dispatch({ type: 'products/fetchAll/fulfilled', payload: [...] })
            o dispatch({ type: 'products/fetchAll/rejected', error })
```

> 📌 Un **thunk** es "una función que devuelve una función". El middleware de
> thunk (incluido en `configureStore`) detecta que despachas una **función** en
> vez de un objeto y la ejecuta pasándole `dispatch` y `getState`.

---

## 12.2 `createAsyncThunk` + `extraReducers` ⭐⭐

```ts
// src/features/products/productsSlice.ts
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit'
import type { RootState } from '../../app/store'

export type Product = {
  _id: string
  name: string
  price: number
  category: string
}

type ProductsState = {
  items: Product[]
  status: 'idle' | 'loading' | 'succeeded' | 'failed'   // mejor que varios booleanos
  error: string | null
}

const initialState: ProductsState = { items: [], status: 'idle', error: null }

const API_URL = import.meta.env.VITE_API_URL

// ─── THUNK: GET /api/products ────────────────────────────────────────────
export const fetchProducts = createAsyncThunk<
  Product[],                 // 1. lo que devuelve si va bien (payload de fulfilled)
  void,                      // 2. argumento que recibe (ninguno)
  { rejectValue: string }    // 3. tipo del error si usamos rejectWithValue
>(
  'products/fetchAll',       // prefijo → genera /pending, /fulfilled, /rejected
  async (_, { signal, rejectWithValue }) => {
    const res = await fetch(`${API_URL}/api/products`, { signal })
    if (!res.ok) return rejectWithValue(`Error ${res.status}`)
    return (await res.json()) as Product[]
  },
  {
    // No volver a pedir si ya está cargando o cargado (útil con StrictMode)
    condition: (_, { getState }) => {
      const { status } = (getState() as RootState).products
      return status === 'idle' || status === 'failed'
    },
  },
)

// ─── THUNK: POST /api/products ───────────────────────────────────────────
export const createProduct = createAsyncThunk<
  Product,
  Omit<Product, '_id'>,      // recibe { name, price, category }
  { rejectValue: string }
>('products/create', async (newProduct, { rejectWithValue }) => {
  const res = await fetch(`${API_URL}/api/products`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(newProduct),
  })
  if (!res.ok) {
    const body = await res.json().catch(() => null)
    return rejectWithValue(body?.error ?? `Error ${res.status}`)   // mensaje de tu API
  }
  return (await res.json()) as Product
})

// ─── SLICE ───────────────────────────────────────────────────────────────
const productsSlice = createSlice({
  name: 'products',
  initialState,
  reducers: {},              // acciones síncronas (si las hubiera)
  // extraReducers: responder a acciones creadas FUERA del slice (los thunks)
  extraReducers: builder => {
    builder
      .addCase(fetchProducts.pending, state => {
        state.status = 'loading'
        state.error = null
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.status = 'succeeded'
        state.items = action.payload
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.status = 'failed'
        state.error = action.payload ?? action.error.message ?? 'Error desconocido'
      })
      .addCase(createProduct.fulfilled, (state, action) => {
        state.items.push(action.payload)       // Immer 👍
      })
  },
})

export default productsSlice.reducer

export const selectProducts = (state: RootState) => state.products.items
export const selectProductsStatus = (state: RootState) => state.products.status
export const selectProductsError = (state: RootState) => state.products.error
```

| `reducers` | `extraReducers` |
|------------|-----------------|
| Define acciones **nuevas** del slice (genera action creators) | Responde a acciones que **ya existen** (thunks, otros slices) |
| Objeto `{ nombre(state, action) {} }` | Función `builder => builder.addCase(...)` |

> ⚠️ En RTK 2 `extraReducers` **solo** acepta la forma `builder`. La forma de
> objeto `{ [fetchProducts.fulfilled]: ... }` de tutoriales viejos ya no funciona.

Registra el slice en el store:

```ts
// src/app/store.ts
reducer: {
  interests: interestsReducer,
  products: productsReducer,
},
```

---

## 12.3 Usarlo en un componente

Compara con el `ProductList` de `06`: los tres `useState` (data, loading, error)
desaparecen del componente y viven en el store.

```tsx
// src/features/products/ProductList.tsx
import { useEffect } from 'react'
import { useAppDispatch, useAppSelector } from '../../app/hooks'
import {
  fetchProducts, selectProducts, selectProductsError, selectProductsStatus,
} from './productsSlice'

export function ProductList() {
  const dispatch = useAppDispatch()
  const products = useAppSelector(selectProducts)
  const status = useAppSelector(selectProductsStatus)
  const error = useAppSelector(selectProductsError)

  useEffect(() => {
    dispatch(fetchProducts())      // la "condition" evita la doble petición
  }, [dispatch])

  if (status === 'loading') return <p>Cargando…</p>
  if (status === 'failed') return <p role="alert">❌ {error}</p>

  return (
    <ul>
      {products.map(p => (
        <li key={p._id}>{p.name} — {p.price} € ({p.category})</li>
      ))}
    </ul>
  )
}
```

### Esperar el resultado de un thunk: `.unwrap()`

En un formulario quieres saber si el POST fue bien para limpiar o mostrar el error:

```tsx
async function handleSubmit(e: FormEvent<HTMLFormElement>) {
  e.preventDefault()
  try {
    await dispatch(createProduct({ name, price: Number(price), category })).unwrap()
    setName(''); setPrice(''); setCategory('')        // ✅ fue bien
  } catch (err) {
    setFormError(err as string)                        // ❌ el rejectValue
  }
}
```

> 📌 Sin `.unwrap()`, `await dispatch(thunk())` **nunca lanza**: devuelve la
> acción `fulfilled` o `rejected`. `.unwrap()` la convierte en una promesa normal
> que lanza si falló.

---

## 12.4 RTK Query: la forma moderna de pedir datos ⭐⭐

Fíjate en todo lo que escribiste en `12.2` para **un** GET: estado `status`,
`error`, `pending/fulfilled/rejected`, evitar duplicados… y aún falta: caché,
refrescar la lista tras un POST, reintentar… **RTK Query** lo hace por ti.

### Definir la API

```ts
// src/features/products/productsApi.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import type { Product } from './productsSlice'   // o en un types.ts compartido

export const productsApi = createApi({
  reducerPath: 'productsApi',                     // dónde vive en el store
  baseQuery: fetchBaseQuery({ baseUrl: `${import.meta.env.VITE_API_URL}/api` }),
  tagTypes: ['Product'],                          // etiquetas para la caché
  endpoints: builder => ({
    // GET /api/products?category=...
    getProducts: builder.query<Product[], string | void>({
      query: category => (category ? `products?category=${category}` : 'products'),
      providesTags: ['Product'],                  // "estos datos son de tipo Product"
    }),
    // GET /api/products/:id
    getProduct: builder.query<Product, string>({
      query: id => `products/${id}`,
      providesTags: (_result, _err, id) => [{ type: 'Product', id }],
    }),
    // POST /api/products
    addProduct: builder.mutation<Product, Omit<Product, '_id'>>({
      query: body => ({ url: 'products', method: 'POST', body }),  // JSON automático
      invalidatesTags: ['Product'],               // → se vuelve a pedir getProducts
    }),
    // DELETE /api/products/:id
    deleteProduct: builder.mutation<void, string>({
      query: id => ({ url: `products/${id}`, method: 'DELETE' }),
      invalidatesTags: ['Product'],
    }),
  }),
})

// Hooks GENERADOS automáticamente: use + NombreEndpoint + Query/Mutation
export const {
  useGetProductsQuery,
  useGetProductQuery,
  useAddProductMutation,
  useDeleteProductMutation,
} = productsApi
```

### Añadirla al store (reducer **y** middleware)

```ts
// src/app/store.ts
import { configureStore } from '@reduxjs/toolkit'
import { productsApi } from '../features/products/productsApi'
import interestsReducer from '../features/interests/interestsSlice'

export const store = configureStore({
  reducer: {
    interests: interestsReducer,
    [productsApi.reducerPath]: productsApi.reducer,   // la caché
  },
  middleware: getDefaultMiddleware =>
    getDefaultMiddleware().concat(productsApi.middleware),  // ⚠️ obligatorio
})
```

### Usarla: el componente queda ridículamente corto

```tsx
import { useGetProductsQuery, useDeleteProductMutation } from './productsApi'

export function ProductList({ category }: { category?: string }) {
  const { data: products = [], isLoading, isError, error, isFetching } =
    useGetProductsQuery(category)
  const [deleteProduct, { isLoading: isDeleting }] = useDeleteProductMutation()

  if (isLoading) return <p>Cargando…</p>
  if (isError) return <p role="alert">❌ Error: {JSON.stringify(error)}</p>

  return (
    <ul style={{ opacity: isFetching ? 0.5 : 1 }}>   {/* refrescando en 2º plano */}
      {products.map(p => (
        <li key={p._id}>
          {p.name} — {p.price} €
          <button disabled={isDeleting} onClick={() => deleteProduct(p._id)}>🗑️</button>
        </li>
      ))}
    </ul>
  )
}
```

```tsx
// Formulario: la mutation devuelve una promesa con .unwrap()
const [addProduct, { isLoading }] = useAddProductMutation()

await addProduct({ name, price: Number(price), category }).unwrap()
// → invalida 'Product' → useGetProductsQuery vuelve a pedir la lista SOLO
```

### ¿Qué te da RTK Query gratis?

| Característica | Explicación |
|----------------|-------------|
| `isLoading` / `isFetching` / `isError` / `isSuccess` | `isLoading` = primera carga; `isFetching` = cualquier petición en curso |
| **Caché** | 2 componentes con `useGetProductsQuery()` → **1** sola petición |
| **Deduplicación** | Adiós al problema de la doble petición con `StrictMode` |
| **Invalidación por tags** | Tras un POST/DELETE, se re-piden solas las queries afectadas |
| Limpieza | Si nadie usa un dato durante 60 s (`keepUnusedDataFor`), se borra de la caché |
| `refetch()` | Botón "recargar" manual |
| `skip` | `useGetProductQuery(id, { skip: !id })` → no pedir hasta tener `id` |
| `pollingInterval` | `{ pollingInterval: 5000 }` → refrescar cada 5 s |

| Concepto | `query` | `mutation` |
|----------|---------|------------|
| Para | **Leer** (GET) | **Cambiar** (POST, PUT, PATCH, DELETE) |
| Se ejecuta | Sola al montar el componente | Cuando llamas a la función |
| Hook | `const { data } = useXQuery(arg)` | `const [hacerX, { isLoading }] = useXMutation()` |
| Caché | `providesTags` | `invalidatesTags` |

---

## 12.5 Enviar el JWT en cada petición

Conectado con tu autenticación (`08`, `PARA ENTREVISTAS/07`): un slice `auth`
guarda el token y RTK Query lo añade a **todas** las peticiones.

```ts
// src/features/auth/authSlice.ts
type AuthState = { token: string | null; email: string | null }

const authSlice = createSlice({
  name: 'auth',
  initialState: { token: null, email: null } as AuthState,
  reducers: {
    loggedIn(state, action: PayloadAction<{ token: string; email: string }>) {
      state.token = action.payload.token
      state.email = action.payload.email
    },
    loggedOut() {
      return { token: null, email: null }
    },
  },
})
```

```ts
// productsApi.ts
baseQuery: fetchBaseQuery({
  baseUrl: `${import.meta.env.VITE_API_URL}/api`,
  prepareHeaders: (headers, { getState }) => {
    const token = (getState() as RootState).auth.token
    if (token) headers.set('Authorization', `Bearer ${token}`)
    return headers
  },
  // credentials: 'include',   // si tu API usa cookie HttpOnly en vez de header
}),
```

> 📌 Es el equivalente en el frontend al middleware de autenticación de Express
> (`EXPRESS/04`) o a un *guard* de NestJS (`NESTJS/07`): un único sitio para algo
> que afecta a todas las rutas.

---

## 12.6 `createAsyncThunk` vs RTK Query

| | `createAsyncThunk` | RTK Query |
|---|---|---|
| Código | Mucho (thunk + extraReducers + status) | Poco (endpoints) |
| Caché y deduplicación | A mano | ✅ Automática |
| Refrescar tras mutar | A mano (`push`, volver a pedir…) | ✅ Tags |
| Control total del flujo | ✅ | Menos |
| Úsalo para | Lógica async que **no** es "pedir datos": login con varios pasos, procesos complejos, coordinar varias acciones | **Datos del servidor** (CRUD de tu API) |

> 🗣️ "Para datos de la API uso RTK Query; los slices normales los dejo para
> estado **de cliente** (carrito, filtros, UI, usuario). Así no duplico en Redux lo
> que ya vive en el servidor."

---

## 12.7 Redux vs las alternativas ⭐

| Herramienta | Tipo de estado | Ventajas | Inconvenientes |
|-------------|----------------|----------|----------------|
| `useState` / `useReducer` | Local | Nativo, simple | No se comparte lejos |
| **Context** | Global que cambia poco | Nativo, sin librerías | Re-render de todos los consumidores; no es un gestor de estado completo |
| **Redux Toolkit** | Global de cliente (+ servidor con RTK Query) | Predecible, DevTools, estándar en empresas, muy escalable | Más conceptos y código que las alternativas |
| **Zustand** | Global de cliente | Minimalista, sin Provider, muy poco código | Menos estructura en equipos grandes |
| **TanStack Query** | **Servidor** (caché de la API) | El estándar para datos remotos | No gestiona estado de cliente |

Zustand en 10 líneas, para que veas la diferencia:

```ts
import { create } from 'zustand'

type InterestsStore = { ids: string[]; toggle: (id: string) => void }

export const useInterests = create<InterestsStore>(set => ({
  ids: [],
  toggle: id => set(s => ({
    ids: s.ids.includes(id) ? s.ids.filter(x => x !== id) : [...s.ids, id],
  })),
}))

// En cualquier componente, sin Provider:
const count = useInterests(s => s.ids.length)
```

### ¿Qué elijo?

```
¿Es dato de la API?  ── sí ──▶ RTK Query / TanStack Query
        │ no
¿Lo usa un solo componente (o padre + hijos cercanos)? ── sí ──▶ useState / lifting
        │ no
¿Cambia poco (usuario, tema, idioma)? ── sí ──▶ Context
        │ no
Mucho estado compartido que cambia a menudo ──▶ Redux Toolkit (o Zustand)
```

---

## 12.8 Errores comunes

| Error / síntoma | Causa | Arreglo |
|-----------------|-------|---------|
| `could not find react-redux context value` | Componente fuera del `<Provider>` | Envuelve `<App />` en `main.tsx` |
| La UI no se actualiza tras el `dispatch` | Mutaste el estado **fuera** de Immer, o mutaste **y** devolviste | Dentro del slice: muta **o** devuelve, no ambas |
| `A non-serializable value was detected in the state` | Guardaste `Date`, `Map`, `Set`, una clase o una función | Guarda `date.toISOString()`, arrays, objetos planos |
| `Selector returned a different result when called with the same parameters` | Selector que devuelve `filter/map/{}` nuevo | `createSelector` o devuelve primitivos |
| La petición se hace 2 veces en desarrollo | `StrictMode` + `useEffect` con `dispatch(thunk())` | `condition` en el thunk, o RTK Query |
| `Middleware for RTK-Query API at reducerPath "productsApi" has not been added to the store` | Falta el `middleware` | `getDefaultMiddleware().concat(api.middleware)` |
| `Property 'products' does not exist on type ...` | Slice no registrado en `reducer: {}` | Añádelo al store (y `RootState` se actualiza solo) |
| `await dispatch(thunk())` no entra en el `catch` | Los thunks no lanzan | `.unwrap()` |
| `extraReducers` "no hace nada" / error de tipos | Sintaxis de objeto antigua | `builder.addCase(...)` |
| `useSelector` devuelve `unknown`/`any` | Usas el hook sin tipar | `useAppSelector` de `app/hooks.ts` |

---

## 12.9 Ejemplo completo: `academia` con Redux

Lo de `11` y `12` junto. Árbol de archivos nuevos/modificados:

```
academia/src/
├── app/
│   ├── store.ts                    ← interests + auth + productsApi
│   └── hooks.ts                    ← useAppSelector / useAppDispatch
├── features/
│   ├── interests/interestsSlice.ts ← toggleInterest, clearInterests, selectores
│   ├── auth/authSlice.ts           ← loggedIn, loggedOut
│   └── products/
│       ├── productsApi.ts          ← RTK Query: GET/POST/DELETE /api/products
│       └── ProductList.tsx
├── components/
│   ├── Header.tsx                  ← lee selectInterestCount
│   └── CourseCard.tsx              ← dispatch(toggleInterest(title))
├── App.tsx                         ← ya NO maneja estado de intereses
└── main.tsx                        ← <Provider store={store}>
```

```ts
// src/app/store.ts (versión final)
import { configureStore } from '@reduxjs/toolkit'
import interestsReducer from '../features/interests/interestsSlice'
import authReducer from '../features/auth/authSlice'
import { productsApi } from '../features/products/productsApi'

export const store = configureStore({
  reducer: {
    interests: interestsReducer,
    auth: authReducer,
    [productsApi.reducerPath]: productsApi.reducer,
  },
  middleware: getDefaultMiddleware =>
    getDefaultMiddleware().concat(productsApi.middleware),
})

export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch
```

```tsx
// src/App.tsx
import { Header } from './components/Header'
import { CourseCard } from './components/CourseCard'
import { ProductList } from './features/products/ProductList'
import { courses } from './data/courses'   // tu array REAL, movido a su archivo

export default function App() {
  return (
    <>
      <Header />                                   {/* sin props */}
      <main className="grid">
        {courses.map(course => (
          <CourseCard key={course.title} {...course} />   // igual que tu App REAL
        ))}
      </main>
      <ProductList />                              {/* datos de tu API */}
    </>
  )
}
```

```
 Flujo al pulsar "Me interesa" en el curso React:

 CourseCard ──dispatch(toggleInterest('React'))──▶ store
                                                     │ interestsSlice.reducer
                                                     ▼
                                       state.interests.ids = ['React']
                                                     │
                     ┌───────────────────────────────┴──────────────────┐
                     ▼                                                  ▼
   CourseCard React: isInterested true → "💜"       Header: count 1 → "💜 1 cursos"
   (las otras CourseCard NO se re-renderizan: su selector sigue devolviendo false)
```

### Ejercicios

| # | Ejercicio | Practicas |
|---|-----------|-----------|
| 1 | Pasa `isInterested` de `useState` a `interestsSlice` | `createSlice`, `useAppSelector`, `useAppDispatch` |
| 2 | Página `/intereses` que liste solo los cursos marcados | `createSelector` + React Router (`08`) |
| 3 | Slice `cart` con `addToCart`, `removeFromCart` y un selector `selectCartTotal` | Estado derivado con selectores |
| 4 | Persiste `interests` y `cart` en `localStorage` | `preloadedState` + `store.subscribe` |
| 5 | `ProductList` con `createAsyncThunk` y luego reescrito con RTK Query | Comparar ambos enfoques |
| 6 | Formulario que cree productos con `useAddProductMutation` y vea la lista refrescarse sola | `invalidatesTags` |
| 7 | Tests de `cartSlice` con Vitest | Reducers puros |

---

## 🎯 Resumen

| Necesito… | Uso |
|-----------|-----|
| Lógica async con control total | `createAsyncThunk` + `extraReducers` (`pending/fulfilled/rejected`) |
| Saber si un thunk falló con `try/catch` | `await dispatch(thunk()).unwrap()` |
| Pedir/cambiar datos de mi API con caché | **RTK Query**: `createApi`, `builder.query`, `builder.mutation` |
| Que la lista se refresque tras un POST | `providesTags` + `invalidatesTags` |
| Mandar el JWT siempre | `prepareHeaders` en `fetchBaseQuery` |
| Elegir herramienta | Servidor → RTK Query/TanStack · Cliente global → Redux/Zustand · Poco cambio → Context |
