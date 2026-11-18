# 05 — OBJETOS, DESTRUCTURING, SPREAD/REST Y MODERNOS

---

## 5.1 ¿Qué es un objeto?

Una **colección de pares `clave: valor`**. Mientras un array guarda cosas **por
posición**, un objeto las guarda **por nombre**.

```js
const persona = {
    nombre: "Hector",
    apellidos: "Franco Magallo",
    edad: 25
};
```

**Anatomía:**

```js
const coche = {
    marca: "Seat",      // clave : valor (string)
    modelo: "León",     // coma entre propiedades
    matricula: "1234ABC"
};
//  ▲         ▲        ▲
// clave    dos puntos  valor
```

> ⚠️ **Coma entre propiedades**, **dos puntos** entre clave y valor, y **llaves**
> alrededor. Si olvidas la última coma, funciona igual (pero si la pones, también).

---

## 5.2 Ejemplos REALES copiados

### **Archivo:** `bikram-objetos/bikram-objetos-hector/index.js` *(copiado tal cual)*

```js
//1
let Coche = {
    marca: "Seat",
    modelo: "León",
    matricula: "1234ABC"
};

// 2
let Casa = {
    codPostal: 28001,
    calle: "Gran Vía",
    portal: 5,
    piso: 3
};

//3
let FullStackDeveloper = {
    lenguajes: ["JavaScript", "HTML", "CSS", "Node.js"],
    proyectos: ["Bikram Objetos", "API Rest"]
};

// 4
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

// 13
let Noticia = {
    titular: "Última hora: JavaScript sigue dominando la web",
    cuerpo: "El lenguaje de programación más usado del mundo cumple otro año de liderazgo..."
};

// 14
let Persona = {
    nombre: "Hector",
    apellidos: "García Lopez",
    edad: 28
};

// 15
let Avion = {
    numPasajeros: 180,
    despegar: function () {
        console.log("despegando");
    },
    volar: function () {
        console.log("llegando al destino");
    },
    aterrizar: function () {
        console.log("aterrizando");
    }
};

// 17
let Pais = {
    numHabitantes: 47000000,
    continente: "Europa",
    gentilicio: "español"
};
```

> 📌 **Objetos con funciones dentro** → se llaman **métodos**:
> ```js
> Perro.ladrar();      // ¡Guau guau!
> Avion.despegar();    // despegando
> ```
> Son funciones que "pertenecen" al objeto.

### **Archivo:** `bikram-objetos/bikram-objetos-hector/JS/objects.js` *(copiado tal cual)*

```js
//Objetos para el alumno
let Portatil = {"marca": "Asus", "modelo": "A33"};
let O_Error = {"codigo": 404};
let Concierto = {"grupos": ["Twenty One Pilots", "AC/DC", "Queen"], "cartelera": ["Queen"], "fecha": null};
let Grupo = {"integrantes": ["Pedro", "Paula", "Soledad"], "numIntegrantes": 3};
let Impresora = {"tinta": {"rojo": 75, "verde": 33, "azul": 2}, imprimiendo: null};
let Pantalla = {"pixeles": [["#333", "#123", "#9AF"], ["#777", "#444", "#444"], ["#333", "#444", "#7A7F9D"]], "dimensiones": "1080X720"};
let Movil = {"especificaciones": { "ancho": 1920, "altura": 1080 }, "temperatura": "50º"};
let powered = (Math.round(Math.random()) ? true : false)
let Led = {"rojo": "FF", "verde": "00", "azul": "00", "encendido": powered};
```

> 📌 **Observa los niveles de anidamiento:**
> - `Impresora.tinta.rojo` → objeto dentro de objeto → `75`
> - `Pantalla.pixeles[0][1]` → array de arrays → `"#123"`
> - `Movil.especificaciones.ancho` → `1920`
> - `Concierto.fecha: null` → propiedad **vacía** a la espera de rellenarla
> - `"clave": valor` con comillas también es válido (JSON-like)

### **Archivo:** `bikram-advance-js/.../index.js` — objetos dentro de arrays

```js
const staff = [
    {
        name: 'Pepe',
        role: 'The Boss',
        hobbies: ['leer', 'ver pelis']
    },
    {
        name: 'Ana',
        role: 'becaria',
        hobbies: ['nadar', 'bailar']
    }
];
```

