# 10 — EJEMPLOS COMPLETOS DE CSS COPIADOS DE TUS PROYECTOS

> 📁 **Los archivos de los proyectos NO se han modificado.** Todo lo de aquí abajo son
> **copias** de sus estilos originales, comentadas para que estudies qué hace cada cosa.

---

## EJEMPLO 1 — Hoja de estilos completa de un proyecto pequeño

**Origen:** `proyecto/cuerpo.css` (20 líneas, copiado íntegro)

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

### 🧠 Qué hace

| Regla | Selector | Efecto |
|-------|----------|--------|
| `p{...}` | etiqueta | **Todos** los párrafos: rojo oscuro, 16px |
| `h1{...}` | etiqueta | Todos los `<h1>`: gris oscuro, 20px |
| `title{...}` | etiqueta | ⚠️ **No hace nada**: `<title>` está en el `<head>` y no se ve |
| `.texto{...}` | clase | Solo los elementos con `class="texto"`: negro, 12px |

### Su HTML (`proyecto/main.html`)

```html
<link rel="stylesheet" href="cuerpo.css">
...
<h1>Bienvenidos a mi pagina personal</h1>
<p>Hola, mi nombre es Hector Franco Magallo y esta es mi pagina personal.</p>
<div>   
    <p class="texto">En esta pagina podras encontrar informacion sobre mi, mis intereses y mis proyectos.</p>
    <a href="proyectos.html">Proyectos</a>
</div>
```

**Observa:**
- El `<p>` con `class="texto"` **recibe las dos reglas**: `p{}` (16px rojo) y `.texto{}` (12px negro).
- Como `.texto` tiene **mayor especificidad** (10) que `p` (1), gana → el párrafo queda **negro de 12px**.
- El enlace `<a>` **no tiene ninguna regla** → se ve azul y subrayado (estilo por defecto del navegador).

---

## EJEMPLO 2 — Reset + variables + layout (CV)

**Origen:** `web-cv-hector/web-cv-hector/css/styles.css` *(fragmentos clave)*

```css
/* ============ 1. TIPOGRAFÍA PROPIA ============ */
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

/* ============ 2. VARIABLES ============ */
:root {
  --font-main: "Inter", sans-serif;
  --color-text-heading: #111111;
  --color-text-body: #555555;
  --color-text-meta: #6d6d6d;
  --color-bg-page: #ffffff;
  --color-bg-tag: #f6f6f6;
  --color-border: #e5e5e5;
}

/* ============ 3. RESET ============ */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

img {
  display: block;
  max-width: 100%;
  height: auto;
}

body {
  font-family: var(--font-main);
  background-color: var(--color-bg-page);
}

/* ============ 4. LAYOUT (FLEX) ============ */
.container {
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 43rem;
  margin: 4rem auto;
  padding-inline: 1.5rem;
}

.profile {
  display: flex;
  flex-direction: row;
  align-items: center;
}

.profile__data {
  margin-left: 1rem;
}

.profile__data > * {
  margin-bottom: 0.2rem;
}

.profile__image {
  border-radius: 50%;
  object-fit: cover;
}

/* ============ 5. TIPOGRAFÍA DE LA PÁGINA ============ */
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
  font-weight: 400;
  font-size: 0.75rem;
  line-height: 1;
  color: var(--color-text-body);
  background: var(--color-bg-tag);
  border-radius: 12px;
  padding: 5px 10px 5px 10px;
}

.section {
  margin-top: 3rem;
}

.section__title {
  font-weight: 400;
  font-size: 0.875rem;
  line-height: 1.4;
  color: var(--color-text-heading);
}

.section__description {
  font-weight: 400;
  font-size: 0.875rem;
  line-height: 1.8;
  color: var(--color-text-body);
}

/* ============ 6. FILA: FECHA + DETALLE ============ */
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
  flex: 0 0 7rem;
}

.work-experience__details {
  flex: 1;
  min-width: 0;
}

.work-experience__title {
  font-weight: 400;
  font-size: 0.875rem;
  line-height: 1.8;
  color: var(--color-text-heading);
  overflow-wrap: break-word;
}

.work-experience__location {
  font-weight: 400;
  font-size: 0.875rem;
  line-height: 1.8;
  color: var(--color-text-body);
}

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

.card {
  display: flex;
  flex-direction: column;
  margin-top: 1.5rem;
  max-width: 152px;
  border: 1px solid var(--color-border);
  border-radius: 8px;
  overflow: hidden;
}

.card__image {
  width: 100%;
  height: 89px;
  object-fit: cover;
}

.card__content {
  padding: 0.75rem;
}

.card__title {
  font-weight: 400;
  font-size: 0.875rem;
  line-height: 1.8;
  color: var(--color-text-heading);
  overflow-wrap: break-word;
}

.card__meta {
  font-weight: 400;
  font-size: 0.875rem;
  line-height: 1.8;
  color: var(--color-text-body);
}

/* ============ 7. RESPONSIVE (AL FINAL) ============ */
@media (min-width: 768px) {
  .images-experience {
    width: 100%;
    flex-direction: row;
    align-items: center;
  }
}
```

