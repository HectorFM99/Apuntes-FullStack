# 02 — ETIQUETAS DE TEXTO Y TÍTULOS

---

## 2.1 Títulos: `<h1>` … `<h6>`

HTML tiene **seis niveles de título**, del más importante (`h1`) al menos (`h6`).

```html
<h1>Título de nivel 1 (el más grande y importante)</h1>
<h2>Título de nivel 2</h2>
<h3>Título de nivel 3</h3>
<h4>Título de nivel 4</h4>
<h5>Título de nivel 5</h5>
<h6>Título de nivel 6 (el más pequeño)</h6>
```

**Reglas de uso:**
- En toda página debería haber **un solo `<h1>`** (el tema principal).
- No se "brincan" niveles: lo correcto es `h1 → h2 → h3`.
- El tamaño real lo decide **CSS**, no la etiqueta. La etiqueta indica **jerarquía/orden**, no tamaño.
- Los buscadores (SEO) y los lectores de pantalla usan esta jerarquía.

### Ejemplo real copiado de un proyecto

**Archivo:** `proyecto/ejercicio/ejercicio.html`

```html
<h1>Ejercicio de HTML</h1>
<h2>Subtítulo de ejemplo</h2>
<h3>Otro subtítulo</h3>
<h4>Subtítulo más pequeño</h4>
<h5>Subtítulo aún más pequeño</h5>
<h6>Subtítulo más pequeño de todos</h6>
```

> 📌 Fíjate: en pantalla `h1` sale enorme y `h6` casi diminuto. Ese es el comportamiento
> por defecto del navegador. Si en CSS pones `h1 { font-size: 20px }`, ese tamaño manda.

---

## 2.2 Párrafos: `<p>`

```html
<p>Este es un párrafo de texto.</p>
<p>Este es otro párrafo. Entre párrafos el navegador mete un espacio automáticamente.</p>
```

### Ejemplo real copiado de un proyecto

**Archivo:** `proyecto/main.html`

```html
<h1>Bienvenidos a mi pagina personal</h1>
<p>Hola, mi nombre es Hector Franco Magallo y esta es mi pagina personal.</p>
<div>   
    <p class="texto">En esta pagina podras encontrar informacion sobre mi, mis intereses y mis proyectos.</p>
    <a href="proyectos.html">Proyectos</a>
</div>
```

**Archivo:** `git/Pagina-Web/index.html`

```html
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
```

> 📌 `class="centrado"` es la clase que en CSS hace `text-align: center`.
> `class="reveal"` es la clase que JavaScript usa para animar la aparición del texto.

---

## 2.3 Saltos y separadores: `<br>` y `<hr>`

```html
<p>
  Primera línea<br>
  Segunda línea (el br fuerza el salto sin crear párrafo nuevo)<br>
  Tercera línea
</p>

<hr>   <!-- línea horizontal que separa secciones -->
```

- `<br>` = **salto de línea** dentro del mismo párrafo.
- `<hr>` = **separador temático**, dibuja una línea.

> ⚠️ `<br>` debe usarse poco. Para separar bloques de contenido usa `<p>`, no muchos `<br>`.

### Ejemplo real copiado de un proyecto

**Archivo:** `proyecto/ejercicio/ejercicio.html`

```html
<br>
<br>
<br>
<br>
<img src="https://cdn.pixabay.com/photo/2015/06/24/15/45/code-820275_960_720.jpg"
     alt="Imagen de código" width="520" height="320">
```

> 📌 Aquí los `<br>` se usan para "bajar" la imagen. Es una forma rápida pero poco
> profesional: en CSS se hace con `margin`.

**Archivo:** `git/Pagina-Web/index.html`

```html
<hr class="reveal">
```

Con CSS (`git/Pagina-Web/css/cuerpo.css`) ese `<hr>` queda así:

```css
hr {
  width: 90%;
  margin: 20px auto;
  border: 0;
  border-top: 10px solid #000000;
}
```

---

## 2.4 Texto con significado especial

| Etiqueta | Qué hace | Ejemplo |
|----------|----------|---------|
| `<strong>` | Importancia **fuerte** (negrita + semántica) | `<strong>Alerta</strong>` |
| `<b>` | Negrita **sin** importancia semántica | `<b>negrita</b>` |
| `<em>` | Énfasis / énfasis vocal (cursiva + semántica) | `<em>esto es importante</em>` |
| `<i>` | Cursiva decorativa (idiomas, nombres técnicos) | `<i>italic</i>` |
| `<mark>` | Resalta con amarillo | `<mark>buscado</mark>` |
| `<small>` | Texto pequeño (copyright, letras pequeñas) | `<small>© 2026</small>` |
| `<del>` | Texto eliminado (tachado) | `<del>precio viejo</del>` |
| `<ins>` | Texto insertado (subrayado) | `<ins>precio nuevo</ins>` |
| `<code>` | Texto de programación | `<code>console.log()</code>` |
| `<abbr>` | Acrónimo | `<abbr title="HyperText Markup Language">HTML</abbr>` |
| `<sub>` / `<sup>` | Subíndice / Superíndice | `H<sub>2</sub>O`, `x<sup>2</sup>` |

