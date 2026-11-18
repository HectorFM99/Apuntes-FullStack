# 06 — DOM Y EVENTOS

> El **DOM** (**D**ocument **O**bject **M**odel) es la representación del HTML como
> **árbol de objetos** que JavaScript puede **leer, modificar y escuchar**.
> Con el DOM es donde JS "se ve" en la página.

---

## 6.1 El árbol del DOM

```html
<html>
  <head>...</head>
  <body>
    <nav class="menu">...</nav>
    <main id="contenido">
      <h1 class="titulo">Hola</h1>
      <p>Párrafo</p>
    </main>
  </body>
</html>
```

```
document
 └─ html
     ├─ head
     └─ body
         ├─ nav.menu
         └─ main#contenido
             ├─ h1.titulo   → "Hola"
             └─ p            → "Párrafo"
```

Cada nodo tiene `textContent`, `innerHTML`, `className`, `children`...

---

## 6.2 Seleccionar elementos

```js
// UNO solo
document.querySelector(".clase")          // primero que coincida con CSS
document.querySelector("#id")
document.querySelector("div.tarjeta")
document.getElementById("texto-escrito")  // solo por id (más rápido)

// VARIOS (devuelve NodeList)
document.querySelectorAll(".reveal")      // TODOS que coincidan con CSS
document.getElementsByClassName("clase")  // por clase (HTMLCollection antigua)
document.getElementsByTagName("p")        // por etiqueta
```

| Método | Devuelve | Se le pasa |
|--------|----------|------------|
| `querySelector` | **1** elemento o `null` | Selector **CSS** igual que en CSS |
| `querySelectorAll` | NodeList (como array) | Selector **CSS** |
| `getElementById` | **1** elemento o `null` | Solo el `id`, **sin `#`** |

> 📌 **Hoy se usa casi solo `querySelector` / `querySelectorAll`** porque aceptan
> cualquier selector CSS que ya sabes escribir.

---

## 6.3 Ejemplos REALES copiados

### **Archivo:** `git/Pagina-Web/js/programacion.js` *(copiado íntegro)*

```js
const botonMenu = document.querySelector(".boton-hamburguesa");
const menuLateral = document.querySelector(".menu-lateral");

botonMenu.addEventListener("click", function () {
  menuLateral.classList.toggle("abierto");
});

document.addEventListener("click", function (evento) {
  const clicEnMenu = menuLateral.contains(evento.target);
  const clicEnBoton = botonMenu.contains(evento.target);

  if (!clicEnMenu && !clicEnBoton) {
    menuLateral.classList.remove("abierto");
  }
});
const observer = new IntersectionObserver(
  (entries, observer) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add("is-visible");
        observer.unobserve(entry.target);
      }
    });
  },
  { threshold: 0.2 }
);

document.querySelectorAll(".reveal").forEach((element) => {
  observer.observe(element);
});

const elemento = document.getElementById("texto-escrito");

if (elemento) {
  const texto = elemento.textContent.trim();
  elemento.textContent = "";

  let posicion = 0;

  function escribir() {
    if (posicion < texto.length) {
      elemento.textContent += texto.charAt(posicion);
      posicion++;
      setTimeout(escribir, 40); // Más milisegundos = más lento
    }
  }

  escribir();
}
```

### 🧠 Desglose completo (trozo por trozo)

**1) Seleccionar y alternar una clase**

```js
const botonMenu = document.querySelector(".boton-hamburguesa");
const menuLateral = document.querySelector(".menu-lateral");

botonMenu.addEventListener("click", function () {
  menuLateral.classList.toggle("abierto");
});
```

| Pieza | Qué hace |
|-------|----------|
| `querySelector(".boton-hamburguesa")` | Busca el primer elemento con esa **clase** |
| `addEventListener("click", fn)` | Registra: "cuando hagas **clic**, ejecuta esto" |
| `classList.toggle("abierto")` | Si **tiene** la clase `abierto` la **quita**; si no, la **pone** |
| `function () { ... }` | Callback: se ejecuta **solo al hacer clic**, no ahora |

