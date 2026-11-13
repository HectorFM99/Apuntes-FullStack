# 02 — 🎨 HTML Y CSS: PREGUNTAS CLAVE

> En una entrevista FullStack junior el HTML/CSS no suele ser lo más preguntado, pero
> **fallar lo básico** (box model, especificidad, flex) da muy mala impresión. Repásalo.

---

## 2.1 HTML semántico

Usar la **etiqueta que describe el significado** del contenido, no solo `<div>` para todo.

```html
<!-- ❌ "Div-itis" -->
<div class="header">
  <div class="nav">...</div>
</div>
<div class="contenido">
  <div class="titulo">Mi artículo</div>
</div>

<!-- ✅ Semántico -->
<header>
  <nav>...</nav>
</header>
<main>
  <article>
    <h1>Mi artículo</h1>
    <section>...</section>
  </article>
  <aside>...</aside>
</main>
<footer>...</footer>
```

| Ventaja | Por qué |
|---------|---------|
| ♿ Accesibilidad | Los lectores de pantalla entienden la estructura |
| 🔎 SEO | Google entiende mejor qué es importante |
| 🧹 Mantenimiento | El código se lee mejor |

> 📌 Solo **un `<h1>`** por página y no saltes niveles (`h1` → `h2` → `h3`).

---

## 2.2 Accesibilidad (a11y)

| Buena práctica | Ejemplo |
|----------------|---------|
| `alt` en imágenes | `<img src="logo.png" alt="Logo de Nightjar">` (`alt=""` si es decorativa) |
| `label` asociado a cada input | `<label for="email">Email</label><input id="email">` |
| Usar `<button>` para acciones | ❌ `<div onclick>` ✅ `<button>` (es enfocable y funciona con teclado) |
| Contraste suficiente | Texto legible sobre su fondo |
| Navegable con teclado | `Tab` recorre todo, foco visible (`:focus-visible`) |
| `lang` en `<html>` | `<html lang="es">` |
| ARIA solo si hace falta | `aria-label` en un botón que solo tiene icono |

---

## 2.3 Box model ⭐

Cada elemento es una caja: **content → padding → border → margin**.

```
┌──────────── margin ────────────┐
│ ┌────────── border ──────────┐ │
│ │ ┌──────── padding ───────┐ │ │
│ │ │        content         │ │ │
│ │ └────────────────────────┘ │ │
│ └────────────────────────────┘ │
└────────────────────────────────┘
```

```css
/* content-box (por defecto): width = solo el contenido */
.caja { width: 200px; padding: 20px; border: 5px solid; } /* ocupa 250px */

/* ✅ border-box: width incluye padding y border */
*, *::before, *::after { box-sizing: border-box; }       /* ocupa 200px */
```

> 📌 **Margin collapse**: los márgenes verticales de dos bloques seguidos no se suman,
> se queda el mayor.

---

## 2.4 Especificidad y cascada ⭐

Cuando dos reglas chocan, gana la de **mayor especificidad**; si empatan, **la última**.

| Selector | Peso (a, b, c) |
|----------|----------------|
| `style="..."` en línea | 1, 0, 0, 0 |
| `#id` | 0, 1, 0, 0 |
| `.clase`, `[atributo]`, `:hover` | 0, 0, 1, 0 |
| `div`, `::before` | 0, 0, 0, 1 |
| `*` | 0 |

```css
#menu .item a   /* 1 id, 1 clase, 1 etiqueta → (1,1,1) */
.menu .item a   /* (0,2,1) → pierde contra el anterior */
```

```css
/* ❌ Abusar de !important: rompe la cascada y luego nadie puede sobreescribir */
.boton { color: red !important; }

/* ✅ Subir especificidad de forma controlada o reorganizar el CSS */
```

---

## 2.5 `display`: block, inline, inline-block

| | Ocupa toda la línea | Acepta `width`/`height` | Ejemplos |
|---|---|---|---|
| `block` | ✅ | ✅ | `div`, `p`, `h1`, `section` |
| `inline` | ❌ | ❌ | `span`, `a`, `strong` |
| `inline-block` | ❌ | ✅ | Botones en línea |
| `none` | Desaparece (no ocupa espacio) | — | — |

> 📌 `display: none` **quita** el elemento del flujo; `visibility: hidden` lo **oculta
> pero sigue ocupando** su espacio.

---

## 2.6 Flexbox vs Grid ⭐

| | **Flexbox** | **Grid** |
|---|---|---|
| Dimensiones | **1D** (fila **o** columna) | **2D** (filas **y** columnas) |
| Ideal para | Barras de navegación, alinear elementos, centrar | Layouts de página, galerías, tarjetas |
| Piensa en… | El **contenido** decide el tamaño | El **contenedor** define la rejilla |

