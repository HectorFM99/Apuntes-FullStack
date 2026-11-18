# 04 — ARRAYS Y MÉTODOS `MAP`, `FILTER`, `REDUCE`

---

## 4.1 ¿Qué es un array?

Una **lista ordenada** de valores bajo un mismo nombre.

```js
const vacio = [];
const numeros = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
const pares = [0, 2, 4, 6, 8];
const bidimensional = [[0, 1, 2], ['a', 'b', 'c']];
const mezcla = ['hola', 'que', 23, 42.33, 'tal'];
```

### Ejemplos REALES copiados (`bikram2-hector/bikram2-hector/index.js`)

```js
// 1
let arrayVacio = [];

// 2
let arrayNumeros = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];

// 3
let arrayNumerosPares = [0, 2, 4, 6, 8];

// 4
let arrayBidimensional = [[0, 1, 2], ['a', 'b', 'c']];

// 15
let arrayNumerosNeg = [0, -1, -2, -3, -4, -5, -6, -7, -8, -9];

// 16
let holaMundo = ['Hola', 'Mundo'];

// 17
let loGuardoTodo = ['hola', 'que', 23, 42.33, 'tal'];

// 18
let arrayDeArrays = [[756, 'nombre'], [225, 'apellido'], [298, 'direccion']];
```

> 📌 **Un array es un `object`** con índice numérico. Los arrays **pueden mezclar tipos**.

---

## 4.2 Acceso y modificación

```js
const frutas = ["manzana", "pera", "uva"];

frutas[0];              // "manzana"   ← ¡EMPIEZA EN 0!
frutas[frutas.length-1];// última
frutas.length;          // 3
frutas[1] = "plátano";  // modificar
```

> ⚠️ **Los índices empiezan en 0.** El último elemento es `arr[arr.length - 1]`.

### Anidamiento

```js
const matriz = [[0, 1, 2], ['a', 'b', 'c']];

matriz[0];       // [0, 1, 2]
matriz[0][1];    // 1
matriz[1][2];    // "c"
```

---

## 4.3 Métodos principales

| Método | Qué hace | ¿Mutu el original? |
|--------|----------|---------------------|
| `push(el)` | Añade al **final** | ✅ Sí |
| `pop()` | Elimina el **último** | ✅ Sí |
| `shift()` | Elimina el **primero** | ✅ Sí |
| `unshift(el)` | Añade al **principio** | ✅ Sí |
| `indexOf(x)` | Posición de `x` (`-1` si no está) | No |
| `includes(x)` | ¿Contiene `x`? | No |
| `join(sep)` | Convierte a **string** unido | No |
| `concat(a)` | Junta arrays | No |
| `slice(a, b)` | Extrae (no modifica) | No |
| `splice(a, n)` | Inserta/elimina (modifica) | ✅ Sí |
| `reverse()` | Invierte | ✅ Sí |
| `sort(fn)` | Ordena (⚠️ por defecto, como texto) | ✅ Sí |
| `forEach(fn)` | Recorre | No |
| `map(fn)` | **Transforma** → array nuevo | No |
| `filter(fn)` | **Filtra** → array nuevo | No |
| `reduce(fn, init)` | **Reduce** a un único valor | No |
| `find(fn)` | Primer elemento que cumple | No |
| `findIndex(fn)` | Índice del primero que cumple | No |
| `some(fn)` | ¿Alguno cumple? | No |
| `every(fn)` | ¿Todos cumplen? | No |
| `flat()` | Aplana arrays anidados | No |

---

## 4.4 Ejemplos REALES: recorrer y transformar con `for`

**Archivo:** `bikram2-hector/bikram2-hector/index.js` *(copiado tal cual)*

