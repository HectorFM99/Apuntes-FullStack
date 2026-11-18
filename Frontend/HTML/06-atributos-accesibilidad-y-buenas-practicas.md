# 06 — ATRIBUTOS, ACCESIBILIDAD Y BUENAS PRÁCTICAS

---

## 6.1 ¿Qué es un atributo?

```html
<a href="https://google.com" target="_blank" rel="noopener noreferrer">Google</a>
   ─┬─  ─────────────────   ──┬──────   ──┬─────────────────────────
    │          │               │            │
  nombre    valor 1          valor 2      valor 3
```

- Van **dentro de la etiqueta de apertura**.
- Formato: `nombre="valor"`.
- Siempre entre **comillas dobles** (aunque en algunos casos HTML las tolera, mejor siempre).
- Las etiquetas vacías (`<img>`, `<br>`) solo llevan apertura, pero sus atributos van igual.

---

## 6.2 Atributos universales (sirven en CUALQUIER etiqueta)

| Atributo | Para qué sirve |
|----------|----------------|
| `class` | Nombre(es) de clase → lo usa **CSS** y **JavaScript**. Puede haber varios separados por espacio |
| `id` | Identificador **único** en toda la página → anclas, `label for`, JS `getElementById` |
| `style` | Estilos en línea (funciona, pero se prefiere el archivo CSS externo) |
| `title` | Tooltip al pasar el ratón |
| `data-*` | Datos **personalizados** guardados en el elemento (`data-color`, `data-origen`) |
| `hidden` | Oculta el elemento |
| `tabindex` | Orden de foco al pulsar `Tab` |
| `dir` | Dirección del texto: `ltr`, `rtl` |
| `lang` | Idioma del contenido de esa parte |

### `class` vs `id`

```html
<div class="tarjeta">...</div>       <!-- clase: se puede repetir -->
<div class="tarjeta destacada">...</div>  <!-- varias clases separadas por espacio -->

<div id="principal">...</div>        <!-- id: ÚNICO en toda la página -->
<a href="#principal">Saltar</a>      <!-- se usa como ancla -->
```

**CSS:**

```css
.tarjeta { ... }          /* .clase */
.destacada { ... }        /* .clase */
#principal { ... }        /* #id */
```

**Especifidad (quién manda):** `#id` (100) > `.clase` (10) > `etiqueta` (1).

### `data-*` — atributos personalizados

```html
<div class="imagen-proyecto imagen-1" role="img" aria-label="Vista previa del Proyecto 1"></div>
```

Y en los proyectos de JS (`varita-magica/varita-magica-hector/scripts.js`):

```js
if (!element.dataset.originalSrc) {
    element.dataset.originalSrc = element.getAttribute('src');
}
element.src = element.dataset.originalSrc;
```

> 📌 El HTML llevaba algo del palo `data-original-src="foto.png"` y en JS se lee como
> `element.dataset.originalSrc`. Es la forma estándar de **guardar información extra**
> en un elemento sin inventar atributos a medida.

---

## 6.3 Atributos de `<a>` — los importantes

```html
<a href="https://website.com" target="_blank" rel="noopener noreferrer">
    website.com
</a>
```

| Atributo | Cuándo usarlo |
|----------|---------------|
| `href` | Siempre (es el destino) |
| `target="_blank"` | Solo si **de verdad** quieres pestaña nueva |
| `rel="noopener noreferrer"` | **Obligatorio** junto a `target="_blank"` (seguridad) |
| `download` | Si el enlace debe descargar un archivo |
| `aria-label` | Si el enlace no tiene texto visible (solo una imagen) |

### Ejemplo real copiado de un proyecto

**Archivo:** `web-cv-hector/web-cv-hector/index.html`

```html
<a class="profile__website" href="https://website.com" target="_blank" rel="noopener noreferrer">website.com</a>
```

---

## 6.4 Atributos de `<img>` — los importantes

```html
<img src="assets/images/foto.png"
     alt="Descripción clara y útil de la foto"
     width="150"
     height="90"
     loading="lazy">
```

| Atributo | Para qué |
|----------|----------|
| `src` | Ruta/URL de la imagen |
| `alt` | **Obligatorio.** Texto si la imagen no carga o para lectores de pantalla |
| `width` / `height` | Tamaño; además **evita el salto de layout** mientras carga |
| `loading="lazy"` | Carga diferida (mejora LCP/velocidad) |
| `decoding="async"` | Decodificación asíncrona |
| `srcset` | Versiones distintas de la imagen según pantalla |

