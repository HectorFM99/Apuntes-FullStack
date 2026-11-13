# 02 — SELECTORES Y ESPECIFICIDAD

> El 90% de los problemas de CSS son **selectores mal puestos** (o falta de
> especificidad). Este documento es el más importante de la carpeta.

---

## 2.1 Selectores básicos

| Selector | Significado | Ejemplo CSS |
|----------|-------------|-------------|
| `*` | **Todos** los elementos | `* { margin: 0; }` |
| `p` | Todas las etiquetas `<p>` | `p { color: #555; }` |
| `.clase` | Todo lo que tenga esa **clase** | `.texto { color: #000; }` |
| `#id` | El elemento con ese **id** | `#texto-escrito { max-width: 90%; }` |
| `h1, h2` | Lista de selectores separados por coma | `h1, h2 { font-weight: 400; }` |

> 📌 **`.clase` es el selector más usado** con diferencia. Permite reutilizar.
> `#id` es muy específico (solo uno en la página) → úsalo solo cuando lo necesites
> (anclas, `<label for>`, o **un elemento muy concreto**).

---

## 2.2 Ejemplos REALES de selectores básicos

**Archivo:** `proyecto/cuerpo.css` *(copiado tal cual)* — selectores de **etiqueta** y de **clase**

```css
p{
    color: #9c0000; 
    font-size: 16px;
}

h1{
    color: #4b4b4b; 
    font-size: 20px;
}

title{
    color: #000000; 
    font-size: 24px;
}

.texto{
    color: #000000; 
    font-size: 12px;
}
```

> 📌 **Análisis:**
> - `p { }` → afecta a **TODOS** los párrafos del sitio.
> - `h1 { }` → a todos los `<h1>`.
> - `.texto` → solo a los elementos con `class="texto"` (en `main.html` hay uno).
> - ⚠️ **`title{...}` no funciona**: `<title>` está en el `<head>` y **no se ve**,
>   así que darle color no tiene efecto. Es un selector inútil aquí.

**Archivo:** `git/Pagina-Web/css/cuerpo.css` — más selectores de etiqueta

```css
ul{
  color: white;
}

html,
body {
  min-height: 100%;
  margin: 0;
}

p {
  font-family: "Roboto", sans-serif;
  margin-left: 25px;
  margin-right: 25px;
  color: white;
}

hr {
  width: 90%;
  margin: 20px auto;
  border: 0;
  border-top: 10px solid #000000;
}
```

> 📌 La coma en `html,\nbody { }` aplica **el mismo bloque** a los dos selectores.

**Archivo:** `git/Pagina-Web/css/cuerpo.css` — selector `#id`

```css
#texto-escrito {
  max-width: 90%;
  margin-inline: auto;
}
```

Ese `#texto-escrito` corresponde a este HTML (`git/Pagina-Web/pages/sobre_mi.html`):

```html
<p class="escritura" id="texto-escrito">Soy Hector, desarrollador web en formación...</p>
```

> 📌 Un mismo elemento puede llevar `class` **e** `id` a la vez.
> `id` para que JS lo encuentre; `class` para que CSS lo estilice.

---

## 2.3 Selectores por **estructura** (descendiente, hijo, hermano)

```css
/* DESCENDIENTE: cualquier <a> dentro de .menu-lateral (a cualquier profundidad) */
.menu-lateral a { ... }

/* HIJO DIRECTO: solo el <ul> que es hijo directo de .menu-lateral */
.menu-lateral > ul { ... }

/* Hermano SIGUIENTE (+): el <ul> que va justo después de un <li> */
li > ul { ... }

/* Todos los hermanos posteriores (~) */
.boton ~ .otro { ... }
```

| Combinador | Nombre | Significado |
|------------|--------|-------------|
| `A B` | Descendiente | `B` está **en algún punto dentro** de `A` |
| `A > B` | Hijo directo | `B` es **hijo inmediato** de `A` |
| `A + B` | Hermano adyacente | `B` viene **justo después** de `A` |
| `A ~ B` | Hermanos generales | `B` viene **después** de `A` (con cosas entre medias) |

### Ejemplos REALES copiados de `git/Pagina-Web/css/cuerpo.css`

```css
/* Quuitar formato de lista */
.menu-lateral > ul {
  list-style: none;
  margin: 0;
  padding: 20px;
}

/* Enlaces del panel */
.menu-lateral a {
  display: block;
  color: white;
  text-decoration: none;
  padding: 12px;
}

/* Efecto al pasar el ratón */
.menu-lateral a:hover {
  background-color: #555;
}

/* Submenús inicialmente cerrados */
.menu-lateral li > ul {
  list-style: none;
  max-height: 0;
  overflow: hidden;
  opacity: 0;
  ...
}

/* Submenú abierto al pasar el ratón */
.menu-lateral li:hover > ul {
  max-height: 200px;
  opacity: 1;
  ...
}
```

