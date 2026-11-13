# 07 — EJEMPLOS COMPLETOS COPIADOS DE TUS PROYECTOS

> 📁 **Los archivos de los proyectos NO se han modificado.** Todo lo de aquí abajo son
> **copias** de su contenido original, comentadas para estudiar.

---

## EJEMPLO 1 — Variables (el archivo JS más pequeño)

**Origen:** `proyecto/programacion.js` *(copiado íntegro — 3 líneas)*

```js
let nombre = "Hector";
let apellido = "Franco Magallo";
let edad = 25;
```

**Qué enseña:**
- `string` entre comillas (`"Hector"`), `number` sin comillas (`25`).
- `let` para valores que pueden cambiar.
- Se puede usar con plantillas en JS o para mostrar datos en HTML.

```js
// Uso posible con plantillas literales
console.log(`Hola, me llamo ${nombre} ${apellido} y tengo ${edad} años.`);
// "Hola, me llamo Hector Franco Magallo y tengo 25 años."
```

---

## EJEMPLO 2 — DOM completo: menú, scroll y máquina de escribir

**Origen:** `git/Pagina-Web/js/programacion.js` *(copiado íntegro)*

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

### 🧠 Los 4 módulos que contiene

| Bloque | Qué hace | Técnicas JS |
|--------|----------|-------------|
| **1. Menú hamburguesa** | Abre/cierra el menú lateral | `querySelector`, `addEventListener("click")`, `classList.toggle` |
| **2. Clic fuera** | Cierra el menú si pulsas fuera | `document.addEventListener`, `evento.target`, `.contains()`, `classList.remove` |
| **3. Reveal** | Aparecen los elementos al hacer scroll | `IntersectionObserver`, `querySelectorAll().forEach()`, `classList.add`, `unobserve` |
| **4. Máquina de escribir** | Escribe el texto letra a letra | `getElementById`, `textContent`, `charAt()`, `setTimeout` recursivo |

### Su HTML correspondiente

```html
<!-- Botón + menú (git/Pagina-Web/index.html) -->
<button class="boton-hamburguesa" type="button" aria-label="Abrir menú">☰</button>
<aside class="menu-lateral"> ... </aside>
<div class="fondo-difuminado"></div>

<!-- Elementos a animar -->
<h1 class="centrado reveal titulo">Bienvenidos a mi página personal</h1>

<!-- Máquina de escribir (pages/sobre_mi.html) -->
<p class="escritura" id="texto-escrito">Soy Hector, desarrollador web en formación...</p>

<!-- Al final del body -->
<script src="js/programacion.js"></script>
```

### Su CSS correspondiente

```css
.menu-lateral            { left: -250px; transition: left 0.3s; }
.menu-lateral.abierto    { left: 0; }
.reveal                  { opacity: 0; transform: translateX(-28px); transition: ... }
.reveal.is-visible       { opacity: 1; transform: translateX(0); }
#texto-escrito           { max-width: 90%; margin-inline: auto; }
```

---

## EJEMPLO 3 — Manipulación directa del DOM

**Origen:** `bichomon/bichomon_hector/scripts.js` *(copiado íntegro)*

```js
// 1. Cambia el título "Generation 1 Pokémon" por "Generasión 1 Pokimon"

const tituloGeneracion1 = document.querySelector('#gen-1');
tituloGeneracion1.textContent = 'Generasión 1 Pokimon';
console.log('1) Nuevo título:', tituloGeneracion1.textContent);


// 2. Cambia el color de fondo de la primera generación de Pokimon (el primer .infocard-list es el de la generación 1)

const primeraGeneracion = document.querySelector('.infocard-list');
primeraGeneracion.style.backgroundColor = '#ffe9a8';
console.log('2) Fondo 1ª generación:', primeraGeneracion.style.backgroundColor);

// 3. Imprime por consola la URL de la página

console.log('3) URL:', window.location.href);


// 4. Imprime por consola el dominio de la página
//    OJO: document.domain está DEPRECATED asi que le pedi a ChatGTP que me diera otra opcion
// 			y me dijo que usara location.hostname, dejo el original como texto por si acaso.

console.log('4) Dominio:', location.hostname);
// console.log('4) Dominio (deprecated):', document.domain) el original lo tenía así, pero da warning en la consola y no es recomendable usarlo segun ChatGPT;

// 5. Imprime todos los nodos de imagen

const imagenes = document.querySelectorAll('img');
console.log('5) Nodos de imagen (' + imagenes.length + '):', imagenes);

// 6. Sustituye el atributo "src" de todas las imágenes por el gif indicado

const GIF = 'https://media.giphy.com/media/2v170e71aanfi/giphy.gif';
document.querySelectorAll('img').forEach(function (img) {
	img.setAttribute('src', GIF);
});
console.log('6) Imágenes con el gif:', document.querySelectorAll('img[src="' + GIF + '"]').length);

// 7. Premium / Extra: cambia el fondo de todos los .infocard-lg-data.text-muted de los Pokimon voladores (.itype.flying)

let voladores = 0;
document.querySelectorAll('.infocard-lg-data.text-muted').forEach(function (data) {
	if (data.querySelector('.itype.flying')) {
		data.style.backgroundColor = '#cfe8ff';
		voladores++;
	}
});
console.log('7) Pokimon voladores con fondo cambiado:', voladores);
```