### Reglas de `alt`

```html
<!-- ✅ Imagen informativa: describe el CONTENIDO -->
<img src="assets/images/image-profile.png" alt="Sara Lawrence" width="92" height="93">

<!-- ✅ Imagen decorativa: alt VACÍO para que el lector la ignore -->
<img src="assets/images/divisor.png" alt="" role="presentation">

<!-- ✅ Imagen que es enlace: describe el DESTINO -->
<a href="proyectos.html"><img src="icono.png" alt="Ir a la página de proyectos"></a>

<!-- ❌ Mal: describe el archivo -->
<img src="foto.png" alt="foto">

<!-- ❌ Muy mal: sin alt -->
<img src="foto.png">
```

### Ejemplos reales copiados de un proyecto

**Archivo:** `web-cv-hector/web-cv-hector/index.html`

```html
<img class="profile__image" alt="Sara Lawrence" src="./assets/images/image-profile.png" width="92" height="93">
```

```html
<img src="./assets/images/design.png"
     alt="Abstract 3D composition of colorful geometric shapes" width="150" height="90">
```

**Archivo:** `git/Pagina-Web/index.html`

```html
<img src="assets/images/imagen-para-enlace.png" alt="Logo de la página">
```

---

## 6.5 Atributos ARIA (accesibilidad)

`ARIA` = **A**ccessible **R**ich **I**nternet **A**pplications.
Solo se usan **cuando la etiqueta semántica no basta**. Regla de oro:

> **"No uses ARIA si existe una etiqueta HTML que haga ese trabajo."**

| Atributo | Qué comunica |
|----------|--------------|
| `aria-label` | Nombre accesible cuando **no hay texto visible** |
| `aria-labelledby` | Apunta al `id` de otro elemento que hace de título |
| `aria-describedby` | Apunta a una descripción |
| `aria-hidden="true"` | **Oculta** este elemento a los lectores de pantalla (es decorativo) |
| `aria-expanded` | Indica si un menú/desplegable está abierto o cerrado |
| `aria-controls` | Indica qué elemento controla |
| `aria-current="page"` | "Este es el enlace de la página actual" |
| `role="..."` | Cambia el rol percibido del elemento |

### Ejemplos reales copiados de tus proyectos

**1) Botón de menú sin texto visible** — `git/Pagina-Web/index.html`:

```html
<button class="boton-hamburguesa" type="button" aria-label="Abrir menú">
    ☰
</button>
```

> 📌 El símbolo `☰` no lo interpreta bien todo el mundo, así que `aria-label`
> le dice al lector de pantalla: **"Abrir menú"**.

**2) Enlace que solo contiene una imagen** — `git/Pagina-Web/index.html`:

```html
<a class="logo-inicio" href="index.html" aria-label="Ir a la página principal">
    <img src="assets/images/imagen-para-enlace.png" alt="Logo de la página">
</a>
```

> 📌 Doble protección: `aria-label` en el `<a>` y `alt` en el `<img>`.

**3) Flecha decorativa** — `web-cv-hector/index.html`:

```html
<h3 class="work-experience__title">Senior Designer at Magic Design Co<span aria-hidden="true">↗</span></h3>
```

> 📌 La flecha es puro adorno → `aria-hidden="true"` evita que se lea "...co flecha diagonal arriba derecha".

**4) Div que hace de imagen** — `git/Pagina-Web/pages/proyectos.html`:

```html
<div class="imagen-proyecto imagen-1" role="img" aria-label="Vista previa del Proyecto 1"></div>
```

> 📌 Un `<div>` es invisible para lectores. Con `role="img"` + `aria-label` se le da
> el mismo comportamiento que tendría un `<img alt="...">`.

**5) Etiquetar varias navegaciones** — `git/Pagina-Web/index.html`:

```html
<nav aria-label="Enlaces legales">
    <a href="#">Aviso legal</a>
    <a href="#">Política de privacidad</a>
    <a href="#">Política de cookies</a>
</nav>
```

---

## 6.6 Atributos de rendimiento