> 📌 `.menu-lateral > ul` (con `>`) solo toca el `<ul>` **directo**. Si hubiera un
> `<ul>` metido más adentro, **no** lo tocaría. Sin `>`, sí lo tocaría.

### Combinador `~` en acción — `git/Pagina-Web/css/cuerpo.css`

```css
/* Se muestra cuando el menú está abierto */
.menu-lateral.abierto ~ .fondo-difuminado {
  opacity: 1;
  visibility: visible;
  pointer-events: auto;
}
```

```html
<aside class="menu-lateral">...</aside>   <!-- hermano 1 -->
<div class="fondo-difuminado"></div>      <!-- hermano 2, DESPUÉS -->
```

> 📌 Cuando el `<aside>` tiene la clase `abierto`, **el `.fondo-difuminado` que viene
> después** se muestra. Es el mecanismo del fondo oscuro del menú hamburguesa.

---

## 2.4 Selectores por atributo

```css
a[href]              { }   /* todo <a> que tenga atributo href */
a[href="https://x"]  { }   /* href exacto */
a[href^="https"]     { }   /* href que EMPIEZA por https */
a[href$=".pdf"]      { }   /* href que TERMINA en .pdf */
a[href*="github"]    { }   /* href que CONTIENE "github" */
input[type="email"]  { }
```

---

## 2.5 Pseudoclases (estado)

Se escriben con `:` y describen un **estado** o una posición.

```css
a:hover { }                 /* al pasar el ratón */
a:visited { }               /* enlaces ya visitados */
a:focus { }                 /* con el foco (teclado) */
a:focus-visible { }         /* con foco SOLO si es visible (teclado, no ratón) */
li:first-child { }          /* el primer <li> */
li:last-child { }           /* el último */
li:nth-child(2) { }         /* el segundo */
li:nth-child(even) { }      /* pares */
input:checked { }           /* casilla marcada */
input:disabled { }          /* deshabilitado */
p::selection { }            /* texto seleccionado con el ratón */
:root { }                   /* el <html> */
```

### Ejemplos REALES copiados de `git/Pagina-Web/css/cuerpo.css`

```css
.enlace-proyecto:hover {
  background: #c92f3b;
}

.contacto-social:nth-child(2) {
  animation-delay: 140ms;
}

.contacto-social:nth-child(3) {
  animation-delay: 280ms;
}

.contacto-social:nth-child(4) {
  animation-delay: 420ms;
}
```

> 📌 Con `nth-child` se consigue que los 4 iconos de contacto **aparezcan escalonados**:
> el 2º espera 140ms, el 3º 280ms... Efecto "cascada" sin JS.

```css
.contacto-social:hover,
.contacto-social:focus-visible {
  transform: translateY(-7px) scale(1.06);
  box-shadow: 0 14px 26px rgb(0 0 0 / 40%);
}

.contacto-social:focus-visible {
  outline: 3px solid white;
  outline-offset: 4px;
}
```

> 📌 `:hover` = ratón. `:focus-visible` = teclado/lector de pantalla. Separarlos es
> buena práctica de accesibilidad: no quieres un anillo blanco cuando alguien hace clic.

### Pseudoclase compuesta: `.clase.estado`

```css
/* git/Pagina-Web/css/cuerpo.css */
.menu-lateral.abierto {
  left: 0;
}

/* git/Pagina-Web/css/cuerpo.css */
.reveal.is-visible {
  opacity: 1;
  filter: blur(0);
  transform: translateX(0);
}
```

> 📌 `.menu-lateral.abierto` = elementos que tienen **LAS DOS** clases.
> Es el patrón clásico: la base `.menu-lateral` lo pone fuera de pantalla
> (`left: -250px`) y `.abierto` lo trae dentro (`left: 0`). JavaScript solo añade
> o quita la clase `abierto`.

```js
/* git/Pagina-Web/js/programacion.js — la otra mitad */
botonMenu.addEventListener("click", function () {
  menuLateral.classList.toggle("abierto");
});
```

---

## 2.6 Pseudoelementos (se escriben con `::`)

Crean contenido que **no existe en el HTML**.

```css
.parrafo::before { content: "★ "; }      /* ANTES del contenido */
.parrafo::after  { content: " ©"; }      /* DESPUÉS del contenido */
.parrafo::first-letter { font-size: 3em; }   /* primera letra */
.parrafo::first-line   { font-weight: bold; }/* primera línea */
input::placeholder { color: #999; }      /* placeholder del campo */
::selection { background: gold; }         /* texto seleccionado */
```

```css
/* Punto decorativo en listas sin tener que tocar el HTML */
.menu-lateral li::before {
  content: "▸ ";
  color: #e63946;
}

/* Línea que crece al pasar el ratón */
.boton::after {
  content: "";
  display: block;
  height: 2px;
  background: currentColor;
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 0.3s;
}
.boton:hover::after {
  transform: scaleX(1);
}
```