---

## 5.3 Acceder y modificar

```js
const Portatil = { marca: "Asus", modelo: "A33" };

// ACCESO
Portatil.marca;            // "Asus"       ← punto (lo normal)
Portatil["marca"];         // "Asus"       ← corchetes (clave dinámica)

// MODIFICAR (añadir o cambiar)
Portatil.modelo = "P345";  // cambia
Portatil.color = "negro";  // añade una nueva

// BORRAR
delete Portatil.color;

// ¿Existe?
"marca" in Portatil;                 // true
Portatil.marca !== undefined;        // true
```

### Ejemplos REALES copiados (`bikram-objetos/.../index.js`)

```js
// 5 - acceso con punto
let marcaPortatil = Portatil.marca;

// 6 - acceso con corchetes (misma clave)
let marcaPortatil2 = Portatil["marca"];

// 7 - acceder a un array dentro del objeto
let grupos = Concierto.grupos;

// 8 - acceder a varias propiedades y crear un array
let RGB = [Led.rojo, Led.verde, Led.azul];

// 9 - MODIFICAR una propiedad
Portatil.modelo = "P345";

// 10 - añadir al array de dentro
Concierto.cartelera.push("Guns N' Roses");

// 11 - cambiar null por un objeto Date
Concierto.fecha = new Date();

// 12 - añadir un objeto COMPLETO como propiedad
Impresora.imprimiendo = {
    nombreArchivo: "documento.pdf",
    copias: 3,
    numPaginas: 10
};

// 18
let codError = O_Error.codigo;

// 19
let integrantes = Grupo.integrantes;

// 20 - objeto anidado
let nivelesTinta = Impresora.tinta;

// 21 - array de arrays
let pixeles = Pantalla.pixeles;

// 22 - corchetes con propiedad anidada
let especificaciones = Movil["especificaciones"];

// 23 a 26 - modificaciones
Grupo.numIntegrantes = 5;
Pantalla.dimensiones = "1920x1080";
Led.encendido = !Led.encendido;
Movil.temperatura = "20º";
```

> 📌 **`Portatil.marca` y `Portatil["marca"]` son idénticos.**
> Los corchetes son necesarios cuando la clave está en una **variable**:
> ```js
> const clave = "marca";
> Portatil[clave];     // "Asus"   ✅
> Portatil.clave;      // undefined ❌ busca la propiedad literal "clave"
> ```

---

## 5.4 Métodos de objeto

```js
const Perro = {
    nombre: "Toby",
    ladrar: function () { console.log("¡Guau guau!"); }
};

Perro.ladrar();     // ¡Guau guau!
```

**Formas de escribir un método:**

```js
const obj = {
  // 1) clásica
  saludar: function () { return "hola"; },
  // 2) shorthand (moderna, la habitual)
  despedir() { return "adiós"; },
  // 3) arrow
  gritar: () => "¡auuu!"
};
```

---

## 5.5 DESTRUCTURING — extraer valores con sintaxis de array/objeto

> **Idea:** "desempaquetar" en una línea lo que de otro modo costaría varias.

### Con arrays

```js
const empleados = ["Luis", "Ana", "Andrea"];

const [primero, segundo, tercero] = empleados;
// primero = "Luis", segundo = "Ana", tercero = "Andrea"

// Saltar posiciones
const [, ana] = empleados;          // ana = "Ana" (posición 1)
const [luis] = empleados;           // luis = "Luis"
```

### Con objetos

```js
const HIGH_TEMPERATURES = { yesterday: 30, today: 35, tomorrow: 32 };

const { today, tomorrow } = HIGH_TEMPERATURES;
// today = 35, tomorrow = 32

// Con alias (renombrar al extraer)
const { today: maximaHoy, tomorrow: maximaManana } = HIGH_TEMPERATURES;
// maximaHoy = 35, maximaManana = 32

// Valores por defecto
const { pais = "España", ciudad = "Madrid" } = {};
```

### Intercambiar valores — el truco clásico

```js
let a = 5;
let b = 3;
[a, b] = [b, a];      // a = 3, b = 5
```

> 📌 Funciona porque primero se crea el array **`[b, a]`** = `[3, 5]` y **después**
> se desempaqueta en `a` y `b`.

