# 07 — PSEUDOCLASES, PSEUDOELEMENTOS Y SELECTORES AVANZADOS

*(En `02-selectores-y-especificidad.md` se vio la base. Aquí el detalle con ejemplos reales.)*

---

## 7.1 Recordatorio rápido

| Símbolo | Nombre | Ejemplo |
|---------|--------|---------|
| `:` | Pseudoclase (estado/posición) | `:hover`, `:nth-child(2)` |
| `::` | Pseudoelemento (contenido inventado) | `::before`, `::after` |

> En CSS2 se escribían con `:`, así que `:before` sigue funcionando.
> **Lo correcto hoy es `::before`.**

---

## 7.2 Pseudoclases de INTERACCIÓN

```css
a:link            { }   /* enlace sin visitar */
a:visited         { }   /* enlace visitado */
a:hover           { }   /* RATÓN encima */
a:active          { }   /* mientras se tiene el botón PULSADO */
a:focus           { }   /* tiene el foco (clic o tabulador) */
a:focus-visible   { }   /* tiene foco SOLO si es de teclado (recomendado) */
a:focus-within     { }   /* foco en el elemento o en alguno de sus hijos */
```

> 📌 **Orden recomendado de escritura** (el orden importa por la cascada):
> `:link → :visited → :hover → :active`
> Truco: **L**o**V**e**HA** = "**LoVe HA**" (lo amas 😄).

### Ejemplos REALES copiados de `git/Pagina-Web/css/cuerpo.css`

```css
.enlace-proyecto:hover {
  background: #c92f3b;
}

.pie-pagina a:hover {
  color: white;
  text-decoration: underline;
}

.menu-lateral a:hover {
  background-color: #555;
}

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

> 📌 **`:hover` + `:focus-visible` juntos** = la página también es usable con
> teclado. Es una **buena práctica de accesibilidad** muy valorada.

### De `web-minimalista-hector/.../css/styles.css` — hover con descendientes

```css
.skip-link:focus {
  top: 0;
}

.brand__avatar:hover .brand__photo {
  transform: rotate(-15deg);
  transition: transform 0.5s cubic-bezier(0.34, 1.9, 0.64, 1);
}
```

> 📌 `.brand__avatar:hover .brand__photo` = "cuando el **avatar** tiene el ratón
> encima, cambia la **foto** que está dentro". Pseudoclase + descendiente.

---

## 7.3 Pseudoclases de ESTRUCTURA (posición en el árbol)

```css
:first-child    { }   /* el primer hijo de su padre */
:last-child     { }   /* el último hijo */
:nth-child(n)   { }   /* el n-ésimo hijo (1 = primero) */
:nth-child(odd) { }   /* hijos 1, 3, 5... */
:nth-child(even){ }   /* hijos 2, 4, 6... */
:nth-child(3n)  { }   /* cada 3 (3, 6, 9...) */
:nth-child(3n+1){ }   /* 1, 4, 7, 10... */
:nth-of-type(h1){ }   /* el n-ésimo <h1> (ignora otras etiquetas) */
:not(.clase)    { }   /* los que NO tengan esa clase */
:empty          { }   /* sin contenido */
:only-child     { }   /* hijo único */
```

### Ejemplo REAL de `nth-child` escalonado — `git/Pagina-Web/css/cuerpo.css`

```css
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

> 📌 Con **una sola animación** definida para `.contacto-social`, estos tres
> selectores consiguen que los iconos aparezcan **uno tras otro** (efecto cascada)
> sin escribir tres clases distintas en el HTML.

### Filas zebradas en tablas

```css
tbody tr:nth-child(even) {
  background-color: #fafafa;
}
```

### Selectores `:not()` y combinados

```css
/* Todo excepto el último */
.tarjeta:not(:last-child) {
  margin-bottom: 24px;
}

/* Menú lateral abierto */
.menu-lateral.abierto {
  left: 0;
}

/* Submenú abierto al pasar el ratón */
.menu-lateral li:hover > ul {
  max-height: 200px;
  opacity: 1;
}
```

