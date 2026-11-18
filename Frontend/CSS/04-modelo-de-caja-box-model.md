# 04 — MODELO DE CAJA (BOX MODEL)

> Todo en CSS es una **caja**. Entender el modelo de caja es entender el 50% de CSS.

---

## 4.1 Las capas de una caja

```
        ┌─────────────────────────────────────────┐
        │                MARGIN                   │  ← espacio HACIA FUERA (transparente)
        │   ┌─────────────────────────────────┐   │
        │   │            BORDER               │   │  ← borde visible
        │   │   ┌─────────────────────────┐   │   │
        │   │   │        PADDING          │   │   │  ← relleno: espacio DENTRO del borde
        │   │   │   ┌─────────────────┐   │   │   │
        │   │   │   │    CONTENIDO    │   │   │   │  ← el texto / imagen / hijo
        │   │   │   │                 │   │   │   │
        │   │   │   └─────────────────┘   │   │   │
        │   │   └─────────────────────────┘   │   │
        │   └─────────────────────────────────┘   │
        └─────────────────────────────────────────┘

   margen    → no se pinta, separa cajas
   borde    → lo que se ve alrededor
   padding  → "aire" entre el borde y el contenido
   contenido→ lo de dentro
```

---

## 4.2 Propiedades

```css
.caja {
  /* CONTENIDO */
  width: 300px;
  height: 200px;

  /* PADDING (relleno interior) */
  padding: 20px;                    /* los 4 lados */
  padding: 10px 20px;               /* vertical horizontal */
  padding: 10px 20px 30px 40px;     /* arriba derecha abajo izquierda (reloj) */
  padding-top: 10px;
  padding-inline: 1.5rem;           /* lógico: izquierda/derecha según idioma */
  padding-block: 2rem;              /* lógico: arriba/abajo */

  /* BORDE */
  border: 1px solid #e5e5e5;        /* grosor estilo color */
  border-radius: 10px;              /* esquinas redondeadas */
  border-top: 2px dashed red;

  /* MARGEN (exterior) */
  margin: 20px;
  margin: 40px auto;                /* vertical auto | horizontal auto → CENTRA */
  margin-top: -100px;               /* NEGATIVO: sube el elemento */
  margin-inline: auto;
}
```

### Estilos de borde disponibles

```
none | hidden | dotted | dashed | solid | double | groove | ridge | inset | outset
```

### Centrado horizontal con `margin: auto`

```css
.container {
  width: 900px;
  margin: 0 auto;   /* arriba-abajo 0, lados auto → centrado */
}
```

Funciona porque `auto` **reparte el sobrante** a izquierda y derecha por igual.
Solo funciona en elementos **de bloque** con **ancho definido**.

---

## 4.3 `box-sizing` — LA propiedad crítica

### Por defecto: `content-box`

```css
width: 300px;
padding: 20px;
border: 5px solid;
/* ANCHO REAL = 300 + 20 + 20 + 5 + 5 = 350 px  😱 */
```

### Con `border-box` (lo que se usa siempre)

```css
width: 300px;
padding: 20px;
border: 5px solid;
/* ANCHO REAL = 300 px  ✅  el padding y el borde CABEN dentro del width */
```

### El reset estándar

**Ejemplo real copiado de `web-minimalista-hector/.../css/styles.css`:**

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

**Ejemplo real copiado de `web-cv-hector/.../css/styles.css`:**

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
```

> 📌 **Ponlo siempre al principio de tu CSS.** Si no, cada vez que sumes `padding`
> a un elemento con `width` fijo, se te saldrá de la caja.

---

## 4.4 `margin` vs `padding` — ¿cuándo uso uno u otro?

| | `margin` | `padding` |
|---|---|---|
| ¿Dónde? | **Fuera** de la caja | **Dentro** de la caja |
| ¿Se ve? | No (transparente) | Sí, si hay fondo |
| ¿Colapsa? | Sí: dos márgenes contiguos se **suman** (se queda el mayor) | No |
| ¿Se colorea? | No | Sí (`background-color`) |
| Uso típico | Separar bloques, centrar con `auto` | Dar aire **al contenido**, hacer botones |

```css
/* margin → separar secciones */
.section {
  margin-top: 3rem;
}