```js
// 11 - Obtener los números pares de un array
function obtenerPares(arr) {
  let pares = [];
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] % 2 === 0) {
      pares.push(arr[i]);
    }
  }
  return pares;
}

// 24 - Obtener los impares
function obtenerImpares(arr) {
  let impares = [];
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] % 2 !== 0) {
      impares.push(arr[i]);
    }
  }
  return impares;
}

// 25 - Sumar todos los elementos
function sumarArray(arr) {
  let total = 0;
  for (let i = 0; i < arr.length; i++) {
    total = total + arr[i];
  }
  return total;
}

// 26 - Multiplicar todos los elementos
function multiplicarArray(arr) {
  let total = 1;
  for (let i = 0; i < arr.length; i++) {
    total = total * arr[i];
  }
  return total;
}

// 12 - Convertir array a texto
function pintarArray(arr) {
  return '[' + arr.join(', ') + ']';
}

// 14 - Eliminar duplicados
function eliminarDuplicados(arr) {
  let resultado = [];
  for (let i = 0; i < arr.length; i++) {
    if (!resultado.includes(arr[i])) {
      resultado.push(arr[i]);
    }
  }
  return resultado;
}

// 13 - NUESTRO PROPIO "map" (para entenderlo)
function arrayMapi(arr, funcion) {
  let resultado = [];
  for (let i = 0; i < arr.length; i++) {
    resultado.push(funcion(arr[i]));
  }
  return resultado;
}
```

> 📌 `arrayMapi` **es** `Array.map` hecho a mano. Compáralo con la versión real:
> ```js
> arr.map(funcion)
> ```
> La única diferencia es que el real viene de serie y está optimizado.

---

## 4.5 `map()` — transformar cada elemento → **array nuevo**

```js
const resultado = array.map(elemento => loQueSea);
```

- Recorre **todos** los elementos.
- Lo que devuelve cada vuelta se guarda en un **array nuevo**.
- El original **no se toca**.
- El array nuevo tiene **la misma longitud**.

### Ejemplos REALES (`bikram-advance-js/bikram-map-filter-reduce-hector/index.js`)

```js
// Ejercicio 1: elevar cada número a sí mismo
const numbers = [4, 5, 6, 7, 8, 9, 10];
const elevados = () => numbers.map(num => Math.pow(num, num));
// [4**4, 5**5, 6**6, ...]

// Ejercicio 2: traducir cada plato a su frase
const foodList = ['Pizza', 'Ramen', 'Paella', 'Entrecot'];

const frasesComida = {
    Pizza: 'Como soy de Italia, amo comer Pizza',
    Ramen: 'Como soy de Japón, amo comer Ramen',
    Paella: 'Como soy de Valencia, amo comer Paella',
    Entrecot: 'Aunque no como carne, el Entrecot es sabroso'
};

const result2 = foodList.map(comida => frasesComida[comida]);

// Ejercicio 3: construir una frase por cada persona
const staff = [
    { name: 'Pepe',   role: 'The Boss',    hobbies: ['leer', 'ver pelis'] },
    { name: 'Ana',    role: 'becaria',     hobbies: ['nadar', 'bailar'] },
    { name: 'Luis',   role: 'programador', hobbies: ['dormir', 'comprar'] },
    { name: 'Carlos', role: 'secretario',  hobbies: ['futbol', 'queso'] }
];

const result3 = staff.map(persona =>
    `${persona.name} es ${persona.role} y le gusta ${persona.hobbies.join(' y ')}`
);
```

**Resultado de `result3`:**
```js
[
  "Pepe es The Boss y le gusta leer y ver pelis",
  "Ana es becaria y le gusta nadar y bailar",
  "Luis es programador y le gusta dormir y comprar",
  "Carlos es secretario y le gusta futbol y queso"
]
```

> 📌 **`map` = "cada uno → algo nuevo"**. Nunca filtra, nunca salta: **transforma 1 a 1**.

---

## 4.6 `filter()` — quedarse con los que cumplen → **array nuevo (más corto)**

```js
const resultado = array.filter(elemento => condicion);
```

### Ejemplos REALES

```js
// Ejercicio 4: los impares
const numbers2 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const result4 = numbers2.filter(num => num % 2 !== 0);
// [1, 3, 5, 7, 9]

// Ejercicio 5: solo los platos vegetarianos
const foodList2 = [
    { name: 'Tempeh',              isVeggie: true },
    { name: 'Cheesbacon burguer',  isVeggie: false },
    { name: 'Tofu burguer',        isVeggie: true },
    { name: 'Entrecot',            isVeggie: false }
];

const esFemenino = nombre => /a$|burguer$/i.test(nombre);

const result5 = foodList2
    .filter(plato => plato.isVeggie)       // solo los vegetarianos
    .map(plato => `Que ric${esFemenino(plato.name) ? 'a' : 'o'} ${plato.name} me voy a comer!`);

// Ejercicio 6: productos de más de 300€ → solo nombres
const inventory = [
    { name: 'Mobile phone',   price: 199 },
    { name: 'TV Samsung',     price: 459 },
    { name: 'Viaje a Cancún', price: 600 },
    { name: 'Mascarilla',     price: 1 }
];

const result6 = inventory
    .filter(producto => producto.price > 300)
    .map(producto => producto.name);
// ["TV Samsung", "Viaje a Cancún"]
```

