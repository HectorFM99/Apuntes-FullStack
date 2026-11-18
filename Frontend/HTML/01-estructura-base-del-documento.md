# 01 — ESTRUCTURA BASE DEL DOCUMENTO HTML

## ¿Qué es HTML?

**HTML** = **H**yper**T**ext **M**arkup **L**anguage (Lenguaje de Marcado de Hipertexto).

No es un lenguaje de programación: es un lenguaje de **etiquetas** (tags) que le dice
al navegador **qué significa** cada parte del contenido (esto es un título, esto es un
párrafo, esto es una imagen...). La parte "cómo se ve" (colores, tamaños, posiciones)
la decide **CSS**, y la parte "cómo se comporta" la decide **JavaScript**.

```
HTML  →  ESTRUCTURA  (qué hay en la página)
CSS   →  DISEÑO      (cómo se ve)
JS    →  COMPORTAMIENTO (qué hace al interactuar)
```

---

## 1.1 La estructura mínima de toda página

Toda página HTML tiene **siempre** esta pinta:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Título que sale en la pestaña del navegador</title>
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
    <!-- Aquí va TODO lo que se ve en pantalla -->
</body>
</html>
```

### Desglose línea por línea

| Línea | Etiqueta | Qué hace |
|-------|----------|----------|
| 1 | `<!DOCTYPE html>` | Le dice al navegador: "esta página es HTML5". Sin esto, el navegador puede interpretarla en modo antiguo. **No es una etiqueta, es una declaración.** |
| 2 | `<html lang="es">` | Etiqueta **raíz**. Envuelve todo el documento. `lang="es"` indica el idioma (útil para traductores automáticos y lectores de pantalla). |
| 3 | `<head>` | Cabecera. **No se ve en pantalla.** Contiene información *sobre* la página. |
| 4 | `<meta charset="UTF-8">` | Codificación de caracteres. Sin esto, las tildes y la ñ se rompen (`Ã±` en vez de `ñ`). |
| 5 | `<meta name="viewport" ...>` | Hace que la página se adapte a móviles. **Imprescindible** en cualquier web moderna. |
| 6 | `<title>` | El texto de la **pestaña** del navegador y lo que aparece en Google. |
| 7 | `<link rel="stylesheet" href="...">` | Conecta el archivo **CSS** externo. |
| 8 | `</head>` | Cierra la cabecera. |
| 9 | `<body>` | Aquí empieza **todo lo visible**: textos, imágenes, botones... |
| 10 | `<!-- comentario -->` | Comentario. El navegador lo ignora, solo sirve para humanos. |
| 11-13 | `</body>`, `</html>` | Cierran el cuerpo y el documento. |

---

## 1.2 Ejemplo real copiado de un proyecto

**Archivo:** `proyecto/main.html` *(copiado tal cual, sin modificar)*

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

> 📌 **Análisis:**
> - `<h1>` → título principal (el más grande).
> - `<p>` → párrafo de texto.
> - `<div>` → contenedor genérico (una caja sin significado).
> - `class="texto"` → nombre de clase para que CSS pueda cogerlo y ponerle estilo.
> - `<a href="proyectos.html">` → enlace a otra página del mismo proyecto.
> - ⚠️ Detalle a corregir: el `<link>` está *antes* de `<head>` (debería ir dentro).

---

## 1.3 El orden correcto: `<head>` vs `<body>`

```
┌──────────────────────────────────────┐
│ <html lang="es">                     │
│  ┌────────────────────────────────┐  │
│  │ <head>   ← NO SE VE            │  │
│  │  meta, title, link CSS,        │  │
│  │  fuentes, scripts de apoyo     │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │ <body>   ← TODO LO VISIBLE     │  │
│  │  header, main, footer,         │  │
│  │  textos, imágenes, formularios │  │
│  └────────────────────────────────┘  │
│ </html>                              │
└──────────────────────────────────────┘
```

---

## 1.4 Ejemplo real con Google Fonts en el `<head>`

**Archivo:** `git/Pagina-Web/index.html` *(copiado tal cual)*

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
```

