# 03 — PROPS Y TIPADO CON TYPESCRIPT

> Las **props** son los **parámetros** de un componente: datos que el padre le
> pasa al hijo, como atributos de HTML. Con TypeScript, además, quedan
> **documentadas y comprobadas**.

---

## 3.1 Qué son las props

```tsx
<CourseCard title="Excel avanzado" hours={25} price={90} />
```

React junta todos esos atributos en **un objeto** y se lo pasa a la función:

```tsx
function CourseCard(props) {
  // props = { title: 'Excel avanzado', hours: 25, price: 90 }
  return <h2>{props.title}</h2>
}
```

| Valor | Cómo se pasa |
|-------|--------------|
| String | `title="Excel"` (comillas) o `title={'Excel'}` |
| Número | `hours={25}` ⚠️ con llaves; `hours="25"` sería un **string** |
| Booleano | `featured={true}` o simplemente `featured` |
| Array/objeto | `tags={['js', 'css']}`, `course={course}` |
| Función | `onSelect={handleSelect}` |
| JSX | `icon={<Star />}` o como `children` |

---

## 3.2 Tipar las props

### Ejemplo REAL (`academia/src/components/CourseCard.tsx`)

```tsx
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
  // ...
}
```

Lo que te da TypeScript:

```tsx
<CourseCard title="Excel" hours="25" />
//                        ^^^^^ ❌ Type 'string' is not assignable to type 'number'
//          ❌ Property 'price' is missing...
```

- **Autocompletado** de props al usar el componente.
- **Errores** si falta una prop obligatoria o tiene el tipo mal.
- **Documentación**: el `type` te dice qué necesita el componente sin leerlo entero.

> 💡 `type` o `interface` valen igual aquí (ver `TYPESCRIPT/03`). La convención
> más común es `NombreComponenteProps`.

### Desestructuración en los parámetros

```tsx
function CourseCard(props: CourseCardProps) { return <h2>{props.title}</h2> }   // sin desestructurar
function CourseCard({ title }: CourseCardProps) { return <h2>{title}</h2> }     // ✅ lo habitual
```

---

## 3.3 Props opcionales y valores por defecto

```tsx
type CourseCardProps = {
  title: string
  price: number
  category?: string          // ? → opcional (puede ser undefined)
  currency?: '€' | '$'       // literal type: solo esos valores
}

export function CourseCard({ title, price, category = 'General', currency = '€' }: CourseCardProps) {
  return (
    <article>
      <h2>{title}</h2>
      <p>{category}</p>
      <p>{price} {currency}</p>
    </article>
  )
}

<CourseCard title="SQL" price={80} />                    // category = 'General', currency = '€'
<CourseCard title="SQL" price={80} currency="$" />
```

> 📌 Los valores por defecto se ponen con la sintaxis normal de
> **desestructuración con default** de JavaScript (`JAVASCRIPT/05`).

---

## 3.4 Spread de props: `{...course}`

### Ejemplo REAL (`academia/src/App.tsx`)

```tsx
const courses = [
  { title: 'HTML y CSS desde cero', hours: 40, price: 120, category: 'Programación', seats: 3 },
  { title: 'Excel avanzado', hours: 25, price: 90, category: 'Productividad', seats: 2 },
  { title: 'React para principiantes', hours: 30, price: 150, category: 'Programación', seats: 0 },
]

{courses.map((course) => (
  <CourseCard key={course.title} {...course} />
))}
```

`{...course}` **reparte** cada propiedad del objeto como una prop:

```tsx
<CourseCard {...course} />
// equivale a
<CourseCard title={course.title} hours={course.hours} price={course.price}
            category={course.category} seats={course.seats} />
```

| Ventaja | Inconveniente |
|---------|---------------|
| Corto y cómodo | Si el objeto trae propiedades de más, también se pasan |
| TS sigue comprobando que el objeto encaje con `CourseCardProps` | Menos explícito al leer |

> 💡 Alternativa igual de válida: pasar el objeto entero
> `<CourseCard course={course} />` con `type Props = { course: Course }`.
> Cuando el modelo viene de una API, suele ser más claro.