```css
/* Centrar algo (la pregunta más típica) */
.padre {
  display: flex;
  justify-content: center; /* eje principal */
  align-items: center;     /* eje cruzado */
}

/* Galería responsive sin media queries */
.galeria {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
}
```

> 📌 No son rivales: es normal usar **Grid para el layout general** y **Flex dentro de
> cada componente**.

---

## 2.7 `position`

| Valor | Se coloca respecto a… | ¿Sale del flujo? |
|-------|----------------------|------------------|
| `static` | Por defecto, flujo normal | ❌ |
| `relative` | Su posición original | ❌ (deja su hueco) |
| `absolute` | El ancestro **posicionado** más cercano (no `static`) | ✅ |
| `fixed` | La ventana (viewport) | ✅ (no se mueve al hacer scroll) |
| `sticky` | Normal hasta un umbral de scroll, luego se "pega" | ❌ |

```css
.tarjeta { position: relative; }          /* referencia */
.tarjeta .badge { position: absolute; top: 8px; right: 8px; }
```

> 📌 `z-index` solo funciona en elementos posicionados (o hijos flex/grid).

---

## 2.8 Unidades

| Unidad | Relativa a | Uso típico |
|--------|-----------|------------|
| `px` | Absoluta | Bordes, detalles finos |
| `%` | El padre | Anchos |
| `em` | `font-size` del **elemento/padre** (se acumula) | Padding de un botón proporcional a su texto |
| `rem` | `font-size` del **`<html>`** (normalmente 16px) | ✅ Tipografía y espaciados |
| `vw` / `vh` | Ancho/alto del viewport | Secciones a pantalla completa |
| `fr` | Fracción del espacio libre en Grid | Columnas |

---

## 2.9 Responsive design

```html
<!-- Imprescindible en el <head> -->
<meta name="viewport" content="width=device-width, initial-scale=1">
```

```css
/* Mobile first: estilos base para móvil, y se amplían */
.contenedor { padding: 1rem; }

@media (min-width: 768px) {
  .contenedor { padding: 2rem; max-width: 960px; margin: 0 auto; }
}
```

| Técnica | Para qué |
|---------|----------|
| **Mobile first** | Empiezas por lo pequeño (más sencillo) y añades con `min-width` |
| Media queries | Cambiar estilos según el tamaño |
| Unidades relativas | `rem`, `%`, `fr`, `clamp()` |
| Imágenes flexibles | `img { max-width: 100%; height: auto; }` |

---

## 2.10 Otras preguntas rápidas

| Pregunta | Respuesta corta |
|----------|-----------------|
| ¿Pseudo-clase vs pseudo-elemento? | `:hover` = **estado**; `::before` = **parte** "virtual" del elemento |
| ¿Dónde pongo el `<script>`? | Al final del `<body>` o en `<head>` con `defer` |
| `defer` vs `async` | `defer`: se ejecuta en orden al acabar de parsear el HTML. `async`: en cuanto se descarga, sin orden |
| ¿Qué es el DOM? | La representación en árbol del HTML que JS puede leer y modificar |
| Variables CSS | `:root { --color-primario: #6c5ce7; }` → `color: var(--color-primario);` |

---

## 🎯 Cómo responderlo en la entrevista

**¿Qué es el HTML semántico y por qué importa?**
> "Usar etiquetas que describen el significado del contenido, como `header`, `nav`,
> `main`, `article` o `button`, en vez de divs genéricos. Mejora la accesibilidad,
> porque los lectores de pantalla entienden la estructura, el SEO y la legibilidad
> del código."

**Explica el box model.**
> "Cada elemento es una caja formada por contenido, padding, borde y margen. Con
> `box-sizing: border-box` el ancho incluye padding y borde, lo que hace los cálculos
> mucho más predecibles; por eso suelo aplicarlo a todo."

**¿Flexbox o Grid?**
> "Flexbox es unidimensional, ideal para alinear elementos en una fila o columna, como
> una navbar. Grid es bidimensional, ideal para el layout de la página o una galería.
> Los combino: Grid para la estructura y Flex dentro de los componentes."

**¿Cómo funciona la especificidad?**
> "Cuando dos reglas afectan al mismo elemento gana la más específica: estilos en
> línea, luego ids, luego clases y pseudo-clases, luego etiquetas. Si empatan, gana la
> última declarada. Intento evitar `!important` porque rompe la cascada."

**¿Cómo haces una web responsive?**
> "Con el meta viewport, enfoque mobile first, unidades relativas como rem y fr,
> Flexbox/Grid con `auto-fill` y `minmax`, y media queries con `min-width` para
> adaptar a pantallas más grandes."