### Ejemplo REAL completo copiado

**Archivo:** `bikram destructuring hector/advanced-js-bikram-destructuring-spread-rest/index.js`

```js
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
```

### 🧠 Desglose de los dos más difíciles

```js
const [, ana] = empleados;
//       ▲
// coma vacía = "deja la posición 0 en la nada, dame la 1"
```

```js
const [{ email: emailLuis }] = empleados;
//      ▲                 ▲
// toma el PRIMER elemento  y de él extrae su propiedad `email`
// renombrándola a `emailLuis`
// equivale a: empleados[0].email
```

### Destructuring en parámetros de función

```js
function mostrarPersona({ nombre, edad }) {
  console.log(`${nombre} tiene ${edad} años`);
}

mostrarPersona({ nombre: "Hector", edad: 25 });   // "Hector tiene 25 años"
```

---

## 5.6 SPREAD `...` (expandir) y REST `...` (recoger)

Aparentemente son iguales, pero van en **sitios opuestos**:

| | **REST** `...` | **SPREAD** `...` |
|---|---|---|
| ¿Dónde? | En la **declaración** (izquierda) | En la **llamada** (derecha) |
| ¿Qué hace? | **Recoge** lo que sobra en un array | **Expande** un array en elementos sueltos |
| Ejemplo | `function f(...nums)` | `f(...arr)` |

### REST — recoger argumentos

```js
function sumaTodo(...numeros) {       // numeros = [1, 2, 3]
  return numeros.reduce((t, n) => t + n, 0);
}
sumaTodo(1, 2, 3);                    // 6
```

### SPREAD — expandir

```js
const a = [1, 2, 3];
const b = [4, 5];
const c = [...a, ...b];               // [1, 2, 3, 4, 5]

function suma(a, b, c) { return a + b + c; }
suma(...[10, 20, 30]);                // 60  (se expanden como 3 argumentos)

const copia = [...original];          // copia superficial de un array
const objCopia = { ...objeto };       // copia superficial de un objeto
```

### Ejemplos REALES copiados (`bikram destructuring hector/.../index.js`)

```js
// 5.- Suma cualquier cantidad de números   ← REST en los parámetros
function sumEveryOther(...numeros) {
    return numeros.reduce((total, num) => total + num, 0);
}

// 6.- Suma solo los números, ignorando los strings   ← REST + filter + reduce
function addOnlyNums(...args) {
    return args
        .filter(arg => typeof arg === "number")
        .reduce((total, num) => total + num, 0);
}

// 7.- Devuelve cuántos argumentos ha recibido   ← REST
function countTheArgs(...args) {
    return args.length;
}

// 8.- Combina dos arrays en uno solo usando spread   ← SPREAD
function combineTwoArrays(array1, array2) {
    return [...array1, ...array2];
}

// 9.- Devuelve un array con los elementos únicos (sin repetidos)   ← SPREAD + Set
function onlyUniques(...args) {
    return [...new Set(args)];
}

// 10.- Combina cualquier cantidad de arrays en uno solo   ← REST + SPREAD + reduce
function combineAllArrays(...arrays) {
    return arrays.reduce((combinado, arr) => [...combinado, ...arr], []);
}

// 11.- Eleva cada argumento al cuadrado y suma todos los resultados
function sumAndSquare(...args) {
    return args.reduce((suma, num) => suma + num ** 2, 0);
}
```

### Desglose de `combineAllArrays`

```js
combineAllArrays([1,2], [3,4], [5]);

// arrays = [[1,2], [3,4], [5]]
// vuelta 1: []       + [1,2]  → [1,2]
// vuelta 2: [1,2]    + [3,4]  → [1,2,3,4]
// vuelta 3: [1,2,3,4]+ [5]    → [1,2,3,4,5]
```

### Desglose de `onlyUniques`

```js
onlyUniques("a", "b", "a", "c");

// args = ["a","b","a","c"]
// new Set(args)  → Set {"a", "b", "c"}     ← Set elimina duplicados
// [...Set]       → ["a", "b", "c"]         ← spread lo convierte en array
```

---

## 5.7 `Set` — colección **sin duplicados**