### Compartir el tipo entre archivos

En `academia` el array `courses` no tiene tipo explícito (TS lo **infiere**).
Cuando el proyecto crece, conviene un tipo de dominio:

```ts
// src/types.ts
export type Course = {
  title: string
  hours: number
  price: number
  category: string
  seats: number
}
```

```tsx
// App.tsx
import type { Course } from './types'
const courses: Course[] = [ /* ... */ ]

// CourseCard.tsx
import type { Course } from '../types'
type CourseCardProps = Course
```

> 📌 `import type` es obligatorio aquí por `"verbatimModuleSyntax": true` en tu
> `tsconfig.app.json`: los imports que solo son tipos deben marcarse como tales.

---

## 3.5 Las props son de solo lectura

```tsx
function CourseCard({ price }: CourseCardProps) {
  price = price * 0.9      // ❌ nunca modifiques una prop
  const finalPrice = price * 0.9   // ✅ crea una variable nueva
}
```

> 📌 **Los datos bajan** (padre → hijo por props). Si el hijo necesita **cambiar**
> algo del padre, el padre le pasa **una función** (3.7).

---

## 3.6 `children`: pasar JSX dentro de un componente

```tsx
import type { ReactNode } from 'react'

type CardProps = {
  title: string
  children: ReactNode     // cualquier cosa pintable: JSX, texto, arrays, null...
}

function Card({ title, children }: CardProps) {
  return (
    <article className="card">
      <h2>{title}</h2>
      {children}
    </article>
  )
}

// Uso: lo que va ENTRE las etiquetas llega como children
<Card title="Contador">
  <Counter />
  <p>Máximo 10</p>
</Card>
```

> 💡 Útil para **contenedores** genéricos: `Layout`, `Modal`, `Card`, `Section`.

---

## 3.7 Funciones como props (callbacks)

Así un hijo **avisa** al padre:

```tsx
type CourseCardProps = Course & {
  onEnroll: (title: string) => void       // tipo de la función
}

function CourseCard({ title, onEnroll }: CourseCardProps) {
  return <button onClick={() => onEnroll(title)}>Inscribirme</button>
}

function App() {
  function handleEnroll(title: string) {
    alert(`Te has inscrito en ${title}`)
  }
  return <CourseCard {...course} onEnroll={handleEnroll} />
}
```

> 📌 Convención: la prop se llama `onAlgo` (`onEnroll`, `onDelete`) y la función
> del padre `handleAlgo` (`handleEnroll`).

---

## 3.8 Tipos de React que más usarás

| Tipo | Para qué |
|------|----------|
| `ReactNode` | `children` o cualquier cosa pintable |
| `ComponentProps<'button'>` | Todas las props de un `<button>` nativo (para envolverlo) |
| `ChangeEvent<HTMLInputElement>` | Evento `onChange` de un input |
| `FormEvent<HTMLFormElement>` | Evento `onSubmit` de un formulario |
| `MouseEvent<HTMLButtonElement>` | Evento `onClick` |
| `Dispatch<SetStateAction<T>>` | Tipo de un `setX` de `useState` si lo pasas por props |

```tsx
import type { ComponentProps } from 'react'

// Un botón propio que acepta TODO lo de un botón normal + "variant"
type ButtonProps = ComponentProps<'button'> & { variant?: 'primary' | 'ghost' }

function Button({ variant = 'primary', ...rest }: ButtonProps) {
  return <button className={`btn btn-${variant}`} {...rest} />
}

<Button type="button" disabled onClick={() => {}}>Guardar</Button>
```

---

## 🎯 Resumen

| Concepto | Clave |
|----------|-------|
| Props | Objeto con los atributos que el padre pasa al hijo |
| Tipado | `type XProps = {...}` + desestructurar en los parámetros |
| Opcional / default | `prop?: T` + `{ prop = valor }` |
| `{...obj}` | Pasa cada propiedad como prop |
| Inmutables | Nunca se modifican; para cambiar algo → callback del padre |
| `children` | Lo que va entre las etiquetas (`ReactNode`) |
| Callbacks | `onX` en el hijo, `handleX` en el padre |
