# 05 — FLEXBOX

**Flexbox** = **Flex**ible **Box** Layout. Sirve para maquetar **en una dimensión**
(fila **o** columna), repartiendo el espacio entre elementos.

---

## 5.1 Los dos roles

```css
.contenedor {
  display: flex;        /* ← EL PADRE es el flex container */
}

.contenedor > * {
  /* ← Los HIJOS son los flex items */
}
```

> ⚠️ **Solo los hijos directos** se vuelven flex items. Los nietos no.

---

## 5.2 Propiedades del CONTENEDOR

```css
.contenedor {
  display: flex;

  /* EJE PRINCIPAL (main axis) */
  flex-direction: row;          /* row (por defecto) | row-reverse | column | column-reverse */
  justify-content: center;      /* reparto a lo LARGO del eje principal */

  /* EJE CRUZADO (cross axis) */
  align-items: center;          /* colocación en el eje CRUZADO */
  align-content: center;        /* reparto de las líneas (solo si hay varias) */

  /* LÍNEAS */
  flex-wrap: wrap;              /* permitir que los hijos pasen a otra línea */
  gap: 12px 24px;               /* hueco entre hijos (fila y columna) */

  /* Eje cruzado con varias líneas */
  align-content: space-between;
}
```

### `justify-content` — reparto en el eje PRINCIPAL

```
flex-start    |□□□             ← pegados a la izquierda (o arriba en column)
center        |    □□□         ← centrados
flex-end      |        □□□     ← a la derecha
space-between |□□□       □□    ← uno a cada extremo, hueco en medio
space-around  |  □□  □□  □□    ← medio hueco en los extremos
space-evenly  |   □□  □□  □□   ← huecos IGUALES
```

### `align-items` — colocación en el eje CRUZADO

```
flex-start   → arriba
center       → centro vertical
flex-end     → abajo
stretch      → estira a toda la altura (por defecto)
baseline     → alineado por la línea base del texto
```

> 📌 **Truco para centrar perfectamente:**
> ```css
> .caja {
>   display: flex;
>   justify-content: center;   /* horizontal */
>   align-items: center;       /* vertical */
> }
> ```
> Centra en **ambos ejes**. Es la forma más sencilla de centrar algo en CSS.

---

## 5.3 Propiedades de los HIJOS (flex items)

```css
.hijo {
  flex-grow: 1;        /* cuánto del SOBRANTE se lleva este (proporcional) */
  flex-shrink: 1;      /* cuánto puede encogerse si no cabe */
  flex-basis: 200px;   /* tamaño inicial antes de repartir */
  flex: 1;             /* shorthand de los tres anteriores (1 1 0%) */

  order: 2;            /* cambia el ORDEN de visualización (no el del HTML) */
  align-self: flex-end;/* alinea SOLO a este hijo en el eje cruzado */
}
```

### `flex: 1` — el atajo que más se usa

```css
.flex: 1;      /* = flex: 1 1 0% → todos a partes iguales */
```

| Valor | Significado |
|-------|-------------|
| `flex: 0` | No crece (tamaño natural) |
| `flex: 1` | Crece por igual |
| `flex: 2` | Crece el doble que un `flex: 1` |
| `flex: auto` | `1 1 auto` → parte del tamaño natural del contenido |

---

## 5.4 Ejemplos REALES copiados de tus proyectos

### 1) Fila: perfil con foto + datos

**HTML** (`web-cv-hector/web-cv-hector/index.html`):

```html
<div class="profile">
    <img class="profile__image" alt="Sara Lawrence" src="./assets/images/image-profile.png" width="92" height="93">
    <div class="profile__data">
        <h1 class="profile__name">Sara Lawrence</h1>
        <p class="profile__subtitle">Design in San Francisco</p>
        <a class="profile__website" href="https://website.com" target="_blank" rel="noopener noreferrer">website.com</a>
    </div>
</div>
```

**CSS** (`web-cv-hector/web-cv-hector/css/styles.css`):

```css
.profile {
  display: flex;
  flex-direction: row;
  align-items: center;      /* foto y texto alineados por el CENTRO vertical */
}

.profile__data {
  margin-left: 1rem;
}
```

> 📌 **Antes de Flexbox** esto se hacía con `float` o con `display: inline-block`.
> Ahora son 3 líneas y queda perfecto.