```html
<p>
  <strong>Importante:</strong> recuerda cerrar las etiquetas.
  El precio era de <del>100€</del> <ins>80€</ins>.
  En este <mark>trecho</mark> hay resaltado.
</p>
```

---

## 2.5 Espacios en blanco y entidades HTML

El navegador **colapsa** los espacios: 10 espacios seguidos = 1 espacio.
Y ciertos caracteres tienen significado especial, así que se escriben con **entidades**:

| Qué quiero escribir | Cómo se escribe |
|---------------------|-----------------|
| `&` (ampersand) | `&amp;` |
| `<` menor que | `&lt;` |
| `>` mayor que | `&gt;` |
| espacio no separable | `&nbsp;` |
| © copyright | `&copy;` |
| flecha ↗ | `&#8599;` |
| ☰ (menú hamburguesa) | directamente `☰` |
| comilla « | `&laquo;` |

### Ejemplo real copiado de un proyecto

**Archivo:** `ejercicio-pagina web/pagina web/index.html`

```html
<h4 class="titulo">Senior Designer at Magic Design Co...&#8599;</h4>
```

`&#8599;` es el código numérico del carácter **↗** (flecha diagonal arriba-derecha).

**Archivo:** `git/Pagina-Web/index.html`

```html
<button class="boton-hamburguesa" type="button" aria-label="Abrir menú">
    ☰
</button>
```

---

## 2.6 Elementos de contenido "meta": `<time>`, `<figure>`, `<blockquote>`

### `<time>` — una fecha con significado

```html
<time datetime="2017">2017</time>
<time datetime="2026-10-19">19 de octubre de 2026</time>
```

El atributo `datetime` da la fecha **en formato máquina** (para buscadores) y lo de
dentro es lo que **se ve** la persona.

**Ejemplo real copiado de `web-cv-hector/web-cv-hector/index.html`:**

```html
<article class="work-experience">
    <div class="work-experience__date"><time datetime="2017">2017</time> — Now</div>
    <div class="work-experience__details">
        <h3 class="work-experience__title">Senior Designer at Magic Design Co<span aria-hidden="true">↗</span></h3>
        <p class="work-experience__location">San Francisco, CA</p>
    </div>
</article>
```

### `<figure>` y `<figcaption>` — imagen con pie de foto

**Ejemplo real copiado de `web-minimalista-hector/.../about.html`:**

```html
<figure class="now__figure">
    <img class="now__photo" src="img/about-portrait.jpg" alt="portrait of woman" loading="lazy">
</figure>
```

### `<blockquote>` — cita larga / `<q>` — cita corta

```html
<blockquote cite="https://ejemplo.com">
    <p>La práctica hace al maestro.</p>
</blockquote>

<p>Como dijo él: <q>la práctica hace al maestro</q>.</p>
```

---

## 2.7 Texto preformateado: `<pre>`

Mantiene **tal cual** los espacios, tabulaciones y saltos de línea.

```html
<pre>
function suma(a, b) {
    return a + b;
}
</pre>
```

---

## 2.8 `<span>` — el contenedor en línea

`<span>` es el hermano en línea de `<div>`: no hace nada por sí mismo, sirve para
**envolver una parte del texto** y poder darle estilo con CSS o cogerla con JavaScript.

```html
<p>
  El precio es <span class="oferta">29,99 €</span> en lugar de 49,99 €.
</p>
```

```css
.oferta {
  color: red;
  font-weight: bold;
}
```

### Ejemplo real copiado de un proyecto

**Archivo:** `web-cv-hector/web-cv-hector/index.html`

```html
<h3 class="work-experience__title">Senior Designer at Magic Design Co<span aria-hidden="true">↗</span></h3>
```

> 📌 `aria-hidden="true"` le dice al lector de pantalla: "esta flecha es decorativa,
> no la leas". Así el lector solo oye "Senior Designer at Magic Design Co".

---

## ✅ Resumen rápido

| Necesidad | Etiqueta |
|-----------|----------|
| Título principal | `<h1>` (uno por página) |
| Subtítulos | `<h2>`, `<h3>`... |
| Párrafo | `<p>` |
| Salto de línea | `<br>` |
| Línea separadora | `<hr>` |
| Énfasis fuerte | `<strong>` / débil `<em>` |
| Resaltar texto | `<mark>` |
| Espacio literal / código raro | Entidades (`&amp;`, `&#8599;`) |
| Envolver texto para darle estilo | `<span>` |
| Fecha con significado | `<time datetime="...">` |

👉 **Siguiente:** `03-enlaces-listas-y-multimedia.md`