> 📌 JS **no cambia estilos directamente**: solo **alterna clases** y el CSS hace
> el resto (`.menu-lateral { left: -250px }` / `.menu-lateral.abierto { left: 0 }`).

**2) Cerrar el menú al clicar fuera**

```js
document.addEventListener("click", function (evento) {
  const clicEnMenu = menuLateral.contains(evento.target);
  const clicEnBoton = botonMenu.contains(evento.target);

  if (!clicEnMenu && !clicEnBoton) {
    menuLateral.classList.remove("abierto");
  }
});
```

| Pieza | Qué hace |
|-------|----------|
| `document.addEventListener("click", ...)` | Escucha clics en **toda** la página |
| `evento.target` | El elemento **concreto** sobre el que se hizo clic |
| `.contains(nodo)` | ¿Ese nodo está **dentro** de mí? |
| `classList.remove("abierto")` | Quita la clase → CSS cierra el menú |

> 📌 **Patrón "click outside":** si el clic fue **fuera** del menú **y** fuera del
> botón, ciérralo. Es el estándar para menús y modales.

**3) Animación al hacer scroll (`IntersectionObserver`)**

```js
const observer = new IntersectionObserver(
  (entries, observer) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add("is-visible");
        observer.unobserve(entry.target);
      }
    });
  },
  { threshold: 0.2 }
);

document.querySelectorAll(".reveal").forEach((element) => {
  observer.observe(element);
});
```

| Pieza | Qué hace |
|-------|----------|
| `new IntersectionObserver(callback, opciones)` | Crea un **vigilante** de visibilidad |
| `{ threshold: 0.2 }` | Actívate cuando el **20%** del elemento sea visible |
| `entry.isIntersecting` | ¿Está **entrando** en pantalla? |
| `entry.target` | El elemento vigilado |
| `classList.add("is-visible")` | Le añade la clase que CSS usa para **animarlo** |
| `observer.unobserve(...)` | **Deja de vigilarlo** (la animación solo pasa una vez) |
| `querySelectorAll(".reveal").forEach(...)` | Vigila **todos** los elementos con esa clase |

**Su CSS** (`git/Pagina-Web/css/cuerpo.css`):

```css
.reveal {
  opacity: 0;
  filter: blur(12px);
  transform: translateX(-28px);
  transition: opacity 1800ms ease, filter 1800ms ease, transform 1800ms ease;
}

.reveal.is-visible {
  opacity: 1;
  filter: blur(0);
  transform: translateX(0);
}
```

**Su HTML:**

```html
<h1 class="centrado reveal titulo">Bienvenidos a mi página personal</h1>
```

> 📌 **Cadena completa:** CSS lo deja invisible → JS vigila → al entrar en pantalla
> JS añade `is-visible` → CSS lo anima a la vista.

**4) Efecto "máquina de escribir" (`getElementById` + `setTimeout`)**

```js
const elemento = document.getElementById("texto-escrito");

if (elemento) {                                  // ⚠️ ¿existe? evita errores
  const texto = elemento.textContent.trim();     // leo TODO el texto
  elemento.textContent = "";                     // lo borro

  let posicion = 0;

  function escribir() {
    if (posicion < texto.length) {               // mientras queden letras
      elemento.textContent += texto.charAt(posicion);  // añado la siguiente
      posicion++;
      setTimeout(escribir, 40);                  // repito en 40ms
    }
  }

  escribir();                                    // arranco
}
```

**Su HTML** (`git/Pagina-Web/pages/sobre_mi.html`):

```html
<p class="escritura" id="texto-escrito">Soy Hector, desarrollador web en formación...</p>
```

**Su CSS:**

```css
#texto-escrito {
  max-width: 90%;
  margin-inline: auto;
}
```

> 📌 **Cómo funciona:** se guarda el texto → se borra → se escribe letra a letra con
> `setTimeout(..., 40)` (cada 40 ms añade una). `if (elemento)` protege para que no
> falle en páginas donde **ese id no existe**.

---

### **Archivo:** `bichomon/bichomon_hector/scripts.js` *(copiado íntegro)*

