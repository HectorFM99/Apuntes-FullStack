# 08 — DISEÑO RESPONSIVE Y MEDIA QUERIES

**Responsive** = la página se adapta a **cualquier tamaño de pantalla**
(móvil, tablet, portátil, TV) sin necesidad de versiones distintas.

---

## 8.1 Base imprescindible: el `<meta viewport>`

Sin esta línea en el HTML, el responsive **no funciona** en móvil:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

| Fragmento | Significado |
|-----------|-------------|
| `width=device-width` | El ancho de la página = el ancho del dispositivo |
| `initial-scale=1.0` | Zoom inicial al 100% (sin zoom forzado) |

**Extras útiles:**

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5.0">
<meta name="theme-color" content="#e63946">   <!-- color de la barra del navegador móvil -->
```

---

## 8.2 Estrategia: **Mobile First** ✅

```css
/* 1. PRIMERO: diseño para MÓVIL (lo mínimo, todo en una columna) */
.caja {
  display: flex;
  flex-direction: column;
  padding: 16px;
}

/* 2. DESPUÉS: añades mejoras para pantallas GRANDES */
@media (min-width: 768px) {
  .caja {
    flex-direction: row;
    padding: 40px;
  }
}
```

**¿Por qué `min-width` y no `max-width`?**
- `min-width: 768px` → "a partir de 768px aplica esto". **Mobile First**: el CSS
  base es el móvil y cada `@media` **añade**.
- `max-width: 767px` → "hasta 767px aplica esto". Obliga a escribir primero el
  diseño de escritorio y luego "desarmarlo".

> 📌 **Ventajas de Mobile First:** menos código, mejor rendimiento en móvil
> (descarga menos CSS), y es lo que exige Google (índice móvil).

---

## 8.3 Sintaxis de `@media`

```css
@media (condición) {
  .selector { ... }
}
```

### Por ANCHO de pantalla

```css
/* Solo pantallas MEDIANAS o mayores */
@media (min-width: 768px) { ... }

/* Solo pantallas GRANDES o mayores */
@media (min-width: 1000px) { ... }
```

### Los puntos de ruptura (breakpoints) habituales

```
0 ──────── 480 ──────── 768 ──────── 1024 ──────── 1280 ───►
   móvil       tablet S      tablet/L       escritorio    widescreen
```

| Nombre | Rango | `@media` típico |
|--------|-------|-----------------|
| Móvil pequeño | < 480px | `max-width: 479px` |
| Móvil | 480–767px | `max-width: 767px` |
| **Tablet** | **768–1023px** | **`min-width: 768px`** |
| Escritorio | ≥ 1024px | `min-width: 1024px` |
| Grande | ≥ 1280px | `min-width: 1280px` |

> ⚠️ **Los breakpoints no son universales.** Se eligen mirando **cuándo se rompe tu
> diseño**, no siguiendo una tabla de dogma. `768px` y `1024px` son los más usados
> porque coinciden con tablets en horizontal y verticales.

### Otras condiciones

```css
@media (orientation: portrait)  { }   /* vertical */
@media (orientation: landscape) { }   /* horizontal */
@media (prefers-color-scheme: dark) { }  /* tema oscuro del sistema */
@media (prefers-reduced-motion: reduce) { }  /* usuario que pide menos animación */
@media (hover: hover) { }             /* dispositivo con ratón */
@media print { }                      /* al imprimir */
```

### Combinar condiciones (se leen como "Y")

```css
@media (min-width: 768px) and (max-width: 1023px) { }   /* solo tablet */
@media (min-width: 1024px) and (orientation: landscape) { }
```

### Varios breakpoints

```css
/* Base = móvil */

@media (min-width: 768px)  { /* tablet */ }
@media (min-width: 1024px) { /* escritorio */ }
```

---

## 8.4 Ejemplos REALES copiados de tus proyectos

### 1) `web-cv-hector/web-cv-hector/css/styles.css` — el final del archivo

```css
.images-experience {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin-top: 1.5rem;
}

@media (min-width: 768px) {
  .images-experience {
    width: 100%;
    flex-direction: row;
    align-items: center;
  }
}
```

> 📌 **Móvil:** las 3 imágenes se apilan **en columna**.
> **Tablet en adelante (`≥768px`):** pasan a **fila**.
> **Cambia solo `flex-direction`.** El HTML no se toca.

---

### 2) `web-minimalista-hector/.../css/styles.css` — varios breakpoints

```css
/* ...estilos base (móvil)... */

@media (min-width: 800px) {
  ...
    flex-direction: row;
  ...
    flex-direction: row;
    align-items: flex-start;
  ...
    flex: 1 0 0;
  ...
}

@media (min-width: 1000px) {
  ...
    flex-direction: row;
  ...
    flex: 1 0 0;
  ...
    flex-direction: column;
  ...
}
```

> 📌 Dos puntos de ruptura: **800px** y **1000px**. Cada uno **añade** sobre el
> diseño base de móvil. Fíjate en `flex: 1 0 0` → crece, no encoge, base 0
> (columnas iguales).

---

### 3) `git/Pagina-Web/css/cuerpo.css` — Grid responsive SIN media query

```css
.proyectos {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 24px;
  max-width: 1100px;
  margin: 40px auto;
  margin-top: -100px;
}
```

```css
.lista-certificados {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 280px), 1fr));
  gap: 24px;
  max-width: 900px;
  margin: 30px auto;
}
```

> 📌 **`auto-fit` + `minmax()` hace el trabajo de media queries:**
> - Pantalla de 300px → 1 columna (por `min(100%, 280px)`).
> - Pantalla de 600px → 2 columnas.
> - Pantalla de 900px → 3 columnas.
>
> Esto es **responsive declarativo**: el navegador calcula solo.

---

### 4) `web-minimalista-hector/.../css/styles.css` — menú que se adapta

```css
.menu {
  display: flex;
  flex-direction: column;      /* MÓVIL: menú en vertical */
  align-items: flex-end;
  gap: 30px;
  line-height: 1.03;
}

