# 06 — CSS GRID

**Grid** sirve para maquetar **en dos dimensiones** (filas **y** columnas a la vez).
Es ideal para tarjetas, layouts completos y rejillas de contenido.

> **Regla práctica:**
> - ¿Necesitas **una fila o una columna**? → **Flexbox**
> - ¿Necesitas **filas Y columnas** con alineación de rejilla? → **Grid**

---

## 6.1 Los dos roles

```css
.contenedor {
  display: grid;                      /* ← EL PADRE es el grid container */
}

.contenedor > * {
  /* ← los HIJOS son los grid items, y se colocan solos en las celdas */
}
```

---

## 6.2 Definir columnas y filas

```css
.contenedor {
  display: grid;

  /* COLUMNAS */
  grid-template-columns: 200px 200px 200px;       /* 3 columnas fijas */
  grid-template-columns: repeat(3, 1fr);           /* igual, abreviado */
  grid-template-columns: 1fr 2fr;                  /* la 2ª doble que la 1ª */
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));  /* RESPONSIVE mágico */
  grid-template-columns: 1fr 300px;                /* fluida + fija */

  /* FILAS */
  grid-template-rows: 100px auto;

  /* HUECO */
  gap: 24px;
  gap: 20px 24px;          /* filas 20px, columnas 24px */

  /* Alineación */
  justify-items: center;    /* items dentro de SU celda (eje columnas) */
  align-items: center;      /* items dentro de SU celda (eje filas) */
  place-items: center;      /* los dos a la vez */
  justify-content: center;  /* la REJILLA entera dentro del contenedor */
  text-align: center;
}
```

### La unidad `fr`

`fr` = **fraction** → "parte del espacio libre".

```css
grid-template-columns: 1fr 1fr 1fr;   /* 3 columnas iguales */
grid-template-columns: 1fr 3fr;       /* 1/4 y 3/4 */
grid-template-columns: 200px 1fr;     /* una fija y la otra se lleva el resto */
```

### `repeat()` y `auto-fit` + `minmax()` — el truco responsive sin `@media`

```css
grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
```

Significado:
- **cada columna como mínimo 240px**, máximo `1fr` (lo que toque).
- El navegador **solo** crea las columnas que caben.
- En móvil: 1 columna. En tablet: 2. En escritorio: 3 o 4.
- **No necesitas ni una `@media`.**

---

## 6.3 Colocación manual de elementos

```css
.item {
  grid-column: 1 / 3;      /* ocupa de la columna 1 a la 3 (columnas 1 y 2) */
  grid-row: 1 / 2;         /* fila 1 */
  grid-column: span 2;     /* ocupa 2 columnas */
  grid-column: 1 / -1;     /* de la primera a la ÚLTIMA (ancho completo) */
}
```

---

## 6.4 Ejemplos REALES copiados de tus proyectos

### 1) Rejilla de proyectos responsive — `git/Pagina-Web/css/cuerpo.css`

```css
.proyectos {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 24px;
  max-width: 1100px;
  margin: 40px auto;
  padding: 0 -25px;
  margin-top: -100px;
}
```

**Su HTML** (`git/Pagina-Web/pages/proyectos.html`):

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

> 📌 **Funcionamiento:**
> - El `<section class="proyectos">` es el **grid container**.
> - Cada `<article class="tarjeta">` es un **grid item** y ocupa una celda.
> - `minmax(240px, 1fr)` → cada tarjeta ocupa **entre 240px y 1 columna fluida**.
> - `auto-fit` → el navegador calcula **cuántas caben** y crea las columnas.
> - El HTML **no cambia nunca**: solo con CSS se pasa de 1 a 4 columnas.
>
> **Resultado:**
> ```
> Móvil (<240px de útil)     Tablet (~500px)         Escritorio (1100px)
> ┌──────────┐              ┌────────┬────────┐      ┌──────┬──────┬──────┐
> │ Tarjeta 1│              │ T. 1   │ T. 2   │      │ T.1  │ T.2  │ T.3  │
> ├──────────┤              ├────────┼────────┤      ├──────┼──────┼──────┤
> │ Tarjeta 2│              │ T. 3   │ T. 4   │      │ T.4  │      │      │
> └──────────┘              └────────┴────────┘      └──────┴──────┴──────┘
> ```

---

### 2) Rejilla de certificados — `git/Pagina-Web/css/cuerpo.css`

```css
.lista-certificados {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 280px), 1fr));
  gap: 24px;
  max-width: 900px;
  margin: 30px auto;
}
```

> 📌 **`min(100%, 280px)`** es un tope de seguridad: si la pantalla es **más estrecha
> que 280px**, usa el 100% en vez de 280px. Así **nunca** se sale de la pantalla.
> Patrón "a prueba de móviles pequeños".