```js
// 1. Cambia el título "Generation 1 Pokémon" por "Generasión 1 Pokimon"

const tituloGeneracion1 = document.querySelector('#gen-1');
tituloGeneracion1.textContent = 'Generasión 1 Pokimon';
console.log('1) Nuevo título:', tituloGeneracion1.textContent);


// 2. Cambia el color de fondo de la primera generación de Pokimon

const primeraGeneracion = document.querySelector('.infocard-list');
primeraGeneracion.style.backgroundColor = '#ffe9a8';
console.log('2) Fondo 1ª generación:', primeraGeneracion.style.backgroundColor);

// 3. Imprime por consola la URL de la página

console.log('3) URL:', window.location.href);


// 4. Imprime por consola el dominio de la página

console.log('4) Dominio:', location.hostname);


// 5. Imprime todos los nodos de imagen

const imagenes = document.querySelectorAll('img');
console.log('5) Nodos de imagen (' + imagenes.length + '):', imagenes);

// 6. Sustituye el atributo "src" de todas las imágenes por el gif indicado

const GIF = 'https://media.giphy.com/media/2v170e71aanfi/giphy.gif';
document.querySelectorAll('img').forEach(function (img) {
	img.setAttribute('src', GIF);
});
console.log('6) Imágenes con el gif:', document.querySelectorAll('img[src="' + GIF + '"]').length);

// 7. Premium / Extra: cambia el fondo de todos los .infocard-lg-data de los Pokimon voladores

let voladores = 0;
document.querySelectorAll('.infocard-lg-data.text-muted').forEach(function (data) {
	if (data.querySelector('.itype.flying')) {
		data.style.backgroundColor = '#cfe8ff';
		voladores++;
	}
});
console.log('7) Pokimon voladores con fondo cambiado:', voladores);
```

### 🧠 Qué demuestra cada punto

| # | Técnica DOM | Línea clave |
|---|-------------|-------------|
| 1 | **Seleccionar por id con selector CSS** + **texto** | `querySelector('#gen-1')` → `.textContent = '...'` |
| 2 | **Estilo en línea** desde JS | `.style.backgroundColor = '#ffe9a8'` |
| 3 | **Objeto `window.location`** | `window.location.href` = URL completa |
| 4 | **Hostname** | `location.hostname` = solo el dominio |
| 5 | **Lista de elementos** + `.length` | `querySelectorAll('img')` |
| 6 | **Recorrer con `forEach`** + **atributo** | `setAttribute('src', GIF)` |
| 7 | **Selector compuesto** + **condición** + **contador** | `.infocard-lg-data.text-muted` + `querySelector('.itype.flying')` |

**Detalles a destacar:**

```js
// Selector compuesto: DOS clases juntas (como en CSS)
document.querySelectorAll('.infocard-lg-data.text-muted')

// Comprobar si dentro hay un hijo concreto
if (data.querySelector('.itype.flying')) { ... }

// Cambiar un ATRIBUTO, no solo el estilo
img.setAttribute('src', GIF);

// Estilo en línea desde JS (equivale a style="background-color: #ffe9a8")
elemento.style.backgroundColor = '#ffe9a8';

// Cadena de selección: 'img[src="URL"]'  = imágenes con ese src exacto
document.querySelectorAll('img[src="' + GIF + '"]')
```

> 📌 **`element.style.propiedad`** aplica **estilo en línea**, que tiene la máxima
> especificidad. Es potente pero, como vimos en CSS, **lo mejor es añadir/quitar
> clases** en vez de pintar estilos uno a uno desde JS.

---

### **Archivo:** `varita-magica/varita-magica-hector/scripts.js` *(copiado íntegro)*

