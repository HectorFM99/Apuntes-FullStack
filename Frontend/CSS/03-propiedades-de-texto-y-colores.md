# 03 — PROPIEDADES DE TEXTO, COLOR Y FONDO

---

## 3.1 Color: las 4 formas de escribirlo

```css
/* 1. Palabras clave */
color: red;
color: white;

/* 2. HEXADECIMAL (#RRGGBB)  → 16.777.216 colores */
color: #e63946;
color: #000;        /* abreviado: #000 = #000000 (negro) */
color: #fff;        /* abreviado: #fff = #ffffff (blanco) */

/* 3. RGB / RGBA  (rojo, verde, azul + alfa/transparencia) */
color: rgb(230, 57, 70);
color: rgb(0 0 0 / 25%);      /* sintaxis moderna: 25% de transparencia */
color: rgba(0, 0, 0, 0.5);    /* sintaxis antigua equivalente */

/* 4. HSL  (Tono Saturación Luminosidad) → muy intuitivo */
color: hsl(0, 100%, 50%);

/* Otras */
color: currentColor;   /* toma el color de texto del padre */
color: inherit;        /* hereda el color del padre */
color: transparent;
```

### Ejemplos REALES copiados de tus proyectos

**Archivo:** `proyecto/cuerpo.css`

```css
p{
    color: #9c0000; 
    font-size: 16px;
}

h1{
    color: #4b4b4b; 
    font-size: 20px;
}
```

**Archivo:** `git/Pagina-Web/css/cuerpo.css` — transparencias con `rgb()`

```css
.pie-pagina {
  margin-top: auto;
  padding: 18px 20px;
  background-color: rgba(20, 20, 20, 0.92);
  color: white;
  text-align: center;
}

.contacto-social {
  ...
  border: 1px solid rgb(255 255 255 / 30%);
  background: rgb(20 20 20 / 75%);
  box-shadow: 0 8px 20px rgb(0 0 0 / 25%);
}
```

> 📌 `rgb(20 20 20 / 75%)` = casi negro, al 75% de opacidad (se transparenta el fondo).

---

## 3.2 Tipografía: `font-*`

```css
body {
  font-family: "Poppins", sans-serif;   /* familia + fallback */
  font-size: 16px;                      /* tamaño */
  font-weight: 400;                     /* grosor: 100..900, o normal/bold */
  font-style: normal;                   /* normal | italic */
  line-height: 1.6;                     /* interlineado (sin unidad = multiplicador) */
  letter-spacing: 0.5px;                /* espacio entre letras */
  word-spacing: 2px;                    /* espacio entre palabras */
  text-transform: uppercase;            /* mayúsculas | lowercase | capitalize */
  text-align: center;                   /* left | right | center | justify */
  color: #333;                          /* color del TEXTO */
}
```

### ⚠️ `font-family` con fallbacks

```css
font-family: "Poppins", sans-serif;
font-family: "Inter", "Segoe UI", Arial, sans-serif;
```

Si el navegador **no tiene** "Poppins", intenta "Segoe UI", luego Arial y por último
cualquier sans-serif. **Siempre** termina con una familia genérica.

### `@font-face` — fuentes propias (ejemplo real)

**Archivo:** `web-cv-hector/web-cv-hector/css/styles.css` *(copiado tal cual)*

```css
@font-face {
  font-family: 'Inter';
  src: url('../assets/fonts/Inter/static/Inter_18pt-Light.ttf') format('truetype');
  font-weight: 300;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Inter';
  src: url('../assets/fonts/Inter/static/Inter_18pt-Regular.ttf') format('truetype');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Inter';
  src: url('../assets/fonts/Inter/static/Inter_18pt-Bold.ttf') format('truetype');
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}
```