---

### 2) Columna: página que empuja el footer abajo del todo

**CSS** (`git/Pagina-Web/css/cuerpo.css`):

```css
body {
  min-height: 100vh;
  display: flex;
  flex-direction: column;     /* columnas: header arriba, footer abajo */
  font-family: "Poppins", sans-serif;
  color: var(--color-texto);
  background: #f4f4f4 url("../assets/images/Fondo.jpg") center / cover no-repeat fixed;
}

.pie-pagina {
  margin-top: auto;           /* ← CLAVE: empuja el footer al FONDO */
  padding: 18px 20px;
  background-color: rgba(20, 20, 20, 0.92);
  color: white;
  text-align: center;
}
```

> 📌 **Patrón "sticky footer":** `<body>` como `flex-direction: column` con
> `min-height: 100vh`, y en el `footer` un `margin-top: auto`.
> Así, aunque el contenido sea corto, **el pie queda pegado al fondo de la ventana**.

---

### 3) Fila con envoltura: redes sociales centradas

**CSS** (`git/Pagina-Web/css/cuerpo.css`):

```css
.contactos-sociales {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;         /* si no caben, pasan a la siguiente línea */
  gap: 28px;
  margin: 36px auto;
}

.contacto-social {
  display: flex;
  width: 130px;
  min-height: 120px;
  flex-direction: column;   /* dentro de CADA tarjeta: icono encima, texto debajo */
  align-items: center;
  justify-content: center;
  gap: 12px;
  border: 1px solid rgb(255 255 255 / 30%);
  border-radius: 18px;
  background: rgb(20 20 20 / 75%);
  color: white;
  font-size: 17px;
  font-weight: 600;
  text-decoration: none;
  box-shadow: 0 8px 20px rgb(0 0 0 / 25%);
  transition: transform 180ms ease, box-shadow 180ms ease;
}
```

> 📌 **Flex anidado:** `.contactos-sociales` es flex **fila** (coloca las tarjetas),
> y cada `.contacto-social` es flex **columna** (coloca icono + texto).
> `gap: 12px` dentro y `gap: 28px` fuera.

---

### 4) Fila: barra superior con botón a la izquierda y logo al centro

**HTML** (`git/Pagina-Web/index.html`):

```html
<nav class="imagen-barra-superior">
  <button class="boton-hamburguesa" type="button" aria-label="Abrir menú">☰</button>
  <a class="logo-inicio" href="index.html" aria-label="Ir a la página principal">
    <img src="assets/images/imagen-para-enlace.png" alt="Logo de la página">
  </a>
</nav>
```

**CSS** (`git/Pagina-Web/css/cuerpo.css`):

```css
.imagen-barra-superior {
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: space-between;   /* botón a la izquierda, hueco, logo */
  gap: 15px;
  min-height: 64px;
  padding: 12px 40px;
  position: relative;
  background: url("../assets/images/Banners superior.jpg") center / cover no-repeat;
  box-shadow: 0 2px 6px rgb(0 0 0 / 90%);
}
```

Y el logo se centra **absolutamente** encima (para que quede en el medio exacto,
no solo en el hueco):

```css
.logo-inicio {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  display: block;
  line-height: 0;
  cursor: pointer;
}
```

---

### 5) Columna: pie con enlaces centrados y que envuelven

**HTML** (`git/Pagina-Web/index.html`):

```html
<footer class="pie-pagina">
  <nav aria-label="Enlaces legales">
    <a href="#">Aviso legal</a>
    <a href="#">Política de privacidad</a>
    <a href="#">Política de cookies</a>
  </nav>
  <p>© 2026 Hector Franco Magallo</p>
</footer>
```

**CSS** (`git/Pagina-Web/css/cuerpo.css`):

```css
.pie-pagina nav {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 12px 24px;        /* 12px entre filas, 24px entre columnas */
}

.pie-pagina a {
  margin: 0;
  color: #d9d9d9;
  text-decoration: none;
  font-size: 14px;
}
```

---

### 6) Filas apiladas: experiencia laboral (fecha + detalle)

**HTML** (`web-cv-hector/index.html`):