```js
// 1. Evitar el comportamiento por defecto al hacer click

document.addEventListener('click', (event) => {
    event.preventDefault();

    const element = event.target;

    // 2.1 Imágenes: cambiamos la imagen por un gif aleatorio
    if (element.tagName === 'IMG') {
        if (!element.dataset.originalSrc) {
            element.dataset.originalSrc = element.getAttribute('src');
        }
        element.src = getRandom(gifs);
        return;
    }

    // 2.2 Párrafos: cambiamos el color del texto y el de fondo
    if (element.tagName === 'P') {
        element.style.color = getRandom(colors);
        element.style.backgroundColor = getRandom(colors);
        return;
    }

    // 2.3 Bloques de article o section: cambiamos el color de fondo
    if (element.tagName === 'ARTICLE' || element.tagName === 'SECTION') {
        element.style.backgroundColor = getRandom(blockColors);
        return;
    }
});


// 3. Al pasar el cursor por encima, y devolver al estado original

document.addEventListener('mouseover', (event) => {
    const element = event.target;

    if (element.tagName === 'IMG') {
        if (!element.dataset.originalSrc) {
            element.dataset.originalSrc = element.getAttribute('src');
        }
        element.src = abracadabraGif;
        return;
    }

    if (element.tagName === 'P') {
        element.style.color = getRandom(colors);
        element.style.backgroundColor = getRandom(colors);
        return;
    }

    if (element.tagName === 'ARTICLE' || element.tagName === 'SECTION') {
        element.style.backgroundColor = getRandom(blockColors);
        return;
    }
});

// Al salir del elemento devolvemos el estado original
document.addEventListener('mouseout', (event) => {
    const element = event.target;

    if (element.tagName === 'IMG' && element.dataset.originalSrc) {
        element.src = element.dataset.originalSrc;
        return;
    }

    if (element.tagName === 'P') {
        element.style.color = '';
        element.style.backgroundColor = '';
        return;
    }

    if (element.tagName === 'ARTICLE' || element.tagName === 'SECTION') {
        element.style.backgroundColor = '';
        return;
    }
});

// 4. Función que devuelve un valor aleatorio de un array

const getRandom = (array) => {
    return array[Math.floor(Math.random() * array.length)];
};

// Comprobación de la función getRandom
console.log(getRandom(['red', 'blue', 'green']));


// 5. Paleta de colores elegida en coolors.co (Magical palette)

const colors = ['#F94144', '#F3722C', '#F8961E', '#F9C74F', '#90BE6D', '#43AA8B', '#577590'];

// Colores de fondo para los bloques article/section
const blockColors = ['#22223B', '#4A4E69', '#2A9D8F', '#264653', '#6D597A', '#355070'];


// 6. Gifs mágicos de la carpeta assets

const gifs = [
    'assets/magic-1.gif',
    'assets/magic-2.gif',
    'assets/magic-3.gif',
    'assets/magic-4.gif',
    'assets/magic-5.gif',
    'assets/magic-6.gif'
];

const abracadabraGif = 'assets/abracadabra.gif';
```

### 🧠 Desglose

| Concepto | Línea | Qué hace |
|----------|-------|----------|
| `event.preventDefault()` | 1ª | **Cancela** la acción por defecto (que un enlace navegue) |
| `event.target` | | El elemento **exacto** sobre el que se hizo clic/pasó el ratón |
| `element.tagName` | | Nombre de la etiqueta en **MAYÚSCULAS** (`"IMG"`, `"P"`) |
| `element.dataset.originalSrc` | | Lee el atributo **`data-original-src`** (guion → camelCase) |
| `getAttribute('src')` | | Lee el atributo `src` |
| `element.style.color = ''` | | **Vaciar** el estilo = quitarlo y volver al original |
| `addEventListener('mouseover'...)` | | Ratón **encima** del elemento |
| `addEventListener('mouseout'...)` | | Ratón **sale** del elemento |
| `getRandom(array)` | | `Math.random() * length` → índice aleatorio |
| `Math.floor(Math.random() * 6)` | | Entero entre 0 y 5 (inclusivo) |

**El truco de `dataset` para restaurar:**

```js
// PRIMERA vez que pasas el ratón: guarda la imagen original
if (!element.dataset.originalSrc) {
    element.dataset.originalSrc = element.getAttribute('src');   // data-original-src="foto.png"
}
element.src = abracadabraGif;

// Al salir (mouseout): la recupera
if (element.tagName === 'IMG' && element.dataset.originalSrc) {
    element.src = element.dataset.originalSrc;    // vuelve a "foto.png"
}
```

**El truco de `style = ''`:**

