# 04 — ESTADO CON `useState` Y EVENTOS

> Las **props** vienen de fuera; el **estado** es la **memoria propia** del
> componente: datos que cambian con el tiempo (un contador, si te interesa un
> curso, lo que escribes en un input). Cuando el estado cambia, React **vuelve a
> renderizar** el componente.

---

## 4.1 ¿Por qué no vale una variable normal?

```tsx
function Counter() {
  let value = 0
  return <button onClick={() => { value++; console.log(value) }}>{value}</button>
}
```

- La consola muestra 1, 2, 3… pero la pantalla **sigue en 0**.
- Motivo 1: cambiar una variable **no avisa a React** de que tiene que repintar.
- Motivo 2: aunque repintara, `let value = 0` se **vuelve a ejecutar** en cada
  render (el componente es una función: cada llamada empieza de cero).

> 📌 `useState` resuelve ambas cosas: **guarda** el valor entre renders y
> **dispara** un nuevo render cuando lo cambias.

---

## 4.2 `useState`

```tsx
import { useState } from 'react'

const [value, setValue] = useState(0)
//     │      │                    └─ valor inicial (solo se usa en el PRIMER render)
//     │      └─ función para cambiarlo (y provocar un re-render)
//     └─ valor actual en ESTE render
```

Es **desestructuración de arrays** (`JAVASCRIPT/05`): `useState` devuelve
`[valor, setter]` y tú eliges los nombres. Convención: `x` / `setX`.

### Ejemplo REAL (`academia/src/components/Counter.tsx`)

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

| Detalle | Por qué está bien hecho |
|---------|-------------------------|
| `setValue((current) => ...)` | **Actualización funcional**: calcula a partir del valor más reciente (4.4) |
| `Math.max(0, ...)` / `Math.min(10, ...)` | Limita el rango **aunque** el botón no estuviera desactivado |
| `disabled={value === 0}` | La UI se **deriva** del estado: no hay que acordarse de desactivar nada |
| `type="button"` | Evita que el botón haga *submit* si algún día va dentro de un `<form>` |
| Cada `<Counter />` | Tiene **su propio** estado independiente |

### Ejemplo REAL (`academia/src/components/CourseCard.tsx`)

```tsx
const [isInterested, setIsInterested] = useState(false);

<button type="button" onClick={() => setIsInterested((current) => !current)}>
  {isInterested ? 'Te interesa' : 'Me interesa'}
</button>
```

- Estado **booleano** que alterna (*toggle*) con `!current`.
- Cada una de las 3 tarjetas recuerda **su** `isInterested`.
- Nombres de booleanos: `isX`, `hasX`, `showX` → se leen como preguntas.

---

## 4.3 Qué pasa al llamar a `setValue`

```
1. Clic en "+"            → setValue(c => c + 1)
2. React programa un render (no es inmediato)
3. React llama otra vez a Counter()  → useState(0) ahora devuelve 1
4. Compara el nuevo JSX con el anterior (Virtual DOM)
5. Cambia en el DOM solo el texto del <span>: "0" → "1"
```

### El estado es una "foto" de cada render

```tsx
function handleClick() {
  setValue(value + 1)
  console.log(value)     // ⚠️ sigue mostrando el valor ANTIGUO
}
```

`value` es una constante de **este** render. El nuevo valor solo existirá en el
**siguiente** render.

---

## 4.4 Actualización funcional: `setX(prev => ...)`

```tsx
// ❌ Las tres usan el mismo "value" de este render (0) → resultado: 1
setValue(value + 1)
setValue(value + 1)
setValue(value + 1)

// ✅ Cada una recibe el resultado de la anterior → resultado: 3
setValue(v => v + 1)
setValue(v => v + 1)
setValue(v => v + 1)
```

> 📌 **Regla práctica**: si el nuevo valor **depende del anterior**, usa la forma
> funcional. Tu `Counter` y tu `CourseCard` ya lo hacen así. 👏

---

## 4.5 Inmutabilidad: arrays y objetos en el estado

React decide si repintar comparando **referencias** (`Object.is`). Si mutas el
objeto, la referencia es la misma → React cree que **no ha cambiado nada**.

```tsx
const [courses, setCourses] = useState<Course[]>([])

courses.push(nuevo)      // ❌ muta: la pantalla no se actualiza
setCourses(courses)      // ❌ misma referencia → React lo ignora
```

### Chuleta de operaciones inmutables

