# 04 — ESTRUCTURA SEMÁNTICA Y LAYOUT

---

## 4.1 ¿Qué es "HTML semántico"?

Significa usar **la etiqueta que describe lo que hay dentro**, en vez de meter todo
en `<div>`.

```html
<!-- ❌ Sin semántica: el navegador no sabe qué es nada de esto -->
<div class="header">...</div>
<div class="main">...</div>
<div class="footer">...</div>

<!-- ✅ Con semántica: el nombre de la etiqueta ya lo dice -->
<header>...</header>
<main>...</main>
<footer>...</footer>
```

**Ventajas:**
1. **SEO** — Google entiende mejor la estructura y posiciona mejor.
2. **Accesibilidad** — Los lectores de pantalla permiten saltar directamente a `<main>` o `<nav>`.
3. **Legibilidad** — Cualquiera entiende el código de un vistazo.
4. **Mantenimiento** — Menos clases decorativas en CSS.

---

## 4.2 El esqueleto semántico de una página

```
┌──────────────────────────────────────────┐
│ <header>   cabecera: logo, menú, título  │
├──────────────────────────────────────────┤
│ <nav>      navegación / enlaces          │
├──────────────────────────────────────────┤
│ <main>     CONTENIDO PRINCIPAL (único)   │
│   ├─ <section>  bloque temático          │
│   │    └─ <article>  contenido repetible │
│   └─ <aside>    contenido lateral        │
├──────────────────────────────────────────┤
│ <footer>   pie: copyright, enlaces leg.  │
└──────────────────────────────────────────┘
```

| Etiqueta | Significado | Regla |
|----------|-------------|-------|
| `<header>` | Cabecera de la página o de una sección | Puede haber varias (una por sección) |
| `<nav>` | Bloque de navegación | Puede haber varias |
| `<main>` | **El contenido principal** de la página | **Solo uno** por página |
| `<section>` | Agrupación de contenido **temática** | Normalmente lleva un `<h2>` dentro |
| `<article>` | Contenido **independiente y repetible** (noticia, tarjeta, comentario) | Se entiende "solo" |
| `<aside>` | Contenido **complementario / lateral** (barra, relacionados) | |
| `<footer>` | Pie de la página o de una sección | Puede haber varias |
| `<div>` | Contenedor **genérico sin significado** | Solo cuando necesitas una caja para CSS |
| `<span>` | Contenedor **en línea sin significado** | Solo para envolver trozos de texto |

---

## 4.3 Ejemplo real completo copiado de un proyecto

**Archivo:** `web-cv-hector/web-cv-hector/index.html` *(fragmento copiado tal cual)*

```html
<body>
    <div class="container">
        <header>
            <div class="profile">
                <img class="profile__image" alt="Sara Lawrence" src="./assets/images/image-profile.png" width="92" height="93">
                <div class="profile__data">
                    <h1 class="profile__name">Sara Lawrence</h1>
                    <p class="profile__subtitle">Design in San Francisco</p>
                    <a class="profile__website" href="https://website.com" target="_blank" rel="noopener noreferrer">website.com</a>
                </div>
            </div>
        </header>
        <main>
            <section class="section">
                <h2 class="section__title">About</h2>
                <p class="section__description">
                    I’m a passionate UX designer striving to create intuitive and engaging
                    experiences. I’m a big believer that things can always be simpler than
                    we think.
                </p>
            </section>
            <section class="section">
                <h2 class="section__title">Work Experience</h2>
                <article class="work-experience">
                    <div class="work-experience__date"><time datetime="2017">2017</time> — Now</div>
                    <div class="work-experience__details">
                        <h3 class="work-experience__title">Senior Designer at Magic Design Co<span aria-hidden="true">↗</span></h3>
                        <p class="work-experience__location">San Francisco, CA</p>
                    </div>
                </article>
            </section>
        </main>
        <footer>
            <section class="section">
                <h2 class="section__title">Contact</h2>
                ...
            </section>
        </footer>
    </div>
</body>
```

### 🧠 Análisis de por qué está bien estructurado

```
div.container          → caja que centra todo (es de CSS, no de contenido: div correcto)
  header               → cabecera con foto y nombre  ✅ semántica
    h1                  → el ÚNICO h1 de la página: "Sara Lawrence"
  main                 → contenido principal  ✅ (solo uno)
    section.section    → "About"  (bloque temático, con su h2)
    section.section    → "Work Experience"
      article           → UNA experiencia laboral (se repite varias veces) ✅
        h3              → jerarquía correcta: h1 → h2 → h3
  footer               → pie con contacto ✅
```

**¿Por qué `<article>` dentro de `<section>`?**
Porque la `section` es "Work Experience" (tema) y cada `article` es una entrada
concreta de ese tema que podría leerse por separado.

---

## 4.4 Ejemplo real de navegación + `<main>` + `<footer>`

**Archivo:** `git/Pagina-Web/index.html` *(copiado tal cual)*

```html
<body>
<nav class="imagen-barra-superior">
  <button class="boton-hamburguesa" type="button" aria-label="Abrir menú">
    ☰
  </button>
  <a class="logo-inicio" href="index.html" aria-label="Ir a la página principal">
    <img src="assets/images/imagen-para-enlace.png" alt="Logo de la página">
  </a>
</nav>

<aside class="menu-lateral">
  <ul>
    <li><a class="boton-menu" href="index.html">Inicio</a></li>
    ...
  </ul>
</aside>
<div class="fondo-difuminado"></div>

<main class="contenido">
  <h1 class="centrado reveal titulo">Bienvenidos a mi página personal</h1>
  <hr class="reveal">
  <p class="centrado reveal">
    Hola, mi nombre es Hector Franco Magallo y esta es mi página personal.
  </p>
</main>
</div>

<footer class="pie-pagina">
  <nav aria-label="Enlaces legales">
    <a href="#">Aviso legal</a>
    <a href="#">Política de privacidad</a>
    <a href="#">Política de cookies</a>
  </nav>
  <p>© 2026 Hector Franco Magallo</p>
</footer>
<script src="js/programacion.js"></script>
</body>
```