---

## 7.4 Pseudoclases de ESTADO de formulario

```css
input:checked        { }   /* marcado */
input:disabled       { }   /* deshabilitado */
input:required       { }   /* obligatorio */
input:invalid        { }   /* no válido */
input:valid          { }   /* válido */
input:placeholder-shown { } /* solo placeholder visible */
```

```css
input:invalid {
  border-color: #e63946;
}
input:valid {
  border-color: #315f43;
}
```

---

## 7.5 PSEUDOELEMENTOS (`::`)

```css
.elemento::before        { content: "..."; }   /* ANTES del contenido */
.elemento::after         { content: "..."; }   /* DESPUÉS del contenido */
.elemento::first-letter  { }                   /* primera letra */
.elemento::first-line    { }                   /* primera línea */
.elemento::selection     { }                   /* texto seleccionado */
input::placeholder       { }                   /* placeholder */
```

> ⚠️ **`::before` y `::after` NECESITAN `content`.** Sin él, no se generan.

### Usos prácticos

```css
/* 1) Viñeta personalizada */
.menu-lateral li::before {
  content: "▸ ";
  color: #e63946;
}

/* 2) Etiqueta antes de un título */
.seccion__titulo::before {
  content: "— ";
}

/* 3) Icono decorativo sin HTML extra */
.boton-descarga::after {
  content: " ↓";
}

/* 4) Línea que crece al hacer hover */
.boton::after {
  content: "";
  display: block;
  height: 2px;
  background: currentColor;
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 0.3s ease;
}
.boton:hover::after {
  transform: scaleX(1);
}

/* 5) Inicial grande tipo revista */
 article > p:first-of-type::first-letter {
  font-size: 3.5em;
  font-weight: 700;
  float: left;
  line-height: 1;
  margin: 0 8px 0 0;
}

/* 6) Marca de agua detrás de un bloque */
.hero::before {
  content: "";
  position: absolute;
  inset: 0;
  background: url("marca.png") center/contain no-repeat;
  opacity: 0.08;
  z-index: -1;
}
```

---

## 7.6 Combinadores — repaso con ejemplos reales

```css
A B        /* descendiente */
A > B      /* hijo directo */
A + B      /* hermano adyacente */
A ~ B      /* hermanos posteriores */
A:hover B  /* descendiente cuando A tiene hover */
```

**Reales de `git/Pagina-Web/css/cuerpo.css`:**

```css
/* Descendiente */
.menu-lateral a { display: block; }

/* Hijo directo */
.menu-lateral > ul { list-style: none; }
.menu-lateral li > ul { max-height: 0; overflow: hidden; }

/* Descendiente con pseudoclase */
.menu-lateral li:hover > ul { max-height: 200px; opacity: 1; }

/* Hermanos posteriores (¡el fundido del menú!) */
.menu-lateral.abierto ~ .fondo-difuminado {
  opacity: 1;
  visibility: visible;
  pointer-events: auto;
}

/* Hijo directo de clase compuesta */
.contenido-tarjeta h2 { margin: 0 0 10px; }
.contenido-tarjeta p  { margin: 0 0 16px; color: #555; line-height: 1.5; }

/* Selectores agrupados por COMA */
.contacto-social:hover,
.contacto-social:focus-visible {
  transform: translateY(-7px) scale(1.06);
}

/* Selectores encadenados (mismas clases juntas) */
.estado-certificado.estado-en-curso {
  background: #315b78;
}
```

---

## 7.7 Selectores por ATRIBUTO

```css
a[href]                        { }   /* cualquiera con href */
a[href^="https"]               { }   /* EMPIEZA por https  (^ = caret) */
a[href$=".pdf"]                { }   /* TERMINA en .pdf    ($ = dollar) */
a[href*="github"]              { }   /* CONTIENE github    (* = asterisco) */
input[type="email"]            { }
img[alt=""]                    { }   /* imágenes sin alt */
[hidden]                       { }
```