### 🧠 Vocabulario DOM cubierto

```js
document.querySelector('#gen-1')          // seleccionar 1 por id
document.querySelector('.clase')          // seleccionar 1 por clase
document.querySelectorAll('img')          // seleccionar TODOS
document.querySelectorAll('.a.b')         // selector compuesto
.querySelectorAll(...).forEach(fn)        // recorrer la lista
element.textContent                       // leer/escribir TEXTO
element.style.backgroundColor             // estilo en línea (camelCase)
element.setAttribute('src', valor)        // cambiar ATRIBUTO
window.location.href                      // URL completa
location.hostname                         // solo dominio
element.querySelector('.hijo')            // buscar DENTRO de un elemento
img[src="..."]                            // selector por atributo
contador++                                // acumulador
```

### Su HTML correspondiente (`bichomon/bichomon_hector/index.html`)

```html
<h2 id="gen-1">Generation 1 Pokémon</h2>
<div class="infocard-list infocard-list-pkmn-lg">
    ...
</div>
```

---

## EJEMPLO 4 — Eventos y aleatoriedad

**Origen:** `varita-magica/varita-magica-hector/scripts.js` *(copiado íntegro)*

```js
// 1. Evitar el comportamiento por defecto al hacer click (por ejemplo, que un enlace no navegue a otra página)

document.addEventListener('click', (event) => {
    event.preventDefault();

    const element = event.target;

    // 2.1 Imágenes: cambiamos la imagen por un gif aleatorio magic-*
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


// 3. Al pasar el cursor por encima, y devolver al estado original cuando el cursor sale del elemento

// 3.1 Imágenes: cambiamos la imagen por abracadabra.gif
document.addEventListener('mouseover', (event) => {
    const element = event.target;

    if (element.tagName === 'IMG') {
        if (!element.dataset.originalSrc) {
            element.dataset.originalSrc = element.getAttribute('src');
        }
        element.src = abracadabraGif;
        return;
    }

    // 3.2 Párrafos: cambiamos el color del texto y el de fondo
    if (element.tagName === 'P') {
        element.style.color = getRandom(colors);
        element.style.backgroundColor = getRandom(colors);
        return;
    }

    // 3.3 Bloques de article o section: fondo distinto al de párrafo
    if (element.tagName === 'ARTICLE' || element.tagName === 'SECTION') {
        element.style.backgroundColor = getRandom(blockColors);
        return;
    }
});

// Al salir del elemento devolvemos el estado original
document.addEventListener('mouseout', (event) => {
    const element = event.target;

    // 3.1 Imágenes: restauramos la imagen original
    if (element.tagName === 'IMG' && element.dataset.originalSrc) {
        element.src = element.dataset.originalSrc;
        return;
    }

    // 3.2 Párrafos: eliminamos los estilos y recuperamos los originales
    if (element.tagName === 'P') {
        element.style.color = '';
        element.style.backgroundColor = '';
        return;
    }

    // 3.3 Bloques de article or section: eliminamos el fondo aplicado
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

// Colores de fondo para los bloques article/section (distintos a los de párrafo)
const blockColors = ['#22223B', '#4A4E69', '#2A9D8F', '#264653', '#6D597A', '#355070'];


// 6. Gifs mágicos de la carpeta assets, elegidos con getRandom en los apartados 2 y 3

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

### 🧠 Técnicas cubiertas

| Técnica | Ejemplo del archivo |
|---------|---------------------|
| Eventos de **documento** | `addEventListener('click'/'mouseover'/'mouseout')` |
| `event.preventDefault()` | Evita que los enlaces naveguen |
| `event.target` | Detectar **qué** elemento se pulsó |
| `tagName` en mayúsculas | `=== 'IMG'`, `=== 'P'`, `=== 'ARTICLE'` |
| `dataset` | Guardar `data-original-src` para restaurar |
| `getAttribute` / `setAttribute` | Leer y cambiar `src` |
| `style.prop = ''` | **Quitar** un estilo en línea |
| Arrow function | `const getRandom = (array) => {...}` |
| Aleatoriedad | `Math.floor(Math.random() * array.length)` |
| Arrays como datos | `colors`, `blockColors`, `gifs` |
| `||` (OR) | `tagName === 'ARTICLE' \|\| tagName === 'SECTION'` |
| `return` temprano | Sale de la función nada más tratar un caso |
| Constantes con `const` | Las paletas y los gifs |

### La función `getRandom` paso a paso

```js
const getRandom = (array) => {
    return array[Math.floor(Math.random() * array.length)];
};

