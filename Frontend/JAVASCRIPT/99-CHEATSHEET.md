# 📋 CHEATSHEET — JAVASCRIPT

> Chuleta rápida de comandos y código. Teoría en los documentos 01–07 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal

| Comando | Qué hace |
|---------|----------|
| `node archivo.js` | Ejecuta un archivo JS fuera del navegador |
| `node` | Consola interactiva (REPL); salir con `.exit` o Ctrl+C dos veces |
| `node --watch archivo.js` | Reejecuta al guardar |
| F12 → *Console* | Consola del navegador para probar código |

```html
<script src="js/programacion.js" defer></script>   <!-- cargar JS en una web -->
<script type="module" src="js/main.js"></script>   <!-- con import/export -->
```

---

## 2. Variables, tipos y conversión

```js
const PI = 3.14;        // no se reasigna (por defecto)
let edad = 25;          // se reasigna
var viejo = 1;          // ❌ evitar: scope de función, hoisting

typeof "hola"     // "string"     typeof 42     // "number"
typeof true       // "boolean"    typeof undefined // "undefined"
typeof null       // "object" (bug histórico)   typeof [] // "object"
Array.isArray([]) // true

Number("42")      // 42           Number("abc")  // NaN
parseInt("42px")  // 42           parseFloat("3.5") // 3.5
String(42)        // "42"         (42).toFixed(2)   // "42.00"
Boolean("")       // false        !!"texto"         // true
```

Falsy: `false 0 "" null undefined NaN` — todo lo demás es truthy.

---

## 3. Operadores

```js
5 + 2; 5 - 2; 5 * 2; 5 / 2; 5 % 2; 2 ** 3;   // 7 3 10 2.5 1 8
x += 1; x++; x--;
5 === "5"   // false ✅ compara valor Y tipo (usar siempre)
5 == "5"    // true  ❌ convierte tipos
!== , > , >= , < , <=
a && b      // Y        a || b   // O (primer truthy)       !a  // NO
valor ?? "por defecto"   // solo si es null/undefined
user?.direccion?.calle   // encadenamiento opcional: undefined en vez de error
const msg = edad >= 18 ? "Adulto" : "Menor";   // ternario
`Hola ${nombre}, tienes ${edad} años`           // template string
```

---

## 4. Condicionales y bucles

```js
if (nota >= 9) { … } else if (nota >= 5) { … } else { … }

switch (dia) {
  case "lunes": …; break;
  default: …;
}

for (let i = 0; i < 10; i++) { }        // clásico
for (const item of array) { }           // valores de un array
for (const clave in objeto) { }         // claves de un objeto
while (cond) { }                        // mientras se cumpla
do { } while (cond);                    // al menos una vez
break;     // sale del bucle
continue;  // salta a la siguiente vuelta
```

---

## 5. Funciones

```js
function sumar(a, b) { return a + b; }           // declaración (hoisting)
const restar = function (a, b) { return a - b; }; // expresión
const doble = (n) => n * 2;                      // flecha: return implícito
const crear = (n) => ({ nombre: n });            // devolver objeto: paréntesis
const saludar = (nombre = "invitado") => `Hola ${nombre}`;  // parámetro por defecto
const total = (...nums) => nums.reduce((a, b) => a + b, 0); // rest: n argumentos

[1, 2, 3].forEach((n) => console.log(n));        // callback
setTimeout(() => console.log("1 s"), 1000);       // una vez
const id = setInterval(() => {}, 1000); clearInterval(id);
(function () { /* IIFE */ })();

// Closure: la función recuerda su entorno
function contador() { let c = 0; return () => ++c; }
const next = contador(); next(); next();  // 1, 2
```

---

## 6. Arrays

```js
const arr = [3, 1, 2];
arr.length; arr[0]; arr.at(-1);         // tamaño, primero, último
arr.push(4); arr.pop();                 // añadir/quitar al final   (MUTAN)
arr.unshift(0); arr.shift();            // añadir/quitar al inicio  (MUTAN)
arr.splice(1, 1);                       // borrar 1 en la posición 1 (MUTA)
arr.slice(0, 2);                        // copia de un trozo
arr.includes(2); arr.indexOf(2);        // ¿está? / posición (-1 si no)
arr.join(", "); "a,b".split(",");       // array ↔ string
arr.sort((a, b) => a - b);              // ordenar números (MUTA)
arr.toSorted((a, b) => a - b);          // sin mutar (ES2023)
arr.reverse(); [...arr];                // invertir / copiar
```

| Método | Devuelve | Ejemplo |
|--------|----------|---------|
| `map` | Array nuevo transformado | `precios.map(p => p * 1.21)` |
| `filter` | Array con los que cumplen | `cursos.filter(c => c.plazas > 0)` |
| `reduce` | Un solo valor | `nums.reduce((acc, n) => acc + n, 0)` |
| `find` | Primer elemento o `undefined` | `users.find(u => u.id === 3)` |
| `findIndex` | Posición o `-1` | |
| `some` / `every` | `true`/`false` (alguno / todos) | `notas.every(n => n >= 5)` |
| `forEach` | Nada (solo efectos) | |
| `flat` | Aplana un nivel | `[[1], [2]].flat()` |