**Su HTML** (`git/Pagina-Web/pages/certificados.html`, patrón):

```html
<section class="lista-certificados">
    <article class="tarjeta-certificado">
        <h2>Nombre del certificado</h2>
        <p class="fecha-certificado">Fecha</p>
        <span class="estado-certificado">Completado</span>
    </article>
    <article class="tarjeta-certificado">
        ...
    </article>
</section>
```

---

### 3) Lista de estudios apilada — `git/Pagina-Web/css/cuerpo.css`

```css
.lista-estudios {
  display: grid;
  gap: 20px;
  max-width: 800px;
  margin: 30px auto;
}
```

> 📌 **Truco:** `display: grid` **sin** `grid-template-columns` crea **una sola
> columna** que ocupa todo el ancho. Es la forma más corta de apilar elementos con
> un `gap` constante (mejor que dar `margin-bottom` a cada hijo).

---

### 4) Grid con filas y columnas combinadas — `git/Pagina-Web/css/cuerpo.css`

```css
.contacto-social {
  display: flex;
  width: 130px;
  min-height: 120px;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  ...
}
```

*(Aquí se usa Flex para la tarjeta individual; Grid para la rejilla de tarjetas.)*

---

### 5) Grid + `place-items` — patrón de centrado en celda

```css
.tarjeta {
  display: grid;
  place-items: center;      /* el contenido queda centrado en SU celda */
  min-height: 200px;
  border: 1px solid #e5e5e5;
  border-radius: 10px;
}
```

---

## 6.5 Ejemplo completo montado (HTML + CSS)

```html
<!-- HTML -->
<section class="rejilla">
    <article class="tarjeta">
        <img src="foto1.png" alt="Foto 1">
        <h3>Tarjeta 1</h3>
        <p>Descripción corta.</p>
        <a href="#">Ver más</a>
    </article>
    <article class="tarjeta"> ... </article>
    <article class="tarjeta"> ... </article>
    <article class="tarjeta"> ... </article>
</section>
```

```css
/* CSS */
.rejilla {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 260px), 1fr));
  gap: 24px;
  max-width: 1100px;
  margin: 40px auto;
}

.tarjeta {
  overflow: hidden;
  background: #fff;
  border: 1px solid #e5e5e5;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgb(0 0 0 / 12%);
}

.tarjeta img {
  display: block;
  width: 100%;
  height: 160px;
  object-fit: cover;
}

.tarjeta h3 { margin: 18px 18px 8px; }
.tarjeta p  { margin: 0 18px 18px; color: #555; line-height: 1.5; }
```

---

## 6.6 Grid vs Flexbox — tabla comparativa

| | **Flexbox** | **Grid** |
|---|---|---|
| Dimensiones | 1 (fila **o** columna) | 2 (filas **y** columnas) |
| Control | Principalmente del **contenedor** | Contenedor define **rejilla**, hijo define **celda** |
| ¿Necesitas filas y columnas? | No | **Sí** |
| Caso ideal | Barra, menú, footer, centrar, filas | Tarjetas, galerías, layout de página completo |
| Responsive | Cambiando `flex-direction` | `auto-fit` + `minmax()` (¡sin media queries!) |

### Combinación habitual (muy usada en tus proyectos)

```css
/* Grid para la REJILLA de tarjetas... */
.proyectos {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 24px;
}

/* ...y Flex DENTRO de cada tarjeta */
.tarjeta-contenido {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
```

---

## 6.7 Errores comunes con Grid

| Error | Solución |
|-------|----------|
| Los `gap` no hacen nada | Recuerda: `gap` solo funciona en **flex y grid** |
| La rejilla se sale de la pantalla | Usa `minmax(min(100%, 280px), 1fr)` |
| Una columna ocupa demasiado | Cambia `1fr` por `minmax(240px, 1fr)` |
| No sé cuántas columnas crear | `repeat(auto-fit, minmax(Xpx, 1fr))` |
| `grid-template-columns` fijo en móvil | Usa `auto-fit` o añade `@media` |

---

## ✅ Resumen rápido

1. `display: grid` en el padre; los hijos pasan a ser **grid items**.
2. `grid-template-columns: repeat(3, 1fr)` → 3 columnas iguales.
3. `repeat(auto-fit, minmax(240px, 1fr))` → **responsive automático sin `@media`**.
4. `gap` separa celdas (filas y columnas).
5. `place-items: center` centra el contenido **dentro de su celda**.
6. `grid-column: 1 / -1` → ancho completo.
7. Grid para la rejilla, Flex para lo de dentro.

👉 **Siguiente:** `07-pseudoclases-y-pseudoelementos.md`
