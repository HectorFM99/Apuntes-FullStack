# 06 — NARROWING, `any` vs `unknown` Y ASERCIONES

> **Narrowing** (estrechamiento) = cuando un valor puede ser varias cosas
> (`string | number`, `Track | undefined`, `unknown`...) y tú lo **compruebas**
> con un `if`, TypeScript **reduce** su tipo dentro de ese bloque.
> Es la forma **segura** de trabajar con datos inciertos.

---

## 6.1 La idea

```ts
function mostrarID(id: number | string) {
  id.toUpperCase();          // ❌ Property 'toUpperCase' does not exist on type 'number'.

  if (typeof id === "string") {
    id.toUpperCase();        // ✅ aquí dentro id es string
  } else {
    id.toFixed(2);           // ✅ aquí dentro id es number
  }
}
```

```
id: number | string
        │
   typeof id === "string" ?
     ├── sí → id: string
     └── no → id: number
```

TS **lee tus `if`** y ajusta el tipo. No es magia: solo funciona con comprobaciones
que TS entiende (las de este archivo).

---

## 6.2 `typeof` — para primitivos

Valores posibles: `"string"`, `"number"`, `"boolean"`, `"undefined"`, `"object"`,
`"function"`, `"bigint"`, `"symbol"`.

### Ejemplo REAL copiado (`ejercicios-typescript.ts`, EJERCICIO 5)

```ts
function mostrarID(id: number | string): void {
    if (typeof id === "string") {
        console.log(id.toUpperCase());
    }else{
        console.log(id.toFixed(2));
    }
}
```

### Ejemplo REAL — validar datos de una petición (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
if (typeof req.query.category === "string") {
    filter.category = req.query.category;
}

if (typeof req.query.city === "string") {
    filter.city = req.query.city;
}
```

> 📌 En Express, `req.query.x` puede ser `string`, un array de strings, un objeto o
> `undefined` (`?city=a&city=b` → array). El `typeof ... === "string"` deja pasar
> solo el caso normal.

> ⚠️ `typeof null === "object"`. Para comprobar objetos, descarta `null` aparte
> (lo verás en 6.8).

---

## 6.3 Comprobaciones de "verdad" (truthiness) y `!== undefined`

```ts
if (valor) { ... }   // descarta undefined, null, "", 0, NaN, false
```

### Ejemplo REAL (`ejercicios-typescript.ts`, EJERCICIO 3)

```ts
function presentar(nombre: string, apellido?:string): string {
    if (apellido) {                      // apellido: string | undefined
        return `Hola, me llamo ${nombre} ${apellido}.`;   // aquí: string
    } else {
        return `Hola, me llamo ${nombre}.`;
    }
}
```

### Ejemplo REAL — comprobación explícita (EJERCICIO 11)

```ts
const usuarioEncontrado = buscarUsuario(1);      // string | undefined
if (usuarioEncontrado !== undefined) {
    console.log(usuarioEncontrado.toUpperCase()); // string
}
```

### Ejemplo REAL — "early return" (`ProyectoReproductor/src/app.ts`)

```ts
play.addEventListener('click', async () => {
  if (!current) return;          // current: Track | undefined
  if (audio.paused) await audio.play();
  else audio.pause();
});
```

### Ejemplo REAL — en una API (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
const listing = await listingsCollection().findOneAndUpdate(
    { _id: new ObjectId(id) },
    { $inc: { views: 1 } },
    { returnDocument: "after" },
);

if (!listing) {
    res.status(404).json({ error: "No existe ningún anuncio con ese id" });
    return;
}

res.json(listing);   // aquí listing ya NO puede ser null
```

> 📌 Patrón **guard clause**: compruebas lo malo, sales con `return`, y el resto de la
> función trabaja con el tipo "limpio". TS entiende el `return` y estrecha el tipo
> para todo lo que viene después.

> ⚠️ Truthiness con números: `if (cantidad)` descarta el `0`. Si `0` es un valor
> válido, usa `if (cantidad !== undefined)`.

---

## 6.4 `instanceof` — para clases y objetos del DOM

### Ejemplo REAL copiado (`git/Pagina-Web/ts/programacion.ts`)

```ts
// Menú lateral hamburguesa: abre/cierra el menú al hacer clic
const botonMenu = document.querySelector(".boton-hamburguesa");
const menuLateral = document.querySelector(".menu-lateral");

if (botonMenu instanceof HTMLElement && menuLateral instanceof HTMLElement) {
  botonMenu.addEventListener("click", function () {
    menuLateral.classList.toggle("abierto");
  });

  document.addEventListener("click", function (evento) {
    const destino = evento.target;
    const clicEnMenu = destino instanceof Node && menuLateral.contains(destino);
    const clicEnBoton = destino instanceof Node && botonMenu.contains(destino);

    if (!clicEnMenu && !clicEnBoton) {
      menuLateral.classList.remove("abierto");
    }
  });
}
```

