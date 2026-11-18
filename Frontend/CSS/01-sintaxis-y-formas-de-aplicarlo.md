# 01 — SINTAXIS CSS Y CÓMO APLICARLO

---

## 1.1 ¿Qué es CSS?

**CSS** = **C**ascading **S**tyle **S**heets** (Hojas de Estilo en Cascada).

- **Cascading** → los estilos se "acumulan" y, si hay conflicto, gana uno por reglas de prioridad.
- **Style Sheet** → es un archivo aparte del HTML que solo contiene estilos.

```
HTML → QUÉ hay          (estructura)
CSS  → CÓMO SE VE       (diseño, colores, tamaños, posición)
JS   → QUÉ HACE         (comportamiento)
```

---

## 1.2 Anatomía de una regla CSS

```css
selector {
    propiedad: valor;
    propiedad: valor;
}
```

```css
/* Selector */        /* Propiedad */ /* Valor */
p                     { color         : red;  }
.clase-mia            { font-size     : 20px; }
#mi-id                { background    : #f4f4f4; }
```

**Partes:**
| Parte | Ejemplo | Qué es |
|-------|---------|--------|
| **Selector** | `p`, `.texto`, `#principal` | A **quién** se aplica |
| **Declaración** | `color: red;` | Un par propiedad + valor |
| **Propiedad** | `color` | Qué aspecto cambia |
| **Valor** | `red`, `20px`, `#f4f4f4` | Cómo queda |
| **Bloque** `{ ... }` | | Agrupa varias declaraciones |

> ⚠️ **¡Cada declaración termina en `;`!** Si olvidas uno, a veces "se come" la siguiente.

> ⚠️ **Las llaves `{}` y el `;` son fijos.** En JS son `()` y `;`, en CSS `{}` y `;`.

---

## 1.3 Los 3 lugares donde puede ir el CSS

### Opción A — En línea (inline): atributo `style`

```html
<p style="color: red; font-size: 20px;">Texto en rojo</p>
```

| ✅ Ventajas | ❌ Inconvenientes |
|------------|-------------------|
| Rápido para probar | Difícil de mantener |
| Máxima especificidad (gana siempre) | Mezcla diseño con estructura |
| | No se puede reutilizar |

**Ejemplo real copiado** (`varita-magica/varita-magica-hector/index.html`):

```html
<table style="border-spacing:8px">...</table>
<table style="width: 217px; display: none; top: 80px; position: absolute; left: 1139px;" class="gstl_50 gssb_c" cellspacing="0" cellpadding="0">...</table>
```

### Opción B — En el `<head>` (interno): etiqueta `<style>`

```html
<head>
    <style>
        h1 {
            color: navy;
            font-size: 32px;
        }
        .destacado {
            background: yellow;
        }
    </style>
</head>
```

| ✅ Ventajas | ❌ Inconvenientes |
|------------|-------------------|
| Todo en un archivo | Solo sirve para **una** página |
| No hay que enlazar nada | Mezcla CSS con HTML |

### Opción C — Archivo externo (el recomendado) ✅

**`css/styles.css`**
```css
h1 {
    color: navy;
}
```

**`index.html`**
```html
<head>
    <link rel="stylesheet" href="css/styles.css">
</head>
```

| ✅ Ventajas | ❌ Inconvenientes |
|------------|-------------------|
| Un solo CSS para **todo** el sitio | Hay que enlazarlo en cada página |
| Cacheo: el navegador lo guarda | Ruta mal escrita = sin estilos |
| HTML y CSS separados y limpios | |

---

## 1.4 Ejemplos REALES de enlace CSS en tus proyectos

**Archivo:** `web-cv-hector/web-cv-hector/index.html`

```html
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mi CV</title>
    <link rel="stylesheet" href="./css/styles.css">
</head>
```

**Archivo:** `git/Pagina-Web/index.html` (raíz del proyecto)

```html
<link rel="stylesheet" href="css/cuerpo.css">
```

**Archivo:** `git/Pagina-Web/pages/sobre_mi.html` (dentro de `pages/`)

```html
<link rel="stylesheet" href="../css/cuerpo.css">
```

> 📌 **La diferencia es la ruta:**
> - Desde la raíz: `css/cuerpo.css`
> - Desde una subcarpeta: `../css/cuerpo.css` (el `../` sube un nivel)

**Archivo:** `proyecto/main.html` (intento con ruta en la raíz)

```html
<html lang="en">
    <link rel="stylesheet" href="cuerpo.css">
<head>
```

> 📌 Este `<link>` está **fuera de `<head>`**, antes de abrirlo. Funciona en la
> mayoría de navegadores (lo "adivinan"), pero **lo correcto** es:
> ```html
> <head>
>     <meta charset="UTF-8">
>     <title>Pagina Personal</title>
>     <link rel="stylesheet" href="cuerpo.css">
> </head>
> ```

---

## 1.5 El orden de carga y la "C" de Cascading

Cuando dos reglas chocan, el navegador decide así:

```
1. IMPORTANCIA     !important en el valor  →  gana (pero úsalo poco)
2. ESPECIFICIDAD   #id  >  .clase  >  etiqueta
3. ORDEN           Si empatan, gana la que venga MÁS ABAJO en el archivo CSS
```

