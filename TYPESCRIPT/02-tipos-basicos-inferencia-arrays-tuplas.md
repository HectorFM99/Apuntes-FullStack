# 02 — TIPOS BÁSICOS, INFERENCIA, ARRAYS Y TUPLAS

> Un **tipo** es la "etiqueta" que dice qué valores puede guardar algo.
> Se escribe con **dos puntos** después del nombre: `nombre: tipo`.

---

## 2.1 Anotaciones de tipo — la sintaxis

```ts
let variable: tipo = valor;

function funcion(parametro: tipo): tipoQueDevuelve { ... }
```

### Ejemplo REAL copiado (`ejercicios-Bikram-Typescript/.../ejercicios-typescript.ts`)

```ts
// EJERCICIO 1
let nombre: string = "Hector";
let edad: number = 27;
let activo: boolean = true;

//edad = "ventisiete"; Error: Type 'string' is not assignable to type 'number'
```

**Desglose:**

```
let nombre : string = "Hector"
     ▲         ▲         ▲
  nombre     tipo      valor
```

> 📌 Una vez que `edad` es `number`, **siempre** será `number`. Puedes cambiar
> su **valor** (`edad = 28` ✅), pero no su **tipo** (`edad = "28"` ❌).

---

## 2.2 Los tipos primitivos

| Tipo | Valores | Ejemplo |
|------|---------|---------|
| `string` | Textos | `"Hector"`, `'hola'`, `` `Hola ${x}` `` |
| `number` | Todos los números (enteros y decimales) | `27`, `12.5`, `-3`, `NaN` |
| `boolean` | `true` / `false` | `true` |
| `null` | "vacío a propósito" | `null` |
| `undefined` | "no tiene valor" | `undefined` |
| `bigint` | Enteros enormes | `100n` |
| `symbol` | Identificadores únicos | `Symbol("id")` |

> ⚠️ Se escriben en **minúscula**: `string`, no `String`. `String` (mayúscula) es el
> objeto envoltorio de JS y **no** lo debes usar como tipo.

```ts
// ❌
let nombre: String = "Ana";
// ✅
let nombre: string = "Ana";
```

> 📌 En TS **no hay** `int` ni `float`: todo es `number`. Si necesitas que sea entero,
> lo validas tú (o con `@IsInt()` en un DTO de Nest, ver `07`).

---

## 2.3 Inferencia — TypeScript adivina el tipo

No hace falta anotar **todo**. Si das un valor inicial, TS **deduce** el tipo solo:

```ts
let edad = 27;           // TS infiere: number
edad = "veintisiete";    // ❌ igual que si hubieras escrito `: number`

const saludo = "hola";   // TS infiere: "hola" (literal, porque const no cambia)
let saludo2 = "hola";    // TS infiere: string
```

### Ejemplo REAL — inferencia en tu ejercicio 2

```ts
// EJERCICIO 2
let precios: number[] = [12.5, 30, 8.99, 45];

function sumarPrecios(arr: number[]): number {
    let suma = 0;                     // ← sin anotar: TS infiere number
    for (let i = 0; i < arr.length; i++) {
        suma += arr[i];
    }
    return suma;
}

console.log(sumarPrecios(precios));
```

> 📌 `let suma = 0` ya es `number`. Escribir `let suma: number = 0` es correcto pero
> **redundante**.

### Ejemplo REAL — inferencia en el reproductor (`ProyectoReproductor/src/app.ts`)

```ts
let activeView = 'discover';      // infiere string
let currentTrackId = 0;           // infiere number
let favorites = new Set<string>();// Set<string> (aquí sí se indica el tipo de dentro)
let currentTracks: Track[] = [];  // ← AQUÍ HACE FALTA anotar (ver 2.5)
```

### ¿Cuándo anotar y cuándo dejar que infiera?

| Situación | ¿Anotar? |
|-----------|----------|
| Variable con valor inicial claro (`let x = 5`) | ❌ No hace falta |
| **Parámetros** de funciones | ✅ **Siempre** (TS no puede adivinarlos) |
| Tipo de retorno de funciones | Opcional, pero recomendable en funciones "públicas" |
| Array vacío `[]` o variable sin valor inicial | ✅ Sí |
| Objetos que deben cumplir un `type`/`interface` | ✅ Sí (así te avisa si falta algo) |

---

## 2.4 `let` vs `const` y los tipos literales

```ts
let estado = "pendiente";     // tipo: string  (puede cambiar a cualquier texto)
const estado2 = "pendiente";  // tipo: "pendiente" (no puede cambiar nunca)
```

Con `const`, el tipo es el **valor exacto**. Esto se llama **tipo literal** y se usa
muchísimo con uniones (ver `03`):

