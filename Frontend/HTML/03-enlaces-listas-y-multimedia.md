# 03 — ENLACES, LISTAS Y MULTIMEDIA

---

## 3.1 Enlaces: `<a>` (anchor)

```html
<a href="https://www.google.com">Ir a Google</a>
```

| Atributo | Qué hace |
|----------|----------|
| `href` | **Destino** del enlace (URL, ruta de archivo o `#ancla`) |
| `target="_blank"` | Abre en una **pestaña nueva** |
| `rel="noopener noreferrer"` | **Seguridad**: impide que la página abierta controle la tuya |
| `title` | Tooltip al pasar el ratón |
| `download` | En lugar de navegar, **descarga** el archivo |

### Tipos de `href`

```html
<!-- Externo: URL completa -->
<a href="https://www.google.com">Google</a>

<!-- Relativo: otro archivo del MISMO proyecto -->
<a href="proyectos.html">Proyectos</a>
<a href="pages/sobre_mi.html">Sobre mí</a>

<!-- Subir un nivel con ../ -->
<a href="../index.html">Volver al inicio</a>

<!-- Ancla: salto dentro de la MISMA página (necesita un id) -->
<a href="#contacto">Ir a contacto</a>
...
<h2 id="contacto">Contacto</h2>

<!-- Correo y teléfono -->
<a href="mailto:nombre@correo.com">Escribir correo</a>
<a href="tel:+34600000000">Llamar</a>

<!-- JavaScript (preventDefault) -->
<a href="#" onclick="alert('hola'); return false;">Pruébame</a>
```

### Ejemplo real copiado de un proyecto — enlace con pestaña nueva

**Archivo:** `proyecto/ejercicio/mi-rincon-favorito.html`

```html
<a href="https://cartooncorp.net/" target="_blank" rel="noopener noreferrer">
    Mi rincón favorito
</a>
```

> 📌 **¿Por qué `rel="noopener noreferrer"`?**
> Al abrir en pestaña nueva con `target="_blank"`, la página destino recibe una
> referencia a tu ventana (`window.opener`) y podría redirigirte. `noopener` lo corta.
> **Añádelo siempre** cuando uses `target="_blank"`.

### Ejemplo real copiado de un proyecto — enlace relativo interno

**Archivo:** `git/Pagina-Web/index.html`

```html
<nav class="imagen-barra-superior">
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
      <a class="boton-menu" href="#">Proyectos</a>   <!-- "#" = enlace sin destino aún -->
      <ul>
        <li><a class="boton-menu" href="pages/proyectos.html">Todos los proyectos</a></li>
      </ul>
    </li>
    <li><a class="boton-menu" href="pages/contactos.html">Contacto</a></li>
  </ul>
</aside>
```

> 📌 **Rutas relativas:** como `index.html` está en la raíz del proyecto y las
> subpáginas están en `pages/`, desde el índice se escribe `pages/sobre_mi.html`,
> y desde **dentro** de `pages/` se escribe `../index.html` (subir un nivel).

### Enlace que envuelve una imagen

**Archivo:** `proyecto/ejercicio/ejercicio.html`

```html
<a href="https://pixabay.com/es/photos/...">
    <img src="https://cdn.pixabay.com/photo/2015/06/24/15/45/code-820275_960_720.jpg"
         alt="Imagen de código" width="520" height="320">
</a>
```

Aquí **la imagen entera es clicable**.

---

## 3.2 Imágenes: `<img>`

```html
<img src="assets/images/foto.png" alt="Descripción de la foto" width="300" height="200">
```

| Atributo | Qué hace |
|----------|----------|
| `src` | Ruta o URL de la imagen (**obligatorio**) |
| `alt` | Texto alternativo si la imagen no carga (**obligatorio para accesibilidad**) |
| `width` / `height` | Tamaño en píxeles. Sirve también para **evitar el salto de layout** |
| `loading="lazy"` | Carga la imagen solo cuando llega al viewport (mejora la velocidad) |
| `srcset` | Versiones distintas de la imagen según pantalla |

> ⚠️ **`<img>` es una etiqueta vacía:** no lleva `</img>`.

### Ejemplos reales copiados de un proyecto

**Archivo:** `web-cv-hector/web-cv-hector/index.html`

```html
<img class="profile__image" alt="Sara Lawrence"
     src="./assets/images/image-profile.png" width="92" height="93">
```

```html
<div class="images-experience">
    <img src="./assets/images/design.png"
         alt="Abstract 3D composition of colorful geometric shapes" width="150" height="90">
    <img src="./assets/images/phone-image.png"
         alt="Smartphone showing a mood board app, on a desk next to a laptop" width="150" height="90">
    <img src="./assets/images/design-screen.png"
         alt="Collection of mobile app screens for a magazine website" width="150" height="90">
</div>
```

> 📌 **Buenas prácticas que se ven aquí:**
> 1. El `alt` **describe** la imagen (no dice "imagen1.png").
> 2. `width` y `height` están puestos → el navegador reserva el hueco y la página no "salta".
> 3. Las rutas usan `./` explícito.

