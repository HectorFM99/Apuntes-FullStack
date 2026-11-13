# 02 — CONDICIONALES Y BUCLES

---

## 2.1 `if` / `else if` / `else`

```js
if (condición) {
    // se ejecuta si la condición es true
} else if (otraCondición) {
    // si no, pero esta sí
} else {
    // si ninguna anterior
}
```

> ⚠️ **Las llaves `{}` son obligatorias** en JS (a diferencia de otros lenguajes).
> Con una sola línea se pueden omitir, pero **no lo hagas**: es fuente de errores.

### Ejemplo REAL copiado (`algoritmia/Algoritmia-Hector/index.js`)

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

### Ejemplo REAL con `else if` (`bikram2-hector/bikram2-hector/index.js`)

```js
// 21
function esPar(num) {
  if (num % 2 === 0) {
    return true;
  } else {
    return false;
  }
}
```

> 📌 Esta función se puede simplificar a una sola línea:
> ```js
> const esPar = num => num % 2 === 0;
> ```
> porque `num % 2 === 0` **ya devuelve** `true` o `false`.

### Ejemplo REAL con `if` anidado dentro de bucle (`bikram2-hector/index.js`)

```js
// 11
function obtenerPares(arr) {
  let pares = [];
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] % 2 === 0) {     // ¿este elemento es par?
      pares.push(arr[i]);       //  → lo añado
    }
  }
  return pares;
}
```

---

## 2.2 Condicionales cortos: ternario y `if` implícito

```js
// Ternario: condición ? valorSiTrue : valorSiFalse
const edad = 20;
const acceso = edad >= 18 ? "permitido" : "denegado";

// Encadenado
const nota = 7;
const nivel = nota >= 9 ? "sobresaliente"
            : nota >= 7 ? "notable"
            : nota >= 5 ? "aprobado"
            : "suspenso";

// if implícito (solo cuando NO hay else)
if (esPar) pares.push(valor);

// Guard clause (early return) — muy legible
function saludar(nombre) {
  if (!nombre) return "Hola, desconocido";   // sale antes
  return `Hola, ${nombre}`;
}
```

**Ejemplo real de ternario dentro de un `map` (`bikram-advance-js/index.js`):**

```js
const esFemenino = nombre => /a$|burguer$/i.test(nombre);

const result5 = foodList2
    .filter(plato => plato.isVeggie)
    .map(plato => `Que ric${esFemenino(plato.name) ? 'a' : 'o'} ${plato.name} me voy a comer!`);
```

---

## 2.3 `switch`

```js
switch (valor) {
  case 1:
    console.log("uno");
    break;               // ⚠️ SIN break, "se cae" al siguiente case
  case 2:
  case 3:
    console.log("dos o tres");   // case sin break = agrupados
    break;
  default:
    console.log("otro");
}
```

> ⚠️ **`break` es obligatorio** si no quieres que siga ejecutando los siguientes cases
> (*fall-through*). Úsalo a propósito solo cuando quieras agrupar casos.

---

## 2.4 El bucle `for`

```js
for (inicio; condición; incremento) {
    // cuerpo
}
```

```js
for (let i = 0; i < 5; i++) {
  console.log(i);     // 0, 1, 2, 3, 4   (¡empieza en 0!)
}
```

**Desglose:**
```
let i = 0          ← inicialización (se ejecuta UNA vez)
i < 5              ← condición (se comprueba ANTES de cada vuelta)
i++                ← incremento (se ejecuta DESPUÉS de cada vuelta)
```

### Ejemplo REAL: tabla de multiplicar (`algoritmia/Algoritmia-Hector/index.js`)

```js
//1
const fijo = 3;
for (let i = 1; i < 10; i++) {
    console.log(fijo * i);
}
```

**Salida:**
```
3, 6, 9, 12, 15, 18, 21, 24, 27
```

> 📌 Aquí `i` empieza en **1** (no en 0) y llega hasta 9 (`i < 10`).

### Ejemplo REAL: `for` con índice sobre un array (`bikram2-hector/index.js`)

```js
// 8
function repetirString(texto, veces) {
  let resultado = '';
  for (let i = 0; i < veces; i++) {
    resultado = resultado + texto;
  }
  return resultado;
}

// 25
function sumarArray(arr) {
  let total = 0;
  for (let i = 0; i < arr.length; i++) {
    total = total + arr[i];
  }
  return total;
}

// 26
function multiplicarArray(arr) {
  let total = 1;
  for (let i = 0; i < arr.length; i++) {
    total = total * arr[i];
  }
  return total;
}
```

> 📌 **Patrón de acumulador:** se empieza en `0` para sumas y en `1` para productos
> (el elemento neutro de la multiplicación), y en cada vuelta se **acumula**.

### Ejemplo REAL: `for` con `if` dentro (`bikram hector/bikram-hector/index.js`)

```js
// 12
let sumaPares = 0;
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) {
    sumaPares += i;
  }
}
```

### Ejemplo REAL: bucle infinito con `for` vacío (`bikram hector/index.js`)

```js
// 10
let contarHasta10_2 = 0;
for (; contarHasta10_2 < 10; contarHasta10_2++) {}
```

> 📌 Un `for` **sin inicialización** (la primera parte está vacía) es válido:
> la variable ya existe fuera. El cuerpo `{}` está vacío: solo cuenta.

### `for` anidado — DOble bucle (`algoritmia/Algoritmia-Hector/index.js`)

```js
//4
function contarCaracteres(data, caracter) {
  let total = 0;
  const objetivo = caracter.toLowerCase();

  for (let i = 0; i < data.length; i++) {      // bucle 1: por cada PALABRA
      const palabra = data[i];
      for (let j = 0; j < palabra.length; j++) {   // bucle 2: por cada CARÁCTER
      if (palabra[j].toLowerCase() === objetivo) {
        total++;
      }
    }
  }

  return total;
}
```