// 1. Math.random()          → 0.7381...  (0 a 1)
// 2. * array.length          → 0.7381 * 6 = 4.42...
// 3. Math.floor(...)         → 4           (entero)
// 4. array[4]                → el 5º elemento del array
```

---

## EJEMPLO 5 — Bucles y algoritmia

**Origen:** `algoritmia/Algoritmia-Hector/index.js` *(copiado tal cual)*

```js
//1
const fijo = 3;
for (let i = 1; i < 10; i++) {
    console.log(fijo * i);
}

//2
function pedirNumeros() {
  const numeros = [];

  while (true) {
    const entrada = prompt("Introduce un número (0 para terminar):");


    const numero = parseInt(entrada);

    if (isNaN(numero)) break;  
    if (numero === 0) break;   
    numeros.push(numero);
  }

  numeros.sort((a, b) => a - b);  
  return numeros;
}

//3
function esBisiesto(año) {
  return (año % 4 === 0 && año % 100 !== 0) || (año % 400 === 0);
}
const año = parseInt(prompt("Introduce un año:"));
if (esBisiesto(año)) {
  console.log(año + " es un año bisiesto.");
} else {
  console.log(año + " no es un año bisiesto.");
}

//4
function contarCaracteres(data, caracter) {
  let total = 0;
  const objetivo = caracter.toLowerCase();

  for (let i = 0; i < data.length; i++) {      
    const palabra = data[i];
    for (let j = 0; j < palabra.length; j++) { 
      if (palabra[j].toLowerCase() === objetivo) {
        total++;
      }
    }
  }

  return total;
}
```

### 🧠 Qué cubre

| Ejercicio | Temas |
|-----------|-------|
| **1** | `const`, `for` de 1 a 9, multiplicar, `console.log` |
| **2** | `function`, `while (true)`, `break`, `prompt`, `parseInt`, `isNaN`, `push`, `sort` con comparador, `return` |
| **3** | `&&` y `\|\|`, `%` (módulo), `if/else`, funciones que devuelven booleano |
| **4** | Bucles **anidados** (`i` y `j`), acceso por índice `palabra[j]`, `.toLowerCase()`, acumulador `total++` |

---

## EJEMPLO 6 — Básicos: variables, booleanos y bucles

**Origen:** `bikram hector/bikram-hector/index.js` *(fragmentos seleccionados)*

```js
// 1
let variableSinValor;

// 2
let booleano1 = true;
let booleano2 = false;

// 3
const PI = 3.14;

// 4
const TAU = 2 * PI;

// 5
let booleanoAnd = booleano1 && booleano2;

// 6
let booleanoNot = !booleano1;

// 7
let booleanoMix0 = (booleano1 || booleano2) && (booleano1 || (!booleano1 && !booleano2));

// 8 
let incrementarDesp = 2;
let resultadoDesp = incrementarDesp++;

// 9 
let incrementarAntes = 2;
let resultadoAntes = ++incrementarAntes; 

// 10
let contarHasta10_2 = 0;
for (; contarHasta10_2 < 10; contarHasta10_2++) {}

// 11
let postI = 0;
let postJ = 0;
for (let i = 0; i < 11; i++) {
  postI += postJ++;
}