```js
element.style.color = '';            // quita el color en línea
element.style.backgroundColor = '';  // quita el fondo en línea
// → el elemento vuelve a tener el aspecto original del CSS
```

---

## 6.4 EVENTOS — el catálogo

### Registrar un evento

```js
elemento.addEventListener("tipoDeEvento", function (evento) {
    // código
});

// Con arrow function
elemento.addEventListener("tipoDeEvento", (e) => { ... });

// Con opciones
elemento.addEventListener("scroll", manejador, { passive: true, once: true });

// Quitar
elemento.removeEventListener("tipoDeEvento", manejador);
```

### Eventos más usados

| Evento | Cuándo se dispara |
|--------|-------------------|
| `click` | Clic (también `dblclick`, `contextmenu`) |
| `mouseover` / `mouseout` | Ratón **entra** / **sale** |
| `mouseenter` / `mouseleave` | Igual pero **sin burbujeo** |
| `mousemove` | Ratón se **mueve** dentro |
| `keydown` / `keyup` | Tecla pulsada / soltada |
| `keypress` (obsoleto) | Tecla escrita |
| `input` | El valor de un campo **cambia** |
| `change` | El campo **pierde foco** con cambio (select, checkbox) |
| `submit` | Se envía un formulario |
| `focus` / `blur` | El elemento **gana / pierde** foco |
| `scroll` | Se hace scroll |
| `load` | La página/recurso **terminó** de cargar |
| `DOMContentLoaded` | El HTML **está parseado** (JS seguro para seleccionar) |
| `resize` | Se redimensiona la ventana |
| `touchstart` / `touchend` | Táctil |

### Objeto `evento` (`e` / `evento`)

```js
boton.addEventListener("click", (e) => {
  e.target;            // elemento pulsado
  e.currentTarget;     // elemento al que se registró el listener
  e.preventDefault();  // evitar comportamiento por defecto
  e.stopPropagation();  // evitar que el evento suba al padre (burbujeo)
  e.key;               // tecla pulsada (teclado)
  e.clientX;           // posición del ratón
  e.type;              // tipo de evento ("click")
});
```

---

## 6.5 `classList` — la forma limpia de manipular clases

```js
elemento.classList.add("nueva");       // añade
elemento.classList.remove("vieja");    // quita
elemento.classList.toggle("activa");   // alterna (¡clave!)
elemento.classList.contains("activa"); // ¿la tiene? → true/false
elemento.className = "solo-esto";      // reemplaza TODAS (menos recomendado)
```

**Reales:**

```js
menuLateral.classList.toggle("abierto");     // git/Pagina-Web
entry.target.classList.add("is-visible");    // git/Pagina-Web
menuLateral.classList.remove("abierto");     // git/Pagina-Web
```

> 📌 **Patrón recomendado:** JS solo toca **clases**; el CSS define los estados.
> Es más limpio, mantenible y accesible que ir pintando `style`.

---

## 6.6 Cambiar contenido y atributos

```js
// TEXTO (seguro: no interpreta HTML)
elemento.textContent = "Nuevo texto";
elemento.textContent += " añadido";

// HTML (peligroso si viene de usuario → riesgo XSS)
elemento.innerHTML = "<strong>negrita</strong>";

// ATRIBUTOS
elemento.getAttribute("src");
elemento.setAttribute("src", "nueva.png");
elemento.removeAttribute("hidden");
elemento.hasAttribute("disabled");

// Propiedades directas
imagen.src = "nueva.png";
enlace.href = "https://...";
input.value = "texto";
checkbox.checked = true;
```

---

## 6.7 Modificar CSS desde JS

```js
// Opción A: estilo en línea (máxima especificidad)
elemento.style.color = "red";
elemento.style.backgroundColor = "#ffe9a8";
elemento.style.transform = "rotate(15deg)";

// Opción B (RECOMENDADA): clases
elemento.classList.add("destacado");
```

> ⚠️ En JS las propiedades con guion se escriben en **camelCase**:
> `background-color` → `backgroundColor`, `font-size` → `fontSize`.

---

## 6.8 Crear y borrar elementos

