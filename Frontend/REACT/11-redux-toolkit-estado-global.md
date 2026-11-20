# 11 — REDUX TOOLKIT: ESTADO GLOBAL

> En `07` subiste el estado al padre (*lifting state up*) y en `08` usaste
> **Context** para datos globales que cambian poco. Cuando la app crece y **mucho
> estado** se comparte entre **muchos componentes**, aparece **Redux**. Hoy se
> usa siempre con **Redux Toolkit (RTK)**, la forma oficial y moderna.
>
> 📌 Ejemplo conductor: el ejercicio 5 de `10` ("Te interesan N cursos") hecho
> con Redux en tu `academia`, sin pasar props.

---

## 11.1 ¿Qué problema resuelve Redux? ⭐

Imagina `academia` con más pantallas:

```
App
 ├── Header        → muestra "❤️ 3 cursos"            (necesita los intereses)
 ├── Catalog
 │    └── CourseCard × N → botón "Me interesa"         (cambia los intereses)
 └── InterestsPage → lista de cursos que te interesan  (necesita los intereses)
```

Tres componentes **lejanos** leen o cambian el **mismo** dato. Opciones:

| Opción | Problema cuando crece |
|--------|----------------------|
| Lifting state up (`07`) | El estado sube hasta `App` y baja por props a través de componentes que no lo usan (*prop drilling*) |
| Context (`08`) | Cada cambio re-renderiza a **todos** los consumidores; con mucho estado acabas con 10 providers anidados y lógica repartida |
| **Redux** | Un **único almacén (store)** fuera de los componentes. Cada componente se suscribe **solo** al trozo que necesita |

> 📌 Redux = **un objeto global con el estado de la app** + **reglas estrictas**
> para cambiarlo. Esas reglas hacen que el estado sea **predecible** y **depurable**
> (puedes ver cada cambio, cuándo y por qué, en las DevTools).

### ¿Cuándo NO usar Redux?

| Situación | Usa mejor |
|-----------|-----------|
| Estado de un solo componente (un input, un modal abierto) | `useState` |
| Dos hermanos comparten un dato | Lifting state up |
| Usuario logueado, tema, idioma (cambian poco) | Context |
| Solo necesitas **datos de la API** con caché | RTK Query (`12`) o TanStack Query |
| App pequeña (como `academia` hoy) | Nada de lo anterior: no hace falta |

> 💡 Dan Abramov (co-creador de Redux): *"No necesitas Redux hasta que sabes que lo
> necesitas"*. Pero en **empresas** lo vas a ver muchísimo, así que hay que saberlo.

---

## 11.2 Los conceptos clave ⭐⭐

| Concepto | Qué es | Analogía (banco) |
|----------|--------|------------------|
| **Store** | El objeto que guarda **todo** el estado global | La caja fuerte |
| **State** | Los datos en un momento dado: `{ interests: { ids: [...] } }` | El dinero que hay dentro |
| **Action** | Objeto plano que **describe qué ha pasado**: `{ type: 'interests/toggle', payload: 'React' }` | El formulario de "ingresar 50 €" |
| **Dispatch** | La función para **enviar** una acción al store | Entregar el formulario en ventanilla |
| **Reducer** | Función **pura** `(estadoAnterior, acción) => estadoNuevo` | El cajero que aplica la operación |
| **Selector** | Función que **lee** un trozo del estado: `state => state.interests.ids` | Consultar el saldo |

### El flujo (siempre en una dirección)

```
   ┌─────────────── COMPONENTE ───────────────┐
   │  useSelector(selectCount)  →  pinta "3"  │◀──────────┐
   │  onClick → dispatch(toggle('React'))     │           │ 4. los componentes
   └──────────────────┬───────────────────────┘           │    suscritos se
                      │ 1. acción                         │    re-renderizan
                      ▼                                   │
              ┌──────────────┐  2. (estado, acción)  ┌────┴─────┐
              │    STORE     │ ────────────────────▶ │ REDUCER  │
              │ estado actual│ ◀──────────────────── │  (puro)  │
              └──────────────┘  3. estado NUEVO      └──────────┘
```

> 📌 Igual que en React: **los datos bajan, los eventos suben**. Nadie modifica el
> estado "a mano": solo el reducer, y solo como respuesta a una acción.

### Las 3 reglas de Redux

