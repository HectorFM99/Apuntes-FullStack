# 📋 CHEATSHEET — CSS

> Chuleta rápida de comandos y código. Teoría en los documentos 01–10 de esta carpeta.

---

## 1. Formas de aplicarlo y sintaxis

```html
<link rel="stylesheet" href="css/styles.css">     <!-- ✅ externo (recomendado) -->
<style> p { color: red; } </style>                <!-- interno, en el head -->
<p style="color: red;">Hola</p>                   <!-- en línea: evitar -->
```

```css
selector {               /* a quién */
  propiedad: valor;      /* qué cambia */
}
/* comentario */
@import url("otro.css"); /* importar otro CSS (al principio) */
```

---

## 2. Selectores

| Selector | Selecciona |
|----------|------------|
| `*` | Todo |
| `p` | Etiqueta |
| `.card` | Clase |
| `#menu` | Id |
| `h1, h2` | Varios (grupo) |
| `nav a` | Descendiente (cualquier nivel) |
| `ul > li` | Hijo directo |
| `h2 + p` | Hermano inmediatamente siguiente |
| `h2 ~ p` | Todos los hermanos siguientes |
| `a[target="_blank"]` | Por atributo |
| `.card.destacada` | Elemento con **ambas** clases |

### Especificidad (de menos a más)

```
elemento (0,0,1)  <  .clase / :hover / [attr] (0,1,0)  <  #id (1,0,0)  <  style="" (inline)  <  !important
```

> 📌 A igual especificidad, gana la **última** regla. Evita `!important` e ids para estilos.

---

## 3. Texto y colores

```css
body {
  font-family: "Poppins", system-ui, sans-serif;  /* con alternativas */
  font-size: 16px;            /* px, rem (relativo al html), em (al padre) */
  font-weight: 700;           /* 400 normal, 700 negrita */
  line-height: 1.5;           /* interlineado sin unidad */
  color: #333;
  text-align: center;         /* left | right | justify */
  text-transform: uppercase;
  text-decoration: none;      /* quitar subrayado a los enlaces */
  letter-spacing: 0.5px;
}

.ejemplo {
  color: #aa3bff;                       /* hex */
  color: rgb(170 59 255);               /* rgb */
  background: rgba(170, 59, 255, 0.1);  /* con transparencia */
  color: hsl(270 100% 60%);             /* tono saturación luz */
  background: linear-gradient(135deg, #667eea, #764ba2);  /* degradado */
  background-image: url("../img/fondo.jpg");
  background-size: cover;               /* cubrir sin deformar */
  background-position: center;
  opacity: 0.8;
}
```

| Unidad | Relativa a |
|--------|------------|
| `px` | Fija |
| `rem` | `font-size` del `html` (16 px por defecto) |
| `em` | `font-size` del padre |
| `%` | Tamaño del padre |
| `vw` / `vh` | 1 % del ancho / alto de la ventana |
| `svh` | Alto de ventana real en móvil |

---

## 4. Box model

```css
*, *::before, *::after { box-sizing: border-box; }  /* width incluye padding y borde */

.card {
  width: 300px;  max-width: 100%;
  padding: 24px;                 /* dentro del borde */
  border: 1px solid #ddd;
  border-radius: 12px;           /* 50% = círculo, 999px = píldora */
  margin: 0 auto;                /* centrar horizontalmente un bloque con ancho */
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
}

/* atajos: arriba derecha abajo izquierda */
margin: 10px;            /* los 4 */
margin: 10px 20px;       /* vertical | horizontal */
margin: 10px 20px 5px 0; /* top right bottom left */
```

| `display` | Comportamiento |
|-----------|----------------|
| `block` | Ocupa toda la línea; acepta width/height |
| `inline` | En línea; ignora width/height y margin vertical |
| `inline-block` | En línea **con** width/height |
| `none` | No se pinta ni ocupa espacio |
| `flex` / `grid` | Contenedor de layout |

```css
.box { overflow: hidden; }        /* auto | scroll: contenido que se sale */
```

### Position

| Valor | Se coloca respecto a |
|-------|----------------------|
| `static` | Flujo normal (por defecto) |
| `relative` | Su propia posición (y sirve de referencia a hijos `absolute`) |
| `absolute` | El ancestro posicionado más cercano |
| `fixed` | La ventana (no se mueve al hacer scroll) |
| `sticky` | Normal hasta un umbral, luego se "pega" (`top: 0`) |

```css
.header { position: sticky; top: 0; z-index: 10; }   /* z-index: quién queda encima */
```

---

## 5. Flexbox (una dimensión)