```css
p { color: blue; }        /* etiqueta: especificidad 1 */
p { color: red; }         /* misma especificidad → gana ESTA (va después) */
```

Ver detalle completo en `02-selectores-y-especificidad.md`.

---

## 1.6 Comentarios en CSS

```css
/* Este es un comentario de una línea */

/*
   Este es un comentario
   que ocupa varias líneas
*/
```

> ⚠️ En CSS **no** existen comentarios de `//`. Si pones `//` el navegador ignora
> esa línea entera y puede romper la regla siguiente.

### Ejemplo real copiado de un proyecto

**Archivo:** `git/Pagina-Web/css/cuerpo.css`

```css
/* Cambia estas rutas por las de tus imágenes */
.imagen-barra-superior {
  z-index: 10;
  display: flex;
  ...
}

/* Barra de arriba */
.barra-superior {
  background-color: #222;
  padding: 12px;
}

/* Panel lateral: inicialmente fuera de la pantalla */
.menu-lateral {
  z-index: 20;
  position: fixed;
  top: 0;
  left: -250px;
  ...
}
```

---

## 1.7 Estructura recomendada de un archivo CSS

El orden en que escribes las reglas **importa** (por la regla 3 de desempate).
Convención habitual:

```css
/* ============================================
   1. RESET / NORMALIZACIÓN
   ============================================ */
*, *::before, *::after { box-sizing: border-box; }
body { margin: 0; }

/* ============================================
   2. VARIABLES
   ============================================ */
:root {
  --color-primario: #e63946;
}

/* ============================================
   3. TIPOGRAFÍA BÁSICA
   ============================================ */
body { font-family: "Poppins", sans-serif; }

/* ============================================
   4. LAYOUT (flex / grid)
   ============================================ */
.container { display: flex; }

/* ============================================
   5. COMPONENTES
   ============================================ */
.boton { ... }

/* ============================================
   6. PÁGINAS CONCRETAS
   ============================================ */

/* ============================================
   7. RESPONSIVE (@media al FINAL)
   ============================================ */
@media (min-width: 768px) { ... }
```

> 📌 **Las `@media` siempre al final.** Así, al escribir la última, tienen acceso a
> todo lo anterior y pueden sobreescribirlo con facilidad.

---

## 1.8 El reset: el primer bloque de casi todo CSS

Los navegadores dan estilos por defecto (márgenes en `body`/`p`, tamaños distintos...).
Para partir de "cero" se usa un **reset**.

**Ejemplo real copiado de `web-minimalista-hector/.../css/styles.css`:**

```css
/* ---------- Reset ---------- */
*,
*::before,
*::after {
  box-sizing: border-box;
}

body,
h1,
h2,
p,
ul,
li,
figure,
a {
  margin: 0;
  padding: 0;
}

ul {
  list-style: none;
}

a {
  color: inherit;
  text-decoration: none;
}

img {
  display: block;
}

body {
  font-family: "Source Serif Pro", sans-serif;
  color: #000;
  background: #fff;
}
```

**Ejemplo real copiado de `web-cv-hector/.../css/styles.css`:**

```css
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
```

> 📌 `*` selecciona **todos** los elementos. Es el selector más "barato" de escribir
> pero el que menos especificidad tiene.

---

## 1.9 Unidades que vas a ver todo el rato

| Unidad | Significado | Ejemplo típico |
|--------|-------------|----------------|
| `px` | Píxeles fijos | `font-size: 16px`, `border: 1px` |
| `%` | Porcentaje **del padre** | `width: 50%` |
| `rem` | Relativo al `font-size` de `<html>` | `font-size: 1.25rem` |
| `em` | Relativo al `font-size` del **propio elemento** | `padding: 1em` |
| `vh` | `%` de la **altura** de la ventana | `min-height: 100vh` |
| `vw` | `%` del **ancho** de la ventana | `width: 90vw` |
| `vmin` / `vmax` | menor / mayor dimensión de la ventana | |
| `ch` | Ancho del carácter "0" | `max-width: 60ch` (≈ 60 caracteres) |
| `fr` | "Fraction" → espacio libre repartido (solo en **Grid**) | `1fr 2fr` |
| `s`, `ms` | Tiempos en `transition` y `animation` | `0.3s`, `1800ms` |

**Regla práctica:**
- Para **tamaños de fuente** → usa `rem` (respeta la configuración del usuario).
- Para **anchos de cajas** → `%, rem, vw, fr`.
- Para **bordes y sombras finas** → `px`.
- **Nunca** inventes `font-size: 5vw` para párrafos largos: dificulta la lectura.

---

## ✅ Resumen rápido

1. Regla = `selector { propiedad: valor; }` y **cada declaración termina en `;`**.
2. Formas de aplicar CSS: `style="..."` (inline), `<style>` (interno), `<link>` (externo ✅).
3. El CSS externo se enlaza con `<link rel="stylesheet" href="ruta">`.
4. Rutas: `css/archivo.css` desde la raíz, `../css/archivo.css` desde una subcarpeta.
5. Desempate: `!important` > especificidad (`#` > `.` > etiqueta) > orden (gana la última).
6. Empieza siempre con un **reset** y pon las `@media` **al final**.

👉 **Siguiente:** `02-selectores-y-especificidad.md`