> 📌 **Análisis:**
> - `font-family: 'Inter'` → nombre con el que se **usará** (puede ser distinto al del archivo).
> - `src: url('ruta') format('truetype')` → el archivo `.ttf`. La ruta es **relativa al CSS** (`../` sube de `css/` a la raíz).
> - `font-weight: 300/400/700` → cada archivo cubre un grosor. Así `font-weight: 700` en cualquier regla **carga el Bold**.
> - `font-display: swap` → mientras carga la fuente, muestra el texto con la de respaldo en vez de dejar la página vacía.

### Fuentes de Google (ejemplo real)

**Archivo:** `git/Pagina-Web/index.html`

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&display=swap" rel="stylesheet">
<link href="https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400;700&display=swap" rel="stylesheet">
```

Y su uso en CSS (`git/Pagina-Web/css/cuerpo.css`):

```css
.titulo{
  font-family: "Dancing Script", cursive;
  font-size: 100px;
  color: white;
}

body {
  font-family: "Poppins", sans-serif;
}

p {
  font-family: "Roboto", sans-serif;
}
```

---

## 3.3 Alineación de texto y `line-height`

```css
p {
  text-align: center;        /* left | right | center | justify */
  line-height: 1.6;          /* 1.6 × el tamaño de fuente */
  text-decoration: none;     /* quita el subrayado de los enlaces */
  text-decoration: underline;
  text-transform: capitalize;
  word-wrap: break-word;     /* rompe palabras largas */
  overflow-wrap: break-word;
}
```

### Ejemplos REALES copiados

**Archivo:** `git/Pagina-Web/css/cuerpo.css`

```css
.centrado{
  text-align: center;
}

.texto {
  color: #555;
  line-height: 1.6;
}

.pie-pagina a {
  margin: 0;
  color: #d9d9d9;
  text-decoration: none;     /* enlaces del pie SIN subrayado */
  font-size: 14px;
}

.pie-pagina a:hover {
  color: white;
  text-decoration: underline; /* al pasar el ratón, SÍ subrayado */
}
```

**Archivo:** `web-cv-hector/css/styles.css`

```css
.work-experience__title {
  font-weight: 400;
  font-size: 0.875rem;
  line-height: 1.8;
  color: var(--color-text-heading);
  overflow-wrap: break-word;   /* evita que el texto desborde la caja */
}
```

> 📌 **Regla de oro de legibilidad:** `line-height` entre **1.4 y 1.8** para párrafos.

---

## 3.4 Propiedades de texto avanzadas

```css
.titulo {
  letter-spacing: -0.18px;      /* tracking: separación entre letras */
  white-space: nowrap;          /* prohibir saltos de línea */
  text-overflow: ellipsis;      /* "..." cuando desborda */
  overflow: hidden;
  max-width: 200px;

  -webkit-text-stroke: 1px black;  /* contorno de texto */
  text-shadow: 2px 2px 4px rgba(0,0,0,.5);  /* sombra */
  text-transform: uppercase;
}
```

**Ejemplo real** (`web-minimalista-hector/.../css/styles.css`):

```css
.brand__name {
  font-size: 20px;
  font-weight: 400;
  line-height: 1.52;
  letter-spacing: -0.18px;
  white-space: nowrap;
}

.menu__link {
  display: block;
  font-family: "Source Code Pro", sans-serif;
  font-size: 16px;
  font-weight: 400;
  line-height: 1.03;
  letter-spacing: -0.72px;
  white-space: nowrap;
  cursor: pointer;
}
```

---

## 3.5 Fondos: `background`

```css
.caja {
  background-color: #f6f6f6;
  background-image: url("../assets/images/fondo.jpg");
  background-size: cover;              /* cubre toda la caja recortando */
  background-size: contain;            /* cabe entera, puede sobrar hueco */
  background-position: center;         /* dónde se alinea */
  background-repeat: no-repeat;        /* no repetir */
  background-attachment: fixed;        /* no se mueve al hacer scroll */
  background-blend-mode: multiply;
}
```

> ⚠️ `background` es un **atajo** que agrupa todas las anteriores.
> Orden recomendado: `color`, `image`, `repeat`, `position`, `size`, `attachment`.

### Shorthand real: `background: url() ... ` combinado

**Archivo:** `git/Pagina-Web/css/cuerpo.css`

```css
body {
min-height: 100vh;
  display: flex;
  flex-direction: column;
  font-family: "Poppins", sans-serif;
  color: var(--color-texto);
  background: #f4f4f4 url("../assets/images/Fondo.jpg") center / cover no-repeat fixed;
}
```

**Desglose del shorthand:**

```css
background:
  #f4f4f4                                  /* background-color (por si no carga la img) */
  url("../assets/images/Fondo.jpg")        /* background-image */
  center                                   /* background-position */
  / cover                                  /* background-size */
  no-repeat                                /* background-repeat */
  fixed;                                   /* background-attachment */