```css
.contenedor {
  display: flex;
  flex-direction: row;            /* column */
  justify-content: space-between; /* eje principal: flex-start | center | space-around | space-evenly */
  align-items: center;            /* eje cruzado: stretch | flex-start | flex-end */
  flex-wrap: wrap;                /* saltar de línea */
  gap: 16px;                      /* espacio entre hijos */
}
.hijo { flex: 1; }                /* reparte el espacio libre */
.hijo { flex: 0 0 200px; }        /* grow shrink basis */
.hijo { align-self: flex-end; order: -1; }

/* Centrar cualquier cosa */
.centrar { display: flex; justify-content: center; align-items: center; min-height: 100vh; }
```

---

## 6. Grid (dos dimensiones)

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);                        /* 3 columnas iguales */
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));  /* responsive sin media queries */
  grid-template-rows: auto 1fr auto;
  gap: 20px;
}
.item { grid-column: 1 / 3; }     /* de la línea 1 a la 3 (2 columnas) */
.item { grid-column: span 2; }

/* Áreas con nombre */
.layout {
  grid-template-areas:
    "header header"
    "aside  main"
    "footer footer";
}
.header { grid-area: header; }
.grid { place-items: center; }    /* align + justify a la vez */
```

| Flex | Grid |
|------|------|
| Fila **o** columna | Filas **y** columnas |
| Menús, barras, centrar | Layouts de página, galerías de tarjetas |

---

## 7. Pseudoclases y pseudoelementos

```css
a:hover { color: purple; }            /* ratón encima */
a:focus-visible { outline: 2px solid; }  /* foco con teclado */
button:active { transform: scale(0.98); }
button:disabled { opacity: .5; cursor: not-allowed; }
input:invalid { border-color: red; }
li:first-child, li:last-child { }
li:nth-child(2n) { background: #eee; } /* pares (odd = impares) */
p:not(.intro) { }
.card:has(img) { }                      /* padre que contiene… */

.titulo::before { content: "★ "; }      /* contenido generado antes */
.titulo::after  { content: ""; display: block; width: 50px; height: 3px; background: #aa3bff; }
::selection { background: #aa3bff; color: #fff; }
input::placeholder { color: #999; }
```

---

## 8. Responsive

```css
/* Mobile first: estilos base para móvil, y se amplía */
.menu { flex-direction: column; }

@media (min-width: 768px) {         /* tablet en adelante */
  .menu { flex-direction: row; }
}
@media (min-width: 1024px) { }      /* escritorio */
@media (max-width: 600px) { }       /* desktop first: hasta 600 px */
@media (prefers-color-scheme: dark) { }      /* modo oscuro del sistema */
@media (prefers-reduced-motion: reduce) { * { animation: none; } }

img { max-width: 100%; height: auto; }      /* imágenes fluidas */
h1 { font-size: clamp(2rem, 5vw, 3.5rem); } /* mín, ideal, máx */
```

| Breakpoint típico | Dispositivo |
|-------------------|-------------|
| `< 600px` | Móvil |
| `768px` | Tablet |
| `1024px` | Portátil |
| `1280px+` | Escritorio |

---

## 9. Variables, transiciones y animaciones

```css
:root {
  --accent: #aa3bff;
  --radius: 12px;
}
.btn { background: var(--accent); border-radius: var(--radius); }
.btn { color: var(--texto, #333); }   /* valor por defecto */

@media (prefers-color-scheme: dark) {
  :root { --accent: #c084fc; }         /* tema oscuro cambiando solo variables */
}

.btn {
  transition: background 0.3s ease, transform 0.2s;   /* animar cambios */
}
.btn:hover { transform: translateY(-2px); }

@keyframes aparecer {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}
.card { animation: aparecer 0.6s ease-out forwards; }
/* animation: nombre duración timing delay iteraciones dirección fill-mode */
.loader { animation: girar 1s linear infinite; }
@keyframes girar { to { transform: rotate(360deg); } }
```

| `transform` | Efecto |
|-------------|--------|
| `translate(x, y)` | Mover |
| `scale(1.1)` | Escalar |
| `rotate(45deg)` | Girar |
| `skew(10deg)` | Inclinar |

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| La regla no se aplica | Mira en DevTools si está tachada: otra más específica o posterior gana |
| El ancho total se pasa de lo esperado | `box-sizing: border-box` |
| `margin: auto` no centra | El elemento necesita `width`/`max-width` y ser `block` |
| `width`/`height` no hacen nada | Es `inline`: usa `inline-block` o `block` |
| `absolute` se coloca respecto a la página | Pon `position: relative` al padre |
| `z-index` no funciona | Necesita `position` distinta de `static` (o ser hijo flex/grid) |
| Scroll horizontal en móvil | Algo con ancho fijo: `max-width: 100%`, revisa `100vw` |
| `::before` no aparece | Falta `content: ""` |