**Por qué hace falta:**

```
document.querySelector(".boton-hamburguesa")  →  Element | null
                                                  (puede no existir en la página)
evento.target                                 →  EventTarget | null
menuLateral.contains(...)                     →  pide un Node | null
```

`instanceof HTMLElement` descarta el `null` **y** confirma que es un elemento HTML.
`destino instanceof Node` convierte el `EventTarget` genérico en un `Node`.

### Comparación con la versión JS original de la misma web

Antes de pasarla a TypeScript, tu web tenía este JavaScript (está copiado en
`Apuntes/JAVASCRIPT/07-ejemplos-completos-de-proyectos.md`; hoy `js/programacion.js`
ya es la **salida compilada** del `.ts`):

```js
// versión JavaScript original
const clicEnMenu = menuLateral.contains(evento.target);
```

```ts
// ts/programacion.ts (versión TypeScript)
const clicEnMenu = destino instanceof Node && menuLateral.contains(destino);
```

> 📌 En JS, si `.menu-lateral` no existiera, el código **reventaría** en ejecución
> (`Cannot read properties of null`). TS te **obliga** a contemplarlo.

### Ejemplo REAL — otra vez en la misma web

```ts
document.querySelectorAll("a, button, .tarjeta, .contacto-social, .tarjeta-estudio, .tarjeta-certificado").forEach((el) => {
  el.addEventListener("mousedown", () => {
    if (el instanceof HTMLElement) {
      el.style.filter = "brightness(1.4) hue-rotate(15deg)";
      setTimeout(() => {
        el.style.filter = "";
      }, 150);
    }
  });
});
```

> 📌 `querySelectorAll` devuelve `Element`s, y `Element` **no tiene** `.style` (un SVG
> también es un Element). `instanceof HTMLElement` lo habilita.

### Ejemplo REAL — errores (`api-rastro/api-rastro/src/index.ts`)

```ts
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  if (err instanceof SyntaxError) {
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }

  console.error(err);
  res.status(500).json({ error: "Error interno del servidor" });
});
```

---

## 6.5 Operadores `?.` y `??`

| Operador | Nombre | Qué hace |
|----------|--------|----------|
| `a?.b` | Encadenamiento opcional | Si `a` es `null`/`undefined`, devuelve `undefined` en vez de romper |
| `a ?? b` | Fusión de nulos | Si `a` es `null`/`undefined`, usa `b` |
| `a \|\| b` | O lógico | Si `a` es **falsy** (`""`, `0`, `false`...), usa `b` |

### Ejemplo REAL (`git/Pagina-Web/ts/programacion.ts`)

```ts
const elemento = document.getElementById("texto-escrito");

if (elemento) {
  const texto: string = elemento.textContent?.trim() ?? "";
  elemento.textContent = "";
  // ...
}
```

```
elemento.textContent             → string | null
elemento.textContent?.trim()     → string | undefined   (si es null, no llama a trim)
... ?? ""                        → string               (si salió undefined, usa "")
```

### Ejemplo REAL (`api-rastro/api-rastro/src/db.ts`)

```ts
db = client.db(process.env.DB_NAME ?? "rastro");
```

> 📌 `process.env.X` siempre es `string | undefined` (la variable puede no estar
> definida). `?? "rastro"` da un valor por defecto.

### `??` vs `||` — la diferencia importa

```ts
const paginas = 0;
paginas || 100;   // 100  ← ¡0 es falsy!
paginas ?? 100;   // 0    ← solo reemplaza null/undefined
```

**Ejemplo REAL (`puta/api-books-nest/src/books/books.service.ts`):**

```ts
[author || null, maxPages ?? null],
```

> 📌 Bien pensado: si `author` es `""` (vacío) se trata como "sin filtro" (`||`), pero
> si `maxPages` fuera `0` se respetaría (`??`).

---

## 6.6 `in` y uniones discriminadas (ejemplo propio)

Cuando la unión es de **objetos**, puedes distinguirlos por una propiedad.

```ts
// Ejemplo propio (no está en tus proyectos)
type Exito = { ok: true; datos: string[] };
type Fallo = { ok: false; error: string };
type Resultado = Exito | Fallo;

function mostrar(r: Resultado) {
  if (r.ok) {
    console.log(r.datos.length);  // r: Exito
  } else {
    console.log(r.error);         // r: Fallo
  }
}

// Con `in`: "¿tiene esta propiedad?"
function mostrar2(r: Resultado) {
  if ("error" in r) console.log(r.error);   // r: Fallo
}
```