### 🎓 Qué enseña este archivo (checklist)

- [x] **`@font-face`** para tipografía propia con pesos variables
- [x] **`:root` + variables** → colores centralizados
- [x] **Reset** con `*` y `box-sizing: border-box`
- [x] **Flexbox** en fila (`.profile`, `.work-experience`) y columna (`.container`)
- [x] **`flex: 0 0 7rem`** → columna fija de fechas
- [x] **`flex: 1` + `min-width: 0`** → contenido flexible sin desbordes
- [x] **`object-fit: cover`** → imágenes recortadas sin deformar
- [x] **`border-radius` + `overflow: hidden`** → tarjeta con esquinas limpias
- [x] **`@media` al final** del archivo
- [x] **BEM** en los selectores (`.work-experience__date`)

---

## EJEMPLO 3 — Página completa con menú lateral, grid y animaciones

**Origen:** `git/Pagina-Web/css/cuerpo.css` *(fragmentos seleccionados y ordenados por tema)*

### 3.1 Reset y base

```css
ul{
  color: white;
}

html,
body {
  min-height: 100%;
  margin: 0;
}

body {
min-height: 100vh;
  display: flex;
  flex-direction: column;
  font-family: "Poppins", sans-serif;
  color: var(--color-texto);
  background: #f4f4f4 url("../assets/images/Fondo.jpg") center / cover no-repeat fixed;
}

p {
  font-family: "Roboto", sans-serif;
  margin-left: 25px;
  margin-right: 25px;
  color: white;
}

.texto {
  color: #555;
  line-height: 1.6;
}

.centrado{
  text-align: center;
}

hr {
  width: 90%;
  margin: 20px auto;
  border: 0;
  border-top: 10px solid #000000;
}

.titulo{
  font-family: "Dancing Script", cursive;
  font-size: 100px;
  color: white;
}
```

> 📌 **Patrón "sticky footer":** `body` es `flex-direction: column` con
> `min-height: 100vh`, y `.pie-pagina` lleva `margin-top: auto` (ver 3.5).

### 3.2 Rejilla de proyectos (responsive sin media queries)

```css
.proyectos {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 24px;
  max-width: 1100px;
  margin: 40px auto;
  padding: 0 -25px;
  margin-top: -100px;
}

.tarjeta {
  overflow: hidden;
  background: white;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgb(0 0 0 / 12%);
}

.imagen-proyecto {
  height: 160px;
  background-size: cover;
  background-position: center;
}

.imagen-1 { background-image: url("../assets/images/Icono1.png"); }
.imagen-2 { background-image: url("../assets/images/comingsoon.jfif"); }
.imagen-3 { background-image: url("../assets/images/comingsoon.jfif"); }

.contenido-tarjeta { padding: 18px; }
.contenido-tarjeta h2 { margin: 0 0 10px; }
.contenido-tarjeta p { margin: 0 0 16px; color: #555; line-height: 1.5; }

.enlace-proyecto {
  display: inline-block;
  padding: 9px 14px;
  background: #e63946;
  color: white;
  text-decoration: none;
  border-radius: 6px;
}

.enlace-proyecto:hover { background: #c92f3b; }
```

### 3.3 Barra superior + botón hamburguesa

