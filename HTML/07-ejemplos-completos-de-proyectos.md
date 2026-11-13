# 07 — EJEMPLOS COMPLETOS COPIADOS DE TUS PROYECTOS

> 📁 **Los archivos de los proyectos NO se han tocado.** Todo lo de aquí abajo son
> **copias** de su contenido para que puedas estudiarlas con calma.

---

## EJEMPLO 1 — Página mínima y bien formada

**Origen:** `proyecto/main.html`

```html
<!DOCTYPE html>
<html lang="en">
    <link rel="stylesheet" href="cuerpo.css">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pagina Personal</title>
</head>
<body>
<h1>Bienvenidos a mi pagina personal</h1>
<p>Hola, mi nombre es Hector Franco Magallo y esta es mi pagina personal.</p>
<div>   
    <p class="texto">En esta pagina podras encontrar informacion sobre mi, mis intereses y mis proyectos.</p>
    <a href="proyectos.html">Proyectos</a>
</div>
</body>
</html>
```

**Qué enseña:**
- `<!DOCTYPE html>` + `lang` + `charset` + `viewport` + `title` → la base obligatoria.
- `<h1>` título principal, `<p>` párrafos.
- `class="texto"` → gancho para CSS.
- `<a href="proyectos.html">` → enlace relativo a otra página.
- **Mejora posible:** el `<link>` del CSS debería ir **dentro** de `<head>`.

---

## EJEMPLO 2 — Página con encabezados, listas, imagen y enlace

**Origen:** `proyecto/ejercicio/mi-rincon-favorito.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mi Rincón Favorito</title>
</head>
<body>
    <h1>Mi Rincón Favorito</h1>
    <h2>Hola a todos</h2>
    <h2>Este es un lugar especial para mí</h2>
    <p>Este es mi rincón favorito.</p>
<div>
   <a href="https://cartooncorp.net/" target="_blank" rel="noopener noreferrer">
     Mi rincón favorito
   </a>
    <div>
        <ul>
            <li>Este es un lugar especial para mí porque me gusta mucho el anime y el manga, y en este lugar puedo encontrar todo lo relacionado con estos temas. Además, me gusta mucho la decoración y el ambiente del lugar, que me hace sentir cómodo y relajado.</li>
            <li>En este lugar puedo encontrar figuras de mis personajes favoritos, así como también ropa y accesorios relacionados con el anime y el manga. También hay una sección de cómics y mangas, donde puedo encontrar nuevas historias y aventuras para disfrutar.</li>
            <li>En resumen, este lugar es especial para mí porque me permite disfrutar de mi pasión por el anime y el manga, y me hace sentir feliz y relajado cada vez que lo visito.</li>
        </ul>
    </div>
    
    <div>
        <p>Estas son algunas actividades que se pueden hacer en este sitio;</p>
            <ol>
                <li>Comprar figuras de anime y manga</li>
                <li>Explorar la sección de cómics y mangas</li>
                <li>Disfrutar del ambiente y la decoración del lugar</li>
                <li>Participar en eventos relacionados con el anime y el manga</li>
            </ol>
    </div>
</div>
</body>
</html>
```

**Qué enseña:**
- Jerarquía `h1` → `h2`.
- Diferencia entre `<ul>` (características) y `<ol>` (actividades con orden).
- Enlace externo con `target="_blank" rel="noopener noreferrer"`.
- `<div>` como contenedores neutros.

---

## EJEMPLO 3 — Formulario con `<label>` y `<select>`

**Origen:** `Ejercicio github/cagada-de-repositorio-nano/contacto/contacto.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Contacto</title>
</head>
<body>
    <img src="https://media1.tenor.com/m/ltMwVmL-Y8oAAAAC/cat-middle-finger.gif" alt="Imagen de contacto">
    <h1>Contacto</h1>
    <p>¡Contáctanos!</p>
    <label for="respuesta">Elige tu respuesta</label>

  <select id="respuesta" name="respuesta">
    <option value="" selected disabled>Elige una respuesta</option>
    <option value="rojo">Si</option>
    <option value="azul">No</option>
  </select>
</body>
</html>
```