```css
/* Resaltar enlaces externos */
a[href^="http"]:not([href*="mitienda.com"])::after {
  content: " ↗";
}
```

---

## 7.8 `:root` y variables (adelanto)

```css
:root {
  --color-primario: #e63946;
  --espacio: 24px;
}

:root, body { }    /* equivalente práctico a html */
```

`:root` apunta al `<html>`, que es el ancestro de todo → las variables declaradas
ahí se pueden usar en **cualquier parte** de la hoja de estilos.
→ Detalle completo en `09-variables-animaciones-y-keyframes.md`.

---

## 7.9 Ejemplo REAL desmontado: el menú lateral animado

**CSS** (`git/Pagina-Web/css/cuerpo.css`) — fragmento copiado:

```css
.menu-lateral {
  z-index: 20;
  position: fixed;
  top: 0;
  left: -250px;                 /* fuera de pantalla */
  width: 220px;
  height: 100%;
  background-color: #333;
  transition: left 0.3s;        /* transición de 0.3s */
}

.menu-lateral.abierto {
  left: 0;                      /* dentro de pantalla */
}

.menu-lateral > ul {
  list-style: none;
  margin: 0;
  padding: 20px;
}

.menu-lateral a {
  display: block;
  color: white;
  text-decoration: none;
  padding: 12px;
}

.menu-lateral a:hover {
  background-color: #555;
}

.menu-lateral li > ul {
  list-style: none;
  max-height: 0;
  overflow: hidden;
  opacity: 0;
  margin: 0;
  padding: 0 0 0 15px;
  transition: max-height 0.3s ease, opacity 0.3s ease, padding 0.3s ease;
}

.menu-lateral li:hover > ul {
  max-height: 200px;
  opacity: 1;
  padding: 8px 0 8px 15px;
}

.fondo-difuminado {
  position: fixed;
  inset: 0;
  z-index: 10;
  background: rgb(0 0 0 / 25%);
  backdrop-filter: blur(3px);
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease;
  pointer-events: none;
}

.menu-lateral.abierto ~ .fondo-difuminado {
  opacity: 1;
  visibility: visible;
  pointer-events: auto;
}
```

**HTML** que lo activa:

```html
<aside class="menu-lateral"> ... </aside>
<div class="fondo-difuminado"></div>
```

**JS** que añade/quita la clase (`git/Pagina-Web/js/programacion.js`):

```js
botonMenu.addEventListener("click", function () {
  menuLateral.classList.toggle("abierto");
});
```

### 🧠 Cómo funciona

```
ESTADO CERRADO                       ESTADO ABIERTO
.menu-lateral  → left: -250px        .menu-lateral.abierto  → left: 0
.fondo-difuminado → opacity 0        .abierto ~ .fondo → opacity 1
```

1. `.menu-lateral` lo mantiene **fuera** de la pantalla.
2. `.abierto` lo trae **dentro** (misma especificidad, pero va más abajo → gana).
3. `~` selecciona el `.fondo-difuminado` que está **después** en el HTML.
4. `transition: left 0.3s` hace que el movimiento sea **suave**.
5. JavaScript solo **alterna la clase** — la animación la hace 100% CSS.

---

## ✅ Resumen rápido

1. `:` = estado/posición (`:hover`, `:nth-child`, `:not()`), `::` = contenido inventado.
2. `:focus-visible` es mejor que `:focus` para estilos de foco (solo teclado).
3. `nth-child(2)`, `odd`, `even`, `3n+1` para posiciones y filas zebradas.
4. `::before` / `::after` **necesitan** `content`.
5. Combinadores: descendiente, `>`, `+`, `~`.
6. Atributos: `[href^=""]`, `[href$=""]`, `[href*=""]`.
7. El patrón `.clase + .clase-estado` (toggle) + JS `classList.toggle()` es el
   estándar para menús, acordeones y modales.

👉 **Siguiente:** `08-responsive-y-media-queries.md`
