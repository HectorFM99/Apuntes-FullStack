# 09 — 🧠 ALGORITMIA Y EJERCICIOS DE LIVE CODING

> En el *live coding* de un junior casi nunca te piden algoritmos difíciles. Te piden
> ejercicios **sencillos** para ver **cómo piensas, cómo te comunicas y si escribes
> código limpio**. 📌 Practica cada ejercicio **escribiéndolo tú sin mirar**.

---

## 9.1 Método para cualquier ejercicio

| Paso | Qué hacer | Qué decir en voz alta |
|------|-----------|----------------------|
| 1️⃣ Entender | Repite el problema con tus palabras | "Entonces, recibo un string y devuelvo…" |
| 2️⃣ Preguntar | Casos límite: ¿vacío? ¿mayúsculas? ¿negativos? | "¿Tengo que ignorar espacios y tildes?" |
| 3️⃣ Ejemplos | Escribe 2-3 entradas y su salida esperada | "Para `'abc'` espero `'cba'`" |
| 4️⃣ Solución simple | Que **funcione**, aunque no sea la óptima | "Primero lo hago con dos bucles" |
| 5️⃣ Probar | Recorre tus ejemplos a mano | "Con `[]` devuelve…" |
| 6️⃣ Mejorar | Complejidad, legibilidad | "Con un `Set` lo bajo a O(n)" |

---

## 9.2 Big O básico ⭐

Mide **cómo crece** el tiempo (o la memoria) cuando crece la entrada `n`.

| Big O | Nombre | Ejemplo | Con n = 1000 |
|-------|--------|---------|--------------|
| `O(1)` | Constante | `arr[5]`, `map.get(k)`, `set.has(x)` | 1 |
| `O(log n)` | Logarítmica | Búsqueda binaria | ~10 |
| `O(n)` | Lineal | Un bucle, `map`, `filter`, `includes` | 1.000 |
| `O(n log n)` | | `sort` | ~10.000 |
| `O(n²)` | Cuadrática | Bucle dentro de bucle | 1.000.000 |
| `O(2ⁿ)` | Exponencial | Fibonacci recursivo sin memo | 😱 |

```js
// O(n²): includes() dentro de un bucle ya es otro bucle
for (const x of a) if (b.includes(x)) { /* ... */ }

// O(n): convertir b en Set → .has() es O(1)
const setB = new Set(b);
for (const x of a) if (setB.has(x)) { /* ... */ }
```

> 📌 Reglas rápidas: quita constantes (`O(2n)` → `O(n)`), quédate con el término mayor
> (`O(n² + n)` → `O(n²)`). Bucles **seguidos** se suman; **anidados** se multiplican.

---

## 9.3 FizzBuzz

> Imprime del 1 al `n`. Múltiplos de 3 → "Fizz", de 5 → "Buzz", de ambos → "FizzBuzz".

```js
function fizzBuzz(n) {
  const resultado = [];
  for (let i = 1; i <= n; i++) {
    if (i % 15 === 0) resultado.push("FizzBuzz");   // ⚠️ primero el caso combinado
    else if (i % 3 === 0) resultado.push("Fizz");
    else if (i % 5 === 0) resultado.push("Buzz");
    else resultado.push(i);
  }
  return resultado;
}
fizzBuzz(15); // [1, 2, "Fizz", 4, "Buzz", ..., 14, "FizzBuzz"]
```

**Complejidad:** O(n). ❌ Error típico: comprobar `% 3` antes que `% 15`.

---

## 9.4 Invertir un string

```js
// ✅ Versión idiomática
const invertir = (str) => str.split("").reverse().join("");

// ✅ Con bucle (por si te piden "sin métodos")
function invertirBucle(str) {
  let resultado = "";
  for (let i = str.length - 1; i >= 0; i--) resultado += str[i];
  return resultado;
}

invertir("hola"); // "aloh"
```

**Complejidad:** O(n). 📌 Con emojis usa `[...str]` en vez de `split("")`.

---

## 9.5 Palíndromo

> ¿Se lee igual al derecho que al revés? Ignorando mayúsculas, espacios y signos.

```js
function esPalindromo(texto) {
  const limpio = texto
    .toLowerCase()
    .normalize("NFD").replace(/[̀-ͯ]/g, "") // quita tildes
    .replace(/[^a-z0-9ñ]/g, "");                      // quita espacios y signos
  return limpio === [...limpio].reverse().join("");
}

esPalindromo("Anita lava la tina");  // true
esPalindromo("Hola");                // false
```

