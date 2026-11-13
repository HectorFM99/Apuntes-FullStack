# 01 — SINTAXIS BÁSICA, VARIABLES Y TIPOS DE DATOS

---

## 1.1 Dónde vive el JavaScript

```html
<!-- Opción A: archivo externo (RECOMENDADA) -->
<script src="js/programacion.js"></script>

<!-- Opción B: al final del <body> (clásico, garantiza que el HTML ya existe) -->
<script src="js/programacion.js"></script>
</body>

<!-- Opción C: en línea (solo para pruebas) -->
<button onclick="alert('hola')">Pulsa</button>

<!-- Opción D: dentro del HTML -->
<script>
  console.log("Hola desde el head");
</script>
```

**Ejemplos REALES copiados de tus proyectos:**

`git/Pagina-Web/index.html` (al final del body):
```html
<script src="js/programacion.js"></script>
</body>
</html>
```

`git/Pagina-Web/pages/sobre_mi.html` (con ruta `../`):
```html
<script src="../js/programacion.js"></script>
```

> 📌 **`defer`** es la opción moderna recomendada si el `<script>` está en el `<head>`:
> ```html
> <head>
>     <script src="app.js" defer></script>
> </head>
> ```
> Se ejecuta **después** de parsear todo el HTML y respeta el orden de los archivos.

---

## 1.2 Comentarios

```js
// Comentario de una línea

/*
   Comentario
   de varias líneas
*/

/** JSDoc — documentación */
/**
 * Suma dos números.
 * @param {number} a
 * @param {number} b
 * @returns {number}
 */
```

### Ejemplo real (`varita-magica/varita-magica-hector/scripts.js`)

```js
// 1. Evitar el comportamiento por defecto al hacer click (por ejemplo, que un enlace no navegue a otra página)

document.addEventListener('click', (event) => {
    event.preventDefault();
    ...
});

// 3. Al pasar el cursor por encima, y devolver al estado original cuando el cursor sale del elemento
```

### Ejemplo real con comentarios dentro de la lógica (`bichomon/bichomon_hector/scripts.js`)

```js
// 4. Imprime por consola el dominio de la página
//    OJO: document.domain está DEPRECATED asi que le pedi a ChatGTP que me diera otra opcion
// 			y me dijo que usara location.hostname, dejo el original como texto por si acaso.

console.log('4) Dominio:', location.hostname);
// console.log('4) Dominio (deprecated):', document.domain) el original lo tenía así, pero da warning en la consola
```

---

## 1.3 Las tres formas de declarar una variable

```js
var nombre = "Hector";    // antigua: tiene SCOPE de función y se eleva (hoisting)
let contador = 0;         // moderna: SCOPE de bloque, se puede reasignar
const PI = 3.14;          // moderna: SCOPE de bloque, NO se puede reasignar
```

| | `var` | `let` | `const` |
|---|---|---|---|
| ¿Se puede reasignar? | Sí | Sí | **No** |
| ¿Se puede redeclarar? | Sí | No | No |
| Scope | Función | **Bloque** | **Bloque** |
| ¿Se eleva? | Sí (inicializada `undefined`) | Sí (sin inicializar) | Sí (sin inicializar) |
| Uso actual | ❌ Evitar | ✅ Valor que cambia | ✅ **Por defecto** |

> 📌 **Regla moderna:** usa **`const` por defecto**. Si el valor va a cambiar, usa `let`.
> **No uses `var`** en código nuevo.

### Ejemplos REALES copiados

**Archivo:** `bikram hector/bikram-hector/index.js`

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

// 13
let variableValorNumerico = 7;

// 14
const MiNombre = "Hector"; 

// 15
const MiNumeroFav = 5;
```

> 📌 Fíjate: `PI` y `TAU` son `const` (no cambian), mientras que
> `variableValorNumerico` es `let` (podría reasignarse).

**Archivo:** `proyecto/programacion.js` *(copiado íntegro)*

```js
let nombre = "Hector";
let apellido = "Franco Magallo";
let edad = 25;
```

### Errores típicos

```js
const edad = 25;
edad = 26;            // ❌ TypeError: Assignment to constant variable

