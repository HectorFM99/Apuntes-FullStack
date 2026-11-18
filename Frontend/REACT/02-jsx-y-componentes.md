# 02 — JSX Y COMPONENTES

> Un **componente** es una función que devuelve **JSX**: algo que parece HTML pero
> es JavaScript. Toda la app es un árbol de componentes dentro de otros.

---

## 2.1 ¿Qué es JSX?

JSX es una **sintaxis** que permite escribir "HTML" dentro de JS/TS. Vite la
traduce a llamadas de función:

```tsx
// Lo que escribes
<h2 className="title">{title}</h2>

// Lo que llega al navegador (simplificado)
jsx('h2', { className: 'title', children: title })
```

> 📌 Por eso un archivo con JSX se llama `.jsx` o `.tsx` (con TypeScript).

---

## 2.2 Reglas de JSX

| Regla | HTML | JSX |
|-------|------|-----|
| 1. **Un solo elemento raíz** | Varios hermanos sueltos | Envuélvelos en un `<div>` o en un **fragment** `<>...</>` |
| 2. **Cierra todas las etiquetas** | `<img>`, `<br>`, `<input>` | `<img />`, `<br />`, `<input />` |
| 3. **`className`** en vez de `class` | `class="card"` | `className="card"` (`class` es palabra reservada de JS) |
| 4. **`htmlFor`** en vez de `for` | `<label for="email">` | `<label htmlFor="email">` |
| 5. **camelCase** en atributos y eventos | `onclick`, `tabindex` | `onClick`, `tabIndex` |
| 6. **`style` es un objeto** | `style="color: red; font-size: 14px"` | `style={{ color: 'red', fontSize: 14 }}` |
| 7. **Comentarios** | `<!-- -->` | `{/* comentario */}` |

### Fragment: devolver varios elementos sin `div` extra

```tsx
function Cabecera() {
  return (
    <>
      <h1>Catálogo de cursos</h1>
      <p>Elige el tuyo</p>
    </>
  )
}
```

> 💡 Los paréntesis tras `return (` permiten escribir el JSX en varias líneas.
> Sin ellos, un `return` seguido de salto de línea devuelve `undefined`.

---

## 2.3 Las llaves `{}`: JavaScript dentro de JSX

Dentro de `{}` va **cualquier expresión** (algo que devuelve un valor):

```tsx
<p>{hours} horas</p>                     {/* variable */}
<p>{price * 1.21} € con IVA</p>          {/* operación */}
<p>{title.toUpperCase()}</p>             {/* llamada a método */}
<p>{seats > 0 ? 'Disponible' : 'Completo'}</p>   {/* ternario */}
<img src={course.image} alt={course.title} />   {/* atributo dinámico */}
```

| ✅ Sí se puede (expresiones) | ❌ No se puede (sentencias) |
|------------------------------|----------------------------|
| Variables, operaciones, ternarios | `if`, `for`, `while`, `switch` |
| Llamadas a funciones, `.map()` | Declarar variables (`const x = ...`) |
| `cond && <Algo />` | |

> 📌 Los `if` y los cálculos se hacen **antes del `return`**, y en el JSX solo
> usas el resultado.

### Qué pinta React según el valor

| Valor dentro de `{}` | Se ve |
|----------------------|-------|
| `'texto'`, `42` | El texto / número |
| `true`, `false`, `null`, `undefined` | **Nada** (útil para condicionales) |
| `0` | ⚠️ **Se pinta "0"** (trampa clásica, ver `05`) |
| Un array de JSX | Cada elemento (así funciona `.map()`) |
| Un objeto `{a: 1}` | ❌ Error: *Objects are not valid as a React child* |

### Seguridad: React escapa por defecto

```tsx
const comentario = '<img src=x onerror="alert(1)">'
<p>{comentario}</p>   // se ve el TEXTO literal, no se ejecuta → protege de XSS
```

> ⚠️ La única forma de inyectar HTML es `dangerouslySetInnerHTML`. Su nombre ya
> te dice que no la uses con datos del usuario.

---

## 2.4 Componentes

Un componente es una **función** que:

1. Su nombre empieza por **Mayúscula** (`CourseCard`, no `courseCard`).
2. Devuelve JSX (o `null` si no quiere pintar nada).
3. Se usa como una etiqueta: `<CourseCard />`.