// 12
let sumaPares = 0;
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) {
    sumaPares += i;
  }
}

// 18
let seisNoEsNueve = 6 !== 9;

// 24
let contarHasta10 = 0;
while (contarHasta10 < 10) {
  contarHasta10++;
}

// 25
let preI = 0;
let preJ = 0;
for (let i = 0; i < 11; i++) {
  preI += ++preJ;
}

// 26
let sumaImpares = 0;
for (let i = 0; i < 10; i++) {
  if (i % 2 !== 0) {
    sumaImpares += i;
  }
}
```

---

## EJEMPLO 7 — Arrays y funciones

**Origen:** `bikram2-hector/bikram2-hector/index.js` *(fragmentos seleccionados)*

```js
// 5
function suma(a, b) {
  return a + b;
}

// 7
function separarPalabras(texto) {
  return texto.split(' ');
}

// 8
function repetirString(texto, veces) {
  let resultado = '';
  for (let i = 0; i < veces; i++) {
    resultado = resultado + texto;
  }
  return resultado;
}

// 9
function esPrimo(num) {
  if (num < 2) {
    return false;
  }
  for (let i = 2; i < num; i++) {
    if (num % i === 0) {
      return false;
    }
  }
  return true;
}

// 10
function ordenarArray(arr) {
  return arr.sort(function (a, b) {
    return a - b;
  });
}

// 11
function obtenerPares(arr) {
  let pares = [];
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] % 2 === 0) {
      pares.push(arr[i]);
    }
  }
  return pares;
}

// 12
function pintarArray(arr) {
  return '[' + arr.join(', ') + ']';
}

// 13  ← callback
function arrayMapi(arr, funcion) {
  let resultado = [];
  for (let i = 0; i < arr.length; i++) {
    resultado.push(funcion(arr[i]));
  }
  return resultado;
}

// 14
function eliminarDuplicados(arr) {
  let resultado = [];
  for (let i = 0; i < arr.length; i++) {
    if (!resultado.includes(arr[i])) {
      resultado.push(arr[i]);
    }
  }
  return resultado;
}

// 22 ← array que guarda FUNCIONES
let arrayFunciones = [suma, resta, multiplicacion];

// 25
function sumarArray(arr) {
  let total = 0;
  for (let i = 0; i < arr.length; i++) {
    total = total + arr[i];
  }
  return total;
}
```

---

## EJEMPLO 8 — Objetos

**Origen:** `bikram-objetos/bikram-objetos-hector/index.js` *(fragmentos seleccionados)*

```js
//1
let Coche = {
    marca: "Seat",
    modelo: "León",
    matricula: "1234ABC"
};

//4 - objeto con MÉTODOS (funciones dentro)
let Perro = {
    nombre: "Toby",
    raza: "Labrador",
    color: "Marron",
    edad: 4,
    ladrar: function () {
        console.log("¡Guau guau!");
    },
    popo: function () {
        return Math.random() * 3;
    }
};

// 5 - acceso con punto
let marcaPortatil = Portatil.marca;

// 6 - acceso con corchetes
let marcaPortatil2 = Portatil["marca"];

// 9 - modificar
Portatil.modelo = "P345";

// 10 - añadir a un array de dentro
Concierto.cartelera.push("Guns N' Roses");

// 11 - asignar objeto Date
Concierto.fecha = new Date();

// 12 - añadir objeto anidado
Impresora.imprimiendo = {
    nombreArchivo: "documento.pdf",
    copias: 3,
    numPaginas: 10
};

// 15 - varios métodos
let Avion = {
    numPasajeros: 180,
    despegar: function () { console.log("despegando"); },
    volar: function () { console.log("llegando al destino"); },
    aterrizar: function () { console.log("aterrizando"); }
};

// 25 - negación de una propiedad booleana
Led.encendido = !Led.encendido;
```

---

## EJEMPLO 9 — Destructuring, spread y rest

**Origen:** `bikram destructuring hector/advanced-js-bikram-destructuring-spread-rest/index.js` *(copiado íntegro)*

```js
//RESUELVE LOS EJERCICIOS AQUÍ

/* ===================== Destructuring ===================== */

// 1.- Extrae la empleada Ana completa
const empleados = [
    { name: "Luis", email: "Luis@gmail.com" },
    { name: "Ana", email: "Ana@gmail.com" },
    { name: "Andrea", email: "Andrea@gmail.com" },
];