**Qué enseña:**
- `label for` ↔ `id` asociados.
- `name` = clave de envío.
- `selected disabled` = opción fantasma obligatoria.

---

## EJEMPLO 4 — Encabezados, imagen, imagen clicable e iframe

**Origen:** `proyecto/ejercicio/ejercicio.html`
*(Se omiten los valores de `alt` originales por contener lenguaje soez; el resto es idéntico)*

```html
<!DOCTYPE html>
<html lang="es">
<body>
    <h1>Ejercicio de HTML</h1>
    <h2>Subtítulo de ejemplo</h2>
    <h3>Otro subtítulo</h3>
    <h4>Subtítulo más pequeño</h4>
    <h5>Subtítulo aún más pequeño</h5>
    <h6>Subtítulo más pequeño de todos</h6>
    <br>
    <br>
    <br>
    <br>
    <img src="https://cdn.pixabay.com/photo/2015/06/24/15/45/code-820275_960_720.jpg" alt="Imagen de código" width="520" height="320">
    <a href="https://pixabay.com/es/photos/..."><img src="https://cdn.pixabay.com/photo/2015/06/24/15/45/code-820275_960_720.jpg" alt="Imagen de código" width="520" height="320"></a>
    <br>
    <br>
    <br>
    <br>
   <iframe
  width="560"
  height="315"
  src="https://youtube/GAHtUidWuDA?list=PL21aNyRMMKn6r3Fg05G2LZWdVOcJqK_5q"
  title="Mi vídeo destacado"
  allowfullscreen>
</iframe>
</body>
</html>
```

**Qué enseña:**
- Los 6 niveles de título en orden.
- `<br>` repetido para separar (forma básica).
- Imagen normal e imagen **envuelta en un `<a>`** (clicable).
- `<iframe>` con `title` y `allowfullscreen`.

---

## EJEMPLO 5 — Página real con semántica, menú lateral y pie

**Origen:** `git/Pagina-Web/index.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pagina Personal</title>

    <link rel="stylesheet" href="css/cuerpo.css">

    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&display=swap" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&family=Roboto:wght@400;700&display=swap" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400;700&display=swap" rel="stylesheet">
</head>
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

    <li>
      <a class="boton-menu" href="pages/sobre_mi.html">Sobre mí</a>
      <ul>
        <li><a class="boton-menu" href="pages/certificados.html">Certificados</a></li>
        <li><a class="boton-menu" href="pages/estudios.html">Estudios Academicos</a></li>
      </ul>
    </li>

    <li>
      <a class="boton-menu" href="#">Proyectos</a>
      <ul>
        <li><a class="boton-menu" href="pages/proyectos.html">Todos los proyectos</a></li>
      </ul>
    </li>

    <li><a class="boton-menu" href="pages/contactos.html">Contacto</a></li>
  </ul>
</aside>
<div class="fondo-difuminado"></div>
<main class="contenido">
  <h1 class="centrado reveal titulo">Bienvenidos a mi página personal</h1>
  <hr class="reveal">
  <p class="centrado reveal">
    Hola, mi nombre es Hector Franco Magallo y esta es mi página personal.
  </p>
  <p class="centrado reveal">
    En esta página podrás encontrar información sobre mí, mis intereses y mis proyectos.
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
</html>
```

**Qué enseña:**
- `<head>` completo con **Google Fonts**.
- `<nav>` barra superior + `<aside>` menú lateral + `<main>` contenido + `<footer>` pie.
- `aria-label` en botón, en logo y en el nav del pie.
- Clases `reveal` → animaciones que activa **JavaScript**.
- `<script>` al final del `<body>` → carga el JS tras existir el HTML.
- Estructura de **rutas relativas** (`css/`, `pages/`, `assets/`, `js/`).

