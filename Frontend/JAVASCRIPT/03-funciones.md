# 03 — FUNCIONES

> Una **función** es un bloque de código con nombre que se **define una vez** y se
> **ejecuta (llama) muchas veces**. Es la unidad básica de organización en JS.

---

## 3.1 Declaración de función (la clásica)

```js
function nombre(parametro1, parametro2) {
    // cuerpo
    return resultado;          // devuelve algo y SALE
}
```

**Puntos clave:**
- `function` + **nombre** + `()` + `{}`
- Los **parámetros** van dentro de los paréntesis (máximo 2-3; si necesitas más,
  pasa un objeto).
- `return` **termina** la función y devuelve un valor.
- Si no hay `return`, devuelve `undefined`.
- Se puede llamar **antes** de declararla (*hoisting*).

### Ejemplo REAL copiado (`bikram2-hector/bikram2-hector/index.js`)

```js
// 5
function suma(a, b) {
  return a + b;
}

// 6
function potenciacion(a, b) {
  return Math.pow(a, b);
}

// 7
function separarPalabras(texto) {
  return texto.split(' ');
}

// 19
function multiplicacion(a, b) {
  return a * b;
}

// 22
function resta(a, b) {
  return a - b;
}

// 20
function division(a, b) {
  return a / b;
}
```

**Uso:**

```js
suma(3, 4);        // 7
potenciacion(2, 5); // 32
separarPalabras("hola que tal");   // ["hola", "que", "tal"]
```

---

## 3.2 `return` — devolver vs. mostrar

```js
// ❌ No devuelve nada
function suma(a, b) {
  console.log(a + b);      // solo MUESTRA por consola
}

// ✅ Devuelve
function suma(a, b) {
  return a + b;
}

const resultado = suma(3, 4);   // resultado = 7  → se puede USAR
```

```js
// return TERMINA la función
function validar(edad) {
  if (edad < 0) return "edad inválida";   // sale aquí
  if (edad < 18) return "menor";          // no llega si ya salió
  return "adulto";
}
```

### Función con varias salidas (`algoritmia/Algoritmia-Hector/index.js`)

```js
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
  return numeros;      // ← devuelve el array completo
}
```

> 📌 Toda la lógica del `while` ocurre **dentro**, y lo único que sale es lo que
> indica el `return`.

---

## 3.3 Ejemplos REALES: funciones con bucles y condiciones

**Archivo:** `bikram2-hector/bikram2-hector/index.js` *(copiado tal cual)*

```js
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

// 23
function ordenarArray2(arr) {
  return arr.sort(function (a, b) {
    return b - a;
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

// 13  ← ¡aquí una función recibe OTRA función como parámetro!
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
```

### 🧠 Observa `arrayMapi` — un **callback**

```js
function arrayMapi(arr, funcion) { ... }
//                    ▲
//        recibe una función como PARÁMETRO
```

Uso:

```js
function doble(n) { return n * 2; }

arrayMapi([1, 2, 3], doble);   // [2, 4, 6]
```

> 📌 Ese es el **origen de `Array.map()`**: una función que recorre un array y le
> aplica **otra** función a cada elemento. Entender esto es entender `map/filter/reduce`.

---

## 3.4 Funciones como valores (arrays de funciones)

**Ejemplo REAL (`bikram2-hector/bikram2-hector/index.js`):**

```js
let arrayFunciones = [suma, resta, multiplicacion];
```

```js
// Uso
arrayFunciones[0](10, 5);   // suma(10, 5) → 15
arrayFunciones[1](10, 5);   // resta(10, 5) → 5
arrayFunciones[2](10, 5);   // multiplicacion(10, 5) → 50
```

> 📌 **Sin paréntesis** (`suma`, no `suma()`) = la **función misma** como valor.
> **Con paréntesis** (`suma()`) = su **resultado**.

---

## 3.5 Expresión de función y `function` anónima

```js
// Declaración (con nombre, se eleva)
function saludar() { return "hola"; }

// Expresión (asignada a una variable)
const saludar2 = function () { return "hola"; };

// Callback anónima (muy común)
arr.sort(function (a, b) { return a - b; });
```