---

## 2.7 ESPECIFICIDAD — ¿quién gana si hay conflicgo?

Puntuación por selector (0–0–0,0–0,0):

| Tipo | Valor | Ejemplo |
|------|-------|---------|
| Estilo en línea `style="..."` | **1000** (por encima de todo) | `style="color:red"` |
| `#id` | **100** | `#principal` |
| `.clase` / `:pseudoclase` / `[atributo]` | **10** | `.tarjeta`, `:hover` |
| `etiqueta` / `::pseudoelemento` | **1** | `p`, `article`, `::before` |
| `*`, combinadores (`>`, `+`, `~`) | **0** | `*`, `.a > .b` |

**Se suman columna a columna. Gana quien tenga más a la izquierda.**

```
#menu .activo a        →  1 (id) + 1 (clase) + 1 (etiqueta)  = 1-1-1
.tarjeta a             →  0 + 2 clases + 1 etiqueta          = 0-2-1
nav ul li a            →  0 + 0 + 4 etiquetas                = 0-0-4  ← PERDE
```

> 📌 Fíjate: `nav ul li a` (4 etiquetas) **pierde** contra `.tarjeta a` (1 clase + 1 etiqueta).
> Por eso, en proyectos grandes, se prefiere **clases** sobre anidamiento profundo.

### Ejemplo de cascada real (mismo selector escrito dos veces)

**Archivo:** `git/Pagina-Web/css/cuerpo.css`

```css
.tarjeta-certificado {
  padding: 24px;
  border: 1px solid rgb(255 255 255 / 25%);
  border-radius: 14px;
  background: rgb(20 20 20 / 75%);
  color: white;
  box-shadow: 0 6px 18px rgb(0 0 0 / 25%);
}

/* ... más abajo en el MISMO archivo ... */

.tarjeta-certificado {
  animation: deslizar-derecha 700ms ease-out both;
}
```

> 📌 **Misma especificidad** → las dos se **aplican** (no se sustituyen): las
> propiedades distintas se suman. Si las dos declararan `background`, ganaría la
> segunda porque está más abajo.

### Errores típicos de especificidad

```css
/* ❌ Problema clásico: "mi clase no funciona" */
.boton { background: red; }

/* Alguien más abajo puso: */
#pagina .contenido p { background: blue; }   /* especificidad mayor → gana */

/* ✅ Solución 1: subir especificidad (sin pasarte) */
#pagina .contenido .boton { background: red; }

/* ✅ Solución 2: colocar tu regla MÁS ABAJO en el archivo */

/* ✅ Solución 3 (con moderación): */
.boton { background: red !important; }
```

> ⚠️ **`!important` rompe la cascada.** Úsalo solo para arreglos puntuales
> (por ejemplo, anular estilos de una librería externa), nunca como estilo habitual.

---

## 2.8 Buenas prácticas de selectores

```css
/* ✅ Buena: plano, legible, reutilizable */
.tarjeta { }
.tarjeta__titulo { }
.tarjeta--destacada { }

/* ✅ Aceptable: anidación corta con significado */
.menu-lateral a { }

/* ❌ Mal: anidamiento profundo y frágil */
.container div ul li a span.icono b { }

/* ❌ Mal: usar #id para estilizar (rompe la reutilización) */
#titulo { }

/* ❌ Mal: !important por todas partes */
p { color: red !important; }
```

### Convención BEM (que usan tus proyectos)

```
.bloque                  → .tarjeta
.bloque__elemento        → .tarjeta__titulo
.bloque--modificador     → .menu-lateral.abierto  /  .estado-en-curso
```

Real, de `git/Pagina-Web/css/cuerpo.css`:

```css
.estado-certificado {
  display: inline-block;
  padding: 6px 10px;
  border-radius: 999px;
  background: #315f43;
  color: white;
  font-size: 13px;
  font-weight: 700;
}

.estado-en-curso {
  background: #315b78;
}
```

Real, de `web-cv-hector/css/styles.css`:

```css
.work-experience__date { ... }
.work-experience__details { ... }
.work-experience__title { ... }
.work-experience__location { ... }
```

---

## ✅ Resumen rápido

1. Selector básico: etiqueta (`p`), clase (`.x`), id (`#x`), universal (`*`).
2. Combinadores: descendiente (espacio), hijo (`>`), hermano siguiente (`+`), hermanos (`~`).
3. Pseudoclases `:` = estados (`:hover`, `:nth-child`, `:focus-visible`).
4. Pseudoelementos `::` = contenido inventado (`::before`, `::after`).
5. Especificidad: **estilo en línea > `#id` > `.clase` > etiqueta > `*`**; en empate, gana la última escrita.
6. Prefiere **clases** reutilizables a `#id` y a selectores anidados profundos.
7. `!important` = último recurso.

👉 **Siguiente:** `03-propiedades-de-texto-y-colores.md`