**Mejora (dos punteros, sin crear el string invertido):**

```js
function esPalindromo2(limpio) {
  let i = 0, j = limpio.length - 1;
  while (i < j) {
    if (limpio[i] !== limpio[j]) return false;
    i++; j--;
  }
  return true;
}
```

**Complejidad:** O(n).

---

## 9.6 Anagrama

> ¿Dos palabras tienen las mismas letras? ("roma" / "amor")

```js
// Opción 1: ordenar → O(n log n)
const normalizar = (s) => s.toLowerCase().replace(/\s/g, "").split("").sort().join("");
const esAnagrama = (a, b) => normalizar(a) === normalizar(b);

// Opción 2: contar letras → O(n) ✅
function esAnagrama2(a, b) {
  if (a.length !== b.length) return false;
  const cuenta = {};
  for (const c of a) cuenta[c] = (cuenta[c] || 0) + 1;
  for (const c of b) {
    if (!cuenta[c]) return false;
    cuenta[c]--;
  }
  return true;
}

esAnagrama("Roma", "amor"); // true
```

---

## 9.7 Contar caracteres (frecuencia)

```js
function contarCaracteres(str) {
  const cuenta = {};
  for (const c of str) {
    cuenta[c] = (cuenta[c] ?? 0) + 1;
  }
  return cuenta;
}
contarCaracteres("banana"); // { b: 1, a: 3, n: 2 }

// Variante: el carácter más repetido
function masRepetido(str) {
  const cuenta = contarCaracteres(str);
  return Object.entries(cuenta).reduce((max, par) => (par[1] > max[1] ? par : max))[0];
}
masRepetido("banana"); // "a"

// Variante con reduce
const contar = (str) => [...str].reduce((acc, c) => ({ ...acc, [c]: (acc[c] ?? 0) + 1 }), {});
```

**Complejidad:** O(n). 📌 La de `reduce` con spread es elegante pero copia el objeto en
cada vuelta; la del bucle es más eficiente. ¡Coméntalo si te preguntan!

---

## 9.8 Duplicados

```js
// Quitar duplicados
const unicos = (arr) => [...new Set(arr)];
unicos([1, 2, 2, 3, 1]); // [1, 2, 3]

// ¿Hay alguno duplicado?
const tieneDuplicados = (arr) => new Set(arr).size !== arr.length;

// ¿Cuáles están duplicados?
function duplicados(arr) {
  const vistos = new Set();
  const repetidos = new Set();
  for (const x of arr) {
    if (vistos.has(x)) repetidos.add(x);
    else vistos.add(x);
  }
  return [...repetidos];
}
duplicados([1, 2, 2, 3, 1, 1]); // [2, 1]
```

```js
// ❌ O(n²): indexOf dentro de filter
arr.filter((x, i) => arr.indexOf(x) !== i);
// ✅ O(n): con Set (arriba)
```

---

## 9.9 Two Sum ⭐

> Dado un array y un `objetivo`, devuelve los **índices** de dos números que sumen el
> objetivo.

```js
// ❌ Fuerza bruta: O(n²)
function twoSumLento(nums, objetivo) {
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === objetivo) return [i, j];
    }
  }
  return null;
}

// ✅ Con Map: O(n)
function twoSum(nums, objetivo) {
  const vistos = new Map(); // valor → índice
  for (let i = 0; i < nums.length; i++) {
    const complemento = objetivo - nums[i];
    if (vistos.has(complemento)) return [vistos.get(complemento), i];
    vistos.set(nums[i], i);
  }
  return null;
}

twoSum([2, 7, 11, 15], 9); // [0, 1]
```

> 📌 Este es **el** ejercicio para demostrar que entiendes Big O: empieza con la
> fuerza bruta, explica que es O(n²), y mejóralo con un `Map`.

---

## 9.10 Fibonacci

> 0, 1, 1, 2, 3, 5, 8, 13… Cada número es la suma de los dos anteriores.

```js
// ❌ Recursivo puro: O(2ⁿ) → fib(50) no termina nunca
const fibLento = (n) => (n < 2 ? n : fibLento(n - 1) + fibLento(n - 2));

// ✅ Iterativo: O(n) tiempo, O(1) memoria
function fib(n) {
  let a = 0, b = 1;
  for (let i = 0; i < n; i++) [a, b] = [b, a + b];
  return a;
}

// ✅ Recursivo con memoización (closure + caché)
function fibMemo(n, memo = {}) {
  if (n < 2) return n;
  if (memo[n]) return memo[n];
  return (memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo));
}

fib(10); // 55
```