```

**Otros reales del mismo archivo:**

```css
.imagen-barra-superior {
  background: url("../assets/images/Banners superior.jpg") center / cover no-repeat;
}

.imagen-1 {
  background-image: url("../assets/images/Icono1.png");
}

.imagen-proyecto {
  height: 160px;
  background-size: cover;
  background-position: center;
}
```

> 📌 Recordatorio: la ruta del `url()` es **relativa al archivo CSS**, no al HTML.

### Gradientes

```css
background: linear-gradient(to right, #e63946, #f4a261);
background: radial-gradient(circle at 50% 0%, #fff, #ddd);
background: linear-gradient(rgba(0,0,0,.4), rgba(0,0,0,.7)), url(fondo.jpg) center/cover;
```

---

## 3.6 `display` — cómo se comporta un elemento

```css
display: block;         /* ocupa todo el ancho, va debajo (div, p, h1) */
display: inline;        /* fluye con el texto (span, a, b) */
display: inline-block;  /* en línea PERO acepta width/height/padding vertical */
display: flex;          /* contenedor flexbox */
display: grid;          /* contenedor grid */
display: none;          /* NO se muestra y ocupa 0 de espacio */
visibility: hidden;     /* NO se muestra pero SIGUE ocupando su espacio */
```

### Ejemplo real — `inline-block` para botones

**Archivo:** `git/Pagina-Web/css/cuerpo.css`

```css
.boton-proyectos {
  display: inline-block;
  background-color: #e63946;
  color: white;
  padding: 10px 16px;
  text-decoration: none;
  border-radius: 6px;
}
```

> 📌 `inline-block` es lo que permite dar `padding` vertical a un `<a>` y que
> se quede junto a otro texto. Un `display: inline` puro **ignoraría** el `padding` vertical.

```css
.contenido-tarjeta p {
  margin: 0 0 16px;
  color: #555;
  line-height: 1.5;
}
```

> 📌 Este shorthand de `margin` son **4 valores**: `arriba derecha abajo izquierda`.
> `margin: 0 0 16px` = arriba 0, lados 0, abajo 16px.

---

## 3.7 Cursor y puntero

```css
a, button { cursor: pointer; }   /* manita */
.no-disponible { cursor: not-allowed; }
.cargando { cursor: wait; }
.mover { cursor: move; }
```

**Ejemplo real** (`git/Pagina-Web/css/cuerpo.css`):

```css
.boton-hamburguesa {
  font-family: "Roboto", sans-serif;
  background: #444;
  color: white;
  border: 0;
  padding: 10px 15px;
  font-size: 22px;
  cursor: pointer;
  border-radius: 5px;
}
```

---

## ✅ Resumen rápido

| Qué quiero | Propiedad |
|------------|-----------|
| Color del texto | `color` |
| Color de fondo | `background-color` |
| Imagen de fondo | `background-image` + `size: cover` + `position: center` + `no-repeat` |
| Tipografía | `font-family`, `font-size`, `font-weight`, `line-height` |
| Alineación | `text-align` |
| Quitar subrayado | `text-decoration: none` |
| Espaciado de letras | `letter-spacing` |
| Mostrar / ocultar | `display: none` vs `visibility: hidden` |
| En línea pero con padding | `display: inline-block` |

👉 **Siguiente:** `04-modelo-de-caja-box-model.md`