> ⚠️ **Nota de estudio:** hay un `</div>` sobrante justo antes del `<footer>`
> (no hay `<div>` que cierre ahí). Es el tipo de error que detecta el validador W3C.

---

## EJEMPLO 6 — Página con tipografías y bloque `<main>` + `<footer>`

**Origen:** `git/Pagina-Web/pages/sobre_mi.html` *(fragmento)*

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="../css/cuerpo.css">
    <link href="https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400;700&display=swap" rel="stylesheet">
    <title>Sobre Mí</title>
</head>
<body>
<nav class="imagen-barra-superior">
  <button class="boton-hamburguesa" type="button" aria-label="Abrir menú">
    ☰
  </button>
  <a class="logo-inicio" href="../index.html" aria-label="Ir a la página principal">
  <img src="../assets/images/imagen-para-enlace.png" alt="Logo de la página">
</a>
</nav>

<aside class="menu-lateral">
  <ul>
    <li><a class="boton-menu" href="../index.html">Inicio</a></li>
    <li>
      <a class="boton-menu" href="sobre_mi.html">Sobre mí</a>
      <ul>
        <li><a class="boton-menu" href="certificados.html">Certificados</a></li>
        <li><a class="boton-menu" href="estudios.html">Estudios Academicos</a></li>
      </ul>
    </li>
    <li>
      <a class="boton-menu" href="#">Proyectos</a>
      <ul>
        <li><a class="boton-menu" href="proyectos.html">Todos los proyectos</a></li>
      </ul>
    </li>
    <li><a class="boton-menu" href="contactos.html">Contacto</a></li>
  </ul>
</aside>
<div class="fondo-difuminado"></div>
<main class="contenido">
  <h1 class="centrado reveal titulo">Sobre Mí</h1>
  <p class="escritura" id="texto-escrito">Soy Hector, desarrollador web en formación...</p>
</main>
<script src="../js/programacion.js"></script>
<footer class="pie-pagina">
  <nav aria-label="Enlaces legales">
    <a href="#">Aviso legal</a>
    <a href="#">Política de privacidad</a>
    <a href="#">Política de cookies</a>
  </nav>

  <p>© 2026 Hector Franco Magallo</p>
</footer>
</body>
</html>
```

**Qué enseña:**
- **Rutas con `../`** → subir un nivel porque esta página está en `pages/`.
- `id="texto-escrito"` → **lo busca JavaScript** para hacer el efecto de máquina de escribir.
- El CSS se enlaza con `href="../css/cuerpo.css"`.

---

## EJEMPLO 7 — Tarjetas de proyecto con `<section>` + `<article>`

**Origen:** `git/Pagina-Web/pages/proyectos.html` *(fragmento)*

```html
<main class=" contenido contenido-proyectos">
    <h1 class="centrado reveal titulo">Proyectos</h1>
    <hr>
    <p class="centrado">Lista de mis proyectos:</p>
</main>
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
    <div class="contenido-tarjeta">
      ...
    </div>
  </article>
</section>
```

**Qué enseña:**
- Patrón **sección repetible** → `<section>` contenedora con varias `<article>` idénticas.
- `<div role="img" aria-label="...">` → el div hace de imagen (el CSS pone el fondo).
- `class="enlace-proyecto"` → botón estilizado con CSS.

---

## EJEMPLO 8 — CV con semántica completa

**Origen:** `web-cv-hector/web-cv-hector/index.html` *(cabecera + una sección)*

```html
<!DOCTYPE html>
<html lang="es">

<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mi CV</title>
    <link rel="stylesheet" href="./css/styles.css">
