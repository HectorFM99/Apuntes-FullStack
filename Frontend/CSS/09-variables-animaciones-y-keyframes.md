# 09 — VARIABLES, TRANSICIONES, TRANSFORMACIONES Y ANIMACIONES

---

## 9.1 Variables CSS (Custom Properties)

### Declarar y usar

```css
:root {
  --color-primario: #e63946;
  --color-texto: #333;
  --espacio: 24px;
  --radio: 10px;
  --sombra: 0 4px 12px rgb(0 0 0 / 12%);
}

.titulo {
  color: var(--color-primario);
  margin-bottom: var(--espacio);
}

.tarjeta {
  border-radius: var(--radio);
  box-shadow: var(--sombra);
}
```

| Concepto | Cómo se escribe |
|----------|-----------------|
| Declarar | `--nombre: valor;` |
| Usar | `var(--nombre)` |
| Valor por defecto | `var(--nombre, 16px)` → si no existe, usa 16px |
| Dónde se declara | Normalmente en `:root` (el `<html>`) para alcance global |

> ⚠️ **Se escriben con `--` y se usan con `var()`.** Son **case-sensitive**:
> `--colorText` ≠ `--colortext`.

### Ventajas

1. **Un solo sitio** para cambiar un color → cambiarlo en 1000 sitios = 1 línea.
2. **Temas** (oscuro/claro) cambiando las variables en un `@media`.
3. **Coherencia** en todo el proyecto.
4. Se pueden **redefinir** dentro de un selector (más abajo en cascada).

---

### Ejemplo REAL y completo: `web-cv-hector/web-cv-hector/css/styles.css`

```css
@font-face {
  font-family: 'Inter';
  src: url('../assets/fonts/Inter/static/Inter_18pt-Regular.ttf') format('truetype');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

:root {
  --font-main: "Inter", sans-serif;

  /* Texto */
  --color-text-heading: #111111;   /* nombre, títulos de sección y de puesto */
  --color-text-body: #555555;      /* párrafos, subtítulo, ubicación */
  --color-text-meta: #6d6d6d;      /* fechas: información secundaria */

  /* Superficies */
  --color-bg-page: #ffffff;        /* fondo de la página */
  --color-bg-tag: #f6f6f6;         /* fondo de la píldora "website.com" */

  /* Bordes */
  --color-border: #e5e5e5;         /* marco de las imágenes */
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: var(--font-main);
  background-color: var(--color-bg-page);
}

.profile__name {
  font-weight: 400;
  font-size: 1.25rem;
  line-height: 1.3;
  color: var(--color-text-heading);
}

.profile__subtitle {
  font-weight: 400;
  font-size: 0.875rem;
  line-height: 1.8;
  color: var(--color-text-body);
}

.profile__website {
  font-size: 0.75rem;
  color: var(--color-text-body);
  background: var(--color-bg-tag);
  border-radius: 12px;
  padding: 5px 10px;
}

.work-experience__date {
  color: var(--color-text-meta);
  flex: 0 0 7rem;
}

.work-experience__location {
  color: var(--color-text-body);
}

.images-experience img {
  border: 1px solid var(--color-border);
  border-radius: 8px;
}

.card {
  border: 1px solid var(--color-border);
  border-radius: 8px;
  overflow: hidden;
}
```

> 📌 **Qué consigue:** 6 colores + 1 fuente definidos **una sola vez**.
> Si mañana el gris de las fechas cambia a `#7a7a7a`, se cambia **una línea**
> y se actualiza en toda la web. Además, los comentarios (`/* fechas... */`)
> documentan para **qué** sirve cada color.

**Ejemplo del efecto "cambio global":**

```css
/* Cambiar TODO el tema de un plumazo */
:root {
  --color-bg-page: #ffffff;
  --color-text-heading: #111111;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg-page: #111111;   /* SOLO cambian las variables */
    --color-text-heading: #f5f5f5;
  }
}
/* El resto del CSS NO se toca */
```