1. **Una única fuente de verdad**: todo el estado global vive en un store.
2. **El estado es de solo lectura**: la única forma de cambiarlo es despachar una acción.
3. **Los cambios se hacen con funciones puras** (reducers): mismo estado + misma
   acción ⇒ siempre el mismo resultado. Sin `fetch`, sin `Math.random()`, sin
   `Date.now()`, sin mutar.

---

## 11.3 El puente: `useReducer` es "Redux en pequeño"

Antes de instalar nada, React ya trae la idea de reducer **para un componente**:

```tsx
import { useReducer } from 'react'

type State = { ids: string[] }
type Action =
  | { type: 'toggle'; payload: string }
  | { type: 'clear' }

function interestsReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'toggle':
      return state.ids.includes(action.payload)
        ? { ids: state.ids.filter(id => id !== action.payload) }   // quitar
        : { ids: [...state.ids, action.payload] }                  // añadir
    case 'clear':
      return { ids: [] }
    default:
      return state
  }
}

function Interests() {
  const [state, dispatch] = useReducer(interestsReducer, { ids: [] })
  return (
    <>
      <button onClick={() => dispatch({ type: 'toggle', payload: 'React' })}>React</button>
      <p>Te interesan {state.ids.length} cursos</p>
    </>
  )
}
```

| | `useReducer` | Redux |
|---|---|---|
| Ámbito | **Un** componente (y lo que le pases por props/context) | **Toda** la app |
| Reducer + acciones + dispatch | ✅ | ✅ |
| DevTools, middleware, async | ❌ | ✅ |

> 💡 Si entiendes este código, entiendes Redux. Lo demás es **dónde** vive el
> estado (fuera de React) y **cómo** se conectan los componentes.

---

## 11.4 Redux "clásico" vs Redux Toolkit

En tutoriales antiguos verás esto (Redux "a mano", **no lo escribas hoy**):

```ts
// ❌ Estilo antiguo: constantes, action creators y switch a mano
const TOGGLE = 'interests/toggle'
export const toggle = (id: string) => ({ type: TOGGLE, payload: id })

function reducer(state = { ids: [] }, action) {
  switch (action.type) {
    case TOGGLE: return { ...state, ids: /* copia inmutable a mano */ }
    default: return state
  }
}
const store = createStore(reducer)   // ⚠️ marcado como "deprecated"
```

**Redux Toolkit** es el paquete oficial que elimina ese *boilerplate*:

| Problema del Redux clásico | Solución en Redux Toolkit |
|----------------------------|---------------------------|
| Mucho código repetido (tipos, action creators, switch) | `createSlice` lo genera todo |
| Copias inmutables a mano (`...spread` anidados) | **Immer**: escribes "mutando" y él crea la copia |
| Configurar DevTools y middleware a mano | `configureStore` lo trae puesto |
| Async con librerías externas | `createAsyncThunk` y **RTK Query** incluidos (`12`) |

> 📌 La documentación oficial de Redux dice literalmente: **usa Redux Toolkit**.
> Si en una entrevista dices "Redux", di "Redux Toolkit".

---

## 11.5 Instalación y estructura

```bash
npm install @reduxjs/toolkit react-redux
```

| Paquete | Qué trae |
|---------|----------|
| `@reduxjs/toolkit` | `configureStore`, `createSlice`, `createAsyncThunk`, `createSelector`, RTK Query (y Redux por dentro) |
| `react-redux` | El puente con React: `<Provider>`, `useSelector`, `useDispatch` |

Estructura recomendada (por **funcionalidad**, no por tipo de archivo):

```
src/
├── app/
│   ├── store.ts          ← configureStore + tipos RootState / AppDispatch
│   └── hooks.ts          ← useAppSelector / useAppDispatch tipados
├── features/
│   ├── interests/
│   │   └── interestsSlice.ts
│   └── products/
│       ├── productsSlice.ts      (doc 12)
│       └── ProductList.tsx
├── components/
│   └── CourseCard.tsx
└── main.tsx              ← <Provider store={store}>
```

> 💡 Cada carpeta de `features/` es como un **módulo** de NestJS (`NESTJS/02`):
> agrupa todo lo de una funcionalidad.

---

## 11.6 `createSlice`: estado + reducers + acciones de una vez ⭐⭐

Un **slice** es una "rebanada" del estado global con sus reducers.