```css
.imagen-barra-superior {
 z-index: 10;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 15px;
  min-height: 64px;
  padding: 12px 40px;
  position: relative;
  background: url("../assets/images/Banners superior.jpg") center / cover no-repeat;
  box-shadow: 0 2px 6px rgb(0 0 0 / 90%);
}

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

.logo-inicio {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  display: block;
  line-height: 0;
  cursor: pointer;
}

.logo-inicio img {
  display: block;
  width: auto;
  height: 40px;
  max-width: 35vw;
  object-fit: contain;
}
```

### 3.4 Menú lateral con transiciones y desplegables

```css
/* Panel lateral: inicialmente fuera de la pantalla */
.menu-lateral {
  z-index: 20;
  position: fixed;
  top: 0;
  left: -250px;
  width: 220px;
  height: 100%;
  background-color: #333;
  transition: left 0.3s;
}

/* Clase que mostrará el menú */
.menu-lateral.abierto { left: 0; }

/* Quitar formato de lista */
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
.menu-lateral a:hover { background-color: #555; }

/* Submenús inicialmente cerrados */
.menu-lateral li > ul {
  list-style: none;
  max-height: 0;
  overflow: hidden;
  opacity: 0;
  margin: 0;
  padding: 0 0 0 15px;
  transition: max-height 0.3s ease, opacity 0.3s ease, padding 0.3s ease;
}

/* Submenú abierto al pasar el ratón */
.menu-lateral li:hover > ul {
  max-height: 200px;
  opacity: 1;
  padding: 8px 0 8px 15px;
}

/* Capa que cubre la página */
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

/* Se muestra cuando el menú está abierto */
.menu-lateral.abierto ~ .fondo-difuminado {
  opacity: 1;
  visibility: visible;
  pointer-events: auto;
}
```

### 3.5 Contenido y pie

```css
.contenido {
  max-width: 900px;
  margin: 40px auto 100px;
  padding: 30px;
  background-color: transparent;
  border-radius: 0px;
  box-sizing: border-box;
  box-shadow: none;
  position: relative;
  top: -100px;
}

.pie-pagina {
 margin-top: auto;
  padding: 18px 20px;
  background-color: rgba(20, 20, 20, 0.92);
  color: white;
  text-align: center;
}

.pie-pagina nav {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 12px 24px;
}

.pie-pagina a {
  margin: 0;
  color: #d9d9d9;
  text-decoration: none;
  font-size: 14px;
}

.pie-pagina a:hover {
  color: white;
  text-decoration: underline;
}

.pie-pagina p {
  margin: 14px 0 0;
  font-size: 13px;
  color: #aaa;
}
```

### 3.6 Animaciones de aparición (`reveal`) + accesibilidad

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

@media (prefers-reduced-motion: reduce) {
  .reveal {
    opacity: 1;
    filter: none;
    transform: none;
    transition: none;
  }
}
```

**Su HTML:**

```html
<h1 class="centrado reveal titulo">Bienvenidos a mi página personal</h1>
```

**Su JavaScript** (`git/Pagina-Web/js/programacion.js`):

```js
const observer = new IntersectionObserver(
  (entries, observer) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add("is-visible");
        observer.unobserve(entry.target);
      }
    });
  },
  { threshold: 0.2 }
);

document.querySelectorAll(".reveal").forEach((element) => {
  observer.observe(element);
});
```

> 📌 **Cómo funciona el trío completo:**
> 1. **CSS** deja `.reveal` invisible (`opacity: 0`, desenfocado, desplazado).
> 2. **JS** (`IntersectionObserver`) vigila cuándo el elemento **entra en pantalla**.
> 3. Al entrar, JS añade la clase **`is-visible`** → CSS lo anima a la vista.
> 4. `prefers-reduced-motion` lo desactiva para quien lo pida.

### 3.7 Contactos: tarjetas con animación en cascada

```css
.contactos-intro { text-align: center; }

.contactos-sociales {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 28px;
  margin: 36px auto;
}

.contacto-social {
  display: flex;
  width: 130px;
  min-height: 120px;
  flex-direction: column;
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
  animation: aparecer-rebotando 700ms both;
  transition: transform 180ms ease, box-shadow 180ms ease;
}