---

## 9.2 `transition` — transiciones suaves

```css
.elemento {
  transition: propiedad duración timing retraso;
}

/* Ejemplos */
transition: all 0.3s ease;
transition: background-color 0.2s ease-in-out;
transition: transform 180ms ease, box-shadow 180ms ease;
transition: left 0.3s;
```

| Valor | Opciones |
|-------|----------|
| **Propiedad** | `all`, o una concreta (`background`, `transform`, `opacity`...) |
| **Duración** | `0.3s`, `180ms`, `1800ms` |
| **Timing** (aceleración) | `ease` (por defecto) · `linear` · `ease-in` · `ease-out` · `ease-in-out` · `cubic-bezier(a,b,c,d)` |
| **Retraso** | `0.5s` (cuánto espera antes de empezar) |

```css
/* shorthand completo */
transition: background-color .3s ease-in-out .1s;
/*           propiedad     dur  curva        retraso */
```

> ⚠️ **Solo funciona en propiedades que se PUEDEN animar.** No transiciona
> `display: none → block` (eso salta de golpe). Sí funcionan `opacity`,
> `transform`, `color`, `background`, `width`, `margin`...

### Ejemplos REALES copiados

**`git/Pagina-Web/css/cuerpo.css` — el menú lateral:**

```css
.menu-lateral {
  position: fixed;
  top: 0;
  left: -250px;
  width: 220px;
  height: 100%;
  background-color: #333;
  transition: left 0.3s;     /* la propiedad "left" se anima 0.3s */
}
```

**`git/Pagina-Web/css/cuerpo.css` — submenú desplegable:**

```css
.menu-lateral li > ul {
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
```

> 📌 Tres propiedades se animan **a la vez**: la altura, la transparencia y el
> relleno → el submenú "se despliega" suavemente.

**`git/Pagina-Web/css/cuerpo.css` — botones con elevación:**

```css
.contacto-social {
  ...
  transition: transform 180ms ease, box-shadow 180ms ease;
}

.contacto-social:hover,
.contacto-social:focus-visible {
  transform: translateY(-7px) scale(1.06);
  box-shadow: 0 14px 26px rgb(0 0 0 / 40%);
}
```

**`web-minimalista-hector/.../css/styles.css` — rebote con `cubic-bezier`:**

```css
.brand__photo {
  transform-origin: 57.59% 33.34%;
  transform: rotate(0deg);
  transition: transform 0.7s cubic-bezier(0.34, 1.9, 0.64, 1);
  /*                          ▲ ese 1.9 > 1 crea "rebote" (overshoot) */
}

.brand__avatar:hover .brand__photo {
  transform: rotate(-15deg);
  transition: transform 0.5s cubic-bezier(0.34, 1.9, 0.64, 1);
}
```

**`git/Pagina-Web/css/cuerpo.css` — fondo difuminado:**

```css
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
```

---

## 9.3 `transform` — mover, girar, escalar (sin afectar al layout)

```css
.elemento {
  transform: translate(X, Y);        /* mover */
  transform: translateX(-50%);       /* centrado clásico */
  transform: translateY(-7px);       /* "subir" al hacer hover */
  transform: rotate(45deg);          /* girar */
  transform: scale(1.2);             /* agrandar 20% */
  transform: scale(0.96);            /* encoger */
  transform: skew(10deg);            /* inclinar */
  transform: rotate(-15deg) scale(1.06);   /* combinado */
}
```

**Ventaja clave:** transform **no altera el flujo** del documento. Si escalas un
botón al 120%, **no mueve** a los botones de al lado (a diferencia de `width`).

### Ejemplos REALES

`git/Pagina-Web/css/cuerpo.css` — centrado absoluto con `translate`:

```css
.logo-inicio {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);   /* compensa SU propio tamaño → centro exacto */
  display: block;
  line-height: 0;
  cursor: pointer;
}
```