```js
const nuevo = document.createElement("p");
nuevo.textContent = "Párrafo nuevo";
nuevo.classList.add("nuevo-parrafo");

document.querySelector("main").appendChild(nuevo);   // añade al final
contenedor.prepend(nuevo);                            // al principio
contenedor.before(nuevo);                             // antes del contenedor
contenedor.after(nuevo);                              // después

nuevo.remove();                                       // borra
contenedor.removeChild(nuevo);                        // borra hijo
```

---

## 6.9 Formularios con JS

```js
const campo = document.querySelector("#nombre");

campo.addEventListener("input", (e) => {
  console.log(e.target.value);        // valor actual mientras escribes
});

const formulario = document.querySelector("form");
formulario.addEventListener("submit", (e) => {
  e.preventDefault();                 // ¡evita que recargue la página!
  const datos = new FormData(formulario);
  console.log(Object.fromEntries(datos));
});
```

---

## 6.10 Esperar a que el DOM esté listo

```js
// Forma moderna (si el script es "defer" o está al final, ya no hace falta)
document.addEventListener("DOMContentLoaded", () => {
  const el = document.querySelector(".clase");
});
```

> 📌 Si tu `<script>` está **antes** del contenido y no lleva `defer`, al ejecutarse
> el HTML **aún no existe** → `querySelector` devuelve `null` → error.
> Por eso tus proyectos ponen el `<script>` **al final del `<body>`**.

---

## 6.11 APIs modernas usadas en tus proyectos

### `IntersectionObserver` — detectar visibilidad

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      entry.target.classList.add("is-visible");
      observer.unobserve(entry.target);
    }
  });
}, { threshold: 0.2 });

document.querySelectorAll(".reveal").forEach((el) => observer.observe(el));
```

### `setTimeout` / `setInterval` — temporizadores

```js
setTimeout(() => { ... }, 40);        // ejecuta UNA vez tras 40ms
setInterval(() => { ... }, 1000);     // ejecuta CADA 1000ms
clearTimeout(id); clearInterval(id);  // cancelar
```

### `localStorage` — persistir datos

```js
localStorage.setItem("tema", "oscuro");
localStorage.getItem("tema");         // "oscuro"
localStorage.removeItem("tema");
```

### `fetch` — peticiones a APIs

```js
fetch("https://api.ejemplo.com/datos")
  .then(res => res.json())
  .then(datos => console.log(datos))
  .catch(error => console.error("Error:", error));
```

---

## 6.12 Errores comunes del DOM

| Error | Causa | Solución |
|-------|-------|----------|
| `Cannot read properties of null` | `querySelector` no encontró nada | Comprueba `if (el)` o revisa la clase/ruta |
| El script corre antes que el HTML | `<script>` en el `<head>` sin `defer` | Mover al final del body o usar `defer` |
| Ruta mal | `src="js/app.js"` desde una subcarpeta | `../js/app.js` |
| El clic recarga la página | Botón/`<a>` dentro de un form | `e.preventDefault()` |
| No se ve el cambio | Olvidaste `classList.add` o `classList.toggle` | Revisa que la clase exista en CSS |
| `element.style.foo` no funciona | Propiedad con guion | Usa camelCase: `backgroundColor` |

---

## ✅ Resumen rápido

1. Selecciona con **`querySelector` / `querySelectorAll`** (aceptan CSS).
2. Escucha con **`addEventListener("evento", callback)`**.
3. **`e.target`** = dónde se clicó; **`e.preventDefault()`** = cancela la acción por defecto.
4. Manipula **clases** con `classList.add/remove/toggle/contains` (lo recomendable).
5. **Texto** → `textContent`; **HTML** → `innerHTML` (con cuidado).
6. **Atributos** → `getAttribute/setAttribute`; estilos en línea → `element.style.prop`.
7. Patrón menú: `toggle("abierto")` + clic fuera con `contains()`.
8. Patrón scroll: `IntersectionObserver` + añadir clase `is-visible`.
9. `dataset` guarda datos extra en el HTML (`data-original-src` → `dataset.originalSrc`).
10. Espera al DOM (script al final del body o `defer`).

👉 **Siguiente:** `07-ejemplos-completos-de-proyectos.md`
