# 09 — BUILD, DESPLIEGUE Y ERRORES COMUNES

> Un frontend de React, una vez compilado, son **archivos estáticos**: un HTML,
> un JS y un CSS. No necesita Node en producción. Eso cambia **cómo** se
> despliega respecto a tu API (`RENDER/`).

---

## 9.1 Qué hace `npm run build`

### Script REAL (`academia/package.json`)

```json
"build": "tsc -b && vite build"
```

```
1. tsc -b        → comprueba los tipos de TODO el proyecto (no genera archivos: noEmit)
                   si hay un error de tipos → el build se para aquí ❌
2. vite build    → traduce TSX → JS, une los módulos, minifica, pone hashes
                   y lo escribe en dist/
```

### Resultado REAL (`academia/dist/`)

```
dist/
├── index.html                    ← el mismo HTML, apuntando a los archivos con hash
├── favicon.svg                   ← copiado de public/
├── icons.svg
└── assets/
    ├── index-7VdiLb7_.js         ← TODO tu código + React, en un archivo minificado
    └── index-t7IoUIW6.css        ← index.css + App.css unidos
```

| Detalle | Por qué |
|---------|---------|
| Hash en el nombre (`-7VdiLb7_`) | Si cambia el código, cambia el nombre → el navegador **no** usa una versión vieja en caché |
| Un solo JS | Menos peticiones; con `import()` dinámico se puede dividir (*code splitting*) |
| `dist/` en `.gitignore` | Se **genera**, no se versiona (como el `dist/` de tu backend) |

```bash
npm run build
npm run preview      # sirve dist/ en http://localhost:4173 para probarlo
```

> ⚠️ Abrir `dist/index.html` con doble clic **no funciona** (rutas absolutas
> `/assets/...` y módulos ES): usa `npm run preview` o un servidor.

---

## 9.2 Frontend vs backend en producción

| | Tu API (Express / NestJS) | Tu frontend (React + Vite) |
|---|---|---|
| Qué se despliega | Un **proceso** Node que escucha en `PORT` | **Archivos** estáticos |
| En Render | *Web Service* | *Static Site* |
| Comando de arranque | `npm start` / `node dist/main.js` | Ninguno (lo sirve un CDN) |
| Se duerme en el plan gratis | ✅ Sí (cold start ~50 s) | ❌ No: los estáticos siempre responden |
| Variables de entorno | Se leen **al arrancar** (`process.env`) | Se **incrustan al hacer el build** (`import.meta.env.VITE_*`) |
| Secretos | ✅ Aquí sí | ❌ Nunca |

---

## 9.3 Desplegar en Render como *Static Site*

1. Sube el proyecto a GitHub (sin `node_modules`, sin `dist`, sin `.env` con nada sensible).
2. Render → **New → Static Site** → elige el repositorio.
3. Configuración:

| Campo | Valor |
|-------|-------|
| Root Directory | La carpeta del front si el repo tiene varias (p. ej. `academia`) |
| Build Command | `npm install && npm run build` |
| Publish Directory | `dist` |
| Environment | `VITE_API_URL=https://<tu-api>.onrender.com` |

4. **Rewrite para SPA** (si usas React Router): *Redirects/Rewrites* →
   Source `/*`, Destination `/index.html`, Action **Rewrite**.
5. En tu **API**, permite el nuevo origen en CORS:
   `CORS_ORIGIN=https://<tu-front>.onrender.com`.

```
 navegador ──▶ https://academia.onrender.com     (Static Site: index.html + JS)
     │
     └─ fetch ─▶ https://<api>.onrender.com/api/products   (Web Service, CORS_ORIGIN = el front)
                          │
                          └─▶ MongoDB Atlas / Supabase
```

> ⚠️ Como `VITE_API_URL` se **incrusta en el build**, si la cambias en el panel
> tienes que **volver a desplegar** (rebuild) para que tenga efecto.

> 💡 Otras plataformas habituales para frontends estáticos: **Vercel**, **Netlify**,
> **GitHub Pages**, Cloudflare Pages. El concepto es el mismo: build command +
> carpeta `dist`.

---

## 9.4 Errores comunes y cómo leerlos

### En la consola del navegador / pantalla

