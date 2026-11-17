# 01 — QUÉ ES REACT Y CÓMO SE CREA UN PROYECTO CON VITE

> React es una **librería de JavaScript para construir interfaces** a base de
> **componentes**. Vite es la **herramienta** que crea el proyecto, lo sirve en
> desarrollo con recarga instantánea y lo empaqueta para producción.

---

## 1.1 El problema que resuelve React

En la carpeta `JAVASCRIPT/06-dom-y-eventos.md` pintabas la página **a mano**:

```js
// JavaScript "vanilla" (imperativo): TÚ dices CÓMO cambiar el DOM paso a paso
const span = document.querySelector('#contador');
let valor = 0;
document.querySelector('#mas').addEventListener('click', () => {
  valor++;
  span.textContent = valor;                 // ¿y si hay 5 sitios que muestran "valor"?
  if (valor === 10) boton.disabled = true;  // hay que acordarse de todo
});
```

Con React describes **QUÉ** quieres ver para cada valor del estado:

```tsx
// React (declarativo): TÚ dices QUÉ se ve; React decide cómo actualizar el DOM
const [value, setValue] = useState(0)
return (
  <>
    <span>{value}</span>
    <button onClick={() => setValue(v => v + 1)} disabled={value === 10}>+</button>
  </>
)
```

| | JS vanilla (imperativo) | React (declarativo) |
|---|---|---|
| Piensas en… | Pasos: busca nodo, cámbiale el texto… | Estados: "si value es 10, el botón está desactivado" |
| Sincronizar datos y pantalla | A mano (fuente de bugs) | Automático |
| Reutilizar trozos de UI | Copiar/pegar HTML o funciones que crean nodos | **Componentes** (`<CourseCard />` × 3) |
| Escala a apps grandes | Difícil | Para eso nació (Facebook, 2013) |

> 📌 **UI = f(estado)**. Un componente es una función que recibe datos y devuelve
> lo que hay que pintar. Si cambian los datos, React vuelve a llamar a la función.

---

## 1.2 Conceptos clave

### SPA (Single Page Application)

```
Web clásica (MPA)                      SPA con React
──────────────────                     ───────────────────────────────
/inicio   → servidor manda HTML         Primera visita → index.html + bundle JS
/cursos   → servidor manda OTRO HTML    Navegar → JS cambia lo que se ve, SIN recargar
/contacto → recarga completa            Datos → fetch a una API que devuelve JSON
```

- El servidor manda **un solo HTML** casi vacío (`<div id="root">`).
- El JavaScript construye toda la página **en el navegador**.
- Los datos llegan como **JSON** desde tu API (Express/NestJS).

### Virtual DOM y reconciliación

1. Cada render, tu componente devuelve un **árbol de objetos JS** (el "Virtual DOM").
2. React lo **compara** con el árbol anterior (*diffing*).
3. Aplica al DOM real **solo los cambios mínimos** (*reconciliación*).

```
Antes:  <span>3</span>   Después: <span>4</span>   →  React solo cambia el texto "3" → "4"
```

> 💡 Tocar el DOM real es lo "caro"; comparar objetos JS es barato.

### Librería, no framework

React solo se encarga de **la vista**. Para rutas, peticiones, formularios
complejos o estado global se añaden librerías (React Router, TanStack Query…) o
se usa un *framework* construido sobre React (**Next.js**, React Router en modo
framework). En el bootcamp usas React "puro" con Vite.

---

## 1.3 ¿Qué es Vite?

| Pieza | Para qué |
|-------|----------|
| **Servidor de desarrollo** (`npm run dev`) | Sirve tus `.tsx` al navegador al vuelo, con **HMR** (*Hot Module Replacement*): guardas y el cambio aparece sin perder el estado |
| **Build** (`npm run build`) | Convierte TS + JSX en JS normal, lo **empaqueta y minifica** en `dist/` |
| **Plugins** | `@vitejs/plugin-react` le enseña a entender JSX y activa el *Fast Refresh* |

> 📌 El navegador **no entiende** ni TypeScript ni JSX. Vite los traduce a
> JavaScript (igual que `tsc` traducía tu backend a `dist/`).

---

## 1.4 Crear el proyecto

```bash
npm create vite@latest academia -- --template react-ts
cd academia
npm install
npm run dev          # → http://localhost:5173
```

> 💡 El puerto por defecto de Vite es **5173**. Es el `origin` que tendrás que
> permitir en el CORS de tu API (ver `06`).

---

## 1.5 Estructura del proyecto

### Estructura REAL (`Vite/academia`)

```
academia/
├── index.html              ← el ÚNICO HTML de la app
├── package.json            ← dependencias y scripts
├── vite.config.ts          ← configuración de Vite (plugin react)
├── tsconfig.json           ← "padre" que referencia a los otros dos
├── tsconfig.app.json       ← TS para el código de src/ (navegador)
├── tsconfig.node.json      ← TS para vite.config.ts (Node)
├── .oxlintrc.json          ← linter (reglas de hooks)
├── public/                 ← archivos que se copian TAL CUAL (favicon.svg)
├── dist/                   ← resultado de npm run build (NO se sube a Git)
└── src/
    ├── main.tsx            ← punto de entrada: monta React en #root
    ├── App.tsx             ← componente raíz
    ├── App.css             ← estilos de App
    ├── index.css           ← estilos globales (variables, modo oscuro)
    ├── assets/             ← imágenes que se IMPORTAN desde el código
    └── components/
        ├── CourseCard.tsx
        ├── Counter.tsx
        └── Footer.tsx
```

