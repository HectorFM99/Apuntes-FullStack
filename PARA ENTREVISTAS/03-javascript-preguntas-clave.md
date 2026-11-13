# 03 — ⚡ JAVASCRIPT: PREGUNTAS CLAVE

> **El tema más preguntado.** Si tienes poco tiempo, empieza por aquí. Las
> estrellas ⭐ marcan lo que sale en casi todas las entrevistas.

---

## 3.1 `var`, `let` y `const` ⭐

| | `var` | `let` | `const` |
|---|---|---|---|
| Ámbito (scope) | **Función** | **Bloque** `{}` | **Bloque** `{}` |
| Reasignar | ✅ | ✅ | ❌ |
| Redeclarar | ✅ | ❌ | ❌ |
| Hoisting | Sí, se inicializa a `undefined` | Sí, pero en **TDZ** (error si la usas antes) | Igual que `let` |

```js
if (true) {
  var a = 1;
  let b = 2;
}
console.log(a); // 1  → var "se escapa" del bloque
console.log(b); // ❌ ReferenceError
```

```js
const usuario = { nombre: "Ana" };
usuario.nombre = "Luis";   // ✅ se puede: cambias el CONTENIDO
usuario = {};              // ❌ TypeError: no puedes reasignar la variable
```

> 📌 **`const` no hace el objeto inmutable**, solo impide reasignar la variable.
> Regla práctica: usa `const` por defecto, `let` si vas a reasignar, `var` nunca.

---

## 3.2 Hoisting

JS "sube" las **declaraciones** al principio de su ámbito antes de ejecutar.

```js
console.log(x); // undefined  (var: declarada e inicializada a undefined)
var x = 5;

console.log(y); // ❌ ReferenceError (let: Temporal Dead Zone)
let y = 5;

saludar();      // ✅ "Hola" → las function declarations suben ENTERAS
function saludar() { console.log("Hola"); }

despedir();     // ❌ ReferenceError → arrow en const: no se puede usar antes
const despedir = () => console.log("Adiós");
```

---

## 3.3 Scope (ámbito)

| Tipo | Dónde se ve la variable |
|------|-------------------------|
| **Global** | En todo el programa |
| **De función** | Dentro de la función |
| **De bloque** | Dentro de `{}` (`let`/`const`) |
| **Léxico** | Una función ve las variables de donde fue **escrita**, no de donde se llama |

```js
const global = "🌍";
function externa() {
  const deFuera = "📦";
  function interna() {
    console.log(global, deFuera); // ✅ ve hacia "fuera" (cadena de scopes)
  }
  interna();
}
```

---

## 3.4 Closures ⭐

Una **closure** es una función que **recuerda las variables del ámbito donde se creó**,
incluso cuando ese ámbito ya terminó.

```js
function crearContador() {
  let cuenta = 0;              // variable "privada"
  return function () {
    cuenta++;
    return cuenta;
  };
}

const contador = crearContador();
contador(); // 1
contador(); // 2  → recuerda "cuenta"
```

**Usos reales:** datos privados, fábricas de funciones, `debounce` (ver `09`),
callbacks de eventos, memoización.

```js
// Pregunta trampa clásica
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i), 100); // 3 3 3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i), 100); // 0 1 2
// let crea una variable NUEVA en cada vuelta → cada closure guarda la suya
```

---

## 3.5 `this` ⭐

`this` depende de **cómo se llama** la función (excepto en las arrow functions).

| Cómo se llama | Valor de `this` |
|---------------|-----------------|
| `obj.metodo()` | `obj` |
| `funcion()` suelta | `undefined` (modo estricto) / `window` (no estricto) |
| `new Clase()` | El objeto nuevo |
| `fn.call(x)` / `fn.apply(x)` / `fn.bind(x)` | `x` |
| **Arrow function** | El `this` **de fuera** (no tiene `this` propio) |
| Listener `function` en DOM | El elemento que recibió el evento |