**Ejemplo REAL** (`bikram2-hector/index.js`):

```js
function ordenarArray(arr) {
  return arr.sort(function (a, b) {
    return a - b;     // a - b = ascendente
  });
}
```

Y su versión inversa:

```js
function ordenarArray2(arr) {
  return arr.sort(function (a, b) {
    return b - a;     // b - a = descendente
  });
}
```

> 📌 `sort()` necesita una **función comparadora**: si devuelve **negativo**, `a`
> va antes; **positivo**, `b` va antes; `0`, no cambian.
> ⚠️ Por defecto `sort()` ordena **como texto** (`[10, 9, 1]`), por eso hace falta
> el comparador `a - b` para ordenar numéricamente.

---

## 3.6 Función flecha (arrow function) — ES6

```js
// Clásica
function suma(a, b) {
  return a + b;
}

// Flecha — equivalente
const suma = (a, b) => {
  return a + b;
};

// Flecha con retorno IMPLÍCITO (una sola expresión: se quitan `{}` y `return`)
const suma = (a, b) => a + b;

// Un solo parámetro: se pueden quitar los paréntesis
const doble = n => n * 2;

// Sin paréntesis: son obligatorios
const nada = () => "hola";

// Objeto: hay que envolverlo en paréntesis
const crear = () => ({ nombre: "Hector" });
```

### Tabla de equivalencias

```js
n => n * 10                    //  function (n) { return n * 10; }
(x, y) => x + y                //  function (x, y) { return x + y; }
() => 42                       //  function () { return 42; }
item => item.price             //  function (item) { return item.price; }
```

---

## 3.7 Ejemplos REALES de arrow functions

### **Archivo:** `bikram-advance-js/bikram-map-filter-reduce-hector/index.js`

```js
// Ejercicio 1
const numbers = [4, 5, 6, 7, 8, 9, 10];
const elevados = () => numbers.map(num => Math.pow(num, num));

// Ejercicio 2
const foodList = ['Pizza', 'Ramen', 'Paella', 'Entrecot'];

const frasesComida = {
    Pizza: 'Como soy de Italia, amo comer Pizza',
    Ramen: 'Como soy de Japón, amo comer Ramen',
    Paella: 'Como soy de Valencia, amo comer Paella',
    Entrecot: 'Aunque no como carne, el Entrecot es sabroso'
};

const result2 = foodList.map(comida => frasesComida[comida]);

// Ejercicio 4
const numbers2 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const result4 = numbers2.filter(num => num % 2 !== 0);
```

**Desglose de `foodList.map(comida => frasesComida[comida])`:**

```
foodList.map(...)              → recorre el array y crea otro nuevo
comida => ...                  → arrow function con 1 parámetro (sin paréntesis)
frasesComida[comida]           → acceso por CLAVE: "Pizza" → frasesComida["Pizza"]
```

Resultado:
```js
[
  'Como soy de Italia, amo comer Pizza',
  'Como soy de Japón, amo comer Ramen',
  'Como soy de Valencia, amo comer Paella',
  'Aunque no como carne, el Entrecot es sabroso'
]
```

> 📌 **Truco:** `frasesComida[comida]` usa el **valor del array como clave** del objeto.
> Es una forma muy elegante de "traducir" valores.

### **Archivo:** `bikram destructuring hector/.../index.js`

```js
// 5.- Suma cualquier cantidad de números
function sumEveryOther(...numeros) {
    return numeros.reduce((total, num) => total + num, 0);
}

// 7.- Devuelve cuántos argumentos ha recibido
function countTheArgs(...args) {
    return args.length;
}

// 9.- Devuelve un array con los elementos únicos (sin repetidos)
function onlyUniques(...args) {
    return [...new Set(args)];
}
```

---

## 3.8 Parámetros por defecto y argumentos variables

```js
// Valor por defecto (si no llega nada, usa 10)
function multiplicar(a, b = 10) {
  return a * b;
}
multiplicar(5);      // 50
multiplicar(5, 2);   // 10

// Rest ... : recoge TODOS los argumentos en un array
function sumaTodo(...numeros) {
  return numeros.reduce((total, n) => total + n, 0);
}
sumaTodo(1, 2, 3, 4);   // 10
```