| Mensaje | Causa | Arreglo |
|---------|-------|---------|
| Pantalla en **blanco** | Un error de JS rompió el render | Abre la consola (F12): el primer error rojo es el importante |
| `Each child in a list should have a unique "key" prop` | `.map()` sin `key` | `key={item.id}` (`05`) |
| `Objects are not valid as a React child` | Pintas un objeto: `{product}` | Pinta un campo: `{product.name}` |
| `Too many re-renders` | `setX()` durante el render u `onClick={setX(1)}` | `onClick={() => setX(1)}` |
| `Cannot read properties of undefined (reading 'map')` | El dato aún no ha llegado o la API devolvió otra cosa | Estado inicial `[]` y comprobar `res.ok` |
| `Rendered fewer hooks than expected` | Hook dentro de un `if` o tras un `return` | Reglas de hooks (`08`) |
| `A component is changing an uncontrolled input to be controlled` | `value={undefined}` y luego un string | Inicializa con `''` |
| `blocked by CORS policy` | La API no permite tu origen | `CORS_ORIGIN` en la API o proxy de Vite (`06`) |
| `Failed to fetch` | API caída, URL mal, `http` vs `https`, o Render dormido | Abre la URL de la API en el navegador |
| `Mixed Content` | Front en `https` llamando a API en `http` | La API también en `https` |
| `import.meta.env.VITE_API_URL` es `undefined` | Nombre sin prefijo `VITE_`, o no reiniciaste `npm run dev` tras crear el `.env` | Prefijo + reiniciar |

### En la terminal / en el build

| Mensaje | Causa | Arreglo |
|---------|-------|---------|
| `error TS6133: 'Footer' is declared but its value is never read` | Importas algo y no lo usas (`noUnusedLocals: true` en tu `tsconfig.app.json`) | Úsalo o borra el import |
| `Type 'string' is not assignable to type 'number'` | Props mal pasadas (`hours="25"`) | `hours={25}` |
| `Failed to resolve import "./components/coursecard"` | Mayúsculas (Windows no distingue, Linux/Render **sí**) | Nombre exacto del archivo |
| Funciona en `dev`, falla en `build` | `vite dev` **no** comprueba tipos | Ejecuta `npm run build` antes de subir |
| 404 al recargar `/cursos/react` en producción | Falta el rewrite a `index.html` | Rewrite `/*` → `/index.html` |

> 📌 La pareja **dev funciona / Render falla** casi siempre es: **mayúsculas en
> imports**, **tipos** (que solo comprueba el build) o **variables de entorno**.
> Igual que con tu backend (`RENDER/05`).

---

## 9.5 Checklist antes de desplegar

- [ ] `npm run build` pasa en local sin errores.
- [ ] `npm run preview` y la app funciona contra la API **desplegada**.
- [ ] `npm run lint` sin errores.
- [ ] Ningún secreto en el código ni en `VITE_*`.
- [ ] `VITE_API_URL` configurada en el panel.
- [ ] CORS de la API permite el dominio del front.
- [ ] Rewrite a `index.html` si hay rutas.
- [ ] `<html lang="es">` y `<title>` con el nombre real (en `academia` aún es `academia`).

---

## 9.6 Calidad: lint y tests

| Herramienta | Qué hace | En tu proyecto |
|-------------|----------|----------------|
| **oxlint** / ESLint | Detecta errores y malas prácticas (reglas de hooks, `key`…) | ✅ `npm run lint` |
| **TypeScript** | Errores de tipos | ✅ `tsc -b` en el build |
| **Vitest** | Tests unitarios (API igual que Jest, integrado con Vite) | ❌ Aún no |
| **React Testing Library** | Testear componentes **como un usuario**: busca por texto/rol, hace clic, comprueba lo que se ve | ❌ Aún no |
| **Playwright** / Cypress | Tests E2E en un navegador real | ❌ Aún no |

```tsx
// Counter.test.tsx — así se testearía tu Counter
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { Counter } from './Counter'

test('no baja de 0 y sube con +', async () => {
  render(<Counter />)
  const menos = screen.getByRole('button', { name: '−' })
  const mas = screen.getByRole('button', { name: '+' })

  expect(menos).toBeDisabled()                 // empieza en 0
  await userEvent.click(mas)
  expect(screen.getByText('1')).toBeInTheDocument()
  expect(menos).toBeEnabled()
})
```

> 📌 La filosofía de Testing Library: **testea lo que ve el usuario**, no el
> estado interno. Por eso busca por **rol** y **texto** (y por eso la
> accesibilidad ayuda también a testear).

---

## 🎯 Resumen

| Concepto | Clave |
|----------|-------|
| `npm run build` | `tsc -b` (tipos) + `vite build` → `dist/` |
| Producción | Archivos estáticos: *Static Site*, sin Node, no se duerme |
| `VITE_*` | Se incrustan en el build: públicas y requieren rebuild |
| SPA + rutas | Rewrite `/*` → `/index.html` |
| Errores típicos | `key`, objetos en JSX, CORS, mayúsculas, `res.ok` |
| Calidad | Lint + tipos + (Vitest + Testing Library) |