```js
const persona = {
  nombre: "Héctor",
  saludarNormal() { console.log(this.nombre); },
  saludarFlecha: () => console.log(this.nombre),
};
persona.saludarNormal(); // "Héctor"
persona.saludarFlecha(); // undefined → la arrow toma el this de fuera del objeto
```

> 📌 Por eso **no uses arrow functions como métodos de objeto**, pero sí dentro de
> ellos (callbacks), donde heredar `this` es justo lo que quieres.

---

## 3.6 `==` vs `===` y coerción de tipos ⭐

```js
0 == "0"          // true  → convierte tipos (coerción)
0 === "0"         // false → compara valor Y tipo
null == undefined // true
null === undefined// false
NaN === NaN       // false → usa Number.isNaN(x)
```

✅ **Usa siempre `===`** salvo que tengas un motivo claro.

### Tipos en JS

| Primitivos (se copian por **valor**) | Referencia (se copian por **referencia**) |
|---|---|
| `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint` | `object` (incluye arrays y funciones) |

```js
typeof null      // "object"  ← bug histórico, pregunta trampa
typeof []        // "object"  → usa Array.isArray([])
typeof function(){} // "function"
```

### Valores *falsy* (todo lo demás es *truthy*)

`false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`

```js
if ([]) console.log("¡entra!");  // [] y {} son TRUTHY
"5" + 1   // "51" (con + gana el string)
"5" - 1   // 4    (con - convierte a número)
```

### `null` vs `undefined`

| `undefined` | `null` |
|-------------|--------|
| "No se ha asignado valor" (lo pone JS) | "Vacío a propósito" (lo pones tú) |

### `??` vs `||`

```js
const paginas = 0;
paginas || 100  // 100 ❌ (0 es falsy)
paginas ?? 100  // 0   ✅ (solo cae si es null/undefined)
```

> 📌 En `api-rastro` usas `process.env.DB_NAME ?? "rastro"`: valor por defecto solo si
> la variable no existe.

---

## 3.7 Valor vs referencia e inmutabilidad

```js
const a = [1, 2, 3];
const b = a;        // ❌ NO es una copia: misma referencia
b.push(4);
console.log(a);     // [1, 2, 3, 4]  😱

const c = [...a];   // ✅ copia superficial (shallow)
const d = structuredClone(objAnidado); // ✅ copia profunda (deep)
```

**Inmutabilidad** = en vez de modificar, **creas uno nuevo**. Evita bugs y es la base
de React/Redux.

```js
// ❌ Muta
usuario.edad = 30;
lista.push(nuevo);

// ✅ Inmutable
const usuario2 = { ...usuario, edad: 30 };
const lista2 = [...lista, nuevo];
```

| Mutan el array | No mutan (devuelven uno nuevo) |
|----------------|-------------------------------|
| `push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse` | `map`, `filter`, `reduce`, `slice`, `concat`, `toSorted`, `toReversed` |

---

## 3.8 Spread, rest y destructuring

```js
// Destructuring de objeto (con renombre y valor por defecto)
const { title, pages: paginas = 0 } = libro;

// Destructuring de array
const [primero, , tercero] = [10, 20, 30];

// Spread: "desparrama"
const todos = [...arr1, ...arr2];
const config = { ...porDefecto, ...delUsuario }; // el último gana

// Rest: "recoge el resto"
function sumar(...numeros) { return numeros.reduce((a, n) => a + n, 0); }
const { password, ...usuarioSinPassword } = usuario; // ✅ quitar un campo
```

> 📌 El último ejemplo es muy útil en backend: **nunca devuelvas el hash de la
> contraseña** en la respuesta de la API.

---

## 3.9 `map`, `filter`, `reduce` (y amigos) ⭐

