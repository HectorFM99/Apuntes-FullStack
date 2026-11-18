# 11 — ⚛️ REACT Y FRONTEND: PREGUNTAS CLAVE

> Si la oferta dice "FullStack", **te van a preguntar React**. Las preguntas se
> repiten mucho: qué es, props vs estado, hooks, `useEffect`, `key`, cómo
> compartir estado y cómo consumir una API. Teoría completa en `../Frontend/REACT/`.

---

## 11.1 ¿Qué es React? ⭐

| Idea | Explicación corta |
|------|-------------------|
| **Librería** (no framework) | Solo se ocupa de la **vista**. Router, peticiones o estado global se añaden aparte |
| **Componentes** | La UI se divide en piezas reutilizables: funciones que devuelven JSX |
| **Declarativo** | Describes **qué** se ve para cada estado; React decide **cómo** cambiar el DOM |
| **UI = f(estado)** | Cambia el estado → React vuelve a llamar al componente → actualiza la pantalla |
| **Virtual DOM** | Árbol de objetos JS que React compara con el anterior para tocar el DOM real lo mínimo (*reconciliación*) |
| **SPA** | Un solo HTML; el JS pinta y cambia las "páginas" sin recargar; los datos llegan como JSON |

### Declarativo vs imperativo

```js
// Imperativo (JS vanilla): pasos
span.textContent = valor
if (valor === 10) boton.disabled = true
```
```tsx
// Declarativo (React): resultado
<span>{value}</span>
<button disabled={value === 10}>+</button>
```

---

## 11.2 JSX

- Sintaxis que parece HTML dentro de JS; se compila a llamadas a funciones.
- Diferencias con HTML: `className`, `htmlFor`, eventos en camelCase (`onClick`),
  etiquetas cerradas (`<img />`), un único elemento raíz (o fragment `<>...</>`).
- Dentro de `{}` solo **expresiones** (variables, ternarios, `.map()`), no `if`/`for`.
- React **escapa** el texto por defecto → protege de XSS. La excepción es
  `dangerouslySetInnerHTML`.

---

## 11.3 Props vs estado ⭐⭐

| | Props | Estado |
|---|---|---|
| Qué son | Parámetros que el **padre** pasa al hijo | Memoria **propia** del componente |
| ¿Quién las cambia? | El padre | El propio componente (`setX`) |
| ¿Se pueden modificar dentro? | ❌ Solo lectura | ✅ Con su setter, nunca mutando |
| Al cambiar | Re-render del hijo | Re-render del componente |
| Ejemplo en `academia` | `title`, `price`, `seats` de `CourseCard` | `isInterested` de `CourseCard`, `value` de `Counter` |

> 🗣️ "Las props son los argumentos de la función y el estado es su memoria. En mi
> catálogo de cursos, cada tarjeta recibe por props el título, precio y plazas, y
> tiene su propio estado para el botón 'Me interesa'."

---

## 11.4 `useState` ⭐

```tsx
const [value, setValue] = useState(0)
setValue(v => v + 1)          // forma funcional cuando depende del valor anterior
```

| Pregunta típica | Respuesta |
|-----------------|-----------|
| ¿Por qué no una variable normal? | Cambiarla no avisa a React y además se reinicia en cada render |
| ¿Es síncrono `setState`? | No: programa un re-render. El valor nuevo se ve en el **siguiente** render |
| ¿Cuándo usar `setX(prev => ...)`? | Cuando el nuevo valor depende del anterior (varias actualizaciones seguidas) |
| ¿Por qué no hacer `array.push` en el estado? | React compara **referencias**; mutar no cambia la referencia → no repinta. Se crea uno nuevo: `[...prev, x]` |

---

## 11.5 `useEffect` ⭐⭐

```tsx
useEffect(() => {
  const controller = new AbortController()
  fetch(url, { signal: controller.signal }).then(...)
  return () => controller.abort()          // limpieza
}, [url])                                   // dependencias
```

| Dependencias | Cuándo se ejecuta |
|--------------|-------------------|
| `[]` | Una vez al montar |
| `[a, b]` | Al montar y cuando cambie `a` o `b` |
| Sin array | Tras **cada** render (casi siempre un error) |