| Operación | ❌ Mutando | ✅ Creando uno nuevo |
|-----------|-----------|---------------------|
| Añadir | `arr.push(x)` | `setArr(prev => [...prev, x])` |
| Añadir al principio | `arr.unshift(x)` | `setArr(prev => [x, ...prev])` |
| Borrar | `arr.splice(i, 1)` | `setArr(prev => prev.filter(c => c.id !== id))` |
| Editar uno | `arr[i].seats = 0` | `setArr(prev => prev.map(c => c.id === id ? { ...c, seats: 0 } : c))` |
| Ordenar | `arr.sort()` | `setArr(prev => [...prev].sort(...))` o `prev.toSorted(...)` |
| Cambiar campo de objeto | `obj.name = 'x'` | `setObj(prev => ({ ...prev, name: 'x' }))` |

> 💡 Son exactamente los métodos de `JAVASCRIPT/04` (`map`, `filter`) y el
> *spread* de `JAVASCRIPT/05`. En React se usan **todo el rato**.

---

## 4.6 Tipar `useState`

```tsx
const [value, setValue] = useState(0)                 // inferido: number
const [isOpen, setIsOpen] = useState(false)           // inferido: boolean
const [courses, setCourses] = useState<Course[]>([])  // ✅ array vacío → indica el tipo
const [user, setUser] = useState<User | null>(null)   // ✅ "todavía no hay"
const [error, setError] = useState<string | null>(null)
```

> ⚠️ `useState([])` sin genérico infiere `never[]` y no podrás meter nada.

### Inicialización perezosa

```tsx
// Se ejecuta en CADA render (aunque solo se use el primero)
const [fav, setFav] = useState(JSON.parse(localStorage.getItem('fav') ?? '[]'))

// ✅ Se ejecuta SOLO la primera vez
const [fav, setFav] = useState<string[]>(() => JSON.parse(localStorage.getItem('fav') ?? '[]'))
```

---

## 4.7 Eventos

```tsx
<button onClick={handleClick}>       {/* ✅ pasas la función */}
<button onClick={handleClick()}>     {/* ❌ la EJECUTAS al renderizar */}
<button onClick={() => handleDelete(id)}>   {/* ✅ si necesitas argumentos: flecha */}
```

| Evento | Elemento típico | Para qué |
|--------|-----------------|----------|
| `onClick` | `button` | Acciones |
| `onChange` | `input`, `select`, `textarea` | Cada tecla / cambio (ver `07`) |
| `onSubmit` | `form` | Enviar formulario |
| `onKeyDown` | `input` | Atajos, Enter |
| `onMouseEnter` / `onMouseLeave` | cualquiera | Hover |
| `onFocus` / `onBlur` | `input` | Validar al salir del campo |

### El objeto evento

```tsx
import type { MouseEvent } from 'react'

function handleClick(e: MouseEvent<HTMLButtonElement>) {
  e.preventDefault()          // igual que en el DOM
  e.stopPropagation()
  console.log(e.currentTarget)
}
```

> 📌 React envuelve el evento nativo en un **SyntheticEvent** con la misma API en
> todos los navegadores. Lo que aprendiste en `JAVASCRIPT/06` sigue valiendo.

---

## 4.8 ¿Qué debe ir en el estado? (y qué NO)

| Pregunta | Si la respuesta es SÍ → **no** es estado |
|----------|------------------------------------------|
| ¿Viene del padre por props? | Úsala tal cual |
| ¿No cambia nunca? | Es una constante (tu array `courses` en `App.tsx`) |
| ¿Se puede **calcular** a partir de otro estado o props? | Calcúlalo en el render (*estado derivado*) |

```tsx
// ❌ Estado redundante: hay que mantener dos cosas sincronizadas
const [courses, setCourses] = useState<Course[]>(data)
const [total, setTotal] = useState(data.length)

// ✅ Derivado: se calcula solo, nunca se desincroniza
const [courses, setCourses] = useState<Course[]>(data)
const total = courses.length
const available = courses.filter(c => c.seats > 0)
```

> 💡 En `CourseCard`, "Últimas plazas" o "Completo" **no** son estado: se derivan
> de la prop `seats`. Correcto.

---

## 🎯 Resumen

| Concepto | Clave |
|----------|-------|
| Estado | Memoria del componente que, al cambiar, provoca un re-render |
| `useState(inicial)` | Devuelve `[valor, setValor]` |
| Foto por render | El valor no cambia hasta el siguiente render |
| `setX(prev => ...)` | Cuando el nuevo valor depende del anterior |
| Inmutabilidad | Nunca mutar: crear arrays/objetos nuevos con `...`, `map`, `filter` |
| Eventos | `onClick={fn}`, no `onClick={fn()}` |
| Estado mínimo | Lo que se puede calcular, se calcula |