> 📌 **Detalle importante:** hay **dos `<nav>`**: uno para el menú principal y otro
> para los enlaces del pie. Por eso cada uno lleva un `aria-label` distinto, para que
> el lector de pantalla distinga uno de otro.

---

## 4.5 Ejemplo de `<aside>` como menú lateral

**Archivo:** `git/Pagina-Web/index.html`

```html
<aside class="menu-lateral">
  <ul>
    <li><a class="boton-menu" href="index.html">Inicio</a></li>
    <li>
      <a class="boton-menu" href="pages/sobre_mi.html">Sobre mí</a>
      <ul>
        <li><a class="boton-menu" href="pages/certificados.html">Certificados</a></li>
        <li><a class="boton-menu" href="pages/estudios.html">Estudios Academicos</a></li>
      </ul>
    </li>
    <li><a class="boton-menu" href="#">Proyectos</a></li>
    <li><a class="boton-menu" href="pages/contactos.html">Contacto</a></li>
  </ul>
</aside>
```

---

## 4.6 Ejemplo de tarjetas con `<section>` + `<article>`

**Archivo:** `git/Pagina-Web/pages/proyectos.html` *(copiado tal cual)*

```html
<section class="proyectos">
    <article class="tarjeta">
        <div class="imagen-proyecto imagen-1" role="img" aria-label="Vista previa del Proyecto 1"></div>
        <div class="contenido-tarjeta">
            <h2>Pagina Web</h2>
            <p>Pagina web programada en HTML, CSS y JavaScript</p>
            <a href="https://github.com/HectorFM99/Pagina-Web" class="enlace-proyecto">Ver proyecto</a>
        </div>
    </article>

    <article class="tarjeta">
        <div class="imagen-proyecto imagen-2" role="img" aria-label="Vista previa del Proyecto 2"></div>
        ...
    </article>
</section>
```

> 📌 **Truco muy útil:** aquí la "imagen" **no es un `<img>`**, es un `<div>` con
> `role="img"` y un `background-image` en CSS. Así:
> - `role="img"` → le dice al lector de pantalla "esto es una imagen".
> - `aria-label` → da el texto alternativo.
> - El CSS (`.imagen-1`, `.imagen-2`) pone la foto de fondo.
>
> Es útil cuando la imagen es **decorativa de fondo** y no de contenido.

---

## 4.7 Enlace de salto (`skip-link`) — accesibilidad

**Archivo:** `web-minimalista-hector/.../about.html`

```html
<body>
  <!-- Enlace para saltar al contenido con el teclado -->
  <a class="skip-link" href="#main">Skip to main content</a>

  <div class="page">
    <header class="header"> ... menú ... </header>

    <main class="content" id="main"> ... contenido ... </main>
  </div>
</body>
```

Con CSS (`web-minimalista-hector/.../css/styles.css`) está oculto hasta que recibe foco:

```css
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

.skip-link:focus {
  top: 0;
}
```

> 📌 **Qué hace:** si alguien navega con `Tab` (teclado o lector de pantalla), en el
> primer tabulador aparece "Skip to main content" y puede saltarse todo el menú.
> Es un estándar de accesibilidad.

---

## 4.8 `div` y `span`: cuándo SÍ usarlos

Usa `<div>` / `<span>` cuando **no exista una etiqueta semántica** para lo que necesitas:

```html
<!-- ✅ Legítimo uso de <div>: una caja para maquetar con CSS -->
<div class="profile">
    <img class="profile__image" src="foto.png" alt="foto">
    <div class="profile__data"> ... </div>
</div>

<!-- ✅ Legítimo uso de <span>: envolver parte de un texto -->
<p>Precio: <span class="precio">29,99 €</span></p>

<!-- ❌ Mal uso: un <div> dentro de un <p> rompe el párrafo -->
<p><div>esto está mal</div></p>
```

> 📌 **Regla:** primero busca una etiqueta semántica. Si no hay ninguna que encaje,
> ahí sí, `<div>`.

---

## 4.9 Relación con CSS: la estructura define los selectores

Toda esta semántica se aprovecha en CSS. Ejemplo real de `web-cv-hector/css/styles.css`:

```css
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

.section {
  margin-top: 3rem;
}

.work-experience {
  display: flex;
  flex-direction: row;
  margin-top: 1rem;
}
```

Y del mismo proyecto, cómo se estiliza el `<footer>` (`git/Pagina-Web/css/cuerpo.css`):

```css
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
```

---

## ✅ Resumen rápido

| Situación | Etiqueta correcta |
|-----------|-------------------|
| Cabecera con logo/menú | `<header>` |
| Menú de enlaces | `<nav>` |
| Contenido principal (uno solo) | `<main>` |
| Bloque con título propio | `<section>` |
| Elemento repetible e independiente | `<article>` |
| Barra lateral / complemento | `<aside>` |
| Pie de página | `<footer>` |
| Caja solo para maquetar | `<div>` |
| Envolver parte de un texto | `<span>` |

👉 **Siguiente:** `05-formularios-y-tablas.md`