`git/Pagina-Web/css/cuerpo.css` — hover de tarjetas:

```css
.contacto-social:hover,
.contacto-social:focus-visible {
  transform: translateY(-7px) scale(1.06);
}
```

`git/Pagina-Web/css/cuerpo.css` — elemento que "aparece" desde fuera:

```css
.reveal {
  opacity: 0;
  filter: blur(12px);
  transform: translateX(-28px);
  transition:
    opacity 1800ms ease,
    filter 1800ms ease,
    transform 1800ms ease;
}

.reveal.is-visible {
  opacity: 1;
  filter: blur(0);
  transform: translateX(0);
}
```

> 📌 **`transform-origin`** define desde dónde gira/escala (por defecto, el centro).
> En `web-minimalista-hector` se usa `transform-origin: 57.59% 33.34%` para que la
> foto gire **alrededor del eje de la cara**, no del centro del rectángulo.

---

## 9.4 `@keyframes` + `animation` — animaciones complejas

```css
@keyframes nombre {
  0%   { opacity: 0; transform: translateY(-70px); }
  60%  { opacity: 1; transform: translateY(12px); }
  100% { opacity: 1; transform: translateY(0); }
}

.elemento {
  animation: nombre duración timing retraso iteraciones dirección relleno;
}
```

| Fragmento | Opciones |
|-----------|----------|
| **Duración** | `700ms`, `2s` |
| **Timing** | `ease`, `linear`, `ease-out`, `cubic-bezier(...)` |
| **Retraso** | `0.2s` |
| **Iteraciones** | `3`, `infinite` |
| **Dirección** | `normal`, `reverse`, `alternate` (ida y vuelta) |
| **Relleno** | `forwards` (se queda en el último estado), `both` (aplica desde el inicio) |

### Ejemplo REAL: animación de aparición con rebote

**Archivo:** `git/Pagina-Web/css/cuerpo.css` *(copiado tal cual)*

```css
.contacto-social {
  ...
  animation: aparecer-rebotando 700ms both;
  transition: transform 180ms ease, box-shadow 180ms ease;
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

/* Animación de aparición con rebote */
@keyframes aparecer-rebotando {
  0% {
    opacity: 0;
    transform: translateY(-70px) scale(0.8);
  }
  60% {
    opacity: 1;
    transform: translateY(12px) scale(1.05);
  }
  80% {
    transform: translateY(-6px) scale(0.98);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}
```

> 📌 **Desglose:** los iconos **caen desde arriba** (`translateY(-70px)`), llegan un
> poco por debajo de su sitio (`12px`), **rebotan** hacia arriba (`-6px`) y se quedan
> en su sitio. `both` hace que la animación mantenga el estado final y aplique el
> estado inicial **antes** de empezar (nunca se ve el "salto" del principio).
> Con `nth-child` se retrasa cada icono para que entren **en cascada**.

### Ejemplo REAL: deslizar desde la izquierda

```css
.tarjeta-certificado {
  animation: deslizar-derecha 700ms ease-out both;
}

.tarjeta-certificado:nth-child(2) {
  animation-delay: 180ms;
}

@keyframes deslizar-derecha {
  from {
    opacity: 0;
    transform: translateX(-60px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}
```

> 📌 **`from` = `0%` y `to` = `100%`.** Es la forma corta cuando solo hay 2 pasos.

### Otras animaciones útiles

```css
/* Girar continuamente */
@keyframes girar { to { transform: rotate(360deg); } }
.cargando { animation: girar 1s linear infinite; }

/* Latido */
@keyframes latir {
  0%, 100% { transform: scale(1); }
  50%      { transform: scale(1.15); }
}
.corazon { animation: latir 1.2s ease-in-out infinite; }

/* Parpadeo */
@keyframes parpadear { 50% { opacity: 0; } }
.cursor { animation: parpadear 1s step-end infinite; }
```

---

## 9.5 `prefers-reduced-motion` — accesibilidad

