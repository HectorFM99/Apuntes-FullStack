# 📋 CHEATSHEET — HTML

> Chuleta rápida de comandos y código. Teoría en los documentos 01–07 de esta carpeta.

---

## 1. Estructura base del documento

```html
<!DOCTYPE html>                                   <!-- modo estándar (HTML5) -->
<html lang="es">                                  <!-- idioma: accesibilidad y SEO -->
<head>
  <meta charset="UTF-8">                          <!-- tildes y ñ -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">  <!-- móvil -->
  <meta name="description" content="Mi web personal">   <!-- resumen para Google -->
  <title>Mi web</title>                           <!-- texto de la pestaña -->
  <link rel="icon" href="img/favicon.png">        <!-- favicon -->
  <link rel="stylesheet" href="css/styles.css">   <!-- CSS externo -->
  <script src="js/programacion.js" defer></script><!-- JS tras parsear el HTML -->
</head>
<body>
  <!-- contenido visible -->
</body>
</html>
```

| Va en `<head>` | Va en `<body>` |
|----------------|----------------|
| `meta`, `title`, `link`, `script defer`, fuentes | Todo lo que se ve: textos, imágenes, formularios |

```html
<!-- Google Fonts en el head -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&display=swap" rel="stylesheet">

<!-- comentario: no se ve en la página -->
```

---

## 2. Texto

```html
<h1>Título principal</h1>   <!-- 1 por página -->
<h2>Sección</h2> … <h6>     <!-- jerarquía, sin saltar niveles -->
<p>Párrafo de texto.</p>
<br>                        <!-- salto de línea -->
<hr>                        <!-- separador temático -->
<strong>importante</strong> <!-- negrita con significado -->
<em>énfasis</em>            <!-- cursiva con significado -->
<mark>resaltado</mark>  <small>letra pequeña</small>  <code>const x = 1</code>
<span class="destacado">trozo en línea</span>   <!-- contenedor sin significado -->
<time datetime="2026-11-17">17 de noviembre</time>
<blockquote cite="url">Cita larga</blockquote>  <q>cita corta</q>
<pre>texto   con   espacios   respetados</pre>
```

| Entidad | Resultado | Entidad | Resultado |
|---------|-----------|---------|-----------|
| `&lt;` `&gt;` | `<` `>` | `&amp;` | `&` |
| `&nbsp;` | espacio que no salta | `&copy;` | © |
| `&euro;` | € | `&quot;` | " |

---

## 3. Enlaces, imágenes y listas

```html
<a href="pages/sobre_mi.html">Sobre mí</a>                 <!-- relativo interno -->
<a href="../index.html">Volver</a>                         <!-- subir una carpeta -->
<a href="https://github.com" target="_blank" rel="noopener noreferrer">GitHub</a>  <!-- pestaña nueva -->
<a href="#contacto">Ir a contacto</a>                      <!-- ancla: id="contacto" -->
<a href="mailto:yo@mail.com">Email</a>  <a href="tel:+34600000000">Llamar</a>
<a href="cv.pdf" download>Descargar CV</a>

<img src="img/foto.jpg" alt="Descripción de la foto" width="300">   <!-- alt SIEMPRE -->
<img src="img/deco.svg" alt="">                                     <!-- decorativa: alt vacío -->
<figure>
  <img src="img/plato.jpg" alt="Paella valenciana">
  <figcaption>Paella del domingo</figcaption>
</figure>
```

```html
<ul> <li>Sin orden</li> <li>●</li> </ul>
<ol> <li>Con número</li> <li>2.</li> </ol>
<ul>
  <li>Menú
    <ul><li>Submenú</li></ul>        <!-- lista anidada DENTRO del li -->
  </li>
</ul>
<dl> <dt>HTML</dt> <dd>Lenguaje de marcado</dd> </dl>
```

---

## 4. Multimedia

```html
<video src="video.mp4" controls width="400" poster="portada.jpg"></video>
<audio src="cancion.mp3" controls></audio>
<iframe src="https://www.youtube.com/embed/ID" title="Vídeo" width="560" height="315"
        allowfullscreen loading="lazy"></iframe>
```

| Atributo | Qué hace |
|----------|----------|
| `controls` | Muestra play/pausa/volumen |
| `autoplay muted` | Autoplay (los navegadores lo exigen silenciado) |
| `loop` | Repite |
| `loading="lazy"` | Carga diferida (img/iframe) |

---

## 5. Estructura semántica