> 📌 Es la excusa perfecta para hablar de **recursividad** (caso base + llamada a sí
> misma) y de **memoización**.

---

## 9.11 Aplanar un array

```js
const anidado = [1, [2, [3, [4]], 5]];

// ✅ Nativo
anidado.flat(Infinity); // [1, 2, 3, 4, 5]

// ✅ Recursivo (lo que te pedirán "sin usar flat")
function aplanar(arr) {
  const resultado = [];
  for (const el of arr) {
    if (Array.isArray(el)) resultado.push(...aplanar(el));
    else resultado.push(el);
  }
  return resultado;
}

// ✅ Con reduce
const aplanar2 = (arr) =>
  arr.reduce((acc, el) => acc.concat(Array.isArray(el) ? aplanar2(el) : el), []);
```

---

## 9.12 Debounce ⭐

> Ejecuta una función solo cuando han pasado `ms` milisegundos **sin** que se vuelva a
> llamar. Típico en un buscador: no llamar a la API en cada tecla.

```js
function debounce(fn, ms) {
  let timer;                         // ← closure: se recuerda entre llamadas
  return function (...args) {
    clearTimeout(timer);             // cancela la llamada anterior
    timer = setTimeout(() => fn.apply(this, args), ms);
  };
}

// Uso: buscar canciones cuando el usuario deja de escribir 300 ms
const buscar = debounce((texto) => fetch(`/api/tracks/search?q=${encodeURIComponent(texto)}`), 300);
input.addEventListener("input", (e) => buscar(e.target.value));
```

**Throttle** (pariente cercano): como mucho **una vez cada `ms`**.

```js
function throttle(fn, ms) {
  let ultimo = 0;
  return function (...args) {
    const ahora = Date.now();
    if (ahora - ultimo >= ms) {
      ultimo = ahora;
      fn.apply(this, args);
    }
  };
}
```

> 📌 Este ejercicio junta **closures**, **`setTimeout`**, **rest params** y **`this`**. Si
> lo explicas bien, demuestras mucho en poco código.

---

## 9.13 Extra: otros que pueden caer

| Ejercicio | Pista |
|-----------|-------|
| Máximo de un array sin `Math.max` | Recorre guardando el mayor. O(n) |
| Contar vocales | `[...str].filter(c => "aeiou".includes(c)).length` |
| Capitalizar cada palabra | `split(" ")` + `map(p => p[0].toUpperCase() + p.slice(1))` |
| Agrupar por propiedad | `reduce` a un objeto `{ clave: [items] }` (o `Object.groupBy`) |
| Paréntesis balanceados | Usa una **pila** (array con `push`/`pop`) |
| Número primo | Prueba divisores hasta `Math.sqrt(n)` |
| Segundo mayor | Un solo recorrido con dos variables |
| Búsqueda binaria | Array **ordenado**, partes por la mitad. O(log n) |

---

## 🎯 Cómo responderlo en la entrevista

**Si te bloqueas:**
> "Déjame pensarlo con un ejemplo pequeño…" → escribe la entrada y la salida a mano y
> busca el patrón. Callar 2 minutos es peor que pensar en voz alta.

**¿Qué es Big O?**
> "Una forma de expresar cómo crece el tiempo o la memoria de un algoritmo cuando crece
> la entrada, quedándonos con el peor caso. Por ejemplo, un bucle es O(n) y dos bucles
> anidados O(n²)."

**¿Cómo mejorarías este algoritmo? (two sum, duplicados…)**
> "La versión con dos bucles es O(n²). Puedo guardar lo que ya he visto en un `Map` o
> `Set`, cuya búsqueda es O(1), y así hacerlo en una sola pasada: O(n) en tiempo a
> cambio de O(n) de memoria."

**¿Qué es la recursividad?**
> "Una función que se llama a sí misma con un problema más pequeño hasta llegar a un  
> caso base. Hay que tener cuidado con el caso base y con repetir cálculos, como en  
> Fibonacci, que se soluciona con memorización o con un bucle."

**¿Debounce vs throttle?**
> "Debounce espera a que dejen de llamar a la función durante X ms y entonces la
> ejecuta una vez, ideal para un buscador. Throttle la ejecuta como mucho una vez cada
> X ms, ideal para el scroll o el resize."