```ts
type estadoTarea = "pendiente" | "en progreso" | "completada";
```

---

## 2.5 Arrays

Dos formas de escribir lo mismo:

```ts
let precios: number[] = [12.5, 30, 8.99, 45];   // ✅ la más usada
let precios2: Array<number> = [12.5, 30];       // forma "genérica", equivalente
```

| Tipo | Significa |
|------|-----------|
| `string[]` | Array de textos |
| `number[]` | Array de números |
| `Libro[]` | Array de objetos `Libro` |
| `(string \| number)[]` | Array que mezcla textos y números |
| `Notificable[]` | Array de cualquier cosa que cumpla `Notificable` |

### Ejemplo REAL — array de objetos (`ejercicios-typescript.ts`, EJERCICIO 10)

```ts
interface Libro{
    Titulo: string;
    Autor: string;
    Paginas: number;
    Disponible: boolean;
}

const libros: Libro[] = [
    { Titulo: "El Quijote", Autor: "Miguel de Cervantes", Paginas: 863, Disponible: true },
    { Titulo: "1984", Autor: "George Orwell", Paginas: 328, Disponible: false },
    { Titulo: "Clean Code", Autor: "Robert C. Martin", Paginas: 464, Disponible: true },
];
```

> 📌 Si añades un libro sin `Paginas`, o con `Disponible: "sí"`, TS te avisa en esa
> línea exacta.

### Ejemplo REAL — el array vacío necesita tipo (`ProyectoReproductor/src/app.ts`)

```ts
let currentTracks: Track[] = [];
let queue: Track[] = [];
```

```ts
// ❌ Sin anotar
let queue = [];          // TS lo trata como any[] → pierdes la ayuda
// ✅
let queue: Track[] = []; // ahora queue.push(algoQueNoEsTrack) da error
```

### Los métodos de array conservan el tipo

```ts
const precios: number[] = [12.5, 30, 8.99];

const dobles = precios.map(p => p * 2);          // number[]
const textos = precios.map(p => `${p} €`);       // string[]  ← TS lo deduce
const caros  = precios.filter(p => p > 10);      // number[]
const total  = precios.reduce((acc, p) => acc + p, 0);  // number
```

**Ejemplo REAL (`ProyectoReproductor/src/app.ts`):**

```ts
favorites = new Set(updated.map(t => t.id));
```

`updated` es `Track[]` → `t` es `Track` → `t.id` es `string` → `new Set(...)` es
`Set<string>`. **Nadie ha escrito un solo tipo** en esa línea y todo está tipado.

### ⚠️ Acceder por índice: `arr[i]` puede no existir

```ts
const nombres: string[] = ["Ana", "Luis"];
const tercero = nombres[5];     // TS (por defecto) dice: string... ¡pero es undefined!
```

Con la opción `noUncheckedIndexedAccess` (la tienes activada en `practica-typescript`),
TS es más honesto: `nombres[5]` es `string | undefined` y te obliga a comprobarlo.

> 📌 Tu `sumarPrecios` (`suma += arr[i]`) compila sin problema con la config por defecto,
> pero con `noUncheckedIndexedAccess` daría error porque `arr[i]` podría ser
> `undefined`. Solución: `for (const precio of arr) { suma += precio; }`.

---

## 2.6 Tuplas — arrays de longitud y tipos fijos

Una **tupla** es un array donde **cada posición tiene su tipo** y el tamaño es fijo.

```ts
let persona: [string, number] = ["Hector", 27];

persona[0].toUpperCase();   // ✅ TS sabe que [0] es string
persona[1].toFixed(2);      // ✅ TS sabe que [1] es number

persona = [27, "Hector"];   // ❌ orden incorrecto
persona = ["Hector"];       // ❌ falta un elemento
```

Uso típico: devolver **dos cosas** de una función (como `useState` de React):

```ts
function minMax(nums: number[]): [number, number] {
  return [Math.min(...nums), Math.max(...nums)];
}

const [minimo, maximo] = minMax([4, 9, 1]);   // destructuring: 1 y 9
```

### Ejemplo REAL — tupla con `as const` (`ProyectoReproductor/src/login.ts`)