// Ana está en la posición 1, por eso dejamos hueco con la coma vacía
const [, ana] = empleados;

// 2.- Extrae el email del empleado Luis (posición 0) directamente con alias
const [{ email: emailLuis }] = empleados;

// 3.- Intercambia los valores de a y b con destructuración
let a = 5;
let b = 3;

// Equivale a: const temporal = [b, a] -> a toma el valor de b y viceversa
[a, b] = [b, a];

// 4.- Guarda las temperaturas en maximaHoy y maximaManana mediante destructuración
const HIGH_TEMPERATURES = {
    yesterday: 30,
    today: 35,
    tomorrow: 32,
};

const { today: maximaHoy, tomorrow: maximaManana } = HIGH_TEMPERATURES;

console.log(maximaHoy);     // 35
console.log(maximaManana);  // 32


// 5.- Suma cualquier cantidad de números
function sumEveryOther(...numeros) {
    return numeros.reduce((total, num) => total + num, 0);
}

// 6.- Suma solo los números, ignorando los strings
function addOnlyNums(...args) {
    return args
        .filter(arg => typeof arg === "number")
        .reduce((total, num) => total + num, 0);
}

// 7.- Devuelve cuántos argumentos ha recibido
function countTheArgs(...args) {
    return args.length;
}

// 8.- Combina dos arrays en uno solo usando spread
function combineTwoArrays(array1, array2) {
    return [...array1, ...array2];
}

// 9.- Devuelve un array con los elementos únicos (sin repetidos)
function onlyUniques(...args) {
    return [...new Set(args)];
}

// 10.- Combina cualquier cantidad de arrays en uno solo
function combineAllArrays(...arrays) {
    return arrays.reduce((combinado, arr) => [...combinado, ...arr], []);
}

// 11.- Eleva cada argumento al cuadrado y suma todos los resultados
function sumAndSquare(...args) {
    return args.reduce((suma, num) => suma + num ** 2, 0);
}
```

---

## EJEMPLO 10 — `map`, `filter` y `reduce`

**Origen:** `bikram-advance-js/bikram-map-filter-reduce-hector/index.js` *(fragmentos seleccionados)*

```js
// Ejercicio 1 - map
const numbers = [4, 5, 6, 7, 8, 9, 10];
const elevados = () => numbers.map(num => Math.pow(num, num));

// Ejercicio 2 - map + acceso por clave de objeto
const foodList = ['Pizza', 'Ramen', 'Paella', 'Entrecot'];
const frasesComida = {
    Pizza: 'Como soy de Italia, amo comer Pizza',
    Ramen: 'Como soy de Japón, amo comer Ramen',
    Paella: 'Como soy de Valencia, amo comer Paella',
    Entrecot: 'Aunque no como carne, el Entrecot es sabroso'
};
const result2 = foodList.map(comida => frasesComida[comida]);

// Ejercicio 3 - map sobre array de objetos + template string
const staff = [
    { name: 'Pepe', role: 'The Boss', hobbies: ['leer', 'ver pelis'] },
    { name: 'Ana', role: 'becaria', hobbies: ['nadar', 'bailar'] },
    { name: 'Luis', role: 'programador', hobbies: ['dormir', 'comprar'] },
    { name: 'Carlos', role: 'secretario', hobbies: ['futbol', 'queso'] }
];
const result3 = staff.map(persona =>
    `${persona.name} es ${persona.role} y le gusta ${persona.hobbies.join(' y ')}`
);

// Ejercicio 4 - filter
const numbers2 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const result4 = numbers2.filter(num => num % 2 !== 0);

// Ejercicio 5 - filter + map + ternario + regex
const foodList2 = [
    { name: 'Tempeh', isVeggie: true },
    { name: 'Cheesbacon burguer', isVeggie: false },
    { name: 'Tofu burguer', isVeggie: true },
    { name: 'Entrecot', isVeggie: false }
];
const esFemenino = nombre => /a$|burguer$/i.test(nombre);
const result5 = foodList2
    .filter(plato => plato.isVeggie)
    .map(plato => `Que ric${esFemenino(plato.name) ? 'a' : 'o'} ${plato.name} me voy a comer!`);