/* padding → hacer un botón "gordo" */
.boton {
  padding: 10px 16px;
  background: #e63946;
}
```

> ⚠️ **Colapso de márgenes:** si dos `<p>` seguidos tienen `margin-bottom: 20px` y
> `margin-top: 20px`, el hueco real será **20px**, no 40px (gana el mayor).
> Solo ocurre **en vertical** y entre elementos de bloque hermanos.

---

## 4.5 Ejemplos REALES copiados de tus proyectos

### `git/Pagina-Web/css/cuerpo.css`

```css
html,
body {
  min-height: 100%;
  margin: 0;              /* anula el margen por defecto de <body> */
}

p {
  font-family: "Roboto", sans-serif;
  margin-left: 25px;
  margin-right: 25px;
  color: white;
}

hr {
  width: 90%;
  margin: 20px auto;      /* 20px arriba/abajo + centrado horizontal */
  border: 0;              /* quita el borde por defecto del <hr> */
  border-top: 10px solid #000000;   /* y lo vuelve a poner a su estilo */
}

.contenido {
  max-width: 900px;
  margin: 40px auto 100px;   /* arriba 40 | lados auto | abajo 100 */
  padding: 30px;
  background-color: transparent;
  border-radius: 0px;
  box-sizing: border-box;
  box-shadow: none;
  position: relative;
  top: -100px;                /* margen negativo equivalente: SUBE el bloque */
}
```

> 📌 **`margin: 40px auto 100px`** → shorthand de 3 valores:
> arriba 40px, izquierda/derecha `auto` (centra), abajo 100px.

```css
.proyectos {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 24px;
  max-width: 1100px;
  margin: 40px auto;
  padding: 0 -25px;
  margin-top: -100px;      /* margin LATERAL a margin-top: gana el último (margin-top) */
}
```

> ⚠️ **Detalle real a estudiar:** aquí hay `margin: 40px auto` **y luego**
> `margin-top: -100px`. Como `margin-top` está **después**, sobreescribe el valor
> superior del shorthand. Ese es el orden de la cascada en acción.
> (Nota: `padding: 0 -25px` no es un valor válido → el navegador lo descarta.)

### `web-cv-hector/css/styles.css`

```css
.container {
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 43rem;
  margin: 4rem auto;          /* centrado + separación vertical */
  padding-inline: 1.5rem;     /* aire lateral dentro de la caja */
}

.profile__data {
  margin-left: 1rem;
}

.profile__data > * {
  margin-bottom: 0.2rem;      /* TODOS los hijos directos del bloque de datos */
}

.profile__website {
  font-weight: 400;
  font-size: 0.75rem;
  line-height: 1;
  color: var(--color-text-body);
  background: var(--color-bg-tag);
  border-radius: 12px;
  padding: 5px 10px 5px 10px;   /* píldora "website.com" */
}

.section {
  margin-top: 3rem;
}
```

### `git/Pagina-Web/css/cuerpo.css` — tarjetas con sombra y radio

```css
.tarjeta {
  overflow: hidden;                     /* recorta lo que se salga */
  background: white;
  border-radius: 10px;                  /* esquinas redondeadas */
  box-shadow: 0 4px 12px rgb(0 0 0 / 12%);   /* sombra */
}

.contenido-tarjeta {
  padding: 18px;
}

.contenido-tarjeta h2 {
  margin: 0 0 10px;
}