```html
<!-- Conectar pronto con un dominio externo (Google Fonts) -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- Carga asíncrona de scripts -->
<script src="js/programacion.js" defer></script>
<script src="otro.js" defer></script>

<!-- Imágenes diferidas -->
<img src="foto.png" alt="foto" loading="lazy">

<!-- Precarga de un recurso crítico -->
<link rel="preload" as="image" href="hero.jpg">
```

| Estrategia de `<script>` | Cuándo se ejecuta |
|--------------------------|-------------------|
| `<script src="x.js">` (normal) | **Bloquea**: se para el HTML hasta cargarlo |
| `defer` | Se ejecuta **después** de parsear todo el HTML, en orden → **el recomendado** |
| `async` | Se ejecuta en cuanto descarga, **interrumpiendo** (solo para scripts independientes) |

**Ejemplo real copiado** (`git/Pagina-Web/index.html`):

```html
<script src="js/programacion.js"></script>
```

> 📌 Está al final del `<body>`, justo antes de `</body>`. Esa es la ubicación
> tradicional que garantiza que el HTML ya existe cuando JS busca los elementos.

---

## 6.7 Buenas prácticas (checklist)

### ✅ SÍ

```html
<!DOCTYPE html>
<html lang="es">                                  <!-- idioma declarado -->
<head>
    <meta charset="UTF-8">                        <!-- acentos y ñ ok -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  <!-- responsive -->
    <title>Página con sentido | MiWeb</title>     <!-- título único y descriptivo -->
    <meta name="description" content="Descripción para Google, máx. 160 caracteres">
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
    <a class="skip-link" href="#main">Saltar al contenido</a>
    <header>...</header>
    <main id="main">
        <h1>Un solo H1 por página</h1>
        <section>
            <h2>Subsección</h2>
            <img src="foto.png" alt="Útil" width="800" height="600" loading="lazy">
        </section>
    </main>
    <footer>...</footer>
    <script src="js/app.js" defer></script>
</body>
</html>
```

### ❌ NO

```html
<html>                                    <!-- sin lang -->
<meta charset="utf-7">                    <!-- codificación rara -->
<title>Página</title>                     <!-- genérico -->
<img src="img123.png">                    <!-- sin alt -->
<div class="div1"><div class="div2">      <!-- divitis: todo anidado sin semántica -->
<h1>Título</h1><h3>Salto de nivel</h3>    <!-- jerarquía saltada -->
<a href="#">clic aquí</a>                 <!-- texto no descriptivo -->
<br><br><br><br>                          <!-- saltos para maquetar -->
<p><div>x</div></p>                       <!-- div dentro de p -->
```

---

## 6.8 Validación: cómo comprobar tu HTML

1. **W3C Validator** → https://validator.w3c.org/ (pega el código o la URL).
2. **DevTools del navegador** → `F12` → pestaña *Elements* / *Console*.
3. **Lighthouse** (en Chrome: `F12` → pestaña *Lighthouse*) → puntuación de
   accesibilidad, SEO y buenas prácticas.

### Errores HTML más comunes (y cómo detectarlos)

| Error | Cómo se ve |
|-------|------------|
| Etiqueta sin cerrar | El resto de la página "desaparece" o se mete dentro de algo raro |
| `id` duplicado | El navegador solo aplica el primero |
| `alt` vacío en imagen informativa | Lighthouse lo marca en accesibilidad |
| `target="_blank"` sin `rel` | Lighthouse lo marca en seguridad |
| Imagen sin `width`/`height` | La página "salta" al cargar (CLS) |
| `<meta viewport>` ausente | En móvil se ve todo microscopic |

---

## ✅ Resumen rápido

1. `class` (repetible, CSS/JS) vs `id` (único, anclas/labels/JS).
2. `alt` **siempre** en `<img>`; vacío solo si es decorativa.
3. `target="_blank"` **siempre** con `rel="noopener noreferrer"`.
4. `aria-label` cuando no hay texto visible; `aria-hidden` cuando es decorativo;
   `role="img"` cuando un `<div>` hace de imagen.
5. `loading="lazy"` en imágenes y `<script defer>` al final.
6. `lang`, `charset`, `viewport`, `title` y `meta description` = base de toda página bien hecha.

👉 **Siguiente:** `07-ejemplos-completos-de-proyectos.md`