```html
<header>
  <nav>
    <ul><li><a href="index.html">Inicio</a></li></ul>
  </nav>
</header>
<main>                          <!-- 1 por página: contenido principal -->
  <section id="proyectos">      <!-- bloque temático con título -->
    <h2>Proyectos</h2>
    <article>…</article>        <!-- contenido independiente (tarjeta, post) -->
  </section>
  <aside>…</aside>              <!-- contenido complementario -->
</main>
<footer>© 2026</footer>
```

| Etiqueta | Uso |
|----------|-----|
| `header` / `footer` | Cabecera / pie (de página o de un artículo) |
| `nav` | Navegación principal |
| `main` | Contenido principal, uno por página |
| `section` | Sección con encabezado |
| `article` | Pieza autónoma reutilizable |
| `div` | Contenedor **sin** significado (solo para estilos) |

---

## 6. Formularios

```html
<form action="/contacto" method="post">
  <label for="nombre">Nombre</label>
  <input type="text" id="nombre" name="nombre" placeholder="Tu nombre" required>

  <label for="email">Email</label>
  <input type="email" id="email" name="email" required>

  <input type="password" name="pass" minlength="8">
  <input type="number" name="edad" min="18" max="99" step="1">
  <input type="date" name="fecha">
  <input type="checkbox" id="acepto" name="acepto"> <label for="acepto">Acepto</label>
  <input type="radio" name="nivel" value="junior" checked> Junior
  <input type="radio" name="nivel" value="senior"> Senior

  <select name="curso">
    <option value="">Elige…</option>
    <option value="react">React</option>
  </select>

  <textarea name="mensaje" rows="4"></textarea>

  <fieldset>
    <legend>Datos extra</legend>
  </fieldset>

  <button type="submit">Enviar</button>
  <button type="reset">Borrar</button>
</form>
```

| Atributo de validación | Qué hace |
|------------------------|----------|
| `required` | Obligatorio |
| `minlength` / `maxlength` | Longitud de texto |
| `min` / `max` / `step` | Rango numérico o de fecha |
| `pattern="[0-9]{5}"` | Expresión regular |
| `type="email"` / `url` | Formato automático |

> 📌 `for` del `label` = `id` del input. `name` = clave con la que viaja el dato.

---

## 7. Tablas

```html
<table>
  <caption>Horario</caption>
  <thead>
    <tr><th scope="col">Día</th><th scope="col">Clase</th></tr>
  </thead>
  <tbody>
    <tr><td>Lunes</td><td>HTML</td></tr>
    <tr><td colspan="2">Festivo</td></tr>     <!-- ocupa 2 columnas -->
  </tbody>
  <tfoot>
    <tr><td>Total</td><td>5 h</td></tr>
  </tfoot>
</table>
```

---

## 8. Atributos globales y accesibilidad

| Código | Qué hace |
|--------|----------|
| `id="contacto"` | Identificador único (anclas, JS, `label for`) |
| `class="card destacada"` | Una o varias clases para CSS/JS |
| `title="texto"` | Tooltip |
| `hidden` | Oculta el elemento |
| `data-precio="20"` | Dato propio (JS: `el.dataset.precio`) |
| `aria-label="Cerrar"` | Nombre accesible de un botón sin texto |
| `aria-hidden="true"` | Oculta a lectores de pantalla (iconos decorativos) |
| `role="alert"` | Anuncia mensajes (errores) |
| `tabindex="0"` | Hace enfocable con teclado |

```html
<button type="button" aria-label="Abrir menú">☰</button>  <!-- botón solo icono -->
<a class="skip" href="#main">Saltar al contenido</a>       <!-- enlace de salto -->
```

✅ Checklist rápido: `lang`, un `h1`, `alt` en imágenes, `label` en inputs, `button` para
acciones y `a` para navegar, contraste suficiente, se puede usar solo con teclado.

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| Tildes y ñ salen raras (`Ã©`) | `<meta charset="UTF-8">` y guardar el archivo en UTF-8 |
| En el móvil se ve diminuto | Falta `<meta name="viewport" …>` |
| Imagen rota | Ruta relativa mal (`../img/` desde `pages/`), mayúsculas en el nombre |
| El CSS/JS no se aplica | `href`/`src` mal o `<script>` en el head sin `defer` |
| Clic en el label no enfoca el input | `for` no coincide con el `id` |
| El formulario no envía un campo | Falta `name` en el input |
| Botón dentro de un form recarga la página | `type="button"` si no es de envío |
| Anidamiento roto / cierres cruzados | Cerrar en orden inverso: `<p><strong>…</strong></p>` |