### Ejemplo REAL (`academia/src/components/Footer.tsx`)

```tsx
export function Footer() {
  return (
    <footer>
      <p>© 2026 Academia · Valencia</p>
    </footer>
  )
}
```

> 📌 ¿Por qué Mayúscula? React distingue así entre **etiquetas HTML**
> (`<footer>`, minúscula) y **tus componentes** (`<Footer>`, mayúscula). Si lo
> escribes en minúscula, React intentará crear una etiqueta HTML `<footer>`
> desconocida... o la equivocada.

> 💡 En tu proyecto `Footer` está **creado pero no se usa** en `App.tsx` (y
> `index.css` ya tiene una clase `.footer` preparada). Para usarlo:
>
> ```tsx
> import { Footer } from './components/Footer'
> // ...dentro del return de App, al final:
> <Footer />
> ```
> y en el componente: `<footer className="footer">`.

---

## 2.5 Exportar e importar componentes

| Tipo | Exportar | Importar |
|------|----------|----------|
| **Nombrado** (tus componentes) | `export function CourseCard() {}` | `import { CourseCard } from './components/CourseCard'` |
| **Por defecto** (tu `App`) | `export default App` | `import App from './App.tsx'` |

### Ejemplo REAL (`academia/src/App.tsx`, imports)

```tsx
import { CourseCard } from './components/CourseCard'
import { Counter } from './components/Counter'
import './App.css'
```

> 💡 Convención: **un componente por archivo**, el archivo se llama igual que el
> componente (`CourseCard.tsx`). Los exports nombrados facilitan el autocompletado
> y los renombrados.

### La regla de oxlint `only-export-components`

```json
"react/only-export-components": ["warn", { "allowConstantExport": true }]
```

El *Fast Refresh* de Vite (recargar un componente sin perder el estado) solo
funciona si el archivo **exporta solo componentes**. Si además exportas funciones
de utilidad, mételas en otro archivo (`utils.ts`).

---

## 2.6 Composición: componentes dentro de componentes

La app es un **árbol**:

```
App
├── h1  "Catálogo de cursos"
├── div.course-list
│   ├── CourseCard (HTML y CSS desde cero)
│   ├── CourseCard (Excel avanzado)
│   └── CourseCard (React para principiantes)
└── section.counter-section
    ├── h2 "Contador"
    └── Counter
```

### Ejemplo REAL (`academia/src/App.tsx`)

```tsx
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

> 📌 Usa **HTML semántico** también en React: `main`, `section`, `article`
> (tu `CourseCard` usa `<article>`), `button` para acciones. React no cambia las
> reglas de accesibilidad de `HTML/06`.

---

## 2.7 ¿Cuándo crear un componente nuevo?

| Señal | Ejemplo |
|-------|---------|
| Se repite | Cada tarjeta de curso → `CourseCard` |
| Tiene su propio estado | El contador → `Counter` |
| El archivo se hace largo (> ~150 líneas) | Separar `CourseList`, `CourseFilters` |
| Tiene un nombre claro de "pieza" | `Header`, `Footer`, `SearchBar` |

> ⚠️ **Nunca** declares un componente **dentro** de otro:
>
> ```tsx
> function App() {
>   function Tarjeta() { ... }   // ❌ se recrea en cada render → pierde su estado
>   return <Tarjeta />
> }
> ```
> Decláralos siempre a nivel de archivo.

---

## 2.8 Componentes puros

Un componente debe ser como una **función pura** (ver `JAVASCRIPT/03`):

- Mismas props y estado → **mismo JSX**.
- **No modifica** nada de fuera durante el render (variables globales, el DOM, props).

```tsx
let contador = 0
function Malo() {
  contador++            // ❌ efecto secundario durante el render
  return <p>{contador}</p>
}
```

> 💡 `StrictMode` renderiza dos veces en desarrollo precisamente para que estos
> errores salten a la vista (el número subiría de 2 en 2).

---

## 🎯 Resumen

| Concepto | Clave |
|----------|-------|
| JSX | HTML dentro de JS; se traduce a llamadas `jsx()` |
| Reglas | Una raíz (o `<>`), todo cerrado, `className`, camelCase |
| `{}` | Solo **expresiones** |
| Componente | Función con Mayúscula que devuelve JSX |
| Composición | Componentes dentro de componentes = árbol |
| Pureza | El render no debe cambiar nada de fuera |