let a = 1;
let a = 2;            // ❌ SyntaxError: ya declarada

if (true) {
  let x = 1;
}
console.log(x);       // ❌ ReferenceError: x es de SCOPE de bloque

if (true) {
  var y = 1;
}
console.log(y);       // ✅ 1 (var escapa del bloque)
```

---

## 1.4 Tipos de datos

### Primitivos

| Tipo | Ejemplo | Descripción |
|------|---------|-------------|
| `string` | `"Hector"`, `'Hola'`, `` `Hola ${x}` `` | Texto (comillas simples, dobles o backticks) |
| `number` | `25`, `3.14`, `-7`, `Infinity`, `NaN` | Números (enteros y decimales son el **mismo** tipo) |
| `boolean` | `true`, `false` | Verdadero / falso |
| `undefined` | `let x;` | Declarada pero sin valor |
| `null` | `x = null;` | Vacío **intencionado** |
| `bigint` | `10n` | Enteros gigantes |
| `symbol` | `Symbol("id")` | Identificador único |

### No primitivo

| Tipo | Ejemplo |
|------|---------|
| `object` | `{}`, `[]`, `function`, `Date`, `RegExp` |
| `array` | `[1, 2, 3]` (es un `object` por debajo) |

### Comprobar el tipo: `typeof`

```js
typeof "Hector"      // "string"
typeof 25            // "number"
typeof true          // "boolean"
typeof undefined     // "undefined"
typeof null          // "object"   ⚠️ BUG histórico de JS
typeof [1,2]         // "object"   ⚠️ array = object
typeof {}            // "object"
```

> ⚠️ **`typeof null` devuelve `"object"`** y **`typeof []` también**.
> Para arrays usa `Array.isArray([1,2])` → `true`.

### Ejemplo real de `typeof` (`bikram destructuring hector/.../index.js`)

```js
// 6.- Suma solo los números, ignorando los strings
function addOnlyNums(...args) {
    return args
        .filter(arg => typeof arg === "number")
        .reduce((total, num) => total + num, 0);
}
```

> 📌 `typeof arg === "number"` filtra **solo** los argumentos numéricos.

---

## 1.5 Conversión de tipos

```js
// A número
Number("25")        // 25
parseInt("25 años") // 25   → entero, corta en el primer no-número
parseFloat("3.14")  // 3.14
+"123"              // 123   (coerción rápida)
Number("hola")      // NaN

// A texto
String(25)          // "25"
(25).toString()     // "25"
25 + ""             // "25"

// A booleano
Boolean(0)          // false  ← ¡cuidado!
Boolean("")         // false
Boolean(null)       // false
Boolean(undefined)  // false
Boolean(NaN)        // false
Boolean("hola")     // true
Boolean(25)         // true

// Comprobación de "no es número"
Number.isNaN(valor)
```

### Ejemplo REAL de `parseInt` + `isNaN` (`algoritmia/Algoritmia-Hector/index.js`)

```js
function pedirNumeros() {
  const numeros = [];

  while (true) {
    const entrada = prompt("Introduce un número (0 para terminar):");

    const numero = parseInt(entrada);

    if (isNaN(numero)) break;   // si no es número, se sale
    if (numero === 0) break;    // si es 0, se sale
    numeros.push(numero);
  }

  numeros.sort((a, b) => a - b);   // ordena de menor a mayor
  return numeros;
}
```

> 📌 **Flujo completo:** `prompt` devuelve **texto** → `parseInt` lo convierte en
> número → `isNaN` detecta si la conversión falló → `push` lo guarda →
> `sort` lo ordena → `return` devuelve el array.

---

## 1.6 Operadores

### Aritméticos

```js
+   -   *   /      // suma resta multiplicación división
%                  // RESTO (módulo) → clave para "¿es par?"
**                 // potencia: 2 ** 3 === 8
++  --             // incrementar / decrementar
```

**Ejemplo REAL (`bikram hector/bikram-hector/index.js`):**

```js
// 12 - Sumar los pares del 0 al 9
let sumaPares = 0;
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) {
    sumaPares += i;
  }
}