> 📌 **Qué aporta este ejemplo:**
> - `rel="stylesheet"` → indica que el enlace es una hoja de estilos.
> - `rel="preconnect"` → optimización: avisa al navegador de que va a pedir cosas a ese dominio.
> - `crossorigin` → necesario para que las fuentes de Google funcionen por CORS.
> - Los `<link>` de Google Fonts **van en el `<head>`**, siempre.

---

## 1.5 Comentarios en HTML

```html
<!-- Este es un comentario de una línea -->

<!--
   Este es un comentario
   que ocupa varias líneas
-->

<body>
    <!-- Los comentarios se pueden meter dentro del body -->
    <h1>Título visible</h1>
    <!-- <p>Este párrafo está desactivado, no se verá</p> -->
</body>
```

**Ejemplo real** (archivo `ejercicio-pagina web/pagina web/index.html`):

```html
<div class="experiencia-laboral"><!--Experienda laboral -->
    <div class="fecha"><!--Fecha de la experienda laboral -->
        2015 — 2017</div>
    <div><!--Datos de la experienda laboral -->
        <h4 class="titulo">Junior Designer at Creative Co&#8599;</h4>
        <p class="ubicacion">Atlanta, GA</p>
        <div class="imagenes-experiencia"><!--Imagenes -->
```

Los comentarios se usan para **etiquetar bloques** y que luego sea fácil encontrar cosas.

---

## 1.6 Anidamiento (etiquetas dentro de etiquetas)

Las etiquetas se **anidan** como las muñecas rusas. La que se abre, se cierra.

```html
<article class="tarjeta">
    <div class="imagen-proyecto imagen-1"></div>
    <div class="contenido-tarjeta">
        <h2>Pagina Web</h2>
        <p>Pagina web programada en HTML, CSS y JavaScript</p>
        <a href="https://github.com/..." class="enlace-proyecto">Ver proyecto</a>
    </div>
</article>
```

**Regla de oro:** el orden de cierre debe ser el **inverso** al de apertura.

```html
✅ <div><p><b>texto</b></p></div>
❌ <div><p><b>texto</p></b></div>
```

---

## 1.7 Etiquetas vacías (sin cierre)

Algunas etiquetas **no tienen contenido** y por eso no se cierran:

```html
<meta charset="UTF-8">
<link rel="stylesheet" href="css/styles.css">
<br>          <!-- salto de línea -->
<hr>          <!-- línea horizontal -->
<img src="foto.png" alt="foto">
```

---

## 1.8 Tipos de etiquetas — resumen visual

```
ETIQUETAS DE BLOQUE (ocupan todo el ancho, una debajo de otra)
  <h1> <h2> <p> <div> <ul> <ol> <li> <section> <article>
  <header> <footer> <nav> <main> <aside> <table> <form> <hr>

ETIQUETAS EN LÍNEA (fluyen con el texto, como una palabra)
  <a> <img> <span> <b> <i> <strong> <em> <time> <label> <select>
```

**Ejemplo de la diferencia:**

```html
<p>
    Me gusta <b>JavaScript</b> y también <span>CSS</span>.
    <!-- "b" y "span" no rompen el párrafo: están EN LÍNEA -->
</p>

<h1>Título</h1>
<p>Párrafo.</p>
<!-- "h1" y "p" son de BLOQUE: cada uno ocupa su propia línea -->
```

---

## ✅ Resumen rápido

1. Toda página empieza con `<!DOCTYPE html>` y termina con `</html>`.
2. Dentro hay `<head>` (invisible) y `<body>` (visible).
3. `charset=UTF-8` evita problemas con tildes y ñ.
4. `viewport` hace la página responsive.
5. Las etiquetas se anidan respetando el orden de cierre.
6. `<!-- -->` es un comentario, el navegador lo ignora.

👉 **Siguiente:** `02-etiquetas-de-texto-y-titulos.md`