| Carpeta | Cómo se usa |
|---------|-------------|
| `public/` | Se referencia con ruta absoluta: `<link href="/favicon.svg">`. No se procesa |
| `src/assets/` | Se importa: `import logo from './assets/react.svg'` → Vite le pone hash para caché |

---

## 1.6 Cómo arranca la app: `index.html` → `main.tsx` → `App.tsx`

### Ejemplo REAL (`academia/index.html`)

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>academia</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

- `<div id="root">` vacío: aquí React pintará **todo**.
- `<script type="module" src="/src/main.tsx">`: Vite intercepta esto y sirve el TSX traducido.

> 💡 Mejora: el contenido está en español → `<html lang="es">` (accesibilidad y
> lectores de pantalla, ver `HTML/06`).

### Ejemplo REAL (`academia/src/main.tsx`)

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css"; // estilos globales
import App from "./App.tsx";

// Busca el <div id="root"> y pinta App dentro
createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

| Línea | Qué hace |
|-------|----------|
| `react-dom/client` | React se divide en `react` (componentes, hooks) y `react-dom` (pintar en el navegador) |
| `import "./index.css"` | Importar un CSS lo **inyecta** en la página (cosa de Vite, no de JS) |
| `getElementById("root")!` | El `!` es la *non-null assertion* de TS: "te aseguro que no es `null`" (ver `TYPESCRIPT/06`) |
| `createRoot(...).render(...)` | Crea la raíz de React y pinta `<App />` dentro |
| `<StrictMode>` | Solo en desarrollo: ejecuta dos veces renders y efectos para **detectar bugs** (ver `08`) |

---

## 1.7 `package.json` y scripts

### Ejemplo REAL (`academia/package.json`)

```json
{
  "name": "academia",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "oxlint",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^19.3.1",
    "react-dom": "^19.3.1"
  },
  "devDependencies": {
    "@types/react": "^19.3.0",
    "@types/react-dom": "^19.3.0",
    "@vitejs/plugin-react": "^6.1.2",
    "oxlint": "^1.90.0",
    "typescript": "~7.0.2",
    "vite": "^8.3.6"
  }
}
```

| Script | Qué hace | Cuándo |
|--------|----------|--------|
| `npm run dev` | Servidor de desarrollo con HMR en `:5173` | Mientras programas |
| `npm run build` | `tsc -b` **comprueba tipos** (no genera JS, `noEmit`) y luego `vite build` genera `dist/` | Antes de desplegar |
| `npm run preview` | Sirve `dist/` en local para probar el build | Para comprobar el resultado final |
| `npm run lint` | Pasa **oxlint** (un linter muy rápido, alternativa a ESLint) | Antes de commitear |

> 📌 Fíjate: si hay un **error de tipos**, `npm run build` falla aunque `npm run dev`
> funcione. Vite en desarrollo **no comprueba tipos**, solo los borra. El editor
> (VS Code) y `tsc -b` son los que te avisan.

### `vite.config.ts` (REAL)

```ts
import react from '@vitejs/plugin-react'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [react()],
})
```

### `tsconfig.app.json` (lo importante)

```jsonc
{
  "compilerOptions": {
    "target": "es2023",
    "lib": ["ES2023", "DOM"],        // tipos del navegador (document, fetch…)
    "types": ["vite/client"],        // tipos de import.meta.env, imports de CSS/SVG
    "moduleResolution": "bundler",   // resuelve imports como lo hace Vite
    "noEmit": true,                  // TS solo comprueba; Vite genera el JS
    "jsx": "react-jsx",              // JSX moderno: NO hace falta import React
    "noUnusedLocals": true,          // error si declaras algo y no lo usas
    "noUnusedParameters": true
  },
  "include": ["src"]
}
```

> 💡 Por `"jsx": "react-jsx"` tus componentes **no** empiezan con
> `import React from 'react'` (eso era obligatorio en React antiguo).

---

## 1.8 React vs JS vanilla vs backend: dónde encaja

```
 FRONTEND (navegador)                  BACKEND (servidor)          BASE DE DATOS
 ┌───────────────────────┐   HTTP      ┌────────────────────┐      ┌───────────────┐
 │ React + Vite (TSX)    │ ──JSON────▶ │ Express / NestJS   │ ───▶ │ Postgres /    │
 │ Componentes, estado   │ ◀──JSON──── │ Rutas, validación  │ ◀─── │ MongoDB       │
 │ puerto 5173 (dev)     │             │ puerto 3000        │      │ Supabase/Atlas│
 └───────────────────────┘             └────────────────────┘      └───────────────┘
```

> 📌 React **no** sustituye a tu API: la **consume**. Nunca pongas contraseñas de
> la base de datos en el frontend: todo lo que hay en el bundle lo puede leer
> cualquiera con las DevTools.

---

## 🎯 Resumen

| Concepto | En una frase |
|----------|--------------|
| React | Librería para construir UIs con componentes; UI = f(estado) |
| Declarativo | Describes el resultado, no los pasos |
| Virtual DOM | Árbol de objetos que React compara para tocar el DOM lo mínimo |
| SPA | Un solo HTML; JS pinta y cambia las "páginas" sin recargar |
| Vite | Servidor de desarrollo con HMR + empaquetador para producción |
| `main.tsx` | Monta `<App />` en `#root` con `createRoot` |
| `npm run build` | Comprueba tipos y genera `dist/` (HTML + JS + CSS estáticos) |