**Archivo:** `web-minimalista-hector/.../about.html` — imagen diferida

```html
<img class="now__photo" src="img/about-portrait.jpg" alt="portrait of woman" loading="lazy">
```

### Imagen decorativa

Si la imagen **no aporta nada** (un icono decorativo), se deja `alt=""` vacío para que
el lector de pantalla la ignore.

---

## 3.3 Listas: `<ul>`, `<ol>`, `<li>`

### Lista sin orden — `<ul>` (bullets ●)

```html
<ul>
    <li>Elemento 1</li>
    <li>Elemento 2</li>
</ul>
```

### Lista ordenada — `<ol>` (números 1, 2, 3...)

```html
<ol>
    <li>Primer paso</li>
    <li>Segundo paso</li>
</ol>
```

### Ejemplo real copiado de un proyecto (¡ambas listas!)

**Archivo:** `proyecto/ejercicio/mi-rincon-favorito.html` *(copiado tal cual)*

```html
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
```

> 📌 `ul` = elementos **sin orden** (comodines, características).
> `ol` = elementos **con orden** (pasos, instrucciones, ranking).

### Listas anidadas (submenú)

**Archivo:** `git/Pagina-Web/index.html`

```html
<li>
  <a class="boton-menu" href="pages/sobre_mi.html">Sobre mí</a>
  <ul>                                    <!-- lista DENTRO de un li -->
    <li><a class="boton-menu" href="pages/certificados.html">Certificados</a></li>
    <li><a class="boton-menu" href="pages/estudios.html">Estudios Academicos</a></li>
  </ul>
</li>
```

### Lista de definición `<dl>`

```html
<dl>
    <dt>HTML</dt>
    <dd>Estructura de la página</dd>
    <dt>CSS</dt>
    <dd>Diseño visual de la página</dd>
</dl>
```

---

## 3.4 Contenido incrustado: `<iframe>`

`<iframe>` mete **una página completa dentro de otra** (YouTube, Google Maps, PDF...).

```html
<iframe
    width="560"
    height="315"
    src="https://www.youtube.com/embed/VIDEO_ID"
    title="Mi vídeo destacado"
    allowfullscreen>
</iframe>
```

| Atributo | Qué hace |
|----------|----------|
| `src` | URL de lo que se incrusta |
| `width` / `height` | Tamaño del marco |
| `title` | Obligatorio por accesibilidad (lo leen los lectores de pantalla) |
| `allowfullscreen` | Permite pasar a pantalla completa |
| `loading="lazy"` | Carga diferida |
| `sandbox` | Restringe lo que puede hacer la página incrustada (seguridad) |

### Ejemplo real copiado de un proyecto

**Archivo:** `proyecto/ejercicio/ejercicio.html` *(copiado tal cual)*

```html
<iframe
  width="560"
  height="315"
  src="https://youtube/GAHtUidWuDA?list=PL21aNyRMMKn6r3Fg05G2LZWdVOcJqK_5q"
  title="Mi vídeo destacado"
  allowfullscreen>
</iframe>
```

> 📌 Para YouTube la URL correcta es
> `https://www.youtube.com/embed/ID_VIDEO` (con `/embed/`).

---

## 3.5 Vídeo y audio nativos

```html
<video src="mi-video.mp4" controls width="640" poster="portada.jpg">
    Tu navegador no soporta vídeo HTML5.
</video>

<video controls width="640">
    <source src="mi-video.webm" type="video/webm">
    <source src="mi-video.mp4" type="video/mp4">
    Tu navegador no soporta vídeo HTML5.
</video>

<audio controls>
    <source src="cancion.mp3" type="audio/mpeg">
</audio>
```

- `controls` → muestra los botones de play/volumen (**sin él no se puede controlar**).
- `poster` → imagen de portada antes de reproducir.
- `muted`, `loop`, `autoplay`, `preload` → otras opciones habituales.

---

## 3.6 Tabla de resumen de multimedia

| Qué quiero | Etiqueta |
|------------|----------|
| Foto | `<img>` |
| Icono decorativo | `<img alt="">` |
| Vídeo con controles | `<video controls>` |
| Audio | `<audio controls>` |
| YouTube / mapa incrustado | `<iframe>` |
| Imagen con pie de foto | `<figure>` + `<figcaption>` |
| Enlace | `<a href>` |
| Lista con viñetas | `<ul>` + `<li>` |
| Lista numerada | `<ol>` + `<li>` |

---

## ✅ Resumen rápido

1. `<a href="...">` navega; añade `target="_blank" rel="noopener noreferrer"` si abres pestaña nueva.
2. `href="#id"` sirve para saltar dentro de la misma página.
3. `<img>` necesita **siempre** `alt`; pon también `width`/`height`.
4. `ul` = sin orden, `ol` = con orden, y se pueden anidar.
5. `<iframe>` incrusta otras páginas (YouTube usa `/embed/`).
6. `loading="lazy"` en imágenes y iframes mejora mucho el rendimiento.

👉 **Siguiente:** `04-estructura-semantica-y-layout.md`