// Ejercicio 6 - filter + map (encadenado)
const inventory = [
    { name: 'Mobile phone', price: 199 },
    { name: 'TV Samsung', price: 459 },
    { name: 'Viaje a Cancún', price: 600 },
    { name: 'Mascarilla', price: 1 }
];
const result6 = inventory
    .filter(producto => producto.price > 300)
    .map(producto => producto.name);

// Ejercicio 7 - reduce (producto)
const numeros3 = [39, 2, 4, 25, 62];
const result7 = numeros3.reduce((acumulador, numero) => acumulador * numero, 1);

// Ejercicio 8 - reduce (unir strings)
const sentenceElements = ['Me','llamo','Hector','y','quiero','sentir','la','fuerza','con','javascript'];
const result8 = sentenceElements.reduce((frase, elemento) => `${frase} ${elemento}`).trim();

// Ejercicio 9 - filter + reduce (suma con filtro)
const books = [
    { name: ' JS for dummies', author: 'Emily A. Vander Veer', price: 20, category: 'code' },
    { name: 'Don Quijote de la Mancha', author: 'Cervantes', price: 14, category: 'novel' },
    { name: 'Juego de tronos', author: 'George R. Martin', price: 32, category: 'Fantasy' },
    { name: 'javascript the good parts', author: 'Douglas Crockford', price: 40, category: 'code' }
];
const result9 = books
    .filter(libro => libro.category === 'code')
    .reduce((total, libro) => total + libro.price, 0);
```

---

## 🎓 Ejercicio final: conectar las 3 capas

Toma `git/Pagina-Web/` (HTML + CSS + JS) y responde:

1. **HTML → JS:** ¿qué selectores CSS usan las 4 partes de `programacion.js`? ¿Encuentras esos elementos en el HTML?
2. **JS → CSS:** ¿qué clases añade JavaScript al DOM? ¿Qué reglas CSS existen para esas clases?
3. ¿Qué pasaría si borras la línea `menuLateral.classList.toggle("abierto")`?
4. ¿Y si cambias `querySelector(".boton-hamburguesa")` por `querySelector(".no-existe")`?
5. ¿Por qué `if (elemento)` antes de la máquina de escribir?

---

## 🧪 Laboratorio: reproduce un efecto

Crea `prueba.html` con esto dentro y ábrelo en el navegador (`F12` → Console):

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Laboratorio JS</title>
  <style>
    body { font-family: sans-serif; padding: 30px; }
    .tarjeta { border: 1px solid #ddd; padding: 16px; margin-bottom: 12px; border-radius: 8px; }
    .tarjeta.oculta { opacity: 0; transform: translateY(20px); transition: .5s; }
    .tarjeta.visible { opacity: 1; transform: translateY(0); }
    .boton { padding: 10px 16px; border: 0; background: #e63946; color: #fff; border-radius: 6px; cursor: pointer; }
  </style>
</head>
<body>
  <h1 id="titulo">Laboratorio</h1>
  <button class="boton" id="mi-boton">Cámbiame</button>
  <div class="tarjeta oculta"><p>Primera tarjeta</p></div>
  <div class="tarjeta oculta"><p>Segunda tarjeta</p></div>
  <div class="tarjeta oculta"><p>Tercera tarjeta</p></div>

  <script>
    // 1) Seleccionar y modificar TEXTO
    const titulo = document.querySelector("#titulo");

    // 2) Evento click + classList.toggle
    document.querySelector("#mi-boton").addEventListener("click", () => {
      titulo.textContent = "¡Me has pulsado!";
      titulo.classList.toggle("visible");
    });

    // 3) Recorrer varios elementos
    document.querySelectorAll(".tarjeta").forEach((t, i) => {
      console.log("Tarjeta", i, t.textContent.trim());
    });

    // 4) IntersectionObserver → efecto reveal (igual que tu proyecto)
    const obs = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          entry.target.classList.add("visible");
          obs.unobserve(entry.target);
        }
      });
    }, { threshold: 0.2 });

    document.querySelectorAll(".tarjeta").forEach((el) => obs.observe(el));
  </script>
</body>
</html>
```

**Pruebas a hacer:**
1. Abre `F12` → **Console** y mira los `console.log`.
2. Pulsa el botón → cambia el título.
3. Haz scroll lentamente → las tarjetas aparecen una a una.
4. En **Elements**, observa cómo cambian las clases en tiempo real.

👉 **Vuelve al índice:** `00-INDICE.md`