// 26 - Sumar los impares del 0 al 9
let sumaImpares = 0;
for (let i = 0; i < 10; i++) {
  if (i % 2 !== 0) {
    sumaImpares += i;
  }
}
```

> 📌 **`i % 2 === 0`** → "el resto de dividir entre 2 es 0" = **número par**.

### Incrementadores (¡cuidado con el orden!)

```js
// 8 y 9 del mismo proyecto
let incrementarDesp = 2;
let resultadoDesp = incrementarDesp++;   // POSTFIJO: primero usa 2, luego suma → resultado = 2

let incrementarAntes = 2;
let resultadoAntes = ++incrementarAntes; // PREFIJO: primero suma, luego usa → resultado = 3
```

**Ejemplo real del mismo archivo:**

```js
// 11
let postI = 0;
let postJ = 0;
for (let i = 0; i < 11; i++) {
  postI += postJ++;     // postJ se usa ANTES de incrementar
}

// 25
let preI = 0;
let preJ = 0;
for (let i = 0; i < 11; i++) {
  preI += ++preJ;       // preJ se incrementa ANTES de usarse
}
```

### Comparación

```js
==     // Igualdad PERMISIVA (convierte tipos antes) → EVITAR
===    // Igualdad ESTRICTA (misma valor Y mismo tipo) → USAR SIEMPRE
!=     // Distinto permisivo
!==    // Distinto estricto
>  <  >=  <=
```

```js
"1" == 1     // true   (convierte el string a número)
"1" === 1    // false  (string !== number)  ✅ más seguro
0 == false   // true
0 === false  // false  ✅
```

**Ejemplo real (`bikram hector/bikram-hector/index.js`):**

```js
// 18
let seisNoEsNueve = 6 !== 9;    // true
```

> 📌 **Regla de oro:** usa siempre **`===` y `!==`**.

### Asignación

```js
=    +=   -=   *=   /=   %=   **=
```

```js
let total = 0;
total = total + 5;   // lo largo
total += 5;          // lo corto  (¡ambos valen 5!)
```

### Operadores de texto

```js
"Hola" + " " + "Mundo"           // "Hola Mundo"  (concatenación)
`Hola ${nombre}, tienes ${edad} años`   // plantilla literal (backticks)
"Hola".length                    // 4
"Hola".toUpperCase()             // "HOLA"
"hola".split("")                 // ["h","o","l","a"]
```

---

## 1.7 Plantillas literales (template strings)

```js
const nombre = "Hector";
const edad = 25;

// ❌ forma antigua
console.log("Hola, me llamo " + nombre + " y tengo " + edad + " años.");

// ✅ forma moderna
console.log(`Hola, me llamo ${nombre} y tengo ${edad} años.`);
```

**Ejemplo REAL (`bikram-advance-js/bikram-map-filter-reduce-hector/index.js`):**

```js
const result3 = staff.map(persona =>
    `${persona.name} es ${persona.role} y le gusta ${persona.hobbies.join(' y ')}`
);
```

Con este `staff`:

```js
const staff = [
    { name: 'Pepe',  role: 'The Boss',     hobbies: ['leer', 'ver pelis'] },
    { name: 'Ana',   role: 'becaria',      hobbies: ['nadar', 'bailar'] },
    { name: 'Luis',  role: 'programador',  hobbies: ['dormir', 'comprar'] },
    { name: 'Carlos',role: 'secretario',   hobbies: ['futbol', 'queso'] }
];
```

Resultado:
```
"Pepe es The Boss y le gusta leer y ver pelis"
"Ana es becaria y le gusta nadar y bailar"
```

> 📌 `${...}` permite **cualquier expresión JS**: operaciones, métodos, ternarios...

**Otro ejemplo real del mismo archivo:**

```js
const result5 = foodList2
    .filter(plato => plato.isVeggie)
    .map(plato => `Que ric${esFemenino(plato.name) ? 'a' : 'o'} ${plato.name} me voy a comer!`);