> 📌 La propiedad común con valores literales (`ok: true` / `ok: false`, o
> `status: "AVAILABLE" | ...`) se llama **discriminante**. Es el patrón más potente
> para modelar estados.

---

## 6.7 Type predicates: `value is Tipo` (tus propios "typeof")

Puedes escribir una función de comprobación que **enseñe a TS** a estrechar el tipo.

### Ejemplo REAL copiado (`api-rastro/api-rastro/src/routes/listings.ts`)

```ts
function isNonEmptyString(value: unknown): value is string {
  return typeof value === "string" && value.trim() !== "";
}

function isValidPrice(value: unknown): value is number {
  return typeof value === "number" && Number.isFinite(value) && value >= 0;
}
```

```
function isValidPrice(value: unknown): value is number
                                       ▲───────────────▲
           "si devuelvo true, entonces value ES un number"
```

**Uso real (mismo archivo):**

```ts
const price = req.body?.price;      // any (el body no tiene tipo)
if (!isValidPrice(price)) {
    res.status(400).json({ error: "price es obligatorio y tiene que ser un número mayor o igual que 0" });
    return;
}
// a partir de aquí, price es number
```

> 📌 Si la función devolviera solo `boolean`, TS **no** estrecharía el tipo después
> del `if`. El `value is number` es lo que lo consigue.

---

## 6.8 `any` vs `unknown`

Los dos admiten **cualquier valor**. La diferencia está en **qué te deja hacer con él**.

```ts
let a: any = "hola";
a.toUpperCase();       // ✅ compila
a.metodoQueNoExiste(); // ✅ compila... y REVIENTA al ejecutar
a.foo.bar.baz;         // ✅ compila... y REVIENTA

let u: unknown = "hola";
u.toUpperCase();       // ❌ 'u' is of type 'unknown'.
if (typeof u === "string") {
  u.toUpperCase();     // ✅ después de comprobarlo
}
```

| | `any` | `unknown` |
|---|-------|-----------|
| Acepta cualquier valor | ✅ | ✅ |
| Puedes usarlo sin comprobar | ✅ (peligroso) | ❌ |
| Se "contagia" a otras variables | ✅ (todo lo que toca se vuelve `any`) | ❌ |
| Cuándo | Casi nunca (migrar JS antiguo) | Datos externos: JSON, `catch`, body de peticiones |

### Ejemplo REAL copiado (`ejercicios-typescript.ts`, EJERCICIO 12)

```ts
let dato: unknown = "texto de ejemplo";

if (typeof dato === "string") {
    console.log(dato.toUpperCase());
}

dato = 42;

if (typeof dato === "string") {
    console.log(dato.toUpperCase());
} else {
    console.log("El dato no es un texto, no se puede convertir a mayúsculas.");
}
```

### Ejemplo REAL — `unknown` en un tipo de datos (`api-rastro/api-rastro/src/types.ts`)

```ts
attributes: Record<string, unknown>;
```

> 📌 "Los atributos son un objeto con claves de texto, y no sé qué hay en cada una"
> (talla, color, km...). Con `unknown`, quien lo lea **tendrá que comprobar** el tipo.

### Ejemplo REAL — validar que algo es un objeto (`api-rastro/.../routes/listings.ts`)

```ts
if (
    body.attributes !== undefined &&
    (typeof body.attributes !== "object" || body.attributes === null || Array.isArray(body.attributes))
) {
    res.status(400).json({ error: "attributes tiene que ser un objeto" });
    return;
}
```

> 📌 Para saber si algo es "un objeto de verdad" hay que descartar **tres** trampas de
> JS: que no sea `"object"`, que sea `null` (`typeof null === "object"`) y que sea un
> array (`typeof [] === "object"`).

### ⚠️ Dónde se cuela `any` en tus proyectos

- `req.body` en Express es `any` → por eso se valida **todo** a mano antes de usarlo.
- `response.json()` de `fetch` devuelve `Promise<any>`.
- `JSON.parse(...)` devuelve `any`.

En estos casos, trata el dato **como si fuera `unknown`**: compruébalo antes de confiar en él.

---

## 6.9 `catch (error)` — el error es `unknown`

Con `strict`, la variable del `catch` es `unknown` (cualquier cosa se puede lanzar:
`throw "texto"`, `throw 42`...).

```ts
try {
  // ...
} catch (error) {
  console.log(error.message);              // ❌ 'error' is of type 'unknown'.

  if (error instanceof Error) {
    console.log(error.message);            // ✅ forma segura
  }
}
```

### Ejemplo REAL (`ProyectoReproductor/src/app.ts`)