.enlace-proyecto {
  display: inline-block;
  padding: 9px 14px;
  background: #e63946;
  color: white;
  text-decoration: none;
  border-radius: 6px;
}
```

> 📌 Fíjate en la receta del **botón**: `display: inline-block` + `padding` +
> `background` + `text-decoration: none` + `border-radius`.

---

## 4.6 `overflow` — cuando el contenido desborda

```css
overflow: visible;   /* por defecto: se sale y se ve */
overflow: hidden;    /* se recorta y NO hay scroll */
overflow: auto;      /* se recorta pero aparece scroll si hace falta */
overflow-x: auto;    /* solo scroll horizontal */
overflow-y: auto;    /* solo scroll vertical */
overflow: scroll;    /* scroll siempre (aunque no haga falta) */
```

### Ejemplos reales

`web-minimalista-hector/.../css/styles.css`:

```css
.brand__avatar {
  position: relative;
  display: block;
  width: 40.169px;
  height: 40px;
  border-radius: 50%;
  overflow: hidden;      /* recorta la foto al círculo */
  box-shadow: 0 2.667px 2.667px 0 rgba(0, 0, 0, 0.25);
}

.page {
  ...
  overflow: hidden;      /* evita barras de scroll no deseadas */
}
```

`git/Pagina-Web/css/cuerpo.css` (submenús):

```css
.menu-lateral li > ul {
  list-style: none;
  max-height: 0;         /* altura 0... */
  overflow: hidden;      /* ...y lo que sobresalga, recortado = MENÚ CERRADO */
  opacity: 0;
  margin: 0;
  padding: 0 0 0 15px;
  transition: max-height 0.3s ease, opacity 0.3s ease, padding 0.3s ease;
}

.menu-lateral li:hover > ul {
  max-height: 200px;     /* crece → se "despliega" */
  opacity: 1;
  padding: 8px 0 8px 15px;
}
```

> 📌 Este es un truco clásico: animar `max-height` de `0` a un valor alto simula un
> **desplegable con transición** sin usar JavaScript.

---

## 4.7 `overflow` + `border-radius` + `box-shadow` — componentes reales

```css
.card {
  display: flex;
  flex-direction: column;
  margin-top: 1.5rem;
  max-width: 152px;
  border: 1px solid var(--color-border);
  border-radius: 8px;
  overflow: hidden;              /* la imagen no se sale de las esquinas redondeadas */
}

.card__image {
  width: 100%;
  height: 89px;
  object-fit: cover;             /* recorta la imagen sin deformarla */
}
```

> 📌 **`object-fit: cover`** es clave para imágenes: rellena el hueco **recortando**,
> en vez de estirarse. El otro modo es `object-fit: contain` (cabe entera).

---

## 4.8 Unidades de medida aplicadas al modelo de caja

```css
width: 100%;            /* % del padre */
width: 50vw;            /* 50% del ancho de la ventana */
min-height: 100vh;      /* altura mínima = toda la pantalla */
max-width: 900px;       /* tope máximo: respeta el %, pero no pasa de 900px */
padding-inline: 1.5rem;  /* rem relativo a <html> */
```

**Ejemplo real** (`git/Pagina-Web/css/cuerpo.css`):

```css
body {
  min-height: 100vh;    /* ocupa al menos toda la pantalla (útil para el footer sticky) */
  display: flex;
  flex-direction: column;
}
```

---

## ✅ Resumen rápido

1. Caja = `margin` (fuera) → `border` → `padding` (dentro) → `contenido`.
2. **`box-sizing: border-box`** en el reset: el `padding` y el `border` caben en el `width`.
3. `margin` separa cajas y centra (`margin: 0 auto`); `padding` da aire al contenido.
4. `margin` colapsa entre hermanos verticales; `padding` no.
5. `overflow: hidden` recorta; `auto` pone scroll.
6. Shorthands: `margin: arriba derecha abajo izquierda` (o `vertical horizontal`).
7. `object-fit: cover` + `border-radius` + `overflow: hidden` = tarjetas con imagen bien recortada.

👉 **Siguiente:** `05-flexbox.md`