```js
const libros = [
  { titulo: "A", paginas: 120, autor: "Gabriel" },
  { titulo: "B", paginas: 300, autor: "Isabel" },
  { titulo: "C", paginas: 90,  autor: "Gabriel" },
];

libros.map(l => l.titulo);                      // ["A", "B", "C"]        → transforma
libros.filter(l => l.paginas < 150);            // [A, C]                 → filtra
libros.reduce((total, l) => total + l.paginas, 0); // 510                 → acumula
libros.find(l => l.autor === "Isabel");          // {titulo: "B", ...}    → el primero
libros.some(l => l.paginas > 200);               // true                  → ¿alguno?
libros.every(l => l.paginas > 50);               // true                  → ¿todos?

// Encadenar
libros
  .filter(l => l.autor === "Gabriel")
  .map(l => l.titulo);                           // ["A", "C"]
```

| Método | Devuelve | Longitud |
|--------|----------|----------|
| `map` | Array nuevo | **Igual** |
| `filter` | Array nuevo | **Menor o igual** |
| `reduce` | **Un valor** (número, objeto, array…) | — |
| `forEach` | `undefined` | Solo para efectos (no devuelve nada) |

---

## 3.10 Prototipos y clases

Cada objeto JS tiene un enlace oculto a otro objeto, su **prototipo**. Si buscas una
propiedad y no está, JS la busca en el prototipo, y así sucesivamente
(**cadena de prototipos**).

```js
const arr = [1, 2];
arr.map  // no está en arr → la encuentra en Array.prototype
```

Las **clases** de ES6 son "azúcar sintáctico" sobre los prototipos:

```js
class Animal {
  constructor(nombre) { this.nombre = nombre; }
  hablar() { return `${this.nombre} hace un ruido`; }
}
class Perro extends Animal {
  hablar() { return `${this.nombre} ladra`; }
}
new Perro("Toby").hablar(); // "Toby ladra"
```

---

## 3.11 Event loop ⭐⭐

JS es **monohilo** (una sola cosa a la vez), pero no se bloquea esperando gracias al
**event loop**.

| Pieza | Qué hace |
|-------|----------|
| **Call stack** | Las funciones que se están ejecutando ahora |
| **Web APIs / APIs de Node** | Hacen el trabajo lento fuera (timers, `fetch`, ficheros) |
| **Microtask queue** | Callbacks de **promesas** (`.then`, `await`) |
| **Macrotask (task) queue** | `setTimeout`, `setInterval`, eventos |
| **Event loop** | Cuando el stack está vacío: **primero TODAS las microtareas**, luego una macrotarea |

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");

// Salida: 1, 4, 3, 2
// Síncrono primero (1, 4) → microtareas (3) → macrotareas (2)
```

> 📌 `setTimeout(fn, 0)` **no** significa "ahora": significa "en cuanto el stack esté
> vacío y no queden microtareas".

---

## 3.12 Callbacks, promesas y async/await ⭐

```js
// 1️⃣ Callbacks → "callback hell" si se anidan
leerUsuario(id, (u) => {
  leerPedidos(u, (p) => {
    leerFactura(p, (f) => { /* 😵 */ });
  });
});

// 2️⃣ Promesas: un objeto que representa un valor FUTURO
//    Estados: pending → fulfilled (resuelta) | rejected (rechazada)
fetch("/api/books")
  .then(res => res.json())
  .then(libros => console.log(libros))
  .catch(err => console.error(err));

// 3️⃣ async/await: lo mismo, pero se lee como código síncrono
async function cargarLibros() {
  try {
    const res = await fetch("/api/books");
    if (!res.ok) throw new Error(`HTTP ${res.status}`); // ⚠️ fetch NO rechaza con 404/500
    return await res.json();
  } catch (err) {
    console.error("Error cargando libros", err);
  }
}
```

### Varias promesas a la vez

| Método | Resuelve cuando… | Si una falla… |
|--------|------------------|---------------|
| `Promise.all` | **Todas** terminan bien | ❌ Rechaza en cuanto una falla |
| `Promise.allSettled` | Todas terminan (bien o mal) | Te da el estado de cada una |
| `Promise.race` | La **primera** que termine | Si la primera falla, rechaza |
| `Promise.any` | La primera que termine **bien** | Solo rechaza si fallan todas |

```js
// ❌ En serie (lento si son independientes)
const a = await getA();
const b = await getB();