> 📌 **Lectura:** `data` es un array de palabras (como `["hola", "mundo"]`).
> El bucle exterior recorre palabras; el interior recorre las letras de cada palabra.
> Se usan `i` y `j` (convención: índices anidados con letras consecutivas).

---

## 2.5 El bucle `while`

```js
while (condición) {
    // se repite MIENTRAS la condición sea true
}
```

> ⚠️ **Peligro de bucle infinito:** si la condición nunca se pone en `false`,
> el navegador se cuelga. Asegúrate de **modificar algo dentro** del bucle.

### Ejemplo REAL: contador (`bikram hector/bikram-hector/index.js`)

```js
// 24
let contarHasta10 = 0;
while (contarHasta10 < 10) {
  contarHasta10++;
}
```

### Ejemplo REAL: `while (true)` con `break` (`algoritmia/Algoritmia-Hector/index.js`)

```js
function pedirNumeros() {
  const numeros = [];

  while (true) {                                     // ← bucle "infinito" a propósito
    const entrada = prompt("Introduce un número (0 para terminar):");
    const numero = parseInt(entrada);

    if (isNaN(numero)) break;    // ← la ÚNICA salida 1
    if (numero === 0) break;     // ← la ÚNICA salida 2
    numeros.push(numero);
  }

  numeros.sort((a, b) => a - b);
  return numeros;
}
```

> 📌 **`while (true)` + `break`** es el patrón para pedir datos hasta que el usuario
> decida parar. Sin esos `break`, el bucle **nunca** terminaría.

### `do...while` — se ejecuta AL MENOS una vez

```js
let intento;
do {
  intento = prompt("Introduce la contraseña:");
} while (intento !== "secreto");
```

---

## 2.6 Control de flujo: `break` y `continue`

```js
// break → SALE completamente del bucle
for (let i = 0; i < 100; i++) {
  if (i === 5) break;      // se sale en 5 → se ejecuta 0,1,2,3,4
}

// continue → SALTA a la siguiente vuelta (no sale del bucle)
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) continue;   // salta los pares
  console.log(i);              // 1, 3, 5, 7, 9
}
```

**Ejemplo REAL de `break` (`algoritmia/Algoritmia-Hector/index.js`):**

```js
if (isNaN(numero)) break;
if (numero === 0) break;
```

---

## 2.7 Bucles modernos (ES6+)

### `for...of` — recorrer **valores**

```js
const frutas = ["manzana", "pera", "uva"];
for (const fruta of frutas) {
  console.log(fruta);      // "manzana", "pera", "uva"
}

// También con texto
for (const letra of "hola") {
  console.log(letra);      // h, o, l, a
}
```

### `for...in` — recorrer **claves**

```js
const persona = { nombre: "Hector", edad: 25 };
for (const clave in persona) {
  console.log(clave, persona[clave]);   // "nombre Hector", "edad 25"
}
```

### `forEach` — el preferido para arrays

```js
const numeros = [1, 2, 3];
numeros.forEach((num, indice) => {
  console.log(indice, num);   // 0 1, 1 2, 2 3
});
```

> 📌 **Cuándo usar cada uno:**
> - `for` clásico → cuando necesitas el **índice**, o `break`/`continue`.
> - `for...of` → cuando solo quieres **los valores** de un iterable.
> - `for...in` → **casi siempre arrays NO**; sí para recorrer **objetos**.
> - `forEach` / `map` / `filter` → cuando trabajas con **arrays** de forma funcional.

---

## 2.8 Ejemplos REALES combinados

### De `bikram2-hector/bikram2-hector/index.js`

```js
// 9 - ¿Es primo?
function esPrimo(num) {
  if (num < 2) {
    return false;
  }
  for (let i = 2; i < num; i++) {
    if (num % i === 0) {     // si se divide entre algo distinto de sí mismo...
      return false;           // ...no es primo (sale enseguida)
    }
  }
  return true;                // si llegó hasta aquí, es primo
}

// 14 - Eliminar duplicados
function eliminarDuplicados(arr) {
  let resultado = [];
  for (let i = 0; i < arr.length; i++) {
    if (!resultado.includes(arr[i])) {   // si todavía NO está
      resultado.push(arr[i]);            // lo añado
    }
  }
  return resultado;
}

// 12 - Pintar array como texto
function pintarArray(arr) {
  return '[' + arr.join(', ') + ']';
}
```

### De `bikram hector/bikram-hector/index.js` — todos los bucles del ejercicio

```js
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

## 2.9 Errores típicos de bucles

| Error | Cómo se ve |
|-------|------------|
| `i <= arr.length` | `Undefined error` al acceder al último elemento (se pasa 1) |
| Olvidar `i++` | **Bucle infinito** → el navegador se cuelga |
| `i = 1` empezando un array | Se **pierde** el primer elemento (`arr[0]`) |
| `break` donde querías `continue` | Sale del bucle entero |
| `switch` sin `break` | Ejecuta los cases siguientes también |

---

## ✅ Resumen rápido

1. `if / else if / else` con `===` en las condiciones.
2. Ternario `cond ? a : b` para asignaciones cortas.
3. `for (let i = 0; ...; i++)` para recorrer con índice; `arr.length` como límite.
4. `while` cuando no sabes **cuántas** vueltas habrá; vigila los `break`.
5. `i % 2 === 0` = par; `i % 2 !== 0` = impar.
6. `break` sale del bucle, `continue` salta a la siguiente vuelta.
7. `for...of` y `forEach` para recorrer arrays de forma limpia.

👉 **Siguiente:** `03-funciones.md`