```js
const s = new Set([1, 2, 2, 3, 3, 3]);
[...s];                 // [1, 2, 3]

s.add(4);
s.has(2);               // true
s.delete(1);
s.size;                 // 3
```

```js
// Quitar duplicados de un array (forma moderna)
const unicos = [...new Set([1, 1, 2, 3, 3])];   // [1, 2, 3]
```

---

## 5.8 `Object` — métodos útiles

```js
const persona = { nombre: "Hector", edad: 25 };

Object.keys(persona);        // ["nombre", "edad"]      → claves
Object.values(persona);      // ["Hector", 25]          → valores
Object.entries(persona);     // [["nombre","Hector"], ["edad", 25]]
Object.assign({}, persona);  // copia superficial
Object.freeze(persona);      // ya no se puede modificar
Object.fromEntries([["a",1],["b",2]]);   // { a: 1, b: 2 }
```

**Recorrer un objeto:**

```js
for (const clave in persona) {
  console.log(clave, persona[clave]);
}

// O con entries + map
Object.entries(persona).map(([k, v]) => `${k}: ${v}`);
```

---

## 5.9 `Date` — fechas

```js
const ahora = new Date();
ahora.getFullYear();     // 2026
ahora.getMonth();        // 0-11  (¡enero es 0!)
ahora.getDate();         // día del mes
ahora.getHours();        // hora
ahora.toISOString();     // "2026-10-19T10:00:00.000Z"

new Date("2026-10-19");
new Date(2026, 9, 19);   // mes 9 = octubre
```

**Ejemplo REAL (`bikram-objetos/.../index.js`):**

```js
// 11
Concierto.fecha = new Date();
```

> 📌 Antes `Concierto.fecha` era `null`; aquí se le asigna **la fecha actual**.

---

## 5.10 JSON — objetos como texto

```js
const obj = { nombre: "Hector", skills: ["JS", "CSS"] };

// Objeto → texto
const texto = JSON.stringify(obj);    // '{"nombre":"Hector","skills":["JS","CSS"]}'

// Texto → objeto
const obj2 = JSON.parse(texto);

// Lo usará fetch / APIs:
fetch("https://api.ejemplo.com/datos")
  .then(respuesta => respuesta.json())      // texto → objeto
  .then(datos => console.log(datos));
```

---

## 5.11 Ejemplo combinado: todo junto

```js
// Objeto anidado de un proyecto real (estilo objects.js)
const tienda = {
    nombre: "MiTienda",
    productos: [
        { id: 1, nombre: "Teclado", precio: 45, stock: true },
        { id: 2, nombre: "Ratón",   precio: 20, stock: false },
        { id: 3, nombre: "Monitor", precio: 199, stock: true }
    ],
    buscarProducto(id) {
        return this.productos.find(p => p.id === id);
    }
};

// Destructuring
const { nombre, productos } = tienda;

// Acceso anidado
console.log(productos[0].precio);            // 45

// filter + map + template string
const baratos = productos
    .filter(p => p.precio < 100)
    .map(p => `${p.nombre}: ${p.precio}€`);

console.log(baratos);                        // ["Teclado: 45€", "Ratón: 20€"]

// Spread para copiar y añadir
const nuevoProducto = { id: 4, nombre: "Webcam", precio: 60, stock: true };
const actualizado = [...productos, nuevoProducto];

// spread + destructuring
const [primerProducto, ...resto] = productos;
```

---

## ✅ Resumen rápido

1. Objeto = `{ clave: valor, ... }`; se accede con `obj.clave` o `obj["clave"]`.
2. Funciones dentro del objeto = **métodos**: `obj.metodo()`.
3. Anidamiento libre: `Impresora.tinta.rojo`, `Movil.especificaciones.ancho`.
4. **Destructuring**: `const { a, b } = obj` y `const [x, y] = arr`.
   Con alias: `{ today: maximaHoy }`; saltando: `[, segundo]`.
5. **SPREAD** `...` **expande** (derecha): `[...arr]`, `f(...arr)`, `{...obj}`.
6. **REST** `...` **recoge** (izquierda): `function f(...args)`.
7. `[...new Set(array)]` = quitar duplicados en una línea.
8. `Object.keys/values/entries` para recorrer objetos.

👉 **Siguiente:** `06-dom-y-eventos.md`