```ts
// src/features/interests/interestsSlice.ts
import { createSlice } from '@reduxjs/toolkit'
import type { PayloadAction } from '@reduxjs/toolkit'

type InterestsState = {
  ids: string[]          // títulos de los cursos que me interesan
}

const initialState: InterestsState = {
  ids: [],
}

export const interestsSlice = createSlice({
  name: 'interests',     // prefijo de los types: 'interests/toggleInterest'
  initialState,
  reducers: {
    toggleInterest(state, action: PayloadAction<string>) {
      const id = action.payload
      const index = state.ids.indexOf(id)
      if (index === -1) state.ids.push(id)     // 😱 ¿push? Sí: Immer (ver abajo)
      else state.ids.splice(index, 1)
    },
    clearInterests(state) {
      state.ids = []
    },
  },
})

// createSlice GENERA los action creators con el mismo nombre que los reducers
export const { toggleInterest, clearInterests } = interestsSlice.actions

// y el reducer que irá al store
export default interestsSlice.reducer
```

```ts
toggleInterest('React')
// → { type: 'interests/toggleInterest', payload: 'React' }   ← acción generada
```

### ¿Pero no había que ser inmutable? → Immer

En `04` aprendiste que **nunca** se muta el estado (`push` ❌). Dentro de
`createSlice` sí puedes escribir `state.ids.push(id)` porque RTK usa **Immer**:

```
 Tú escribes:     state.ids.push('React')        (sobre un "borrador")
 Immer produce:   { ...state, ids: [...state.ids, 'React'] }   (copia nueva)
```

| Regla Immer | Ejemplo |
|-------------|---------|
| ✅ Mutar el `state` (borrador) | `state.ids.push(x)`, `state.user.name = 'Ana'` |
| ✅ **Devolver** un estado nuevo | `return { ids: [] }` (útil para "resetear") |
| ❌ Mutar **y** devolver a la vez | `state.ids.push(x); return state` → error |
| ❌ Reasignar `state` | `state = { ids: [] }` → no hace nada (es una variable local) |

> ⚠️ "Mutar" **solo** funciona dentro de `createSlice`/`createReducer`. En un
> `useState` normal sigue estando prohibido.

### `PayloadAction<T>`

Tipa el `payload` de la acción. Así `dispatch(toggleInterest(42))` da error de
TypeScript porque esperaba un `string`.

```ts
addCourse(state, action: PayloadAction<{ title: string; price: number }>) { ... }
reservarPlaza(state, action: PayloadAction<string>) { ... }
reset() { return initialState }                 // sin payload
```

---

## 11.7 `configureStore` y los tipos del store

```ts
// src/app/store.ts
import { configureStore } from '@reduxjs/toolkit'
import interestsReducer from '../features/interests/interestsSlice'

export const store = configureStore({
  reducer: {
    interests: interestsReducer,     // state.interests
    // products: productsReducer,    // state.products (doc 12)
  },
})

// Tipos inferidos del propio store (no los escribas a mano)
export type RootState = ReturnType<typeof store.getState>
// → { interests: { ids: string[] } }
export type AppDispatch = typeof store.dispatch
```

`configureStore` ya incluye:

| Incluido de serie | Para qué |
|-------------------|----------|
| Redux DevTools | Ver cada acción y el estado en el navegador (`11.11`) |
| Middleware de **thunk** | Acciones asíncronas (`12`) |
| Comprobación de **inmutabilidad** (solo dev) | Avisa si mutas el estado fuera de Immer |
| Comprobación de **serializable** (solo dev) | Avisa si guardas `Date`, `Map`, funciones, clases… |

### Hooks tipados (hazlo una vez y úsalos siempre)

```ts
// src/app/hooks.ts
import { useDispatch, useSelector } from 'react-redux'
import type { AppDispatch, RootState } from './store'

export const useAppDispatch = useDispatch.withTypes<AppDispatch>()
export const useAppSelector = useSelector.withTypes<RootState>()
```

> 📌 Así no tienes que escribir `(state: RootState) => ...` en cada componente y
> el `dispatch` conoce los thunks (`12`). En código antiguo verás:
> `export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector`.

---

## 11.8 `<Provider>`: conectar React con el store

Igual que el `AuthProvider` de `08`, pero para **todo** Redux:

```tsx
// src/main.tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { Provider } from 'react-redux'
import { store } from './app/store'
import './index.css'
import App from './App.tsx'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </StrictMode>,
)
```

> ⚠️ Cualquier componente que use `useAppSelector` **fuera** del `<Provider>`
> da: *"could not find react-redux context value"*.

---

## 11.9 Leer y cambiar el estado: `useSelector` y `useDispatch` ⭐⭐

### `CourseCard`: de estado local a Redux

Tu `CourseCard` REAL tenía `const [isInterested, setIsInterested] = useState(false)`.
Con Redux, el "me interesa" pasa al store:

```tsx
// src/components/CourseCard.tsx
import { useAppDispatch, useAppSelector } from '../app/hooks'
import { toggleInterest } from '../features/interests/interestsSlice'

type CourseCardProps = {
  title: string
  hours: number
  price: number
  category: string
  seats: number
}

export function CourseCard({ title, hours, price, category, seats }: CourseCardProps) {
  // LEER: me suscribo SOLO a "¿este curso está en la lista?" → boolean
  const isInterested = useAppSelector(state => state.interests.ids.includes(title))
  // ESCRIBIR
  const dispatch = useAppDispatch()

  return (
    <article className="card">
      <span className="badge">{category}</span>
      <h2>{title}</h2>
      <p>{hours} h · {price} €</p>
      {seats > 0 && seats <= 3 && <p className="warning">¡Últimas plazas!</p>}

      <button onClick={() => dispatch(toggleInterest(title))}>
        {isInterested ? '💜 Te interesa' : '🤍 Me interesa'}
      </button>
    </article>
  )
}
```

### `Header`: lee el mismo dato sin recibir props

```tsx
// src/components/Header.tsx
import { useAppDispatch, useAppSelector } from '../app/hooks'
import { clearInterests } from '../features/interests/interestsSlice'

export function Header() {
  const count = useAppSelector(state => state.interests.ids.length)
  const dispatch = useAppDispatch()

  return (
    <header>
      <h1>Academia</h1>
      <span>💜 {count} cursos</span>
      {count > 0 && <button onClick={() => dispatch(clearInterests())}>Vaciar</button>}
    </header>
  )
}
```

> 📌 `App` **no** sabe nada de los intereses: ni estado, ni props, ni callbacks.
> `CourseCard` y `Header` se comunican a través del store. Eso es lo que Redux
> aporta frente a *lifting state up*.

| Hook | Qué hace | Cuándo re-renderiza |
|------|----------|---------------------|
| `useAppSelector(fn)` | Devuelve lo que devuelva `fn(state)` | Solo si **ese valor** cambia (comparación `===`) |
| `useAppDispatch()` | Devuelve `dispatch` | Nunca (es siempre la misma función) |

---

## 11.10 Selectores ⭐

### Ponles nombre y guárdalos en el slice

```ts
// interestsSlice.ts (al final)
import type { RootState } from '../../app/store'

export const selectInterestIds = (state: RootState) => state.interests.ids
export const selectInterestCount = (state: RootState) => state.interests.ids.length
export const selectIsInterested = (state: RootState, title: string) =>
  state.interests.ids.includes(title)
```

```tsx
const count = useAppSelector(selectInterestCount)
const isInterested = useAppSelector(state => selectIsInterested(state, title))
```

> 💡 Si mañana cambias la forma del estado (`ids` → `byId`), solo cambias el
> selector, no 20 componentes.

### ⚠️ La trampa: selectores que devuelven un objeto/array **nuevo**

```tsx
// ❌ filter() crea un array nuevo CADA vez → useSelector ve "cambio" siempre
//    → re-render en CADA acción de la app (y un warning en desarrollo)
const myCourses = useAppSelector(state =>
  courses.filter(c => state.interests.ids.includes(c.title))
)
```

Solución: **`createSelector`** (memoriza, como `useMemo` de `08`):

```ts
import { createSelector } from '@reduxjs/toolkit'

// (suponiendo que los cursos también estuvieran en un slice `courses`)
const selectCourses = (state: RootState) => state.courses.items

export const selectInterestingCourses = createSelector(
  [selectCourses, selectInterestIds],                 // entradas
  (courses, ids) => courses.filter(c => ids.includes(c.title)),  // solo se recalcula
)                                                     // si cambian las entradas
```