```ts
for (const [id, endpoint] of [['login-form', 'login'], ['register-form', 'register']] as const) {
  $<HTMLFormElement>(`#${id}`).addEventListener('submit', async event => {
    // ...
  });
}
```

**Qué hace `as const` aquí:**

```
Sin as const → string[][]   (arrays de textos cualesquiera)
Con as const → readonly [readonly ["login-form", "login"], readonly ["register-form", "register"]]
```

> 📌 `as const` convierte el array en **tuplas de solo lectura con valores literales**.
> Así `endpoint` es `"login" | "register"`, no un `string` cualquiera. Lo verás más
> en `03` y `06`.

---

## 2.7 `null` y `undefined`

Con `strict` activado, `null` y `undefined` **no caben** en otros tipos. Si algo
puede faltar, hay que decirlo con una **unión**:

```ts
let titulo: string = null;            // ❌
let titulo: string | null = null;     // ✅
```

### Ejemplo REAL (`practica-typescript/precio.ts`)

```ts
//2
type libro = {
    titulo: string | null;
    autor: string | null;
    pagina: number | null;
    disponible: boolean | null;
}
```

> 📌 Aquí **todos** los campos pueden ser `null`. Funciona, pero piensa si de verdad
> es así: ¿un libro sin título? Usa `| null` solo donde el dato **realmente** puede
> no existir. Si no, TS te obligará a comprobar `null` por todas partes.

### Ejemplo REAL — una función que puede no encontrar nada (EJERCICIO 11)

```ts
function buscarUsuario(identificador: number): string | undefined {
    if (identificador === 1) {
        return "Ana";
    }
    return undefined;
}

const usuarioEncontrado = buscarUsuario(1);
if (usuarioEncontrado !== undefined) {
    console.log(usuarioEncontrado.toUpperCase());
}
```

```ts
// ❌ Sin comprobar
buscarUsuario(2).toUpperCase();
// 'buscarUsuario(...)' is possibly 'undefined'.
```

| | `undefined` | `null` |
|---|-------------|--------|
| Significa | "no se ha asignado" / "no existe" | "vacío a propósito" |
| Lo produce JS solo | Sí (parámetros no pasados, propiedades que no existen) | No, lo pones tú |
| Uso típico en TS | Propiedades opcionales `x?:`, búsquedas sin resultado | Datos de BBDD (`NULL` de SQL) |

---

## 2.8 Tipos especiales: `any`, `unknown`, `void`, `never`

| Tipo | Significado | ¿Usarlo? |
|------|-------------|----------|
| `any` | "Cualquier cosa, **apaga** TypeScript" | ❌ Evítalo |
| `unknown` | "Cualquier cosa, pero **compruébalo** antes de usarlo" | ✅ Para datos externos |
| `void` | La función **no devuelve** nada útil | ✅ |
| `never` | **Nunca** ocurre (función que siempre lanza error, caso imposible) | Avanzado |

```ts
function mostrarLibro(libro: libro): void {      // ← real, practica-typescript/precio.ts
  console.log(`${libro.titulo} - ${libro.autor}`);
}

function fallar(mensaje: string): never {
  throw new Error(mensaje);                      // nunca llega a "terminar"
}
```

> 📌 `any` vs `unknown` tiene su propio apartado en `06`. Resumen: **`unknown` es el
> `any` seguro**.

### ⚠️ Un detalle de tus ejercicios (EJERCICIO 5)

```ts
function mostrarID(id: number | string): void { ... }

console.log(mostrarID(123));   // imprime lo de dentro... y luego "undefined"
```

> 📌 `mostrarID` ya hace `console.log` dentro y devuelve `void`. Envolverla en otro
> `console.log` imprime `undefined`. Basta con `mostrarID(123);`.

---

## 2.9 Objetos "en línea"

Puedes tipar un objeto sin crear un `type` aparte:

```ts
let punto: { x: number; y: number } = { x: 1, y: 2 };
```

### Ejemplo REAL (`ProyectoReproductor/src/app.ts`)

```ts
const user = await request<{ email: string }>('/api/auth/me');
$('#account-email').textContent = user.email;
```

### Ejemplo REAL (`api-rastro/api-rastro/src/types.ts`)

```ts
seller: { id: ObjectId; name: string };
photos: { url: string; order: number }[];
```

> 📌 `{ url: string; order: number }[]` = "array de objetos con `url` y `order`".
> Cuando el mismo objeto aparece **más de una vez**, sácalo a un `type` con nombre
> (ver `03`).

---

## ✅ Resumen rápido

1. `nombre: tipo`. Primitivos en **minúscula**: `string`, `number`, `boolean`.
2. TS **infiere** el tipo si das valor inicial → no anotes lo obvio.
3. **Parámetros siempre anotados**. Arrays vacíos también (`let x: Track[] = []`).
4. Arrays: `number[]` o `Array<number>`. Tuplas: `[string, number]`.
5. `as const` → valores literales y de solo lectura.
6. Con `strict`, lo que puede faltar se escribe `T | null` o `T | undefined`.
7. `void` = no devuelve nada. `any` ❌, `unknown` ✅.

👉 **Siguiente:** `03-union-literal-type-e-interface.md`