@media (min-width: 800px) {
  .menu {
    flex-direction: row;       /* GRANDE: menú en horizontal */
    align-items: center;
    gap: 40px;
  }
}
```

---

## 8.5 Unidades fluidas (complemento al responsive)

```css
/* ❌ Rígido */
.contenedor { width: 1000px; }

/* ✅ Fluidos */
.contenedor { width: 90%; max-width: 1000px; }   /* 90% hasta un tope */
.contenedor { width: min(90%, 1000px); }          /* igual, forma moderna */

/* ✅ Texto que se adapta sin perder legibilidad */
.titulo {
  font-size: clamp(1.5rem, 4vw, 3rem);
}
```

### `clamp(MIN, PREFERIDO, MÁX)`

```css
font-size: clamp(16px, 2.5vw, 24px);
/*         mínimo   ideal   máximo  */
```

Nunca baja de 16px, nunca sube de 24px, y en medio crece con la pantalla.
**Sustituye a muchas `@media` de tipografía.**

### Otras funciones útiles

```css
width: min(90%, 1000px);      /* el MENOR de los dos */
width: max(300px, 40%);       /* el MAYOR de los dos */
width: calc(100% - 40px);     /* aritmética: 100% menos 40px */
padding: max(16px, 4vw);      /* padding que nunca baja de 16px */
gap: clamp(12px, 3vw, 40px);
```

### Ejemplo real de `max()` y `calc()` en tus proyectos

`web-minimalista-hector/.../css/styles.css`:

```css
.brand__avatar {
  width: 40.169px;
  height: 40px;
}

.intro {
  width: 100%;
  max-width: 1500px;
  padding: 80px 0;
}
```

`git/Pagina-Web/css/cuerpo.css`:

```css
.contenido {
  max-width: 900px;
  margin: 40px auto 100px;
  padding: 30px;
  box-sizing: border-box;
}
```

---

## 8.6 Imágenes responsivas

```css
/* ❌ La imagen se sale o se deforma */
img { width: 100%; height: auto; }

/* ✅ Recorta sin deformar y llena su caja */
img {
  width: 100%;
  height: 200px;
  object-fit: cover;      /* recorta | contain = cabe entera */
  display: block;
}
```

**Ejemplo real** (`web-cv-hector/css/styles.css`):

```css
img {
  display: block;
  max-width: 100%;     /* NUNCA más ancha que su contenedor */
  height: auto;
}
```

Y en HTML, imágenes con `srcset` para móviles:

```html
<img src="foto-800.jpg"
     srcset="foto-400.jpg 400w, foto-800.jpg 800w, foto-1600.jpg 1600w"
     sizes="(max-width: 600px) 100vw, 50vw"
     alt="Descripción"
     loading="lazy">
```

---

## 8.7 Patrones responsive habituales

### 1) Fila que pasa a columna

```css
.tarjeta {
  display: flex;
  flex-direction: column;             /* móvil */
}
@media (min-width: 768px) {
  .tarjeta { flex-direction: row; }   /* escritorio */
}
```

### 2) Rejilla que se multiplica

```css
.rejilla {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 260px), 1fr));
  gap: 24px;
}
```

### 3) Menú hamburguesa (oculto en móvil)

```css
.menu { display: none; }
.menu.abierto { display: block; }

@media (min-width: 900px) {
  .boton-hamburguesa { display: none; }   /* en grande no hace falta */
  .menu { display: flex; }
}
```

### 4) Texto que escala

```css
h1 { font-size: clamp(1.75rem, 5vw, 3rem); }
p  { font-size: clamp(1rem, 2.5vw, 1.125rem); }
```

### 5) Evitar scroll horizontal

```css
html, body { overflow-x: hidden; }
img, video, iframe { max-width: 100%; }
```

---

## 8.8 Cómo probar el responsive

1. **DevTools**: `F12` → icono de móvil/tablet (o `Ctrl+Shift+M`).
2. **Arrastrar** el borde de la ventana del navegador.
3. **Lighthouse** → pestaña *Emulated Form Factor: Mobile*.
4. Probar con un **dispositivo real** (el emulador no detecta todos los problemas).

### Errores típicos de responsive

| Error | Causa | Solución |
|-------|--------|----------|
| Se sale de pantalla en móvil | Anchura fija en `px` | `width: 100%` / `max-width` |
| La media query "no hace nada" | Faltaba el `<meta viewport>` | Añadirlo al `<head>` |
| Texto diminuto | `font-size` en `px` | Usar `rem` o `clamp()` |
| Imagen deformada | `width` + `height` sin `object-fit` | `object-fit: cover` |
| Menú roto | `overflow` visible | `overflow: hidden` donde toque |
| Toca dos veces para clicar | Elementos < 44px | Mínimo táctil de ~44px |

---

## ✅ Resumen rápido

1. **Sin `<meta name="viewport">` no hay responsive.**
2. **Mobile First**: escribe base para móvil y usa `@media (min-width: ...)` para crecer.
3. Breakpoints habituales: **768px** (tablet) y **1024px** (escritorio).
4. `repeat(auto-fit, minmax(min(100%, 240px), 1fr))` → rejilla responsive sin `@media`.
5. `clamp(16px, 4vw, 32px)` → tipografía fluida sin `@media`.
6. `max-width` + `margin: 0 auto` + `width: 90%` = contenedor centrado y seguro.
7. `object-fit: cover` para imágenes dentro de cajas fijas.

👉 **Siguiente:** `09-variables-animaciones-y-keyframes.md`