> 📌 **`filter` = "cuáles pasan el corte"**. Devuelve `true` para **quedárselo**.
> Ojo: `plato.isVeggie` **ya es un booleano**, por eso no hace falta
> `plato => plato.isVeggie === true`.

---

## 4.7 `reduce()` — acumular en UN solo valor

```js
const resultado = array.reduce((acumulador, elemento) => {
    // lógica
    return nuevoAcumulador;
}, valorInicial);
```

- `acumulador` → el "cubo" donde vas guardando el resultado.
- `valorInicial` → con qué valor arranca (si no se pone, arranca con el primer elemento).

### Ejemplos REALES

```js
// Ejercicio 7: producto de todos
const numeros3 = [39, 2, 4, 25, 62];
const result7 = numeros3.reduce((acumulador, numero) => acumulador * numero, 1);
// 39 * 2 * 4 * 25 * 62 = 483.600
//                    ▲ valor inicial 1 (neutro de multiplicar)

// Ejercicio 8: juntar todas las palabras en una frase
const sentenceElements = ['Me','llamo','Hector','y','quiero','sentir','la','fuerza','con','javascript'];

const result8 = sentenceElements.reduce((frase, elemento) => `${frase} ${elemento}`).trim();
// "Me llamo Hector y quiero sentir la fuerza con javascript"

// Ejercicio 9: precio total de los libros de código
const books = [
    { name: ' JS for dummies',            author: 'Emily A. Vander Veer', price: 20, category: 'code' },
    { name: 'Don Quijote de la Mancha',   author: 'Cervantes',            price: 14, category: 'novel' },
    { name: 'Juego de tronos',            author: 'George R. Martin',     price: 32, category: 'Fantasy' },
    { name: 'javascript the good parts',  author: 'Douglas Crockford',    price: 40, category: 'code' }
];

const result9 = books
    .filter(libro => libro.category === 'code')       // solo los de código
    .reduce((total, libro) => total + libro.price, 0); // suma de precios
// 20 + 40 = 60
```

### Paso a paso de `reduce` con `numeros3`

```
valor inicial: 1

vuelta 1: acumulador=1,  número=39  → 1 * 39  = 39
vuelta 2: acumulador=39, número=2   → 39 * 2  = 78
vuelta 3: acumulador=78, número=4   → 78 * 4  = 312
vuelta 4: acumulador=312, número=25 → 312 * 25 = 7800
vuelta 5: acumulador=7800, número=62→ 7800 * 62 = 483600

resultado: 483600
```

> 📌 **`reduce` = "muchos → uno"**: sumas, productos, contar, agrupar, construir objetos...

### Los 3 patrones de `reduce`

```js
// 1) Sumar
arr.reduce((t, n) => t + n, 0)

// 2) Multiplicar
arr.reduce((t, n) => t * n, 1)

// 3) Construir un array/objeto
arr.reduce((acc, x) => [...acc, x], [])
arr.reduce((acc, x) => ({ ...acc, [x.clave]: x.valor }), {})
```

---

## 4.8 Encadenar métodos (el patrón "pipe")

**Ejemplo REAL** (`bikram-advance-js/index.js`):

```js
const result9 = books
    .filter(libro => libro.category === 'code')
    .reduce((total, libro) => total + libro.price, 0);
```

```js
const result6 = inventory
    .filter(producto => producto.price > 300)
    .map(producto => producto.name);

const result5 = foodList2
    .filter(plato => plato.isVeggie)
    .map(plato => `Que ric${esFemenino(plato.name) ? 'a' : 'o'} ${plato.name} me voy a comer!`);
```

**Dónde `filter` y `map` encajan:**

```
original   →  filter  →  quedarme con unos  →  map  →  transformar  →  resultado
 [4 arr]       [2 arr]                          [2 arr]
```