.contacto-social img {
  width: 42px;
  height: 42px;
  object-fit: contain;
}

.contacto-social:nth-child(2) { animation-delay: 140ms; }
.contacto-social:nth-child(3) { animation-delay: 280ms; }
.contacto-social:nth-child(4) { animation-delay: 420ms; }

.contacto-social:hover,
.contacto-social:focus-visible {
  transform: translateY(-7px) scale(1.06);
  box-shadow: 0 14px 26px rgb(0 0 0 / 40%);
}

.contacto-social:focus-visible {
  outline: 3px solid white;
  outline-offset: 4px;
}

@keyframes aparecer-rebotando {
  0%   { opacity: 0; transform: translateY(-70px) scale(0.8); }
  60%  { opacity: 1; transform: translateY(12px) scale(1.05); }
  80%  { transform: translateY(-6px) scale(0.98); }
  100% { opacity: 1; transform: translateY(0) scale(1); }
}

@media (prefers-reduced-motion: reduce) {
  .contacto-social {
    animation: none;
    transition: none;
  }
}
```

### 3.8 Certificados: grid + animación lateral

```css
.lista-certificados {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 280px), 1fr));
  gap: 24px;
  max-width: 900px;
  margin: 30px auto;
}

.tarjeta-certificado {
  padding: 24px;
  border: 1px solid rgb(255 255 255 / 25%);
  border-radius: 14px;
  background: rgb(20 20 20 / 75%);
  color: white;
  box-shadow: 0 6px 18px rgb(0 0 0 / 25%);
}

.tarjeta-certificado h2 {
  margin: 18px 0 12px;
  font-size: 21px;
}