| Pregunta típica | Respuesta |
|-----------------|-----------|
| ¿Para qué sirve? | Sincronizar el componente con algo **externo**: APIs, timers, suscripciones, `document.title` |
| ¿Qué es la función de limpieza? | Se ejecuta antes de repetir el efecto y al desmontar: `clearInterval`, `abort`, quitar listeners |
| ¿Por qué mi petición sale dos veces? | `StrictMode` en desarrollo monta/desmonta/monta para detectar efectos sin limpieza. En producción sale una |
| ¿Puede ser `async` la función del efecto? | No: devolvería una promesa. Se declara una función `async` **dentro** y se llama |
| ¿Cómo provocas un bucle infinito? | `setX` dentro de un efecto sin dependencias, o con un objeto nuevo en las deps |
| ¿Equivalente en clases? | `componentDidMount`, `componentDidUpdate` y `componentWillUnmount` en uno |

> 🗣️ "No todo necesita un efecto: si algo se puede calcular a partir del estado o
> las props, lo calculo en el render."

---

## 11.6 La prop `key` ⭐

- Identifica cada elemento de una lista entre renders para que React sepa cuál es cuál
  al añadir, borrar o reordenar, y **conserve su estado**.
- Debe ser **única entre hermanos** y **estable**: el `id` de la base de datos (`_id`
  de Mongo, `id` de Postgres).
- ❌ El **índice** da bugs si la lista se reordena o se borran elementos (el estado
  "salta" a otro elemento). ❌ `Math.random()` recrea todo en cada render.

---

## 11.7 Compartir estado entre componentes ⭐⭐

| Técnica | Cuándo |
|---------|--------|
| **Lifting state up**: subir el estado al padre común y bajarlo por props | Dos hermanos necesitan el mismo dato (formulario que crea + lista que muestra) |
| **Callbacks** como props (`onCreated`, `onDelete`) | El hijo avisa al padre de un evento |
| **Context** (`createContext` + `useContext`) | Datos globales que cambian poco: usuario logueado, tema, idioma. Evita el *prop drilling* |
| Librería de estado (Zustand, Redux Toolkit) | Apps grandes con mucho estado compartido |
| Librería de datos de servidor (TanStack Query) | Caché de datos de la API |

> 🗣️ "React tiene flujo de datos **unidireccional**: los datos bajan por props y los
> eventos suben por callbacks. Si dos componentes necesitan el mismo estado, lo subo
> al padre común."

---

## 11.8 Otros hooks

| Hook | Para qué | Pregunta trampa |
|------|----------|-----------------|
| `useRef` | Acceder al DOM (`focus()`) o guardar un valor que **no** provoca render | ¿Diferencia con `useState`? Cambiar un ref no re-renderiza |
| `useMemo` | Memorizar el **resultado** de un cálculo caro | ¿Siempre? No: solo si hay un problema de rendimiento |
| `useCallback` | Memorizar una **función** (para hijos con `memo` o deps de efectos) | `useCallback(fn, deps)` ≈ `useMemo(() => fn, deps)` |
| `useContext` | Leer un context | Cada cambio del context re-renderiza a todos sus consumidores |
| Custom hook (`useFetch`) | Reutilizar lógica con estado entre componentes | Empieza por `use` y puede llamar a otros hooks |

### Reglas de los hooks

1. Solo en el **nivel superior** (nunca en `if`, bucles ni tras un `return` condicional).
2. Solo en **componentes** o en **custom hooks**.

**Por qué:** React identifica cada hook por el **orden** de llamada. En `academia`
el linter tiene `react/rules-of-hooks` como **error**.

---

## 11.9 ¿Cuándo se re-renderiza un componente?

1. Cambia su **estado**.
2. Se re-renderiza su **padre**.
3. Cambia un **context** que consume.

> Re-render ≠ tocar el DOM: React solo aplica las diferencias. Para evitar renders
> caros: `memo`, `useMemo`, `useCallback`… o el **React Compiler**, que los añade solo.

---

## 11.10 Formularios

- **Controlado**: el valor vive en el estado (`value` + `onChange`). Es lo habitual:
  permite validar, transformar y limpiar.
- **No controlado**: el DOM guarda el valor y lo lees con un `ref` o `FormData`
  (`defaultValue`).
- `onSubmit` en el `<form>` + `e.preventDefault()` para no recargar la página.
- Validar en el cliente (experiencia de usuario) **y** en el servidor (seguridad).
- En proyectos grandes: React Hook Form + Zod. React 19 añade *Actions* (`useActionState`).

---

## 11.11 Consumir una API desde React ⭐

```tsx
const [data, setData] = useState<Product[]>([])
const [loading, setLoading] = useState(true)
const [error, setError] = useState<string | null>(null)

useEffect(() => {
  fetch(`${import.meta.env.VITE_API_URL}/api/products`)
    .then(res => { if (!res.ok) throw new Error(`Error ${res.status}`); return res.json() })
    .then(setData)
    .catch(err => setError(err.message))
    .finally(() => setLoading(false))
}, [])
```