| Selector | Devuelve | ¿Necesita `createSelector`? |
|----------|----------|-----------------------------|
| `state => state.interests.ids` | Una referencia que ya existe | ❌ No |
| `state => state.interests.ids.length` | Un número (primitivo) | ❌ No |
| `state => state.x.filter(...)` / `.map(...)` / `{ a, b }` | Algo **nuevo** | ✅ Sí |

> 📌 Regla de oro: guarda en el store el **mínimo** estado; lo demás (totales,
> filtros, conteos) es **estado derivado** que calculas con selectores (igual que
> en `05`).

---

## 11.11 Redux DevTools

Instala la extensión **Redux DevTools** (Chrome/Firefox). `configureStore` la
activa sola en desarrollo.

| Pestaña | Qué ves |
|---------|---------|
| Lista de acciones | Cada `dispatch`: `interests/toggleInterest` con su `payload` |
| *State* | El estado completo en ese momento |
| *Diff* | Qué cambió exactamente con esa acción |
| *Time travel* (slider) | Volver atrás y adelante en el tiempo |

> 💡 Esto es lo que hace Redux "predecible": ante un bug, ves **qué acción** lo
> provocó y **cómo** quedó el estado. Con `useState` repartido es mucho más difícil.

---

## 11.12 Persistir en `localStorage` (versión sencilla)

El ejercicio 6 de `10`, pero con Redux: cargar al arrancar y guardar al cambiar.

```ts
// src/app/store.ts
const saved = localStorage.getItem('interests')

export const store = configureStore({
  reducer: { interests: interestsReducer },
  preloadedState: saved ? { interests: JSON.parse(saved) } : undefined,
})

// subscribe: se ejecuta después de CADA acción
store.subscribe(() => {
  localStorage.setItem('interests', JSON.stringify(store.getState().interests))
})
```

> 💡 En proyectos grandes se usa la librería **`redux-persist`**, pero para
> entender la idea esto basta. Nunca guardes en `localStorage` datos sensibles
> (`08`, tabla del JWT).

---

## 11.13 Testear un reducer (es lo más fácil del mundo)

Un reducer es una **función pura**: le das estado + acción y compruebas el resultado.
Sin renderizar nada (Vitest, como en `99-CHEATSHEET`):

```ts
// src/features/interests/interestsSlice.test.ts
import { describe, expect, it } from 'vitest'
import reducer, { toggleInterest, clearInterests } from './interestsSlice'

describe('interestsSlice', () => {
  it('añade un curso que no estaba', () => {
    const state = reducer({ ids: [] }, toggleInterest('React'))
    expect(state.ids).toEqual(['React'])
  })

  it('quita un curso que ya estaba', () => {
    const state = reducer({ ids: ['React'] }, toggleInterest('React'))
    expect(state.ids).toEqual([])
  })

  it('vacía la lista', () => {
    expect(reducer({ ids: ['React', 'Git'] }, clearInterests()).ids).toEqual([])
  })

  it('devuelve el estado inicial con una acción desconocida', () => {
    expect(reducer(undefined, { type: 'desconocida' })).toEqual({ ids: [] })
  })
})
```

---

## 🎯 Resumen

| Pieza | Código | Para qué |
|-------|--------|----------|
| Slice | `createSlice({ name, initialState, reducers })` | Estado + reducers + acciones generadas |
| Acción | `toggleInterest('React')` | `{ type: 'interests/toggleInterest', payload: 'React' }` |
| Store | `configureStore({ reducer: { interests } })` | El almacén global (+ DevTools + thunk) |
| Tipos | `RootState`, `AppDispatch` | Inferidos del store |
| Hooks tipados | `useAppSelector`, `useAppDispatch` | `.withTypes<...>()` |
| Provider | `<Provider store={store}>` en `main.tsx` | Conectar React con el store |
| Leer | `useAppSelector(selectInterestCount)` | Re-render solo si cambia ese valor |
| Escribir | `dispatch(toggleInterest(title))` | Pasa por el reducer |
| Derivados | `createSelector([...], fn)` | Memorizar filtros/cálculos |
| Immer | `state.ids.push(x)` dentro del slice | "Mutar" de forma segura |

> 🗣️ Siguiente paso → `12`: cómo pedir datos a **tu API** con Redux
> (`createAsyncThunk` y **RTK Query**).