> ⚠️ **Orden importa:**
> ```js
> arr.filter(x).map(x)   // primero filtra (menos elementos que transformar) ✅ más rápido
> arr.map(x).filter(x)   // primero transforma TODOS y luego descarta ❌
> ```

---

## 4.9 Otros métodos útiles con ejemplos

```js
// join — array a texto
['a','b','c'].join(', ')      // "a, b, c"
// Ejemplo real (bikram2): pintarArray → '[' + arr.join(', ') + ']'

// includes / indexOf
[1,2,3].includes(2)           // true
[1,2,3].indexOf(9)            // -1

// find — primer elemento que cumple
books.find(libro => libro.category === 'code')

// some / every
[1,2,3].some(n => n > 2)      // true
[1,2,3].every(n => n > 2)     // false

// sort — ¡cuidado, ordena como TEXTO!
[10, 9, 1].sort()             // [1, 10, 9]   ❌
[10, 9, 1].sort((a,b) => a-b) // [1, 9, 10]   ✅

// flat — arrays anidados a uno solo
[[1,2],[3,4]].flat()          // [1, 2, 3, 4]
```

**`sort` real (`bikram2-hector/index.js`):**

```js
function ordenarArray(arr) {
  return arr.sort(function (a, b) {
    return a - b;      // ascendente
  });
}

function ordenarArray2(arr) {
  return arr.sort(function (a, b) {
    return b - a;      // descendente
  });
}
```

**`sort` real de números (`algoritmia/Algoritmia-Hector/index.js`):**

```js
numeros.sort((a, b) => a - b);
```

---

## 4.10 `Array.map/filter/reduce` vs `for` — cuándo usar cada cosa

| Situación | Usa |
|-----------|-----|
| Transformar **todos** los elementos | `map` |
| **Quedarte** con algunos | `filter` |
| **Acumular** a un valor | `reduce` |
| Necesitas `break` / `continue` | `for` clásico |
| Solo recorrer sin resultado | `forEach` o `for...of` |
| Modificar el array original | `push`, `pop`, `splice`, `sort` |

### Convertir un `for` en `map/filter`

```js
// ❌ Con for
let pares = [];
for (let i = 0; i < arr.length; i++) {
  if (arr[i] % 2 === 0) pares.push(arr[i]);
}

// ✅ Con filter
const pares = arr.filter(n => n % 2 === 0);

// ❌ Con for
let dobles = [];
for (let i = 0; i < arr.length; i++) {
  dobles.push(arr[i] * 2);
}

// ✅ Con map
const dobles = arr.map(n => n * 2);
```

---

## 4.11 Errores comunes

| Error | Causa |
|-------|-------|
| `[1,2,3]` sale `[1,2,3,4]` tras un `push` | `push` muta el original |
| `sort()` da `[1, 10, 2]` | Falta el comparador `(a,b) => a-b` |
| `undefined` en el array | `map` con un `return` olvidado (o con `{}` y sin `return`) |
| `arr.map` → `TypeError` | No es un array (es `object` o `undefined`) |
| Último elemento `undefined` | Usaste `i <= arr.length` |
| `reduce` sin inicial da error en array vacío | Pasa siempre el `valorInicial` |

```js
// ❌ Clásico error de arrow con llaves
arr.map(n => { n * 2 })     // devuelve [undefined, undefined...]

// ✅ retorno implícito
arr.map(n => n * 2)

// ✅ o con llaves Y return
arr.map(n => { return n * 2 })
```

---

## ✅ Resumen rápido

1. Array = lista ordenada, **índices desde 0**, puede mezclar tipos.
2. `push/pop` modifican; `map/filter/reduce/slice` **devuelven algo nuevo**.
3. **`map`** = transformar 1 a 1 (misma longitud).
4. **`filter`** = quedarse con los que devuelven `true` (menos elementos).
5. **`reduce`** = acumular en un valor único (suma con `0`, producto con `1`).
6. **Encadena:** `.filter().map().reduce()` — filtra **antes** de transformar.
7. `sort()` necesita `(a, b) => a - b` para ordenar números.
8. `arrayMapi` de tu proyecto **es** `Array.map` hecho a mano: compáralos.

👉 **Siguiente:** `05-objetos-y-modernos.md`