**Ejemplo REAL copiado de `git/Pagina-Web/css/cuerpo.css`:**

```css
@media (prefers-reduced-motion: reduce) {
  .reveal {
    opacity: 1;
    filter: none;
    transform: none;
    transition: none;
  }
}
```

Y otra vez en el mismo archivo:

```css
@media (prefers-reduced-motion: reduce) {
  .contacto-social {
    animation: none;
    transition: none;
  }
}
```

Y en las tarjetas de certificados:

```css
@media (prefers-reduced-motion: reduce) {
  .tarjeta-certificado {
    animation: none;
  }
}
```

> 📌 **Muy importante:** hay personas con **mareo, vértigo o problemas vestibulares**
> a las que las animaciones les causan malestar. Si el sistema operativo marca
> "reducir movimiento", estos `@media` **desactivan** las animaciones y muestran
> el contenido directamente en su sitio final.
> Es un criterio de **accesibilidad (WCAG 2.3.3)** y aparece en Lighthouse.

---

## 9.6 Otras propiedades visuales útiles

```css
/* Sombra */
box-shadow: 0 4px 12px rgb(0 0 0 / 12%);
/*          X  Y   blur  color / opacidad */

/* Fondo difuminado (detrás de un elemento) */
backdrop-filter: blur(3px);

/* Opacidad */
opacity: 0.5;

/* Filtros */
filter: blur(12px) grayscale(100%) brightness(1.2);

/* Capas */
z-index: 20;          /* mayor = encima */

/* Listón puntero */
cursor: pointer;

/* Recorte con esquinas */
overflow: hidden;
border-radius: 50%;   /* círculo */
```

### Ejemplo REAL de `z-index` + `backdrop-filter` — `git/Pagina-Web/css/cuerpo.css`

```css
.imagen-barra-superior {
  z-index: 10;
  ...
}

.menu-lateral {
  z-index: 20;      /* el menú va ENCIMA de la barra */
  position: fixed;
  ...
}

.fondo-difuminado {
  z-index: 10;
  position: fixed;
  inset: 0;
  background: rgb(0 0 0 / 25%);
  backdrop-filter: blur(3px);
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease;
  pointer-events: none;
}
```

**Jerarquía de capas del proyecto:**

```
z-index: 20   menú lateral        (lo más alto)
z-index: 10   barra superior
z-index: 10   fondo difuminado
z-index: auto contenido normal    (lo más bajo)
```

> 📌 `z-index` **solo funciona con `position` distinta de `static`**
> (`relative`, `absolute`, `fixed`, `sticky`).

---

## 9.7 Rendimiento: qué animar

| ✅ **Barato** (GPU) | ❌ **Carpeta** (fuerza reflow) |
|---------------------|-------------------------------|
| `transform` | `width`, `height` |
| `opacity` | `top`, `left`, `margin` |
| `filter` (con cuidado) | `padding` |
| | `box-sizing` |

> 📌 Regla práctica: **para mover cosas, usa `transform: translate()` en vez de
> `left/top`.** Para desvanecer, `opacity`, no `display: none`.

---

## ✅ Resumen rápido

1. Variables: `--nombre: valor` en `:root`, se usan con `var(--nombre)`.
2. `transition: prop duración curva retraso` → anima cambios de estado (hover, clase).
3. `transform` mueve/gira/escala **sin afectar al layout**.
4. `@keyframes` define los pasos; `animation: nombre duración ... forwards/both`.
5. `animation-delay` + `nth-child` = efecto **cascada** sin JS.
6. `transform: translate(-50%, -50%)` sobre `left: 50%; top: 50%` = **centrado perfecto**.
7. **Siempre** añade `@media (prefers-reduced-motion: reduce)`.
8. Anima `transform` y `opacity` (van por GPU); evita animar `width`/`left`.

👉 **Siguiente:** `10-ejemplos-completos-de-proyectos.md`