// ✅ En paralelo
const [a, b] = await Promise.all([getA(), getB()]);
```

---

## 3.13 Event delegation (delegación de eventos)

Los eventos **burbujean** (*bubbling*): del elemento clicado suben hacia sus padres.
En vez de poner un listener en cada hijo, pones **uno en el padre**.

```js
// ❌ Un listener por cada <li> (y los nuevos no lo tienen)
document.querySelectorAll("li").forEach(li =>
  li.addEventListener("click", () => li.classList.toggle("hecho"))
);

// ✅ Un solo listener en el padre: funciona también con elementos añadidos después
document.querySelector("ul").addEventListener("click", (e) => {
  const li = e.target.closest("li");
  if (li) li.classList.toggle("hecho");
});
```

| Método | Para qué |
|--------|----------|
| `e.target` | El elemento que **originó** el evento |
| `e.currentTarget` | El elemento que **tiene** el listener |
| `e.stopPropagation()` | Para el burbujeo |
| `e.preventDefault()` | Evita el comportamiento por defecto (enviar form, seguir enlace) |

---

## 3.14 Otras preguntas rápidas

| Pregunta | Respuesta corta |
|----------|-----------------|
| Arrow vs function | Arrow: sin `this` propio, sin `arguments`, no sirve con `new`, sintaxis corta |
| ¿Qué es una función de orden superior? | Recibe o devuelve funciones (`map`, `filter`, `addEventListener`) |
| ¿Qué es una función pura? | Mismo input → mismo output, sin efectos secundarios |
| `slice` vs `splice` | `slice` copia un trozo (no muta); `splice` quita/inserta (muta) |
| ¿Qué es el optional chaining? | `user?.direccion?.calle` → `undefined` en vez de error |
| Módulos ES vs CommonJS | `import/export` vs `require/module.exports` |
| Debounce vs throttle | Debounce: ejecuta al **dejar** de llamarse. Throttle: como mucho 1 vez cada X ms |

---

## 🎯 Cómo responderlo en la entrevista

**¿Diferencia entre `var`, `let` y `const`?**
> "`var` tiene ámbito de función y se puede redeclarar; `let` y `const` tienen ámbito de
> bloque. `const` no se puede reasignar, aunque si es un objeto sí puedo modificar su
> contenido. Uso `const` por defecto, `let` si necesito reasignar y nunca `var`."

**¿Qué es una closure?**
> "Una función que recuerda las variables del ámbito donde se creó, aunque ese ámbito
> ya haya terminado. Por ejemplo, una función `crearContador` que devuelve otra función
> que incrementa una variable privada. Se usa para encapsular datos o en un debounce."

**¿Cómo funciona `this`?**
> "Depende de cómo se llama la función: en un método es el objeto, con `new` es la
> instancia, con `call`/`apply`/`bind` lo eliges tú. Las arrow functions no tienen
> `this` propio y usan el del contexto donde se escribieron."

**Explica el event loop.**
> "JS es monohilo. Lo síncrono va al call stack; las operaciones asíncronas se delegan
> al navegador o a Node y, al terminar, su callback va a una cola. Cuando el stack está
> vacío, el event loop ejecuta primero todas las microtareas, que son las promesas, y
> luego las macrotareas como `setTimeout`. Por eso una promesa resuelta se ejecuta antes
> que un `setTimeout` de 0."

**¿Promesas vs async/await?**
> "async/await es azúcar sintáctico sobre promesas: hace que el código asíncrono se lea
> como síncrono y permite usar try/catch. Si tengo varias peticiones independientes las
> lanzo en paralelo con `Promise.all`."

**¿`==` vs `===`?**
> "`==` convierte los tipos antes de comparar y da resultados raros como `0 == ''` true.
> `===` compara valor y tipo, por eso siempre uso `===`."

**¿Qué es la delegación de eventos?**
> "Aprovechar el bubbling para poner un único listener en el padre y detectar con
> `event.target` qué hijo se pulsó. Es más eficiente y funciona con elementos que se
> añaden dinámicamente."