.fecha-certificado { margin: 0; color: #d9d9d9; line-height: 1.5; }

.estado-certificado {
  display: inline-block;
  padding: 6px 10px;
  border-radius: 999px;
  background: #315f43;
  color: white;
  font-size: 13px;
  font-weight: 700;
}

.estado-en-curso { background: #315b78; }

.tarjeta-certificado { animation: deslizar-derecha 700ms ease-out both; }
.tarjeta-certificado:nth-child(2) { animation-delay: 180ms; }

@keyframes deslizar-derecha {
  from { opacity: 0; transform: translateX(-60px); }
  to   { opacity: 1; transform: translateX(0); }
}

@media (prefers-reduced-motion: reduce) {
  .tarjeta-certificado { animation: none; }
}
```

---

## EJEMPLO 4 — Reset + transiciones con "rebote"

**Origen:** `web-minimalista-hector/web-minimalista-hector/css/styles.css` *(fragmento)*

```css
/* ---------- Reset ---------- */
*,
*::before,
*::after {
  box-sizing: border-box;
}

body, h1, h2, p, ul, li, figure, a {
  margin: 0;
  padding: 0;
}

ul { list-style: none; }
a  { color: inherit; text-decoration: none; }
img { display: block; }

body {
  font-family: "Source Serif Pro", sans-serif;
  color: #000;
  background: #fff;
}

/* ---------- Enlace de salto (accesibilidad) ---------- */
.skip-link {
  position: absolute;
  top: -100px;
  left: 0;
  z-index: 10000;
  padding: 8px 12px;
  background: #fff;
  font-family: "Source Code Pro", sans-serif;
  font-size: 14px;
}

.skip-link:focus { top: 0; }

/* ---------- Navegación ---------- */
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

.brand__avatar {
  position: relative;
  display: block;
  width: 40.169px;
  height: 40px;
  border-radius: 50%;
  overflow: hidden;
  box-shadow: 0 2.667px 2.667px 0 rgba(0, 0, 0, 0.25);
}

.brand__photo {
  position: absolute;
  top: -56.81%;
  left: -99.7%;
  width: 259.92%;
  height: 320.36%;
  max-width: none;

  /* El giro sale del centro del círculo (el eje de la foto), no del centro de la imagen */
  transform-origin: 57.59% 33.34%;
  transform: rotate(0deg);

  /* Rebote cuando vuelve a su posición original */
  transition: transform 0.7s cubic-bezier(0.34, 1.9, 0.64, 1);
}

/* Al pasar el ratón la foto se tuerce 15 grados a la izquierda y rebota al llegar */
.brand__avatar:hover .brand__photo {
  transform: rotate(-15deg);
  transition: transform 0.5s cubic-bezier(0.34, 1.9, 0.64, 1);
}

.menu {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 30px;
  line-height: 1.03;
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

.menu__link--home    { color: #AB0782; }
.menu__link--about   { color: #7F750E; }
.menu__link--contact { color: #5F00AD; }
```

### 🎓 Detalles interesantes

- `top: -100px` + `:focus { top: 0 }` → el enlace de salto está fuera de pantalla y
  **solo aparece al recibir foco con Tab**.
- `border-radius: 50%` + `overflow: hidden` → convierte el contenedor en **círculo**.
- `transform-origin: 57.59% 33.34%` → la foto gira **alrededor de la cara**, no del rectángulo.
- `cubic-bezier(0.34, 1.9, 0.64, 1)` → el segundo valor `1.9 > 1` produce **rebote (overshoot)**.
- `.menu__link--home` → **modificador BEM** para poner color solo a ese enlace.

---

## 🎓 Ejercicio final: relaciona HTML con CSS

Toma `git/Pagina-Web/index.html` y `git/Pagina-Web/css/cuerpo.css` y responde:

1. ¿Qué selector estiliza `.boton-hamburguesa`? ¿Es de etiqueta, clase o id?
2. ¿Por qué `.menu-lateral.abierto` gana sobre `.menu-lateral`?
3. ¿Qué propiedad hace que el menú se deslice? ¿Y que los submenús se desplieguen?
4. ¿Qué usan `.proyectos` y `.lista-certificados` para ser responsive **sin `@media`**?
5. ¿Qué clase añade JavaScript a los elementos `.reveal` y qué hace el CSS con ella?
6. Busca un ejemplo de `!important`... ¿hay alguno? ¿Por qué crees que no hace falta?

---

## 🧪 Laboratorio: reproducir un componente

Copia este HTML en un archivo `prueba.html` y pégale este CSS en un `<style>`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Laboratorio</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    .rejilla {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(min(100%, 260px), 1fr));
      gap: 24px;
      max-width: 1100px;
      margin: 40px auto;
      padding: 0 20px;
    }

    .tarjeta {
      overflow: hidden;
      background: #fff;
      border-radius: 10px;
      box-shadow: 0 4px 12px rgb(0 0 0 / 12%);
      transition: transform 180ms ease, box-shadow 180ms ease;
      animation: aparecer 700ms both;
    }

    .tarjeta:nth-child(2) { animation-delay: 140ms; }
    .tarjeta:nth-child(3) { animation-delay: 280ms; }

    .tarjeta:hover {
      transform: translateY(-7px) scale(1.03);
      box-shadow: 0 14px 26px rgb(0 0 0 / 25%);
    }

    .tarjeta__cuerpo { padding: 18px; }
    .tarjeta__titulo { margin-bottom: 8px; }
    .tarjeta__texto  { color: #555; line-height: 1.6; }

    @keyframes aparecer {
      from { opacity: 0; transform: translateY(30px); }
      to   { opacity: 1; transform: translateY(0); }
    }

    @media (prefers-reduced-motion: reduce) {
      .tarjeta { animation: none; transition: none; }
    }
  </style>
</head>
<body>
  <section class="rejilla">
    <article class="tarjeta"><div class="tarjeta__cuerpo">
      <h2 class="tarjeta__titulo">Tarjeta 1</h2>
      <p class="tarjeta__texto">Redimensiona la ventana para ver cómo cambian las columnas.</p>
    </div></article>
    <article class="tarjeta"><div class="tarjeta__cuerpo">
      <h2 class="tarjeta__titulo">Tarjeta 2</h2>
      <p class="tarjeta__texto">Pasa el ratón por encima para ver la transición.</p>
    </div></article>
    <article class="tarjeta"><div class="tarjeta__cuerpo">
      <h2 class="tarjeta__titulo">Tarjeta 3</h2>
      <p class="tarjeta__texto">Abre DevTools y prueba a cambiar las variables.</p>
    </div></article>
  </section>
</body>
</html>
```

👉 **Vuelve al índice:** `00-INDICE.md`