| Punto que debes mencionar | Por qué |
|---------------------------|---------|
| Estados **loading / error / vacío / datos** | La UI debe contemplar los 4 casos |
| Comprobar `res.ok` | `fetch` no lanza error con 404 o 500 |
| Cancelar con `AbortController` | Evita *race conditions* cuando cambia el filtro |
| `VITE_API_URL` | URL distinta en local y producción |
| CORS | El navegador bloquea otro origen; se permite en el **backend** (o proxy de Vite en dev) |
| Secretos | Nada secreto en el frontend: todo el bundle es público |

---

## 11.12 Rutas, estilos y herramientas

| Tema | Respuesta corta |
|------|-----------------|
| ¿Cómo haces varias páginas en una SPA? | React Router: `<Routes>`, `<Route path>`, `<Link>`, `useParams`, `useNavigate` |
| ¿Por qué recargar `/cursos/1` da 404 en producción? | El servidor busca ese archivo; hay que reescribir todas las rutas a `index.html` |
| ¿Cómo das estilos? | CSS global, CSS Modules (ámbito local), Tailwind (utilidades); `style={{}}` para valores dinámicos |
| ¿Qué es Vite? | Servidor de desarrollo con HMR + empaquetador (build a `dist/`) |
| ¿Create React App? | Obsoleto; hoy se usa Vite o un framework como Next.js |
| ¿Qué es Next.js? | Framework sobre React con rutas por archivos, renderizado en servidor (SSR/SSG) y Server Components; mejor SEO y primera carga |
| ¿CSR vs SSR? | CSR: el navegador construye la página con JS (tu Vite). SSR: el servidor manda el HTML ya construido (Next.js) |
| ¿Cómo testearías un componente? | Vitest + React Testing Library: renderizar, buscar por rol/texto, simular clics, comprobar lo que ve el usuario |
| ¿Clases o funciones? | Componentes de función + hooks (desde 2019). Las clases quedan en código antiguo |

---

## 11.13 Tu proyecto para contarlo

**`Vite/academia`** — catálogo de cursos con React 19 + TypeScript + Vite:

| Lo que puedes decir | Dónde se ve |
|---------------------|-------------|
| "Componentes tipados con TypeScript" | `type CourseCardProps` |
| "Listas con `.map()` y `key`, y spread de props" | `courses.map(course => <CourseCard key={course.title} {...course} />)` |
| "Renderizado condicional" | Badge "Últimas plazas" con `seats > 0 && seats <= 3`, "Completo" con ternario |
| "Estado local con actualizaciones funcionales" | `Counter` (0–10 con `Math.max/min`) e `isInterested` |
| "Linter con reglas de hooks" | `.oxlintrc.json` |
| "Build con comprobación de tipos" | `"build": "tsc -b && vite build"` |

> 🗣️ "Lo siguiente que haría es conectarlo a mi API de Express con MongoDB Atlas
> desplegada en Render: un `useEffect` con `fetch`, estados de carga y error, y
> configurar CORS y `VITE_API_URL` para producción." → Une front y back en una frase.

---

## 🎯 Cómo responderlo en la entrevista

**¿Qué es React y por qué usarlo?**
> "Es una librería de JavaScript para construir interfaces con componentes
> reutilizables. Es declarativa: describo cómo se ve la UI para cada estado y React
> se encarga de actualizar el DOM de forma eficiente con el Virtual DOM. Así no tengo
> que sincronizar a mano datos y pantalla, que es donde aparecen los bugs con JS puro."

**¿Diferencia entre props y estado?**
> "Las props vienen del padre y son de solo lectura; el estado es interno del
> componente y se cambia con su setter. Ambos provocan un re-render cuando cambian."

**¿Cómo funciona `useEffect`?**
> "Ejecuta código después del render para sincronizarse con algo externo, como una
> API. El array de dependencias dice cuándo repetirlo: vacío, solo al montar. Puede
> devolver una función de limpieza para cancelar peticiones o timers."

**¿Para qué sirve la `key`?**
> "Para que React identifique cada elemento de una lista entre renders. Uso el id de
> la base de datos; el índice puede dar errores si la lista se reordena."

**¿Cómo compartes estado entre dos componentes?**
> "Lo subo al padre común y lo paso por props, con callbacks para que los hijos lo
> modifiquen. Si muchos componentes lejanos lo necesitan, como el usuario logueado,
> uso Context."