```html
<article class="work-experience">
    <div class="work-experience__date"><time datetime="2017">2017</time> — Now</div>
    <div class="work-experience__details">
        <h3 class="work-experience__title">Senior Designer at Magic Design Co...</h3>
        <p class="work-experience__location">San Francisco, CA</p>
    </div>
</article>
```

**CSS** (`web-cv-hector/css/styles.css`):

```css
.work-experience {
  display: flex;
  flex-direction: row;
  margin-top: 1rem;
}

.work-experience__date {
  font-weight: 400;
  font-size: 0.875rem;
  line-height: 1.8;
  color: var(--color-text-meta);
  flex: 0 0 7rem;        /* NO crece, NO encoge, base fija de 7rem → columna de fechas */
}

.work-experience__details {
  flex: 1;               /* TODO el resto del ancho para los detalles */
  min-width: 0;          /* evita que el contenido desborde (bug clásico de flex) */
}
```

> 📌 **Patrón muy útil:** `flex: 0 0 7rem` crea una **columna fija** de 7rem para las
> fechas, y `flex: 1` hace que el segundo bloque ocupe todo lo demás. Cualquier
> texto largo no rompe la maquetación gracias a `min-width: 0`.

---

### 7) El mismo bloque, cambiado a columna en pantallas grandes

**CSS** (`web-cv-hector/css/styles.css`):

```css
.images-experience {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin-top: 1.5rem;
}

.images-experience img {
  border: 1px solid var(--color-border);
  border-radius: 8px;
  flex: 0 0 150px;
  width: 150px;
  height: 84px;
  object-fit: cover;
  max-width: 100%;
}

@media (min-width: 768px) {
  .images-experience {
    width: 100%;
    flex-direction: row;        /* en pantallas medianas: en FILA */
    align-items: center;
  }
}
```

> 📌 **Cambia de `column` a `row` según el ancho de pantalla.** Ese es el corazón
> del diseño responsive con Flexbox.

---

### 8) Contenedor de página en columna

**CSS** (`web-cv-hector/css/styles.css`):

```css
.container {
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 43rem;
  margin: 4rem auto;
  padding-inline: 1.5rem;
}
```

Y el mismo patrón en `web-minimalista-hector/.../css/styles.css`:

```css
.page {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  padding: 0 20px;
  background: #fff;
  overflow: hidden;
}

.nav {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 20px 20px 60px;
  border-bottom: 0.5px solid #4F4F4F;
}

.brand {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  gap: 20px;
}

.menu {
  display: flex;
  flex-direction: column;
  align-items: flex-end;      /* menú pegado a la derecha */
  gap: 30px;
  line-height: 1.03;
}
```

---

## 5.5 Ejemplo resumido: las 3 recetas más útiles

```css
/* 1) CENTRAR algo perfectamente */
.centro {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

/* 2) Fila que envuelve (galería / chips / botones) */
.fila {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

/* 3) Footer pegado abajo */
body {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}
footer { margin-top: auto; }
```

---

## 5.6 Errores comunes con Flexbox

| Error | Síntoma | Solución |
|-------|---------|----------|
| Esperar que `width` funcione en flex items | No respeta el ancho | Usa `flex-basis` o `flex: 0 0 200px` |
| Olvidar `flex-wrap: wrap` | Los hijos se encogen en vez de pasar de línea | `flex-wrap: wrap` |
| Contenido que desborda | Texto largo rompe la caja | `min-width: 0` en el hijo |
| Centrar verticalmente sin flex | Complicado | Pasa el padre a `display: flex` |
| Esperar que los `gap` afecten a `<p>` consecutivos | No pasa nada | `gap` solo afecta a **hijos flex/grid** |

---

## ✅ Resumen rápido

1. `display: flex` en el **padre**; solo los **hijos directos** son flex items.
2. `flex-direction` define el eje principal (`row` o `column`).
3. `justify-content` reparte en el **eje principal**; `align-items` coloca en el **cruzado**.
4. `gap` es la separación entre hijos (mucho mejor que márgenes).
5. `flex: 1` = crece; `flex: 0 0 7rem` = tamaño fijo.
6. `flex-wrap: wrap` permite salto de línea.
7. Para centrar en los dos ejes: `justify-content: center` + `align-items: center`.

👉 **Siguiente:** `06-grid.md`