```js
// Encadenar
const totalDisponibles = cursos
  .filter(c => c.plazas > 0)
  .map(c => c.precio)
  .reduce((acc, p) => acc + p, 0);
```

---

## 7. Objetos, destructuring, spread, Set, JSON

```js
const curso = { titulo: "React", horas: 30, precio: 150 };
curso.titulo; curso["horas"];           // leer
curso.plazas = 3;  delete curso.plazas; // añadir / borrar
"titulo" in curso;                      // ¿existe la clave?
Object.keys(curso); Object.values(curso); Object.entries(curso);
const persona = { nombre: "Ana", saludar() { return `Hola, soy ${this.nombre}`; } };

// Destructuring
const { titulo, precio: coste, plazas = 0 } = curso;  // renombrar y default
const [primero, , tercero, ...resto] = [1, 2, 3, 4, 5];

// Spread (expandir) — copias sin mutar
const copia = { ...curso, precio: 120 };   // copia y sobrescribe
const todos = [...arr1, ...arr2];
Math.max(...[3, 7, 1]);                    // 7

const unicos = [...new Set([1, 1, 2])];   // [1, 2] sin duplicados

JSON.stringify(curso);                    // objeto → texto
JSON.parse('{"a":1}');                    // texto → objeto
structuredClone(objeto);                  // copia profunda

const hoy = new Date();
hoy.getFullYear(); hoy.toLocaleDateString("es-ES"); Date.now();
Math.random(); Math.floor(Math.random() * 10) + 1;  // entero 1–10
Math.round(2.5); Math.ceil(); Math.min();
```

---

## 8. DOM

```js
const titulo = document.querySelector("#titulo");      // primero que coincide
const items = document.querySelectorAll(".item");      // NodeList (forEach)
document.getElementById("menu");

titulo.textContent = "Nuevo";          // texto (seguro)
titulo.innerHTML = "<b>Nuevo</b>";     // HTML (⚠️ XSS con datos del usuario)
img.src = "foto.jpg"; img.setAttribute("alt", "…"); el.dataset.precio;
el.style.color = "red";                // CSS en línea
el.classList.add("activo");            // remove | toggle | contains
input.value;                           // valor de un input

const li = document.createElement("li");
li.textContent = "Nuevo";
lista.append(li);                      // prepend | before | after
li.remove();
```

### Eventos

```js
boton.addEventListener("click", (e) => {
  e.preventDefault();      // evita la acción por defecto (enviar form, seguir enlace)
  e.target;                // elemento que originó el evento
});
form.addEventListener("submit", (e) => { e.preventDefault(); const datos = new FormData(form); });
input.addEventListener("input", (e) => console.log(e.target.value));
window.addEventListener("scroll", () => window.scrollY);
document.addEventListener("keydown", (e) => e.key === "Escape" && cerrar());
document.addEventListener("DOMContentLoaded", init);   // DOM listo

// Delegación: un listener en el padre
lista.addEventListener("click", (e) => {
  if (e.target.matches("button.borrar")) e.target.closest("li").remove();
});
```

| Evento | Cuándo |
|--------|--------|
| `click`, `dblclick` | Clic |
| `input` / `change` | Al escribir / al confirmar cambio |
| `submit` | Envío de formulario |
| `keydown` / `keyup` | Teclado |
| `mouseenter` / `mouseleave` | Hover |
| `scroll`, `resize` | Ventana |
| `DOMContentLoaded` | HTML parseado |

---

## 9. Asincronía y fetch

```js
fetch("https://api.ejemplo.com/datos")
  .then((res) => res.json())
  .then((data) => console.log(data))
  .catch((err) => console.error(err));

async function cargar() {
  try {
    const res = await fetch(url);
    if (!res.ok) throw new Error(`Error ${res.status}`);  // fetch no falla en 404/500
    return await res.json();
  } catch (err) {
    console.error(err);
  }
}

await Promise.all([fetch(a), fetch(b)]);           // en paralelo
localStorage.setItem("tema", "oscuro"); localStorage.getItem("tema");
```

---

## 10. Consola

```js
console.log("valor", x); console.error("fallo"); console.warn("ojo");
console.table(arrayDeObjetos);   // tabla
console.time("t"); console.timeEnd("t");
debugger;                        // pausa en DevTools
```

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `Cannot read properties of null` | El `querySelector` no encontró nada: selector mal o script antes del HTML (`defer`) |
| `x is not defined` | Variable mal escrita o fuera de su scope |
| `Assignment to constant variable` | Reasignas una `const`: usa `let` |
| `"5" + 1 = "51"` | Convierte con `Number()` antes de sumar |
| `map`/función devuelve `undefined` | Flecha con llaves sin `return` |
| Comparación rara (`0 == ""` es `true`) | Usa siempre `===` |
| Bucle infinito | La condición nunca cambia: revisa el `i++` |
| El listener se ejecuta al cargar | `addEventListener("click", fn)`, no `fn()` |
