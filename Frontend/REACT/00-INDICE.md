# ⚛️ CARPETA REACT — ÍNDICE DE ESTUDIO

Teoría de **React** desde cero: qué problema resuelve, cómo se crea un proyecto
con **Vite + TypeScript**, componentes, JSX, props, estado, eventos, listas,
efectos, formularios, consumir **tus propias APIs** (Express / NestJS) y
desplegar el frontend. Ejemplos reales **copiados de tu proyecto**
`Vite/academia` (catálogo de cursos).

> ⚠️ El proyecto **NO** se ha modificado. Solo se han copiado fragmentos de su código.
> 📌 Lo que no aparece en `academia` (efectos, fetch, formularios, router…) se
> explica con ejemplos nuevos que **conectan** con tus APIs de las carpetas
> `EXPRESS`, `NESTJS`, `MONGODB-ATLAS` y `RENDER`.

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a aprender |
|---|---------|--------------------|
| 1 | `01-que-es-react-y-vite.md` | Qué es React, SPA, declarativo vs imperativo, Virtual DOM, crear el proyecto con Vite, estructura, `index.html` → `main.tsx` → `App.tsx`, scripts |
| 2 | `02-jsx-y-componentes.md` | Reglas de JSX, `{}` para expresiones, `className`, fragments, componentes como funciones, exportar/importar, composición |
| 3 | `03-props-y-tipado-con-typescript.md` | Props, desestructuración, `type Props`, opcionales y por defecto, spread `{...course}`, `children`, funciones como props |
| 4 | `04-estado-useState-y-eventos.md` | `useState`, actualización funcional, inmutabilidad (arrays/objetos), eventos (`onClick`, `onChange`), re-render |
| 5 | `05-renderizado-condicional-y-listas.md` | `&&`, ternario, `return` temprano, `.map()` y la prop `key`, `.filter()`, estado derivado |
| 6 | `06-useEffect-y-consumir-apis.md` | Ciclo de vida, `useEffect` y sus dependencias, limpieza, `fetch` a tu API, loading/error, `VITE_API_URL`, CORS |
| 7 | `07-formularios-y-comunicacion-entre-componentes.md` | Inputs controlados, `onSubmit`, validación, POST a la API, *lifting state up*, flujo de datos unidireccional |
| 8 | `08-hooks-avanzados-context-router-y-estilos.md` | Reglas de los hooks, `useRef`, `useMemo`, `useCallback`, `useContext`, custom hooks, React Router, formas de dar estilos, `StrictMode` |
| 9 | `09-build-despliegue-y-errores-comunes.md` | `npm run build`, `dist/`, `preview`, desplegar en **Render** (Static Site), variables `VITE_`, errores típicos y cómo leerlos |
| 10 | `10-ejemplos-completos-de-proyectos.md` | Tu `academia` archivo por archivo + la versión "full stack" conectada a una API |
| 📋 | `99-CHEATSHEET.md` | Chuleta de comandos y código para consultar rápido |

---

## 🗂️ Proyecto del que se han extraído los ejemplos

| Proyecto | Archivo | Temas que cubre |
|----------|---------|-----------------|
| `Vite/academia/` | `index.html` | El único HTML de la SPA, `<div id="root">`, `<script type="module">` |
| ″ | `src/main.tsx` | `createRoot`, `StrictMode`, CSS global, el `!` de TypeScript |
| ″ | `src/App.tsx` | Datos en un array, `.map()` con `key`, spread de props, composición |
| ″ | `src/components/CourseCard.tsx` | Props tipadas, `useState` booleano, `&&`, ternario, `onClick` |
| ″ | `src/components/Counter.tsx` | `useState` numérico, **actualización funcional**, `disabled`, límites 0–10 |
| ″ | `src/components/Footer.tsx` | Componente mínimo sin props (definido pero **no usado** todavía) |
| ″ | `src/App.css`, `src/index.css` | CSS global con variables, `grid auto-fit`, modo oscuro |
| ″ | `package.json`, `vite.config.ts`, `tsconfig.*.json`, `.oxlintrc.json` | Scripts, plugin de React, `"jsx": "react-jsx"`, regla `rules-of-hooks` |

---

## 🚀 Orden de estudio recomendado

1. `01` → entender **qué** es React y **por qué** existe (sin esto, todo parece magia).
2. `02` y `03` → componentes y props: el 50 % de React.
3. `04` → estado: el otro 50 %. **Imprescindible.**
4. `05` → pintar listas y condiciones (lo que hace tu `academia`).
5. `06` → hablar con tu backend. Aquí se une todo el bootcamp.
6. `07` → formularios y cómo se pasan datos entre componentes.
7. `08` → herramientas para cuando la app crece.
8. `09` → ponerlo en internet.
9. `10` → repaso con el código real.

---

## 💡 La idea en una imagen

```
           ┌──────────────────────── NAVEGADOR ────────────────────────┐
           │ index.html → <div id="root"></div>                        │
           │                    ▲                                      │
           │   main.tsx: createRoot(root).render(<App />)              │
           │                    │                                      │
           │   App ── courses.map ──▶ <CourseCard {...course} /> ×3    │
           │    │                      (props: title, price, seats…)   │
           │    │                      (estado: isInterested)          │
           │    └──────────────────▶ <Counter />  (estado: value)      │
           │                                                           │
           │   UI = f(props, estado)  → cambia el estado → React       │
           │                            vuelve a llamar a la función   │
           │                            y actualiza SOLO lo distinto   │
           └───────────────────────────┬───────────────────────────────┘
                                       │ fetch('/api/courses')  (doc 06)
                                       ▼
                     Tu API Express / NestJS en Render → Atlas / Supabase
```

> 📌 La frase que resume React: **la interfaz es una función del estado**.
> Tú no tocas el DOM; cambias datos y React se encarga de pintar.