```

Aquí hay un **operador ternario** dentro de la plantilla:
`condición ? valorSiTrue : valorSiFalse`.

---

## 1.8 Operadores lógicos y ternario

```js
&&    // Y lógico (ambos deben ser true)
||    // O lógico (al menos uno true)
!     // NOT (invierte)
? :   // ternario: condición ? siTrue : siFalse
??    // nullish coalescing: devuelve el valor si es null/undefined
?.    // optional chaining: accede sin fallar si es null/undefined
```

**Ejemplo REAL (`bikram hector/bikram-hector/index.js`):**

```js
// 5
let booleanoAnd = booleano1 && booleano2;

// 6
let booleanoNot = !booleano1;

// 7
let booleanoMix0 = (booleano1 || booleano2) && (booleano1 || (!booleano1 && !booleano2));

// 16
let booleanoOr = booleano1 || booleano2;

// 17
let booleanoMix1 = (booleano1 && (TAU / 2 === PI)) || (variableValorNumerico >= MiNumeroFav);
```

**Ejemplo REAL de bisiesto (`algoritmia/Algoritmia-Hector/index.js`):**

```js
function esBisiesto(año) {
  return (año % 4 === 0 && año % 100 !== 0) || (año % 400 === 0);
}
const año = parseInt(prompt("Introduce un año:"));
if (esBisiesto(año)) {
  console.log(año + " es un año bisiesto.");
} else {
  console.log(año + " no es un año bisiesto.");
}
```

> 📌 Este `return` es un ejemplo perfecto de **paréntesis + `&&` + `||`**:
> se lee "es bisiesto si es divisible entre 4 **y no** entre 100, **o** es divisible entre 400".

---

## 1.9 La consola: tu mejor amiga

```js
console.log("valor normal");
console.log("múltiples:", a, b, c);
console.table([{ nombre: "Ana", edad: 20 }, { nombre: "Luis", edad: 30 }]);
console.error("error");
console.warn("aviso");
console.time("medir");
console.timeEnd("medir");
console.group("grupo"); console.groupEnd();
console.dir(objeto);
```

**Ejemplos REALES (`bichomon/bichomon_hector/scripts.js`):**

```js
const tituloGeneracion1 = document.querySelector('#gen-1');
tituloGeneracion1.textContent = 'Generasión 1 Pokimon';
console.log('1) Nuevo título:', tituloGeneracion1.textContent);

console.log('3) URL:', window.location.href);

const imagenes = document.querySelectorAll('img');
console.log('5) Nodos de imagen (' + imagenes.length + '):', imagenes);
```

**`varita-magica/varita-magica-hector/scripts.js`:**

```js
// Comprobación de la función getRandom
console.log(getRandom(['red', 'blue', 'green']));
```

> 📌 Cómo **depurar**: `console.log()` de lo que sospeches + `F12` → pestaña *Console*.
> También `debugger;` en el código pausa la ejecución en las DevTools.

---

## 1.10 Errores comunes al empezar

| Error | Causa | Solución |
|-------|-------|----------|
| `Uncaught ReferenceError: x is not defined` | Variable no declarada o fuera de scope | Declárala con `let`/`const` |
| `Uncaught TypeError: Cannot read properties of undefined` | Acceder a propiedad de algo inexistente | Comprueba antes con `if (x)` o `x?.prop` |
| `Assignment to constant variable` | Reasignar un `const` | Usa `let` |
| SyntaxError: Unexpected token | Falta `;`, `}` o `)` | Revisa el bloque anterior |
| No pasa nada al pulsar | El JS se cargó **antes** que el HTML | Mueve el `<script>` al final del `<body>` o usa `defer` |
| Ruta no encontrada | `src` mal escrito | Revisa `js/archivo.js` vs `../js/archivo.js` |

---

## ✅ Resumen rápido

1. `const` por defecto, `let` si cambia, `var` no.
2. Tipos: `string`, `number`, `boolean`, `undefined`, `null`, `object`.
3. `typeof` para comprobar tipos; `Array.isArray()` para arrays.
4. `parseInt()` + `isNaN()` para leer números de `prompt()`/`input`.
5. Usa **`===`** y **`!==`** siempre.
6. `i % 2 === 0` = par.
7. Plantillas con **backticks** y `${expresión}`.
8. `&&`, `||`, `!` y el ternario `cond ? a : b`.
9. `console.log()` para depurar.

👉 **Siguiente:** `02-condicionales-y-bucles.md`