**Ejemplo REAL (`bikram destructuring hector/.../index.js`):**

```js
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

// 8.- Combina dos arrays en uno solo usando spread
function combineTwoArrays(array1, array2) {
    return [...array1, ...array2];
}
```

---

## 3.9 Alcance (scope) y `const`/`let`

```js
const global = "soy global";

function miFuncion() {
  const local = "soy local";
  console.log(global);   // ✅ ve lo de fuera
  console.log(local);    // ✅ ve lo suyo
}

console.log(local);      // ❌ ReferenceError: no existe fuera
```

| Scope | Dónde vive |
|-------|-----------|
| **Global** | Fuera de toda función → visible en todo el archivo |
| **De función** | Dentro de `function` o `{}` de arrow/`for`/`if` con `let` |
| **De bloque** | `{ ... }` de un `if`, `for`, `while` con `let`/`const` |

> 📌 **Regla:** usa el **scope más pequeño posible**. Deja las variables donde se usan.

### Sombreado (shadowing)

```js
const color = "rojo";

function pintar() {
  const color = "azul";   // "sombrea" a la de fuera SOLO dentro de la función
  console.log(color);     // "azul"
}

console.log(color);       // "rojo"
```

---

## 3.10 Callbacks y funciones de orden superior

```js
// Función de orden superior: recibe o devuelve una función
function aplicarOperacion(a, b, operacion) {
  return operacion(a, b);
}

aplicarOperacion(5, 3, suma);            // 8
aplicarOperacion(5, 3, (x, y) => x * y); // 15
```

**Ejemplo REAL (`bikram2-hector/index.js`):**

```js
let arrayFunciones = [suma, resta, multiplicacion];
```

### Patrón de callback en eventos (avance del DOM)

**Archivo:** `git/Pagina-Web/js/programacion.js`

```js
botonMenu.addEventListener("click", function () {
  menuLateral.classList.toggle("abierto");
});
```

> 📌 `addEventListener("click", función)` → la función **no se llama ahora**;
> se guarda y el navegador la ejecuta **cuando haces clic**. Ese es el concepto
> de **callback** en su forma más pura.

---

## 3.11 Funciones autoinvocadas (IIFE)

```js
(function () {
  console.log("Se ejecuta sola al cargar");
})();

// Moderno
(() => {
  console.log("IIFE con arrow");
})();
```

Se usan para tener un **scope privado** sin contaminar el global.

---

## 3.12 Errores comunes con funciones

| Error | Solución |
|-------|----------|
| `undefined` al llamar | Olvidaste el `return` |
| Se ejecuta sola y no cuando la llamas | Pusiste `()` al usarla como callback: `addClick(fn())` en vez de `fn` |
| `is not a function` | La variable no contiene una función, o está declarada después |
| Parámetros vacíos | Falta pasar argumentos → `undefined` dentro |
| Duplicar código | Sácalo a una función |

```js
// ❌ Se ejecuta AL REGISTRAR, no al clicar
boton.addEventListener("click", mostrarAlerta());

// ✅ Se ejecuta al CLICAR
boton.addEventListener("click", mostrarAlerta);

// ✅ También (función anónima)
boton.addEventListener("click", () => mostrarAlerta());
```

---

## ✅ Resumen rápido

1. `function nombre(params) { return valor; }`
2. `return` devuelve **y termina**; sin `return` → `undefined`.
3. Arrow: `(a, b) => a + b` (retorno implícito con `=>` sin `{}`).
4. Los **callbacks** son funciones pasadas como parámetro → base de `map`, `filter`, `reduce` y eventos.
5. Una función **sin `()`** es un valor; **con `()`** es su resultado.
6. `...args` recoge argumentos en un array (rest).
7. `b = 10` da valores por defecto.
8. Scope: `const`/`let` viven en su bloque.

👉 **Siguiente:** `04-arrays-y-metodos-map-filter-reduce.md`