```ts
} catch (error) {
    setStatus((error as Error).message, true);
}
```

> 📌 `(error as Error)` funciona porque **tú** sabes que `request()` siempre lanza
> `new Error(...)`. Pero es una **promesa a TS**, no una comprobación. La versión 100%
> segura es `error instanceof Error ? error.message : "Error desconocido"`.

---

## 6.10 Aserciones: `as`, `!` y `as const`

Una **aserción** le dice a TS: "confía en mí, sé más que tú sobre este tipo".
**No comprueba nada** en ejecución.

### `as Tipo`

**Ejemplo REAL (`ProyectoReproductor/src/app.ts`):**

```ts
const audio = $('#audio') as HTMLAudioElement, play = $('#play-button') as HTMLButtonElement, progress = $('#progress') as HTMLInputElement;
```

> 📌 `$('#audio')` devuelve un `Element` genérico; tú **sabes** que en el HTML es un
> `<audio>`. Con `as HTMLAudioElement` ya puedes usar `audio.play()`, `audio.paused`...
> Si un día cambias el HTML y ya no es `<audio>`, TS **no** te avisará.

**Ejemplo REAL — respuesta de una API externa (`ProyectoReproductor/server/index.ts`):**

```ts
const payload=await response.json() as {headers?:{status?:string;error_message?:string};results?:Array<Record<string,unknown>>};
```

### `!` — "esto NO es null"

**Ejemplo REAL (`ProyectoReproductor/src/app.ts`):**

```ts
const $ = <T extends Element>(selector: string) => document.querySelector<T>(selector)!;

albumMap.get(alb)!.push(t);
```

```
document.querySelector(...)    → T | null
document.querySelector(...)!   → T          ("te prometo que existe")

albumMap.get(alb)              → Track[] | undefined
albumMap.get(alb)!             → Track[]    (la línea anterior acaba de hacer set)
```

> ⚠️ Si te equivocas, el error aparece **en ejecución** (`Cannot read properties of null`),
> justo lo que TS intentaba evitar. Úsalo solo cuando sea **imposible** que falle.

### `as const` — congelar valores literales

**Ejemplo REAL (`api-rastro/api-rastro/src/routes/listings.ts`):**

```ts
for (const field of ["title", "description", "category", "city"] as const) {
    if (!isNonEmptyString(body[field])) {
      res.status(400).json({ error: `${field} es obligatorio y tiene que ser un texto no vacío` });
      return;
    }
}
```

```
Sin as const → field: string
Con as const → field: "title" | "description" | "category" | "city"
```

### Comparación: aserción vs narrowing

```ts
// ❌ Aserción: compila siempre, puede fallar en ejecución
const boton = document.querySelector(".boton") as HTMLButtonElement;
boton.disabled = true;

// ✅ Narrowing: si no existe o no es un botón, no entra
const boton2 = document.querySelector(".boton");
if (boton2 instanceof HTMLButtonElement) {
  boton2.disabled = true;
}
```

> 📌 Tu **Pagina-Web** usa narrowing (`instanceof`) y tu **ProyectoReproductor** usa
> aserciones (`as`, `!`). Las dos compilan; la primera es más segura, la segunda más
> corta. Prioriza narrowing, sobre todo con datos que vienen de fuera.

### 🚩 La aserción "doble" — señal de alarma

**Ejemplo REAL (`ProyectoReproductor/server/index.ts`):**

```ts
function run(sql: string, params: unknown[] = []) { db.run(sql, params as never[]); persist(); }
```

> 📌 `as never[]` es un "truco" para forzar a TS a aceptar los parámetros cuando los
> tipos de la librería no encajan. Funciona, pero apaga la comprobación. Cuando veas
> `as never` o `as unknown as X`, es que se está **peleando** con el sistema de tipos.

---

## ✅ Resumen rápido

1. **Narrowing**: compruebas con `if` y TS ajusta el tipo dentro.
2. `typeof` (primitivos), `instanceof` (clases / DOM), `if (x)` / `!== undefined` (nulos), `in` (propiedades).
3. **Guard clauses**: comprueba lo malo y haz `return`; el resto queda "limpio".
4. `value is Tipo` → tus propias funciones de comprobación.
5. `?.` evita romper con `null`; `??` da valor por defecto (solo `null`/`undefined`).
6. `unknown` ✅ = cualquier cosa, **compruébala**. `any` ❌ = apaga TypeScript.
7. `catch (error)` → `error` es `unknown` → `error instanceof Error`.
8. `as`, `!` = "confía en mí" (sin comprobar). `as const` = valores literales fijos.

👉 **Siguiente:** `07-modulos-y-typescript-con-node-express.md`