</head>

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
                <article class="work-experience">
                    <div class="work-experience__date"><time datetime="2015">2015</time> — <time datetime="2017">2017</time></div>
                    <div class="work-experience__details">
                        <h3 class="work-experience__title">Junior Designer at Creative Co<span aria-hidden="true">↗</span></h3>
                        <p class="work-experience__location">Atlanta, GA</p>
                        <div class="images-experience">
                            <img src="./assets/images/design.png"
                                alt="Abstract 3D composition of colorful geometric shapes" width="150" height="90">
                            <img src="./assets/images/phone-image.png"
                                alt="Smartphone showing a mood board app, on a desk next to a laptop" width="150" height="90">
                            <img src="./assets/images/design-screen.png"
                                alt="Collection of mobile app screens for a magazine website" width="150" height="90">
                        </div>
                    </div>
                </article>
            </section>
        </main>
        <footer>
            <section class="section">
                <h2 class="section__title">Contact</h2>
                <div class="work-experience">
                    <div class="work-experience__date">Threads</div>
                    <div class="work-experience__details">
                        <p class="work-experience__title">username<span aria-hidden="true">↗</span></p>
                    </div>
                </div>
            </section>
        </footer>
    </div>
</body>

</html>
```

**Qué enseña:**
- Jerarquía perfecta: `h1` (nombre) → `h2` (secciones) → `h3` (puestos).
- `<time datetime>` para fechas con significado.
- `<span aria-hidden>` para iconos decorativos.
- `<img width height>` para evitar saltos de layout.
- BEM en los nombres de clase: `work-experience__date`, `card__title`...

---

## EJEMPLO 9 — Skip-link y navegación accesible

**Origen:** `web-minimalista-hector/web-minimalista-hector/about.html` *(fragmento)*

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>About — Sarah Chen</title>

  <!-- Tipografías -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Source+Code+Pro:wght@400;500&family=Source+Serif+Pro:wght@300;400&display=swap">

  <!-- Mis estilos -->
  <link rel="stylesheet" href="css/styles.css">
</head>
<body>

  <!-- Enlace para saltar al contenido con el teclado -->
  <a class="skip-link" href="#main">Skip to main content</a>

  <div class="page">

    <!-- Navegación -->
    <header class="header">
      <nav class="nav">
        <a class="brand" href="index.html">
          <h1 class="brand__name">Sarah Chen</h1>
          <span class="brand__avatar">
            <img class="brand__photo" src="img/portrait.jpg" alt="portrait of woman">
          </span>
        </a>

        <ul class="menu">
          <li><a class="menu__link menu__link--home" href="index.html">Home</a></li>
          <li><a class="menu__link menu__link--about" href="about.html">About</a></li>
          <li><a class="menu__link menu__link--contact" href="mailto:email@ejemplo.com">Contact</a></li>
        </ul>
      </nav>
    </header>

    <main class="content" id="main">
      <section class="intro intro--about">
        <p class="intro__text">During my time at Aetheria School of Visual Arts, ...</p>
        <p class="intro__text">I thrive in collaborative environments, ...</p>
      </section>

      <section class="now">
        <div class="now__head">
          <p class="now__title">Right now I am...</p>
        </div>
        <div class="now__list">
          <div class="now__row">
            <p class="now__label">asdf to:</p>
            <p class="now__label">Relationships by Haim</p>
          </div>
          <figure class="now__figure">
            <img class="now__photo" src="img/about-portrait.jpg" alt="portrait of woman" loading="lazy">
          </figure>
        </div>
      </section>
```

**Qué enseña:**
- `skip-link` + `id="main"` → navegación con teclado.
- `mailto:` como enlace de contacto.
- `<figure>` para imagen con contexto.
- `loading="lazy"` en imágenes.
- Comentarios HTML explicativos (`<!-- Tipografías -->`).

---

## 🎓 Ejercicio final: desmonta una página

Toma `git/Pagina-Web/index.html` y responde por escrito:

1. ¿Cuántos `<h1>` hay? ¿Debería haber más de uno?
2. ¿Qué elementos son **semánticos** y cuáles son `<div>` genéricos?
3. ¿Qué atributos de accesibilidad hay y para qué sirve cada uno?
4. ¿Qué rutas relativas aparecen y qué significan `./`, `../` y `pages/`?
5. ¿Qué clases están ahí **solo** para que JavaScript las busque?

👉 **Vuelve al índice:** `00-INDICE.md`
