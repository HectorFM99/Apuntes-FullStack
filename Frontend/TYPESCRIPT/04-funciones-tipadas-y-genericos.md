# 04 — FUNCIONES TIPADAS Y GENÉRICOS

> En TypeScript una función declara **qué recibe** (parámetros tipados) y **qué
> devuelve** (tipo de retorno). Los **genéricos** (`<T>`) son funciones que
> funcionan con **cualquier tipo** sin perder la información de cuál es.

---

## 4.1 Parámetros y tipo de retorno

```ts
function nombre(param1: Tipo1, param2: Tipo2): TipoRetorno {
  return valor;
}
```

### Ejemplo REAL copiado (`practica-typescript/precio.ts`)

```ts
//1
function calcularDescuento(precio: number, porcentaje: number): number {
const descuento = precio * (porcentaje / 100);
return precio - descuento;
}
console.log(calcularDescuento(100, 20));
```

**Desglose:**

```
function calcularDescuento( precio: number, porcentaje: number ): number {
                            ▲─────────────▲  ▲─────────────────▲   ▲
                            parámetro 1      parámetro 2           devuelve number
```

**Qué comprueba TS:**

```ts
calcularDescuento(100, 20);       // ✅ 80
calcularDescuento("100", 20);     // ❌ Argument of type 'string' is not assignable to parameter of type 'number'.
calcularDescuento(100);           // ❌ Expected 2 arguments, but got 1.
calcularDescuento(100, 20, 5);    // ❌ Expected 2 arguments, but got 3.
const x: string = calcularDescuento(100, 20);  // ❌ Type 'number' is not assignable to type 'string'.
```

> 📌 En JS, pasar de menos o de más argumentos **no da error**. En TS, sí.

### El tipo de retorno también protege el interior

```ts
function calcularDescuento(precio: number, porcentaje: number): number {
  if (porcentaje > 100) {
    return "Descuento inválido";   // ❌ Type 'string' is not assignable to type 'number'.
  }
  return precio - precio * porcentaje / 100;
}
```

> 📌 Si **no** pones `: number`, TS lo infiere del `return`. Pero si lo pones, TS te
> avisa cuando por error devuelves otra cosa. Para funciones importantes, **ponlo**.

---

## 4.2 `void` — funciones que no devuelven nada

### Ejemplo REAL (`ejercicios-typescript.ts`, EJERCICIO 7)

```ts
function mostrarProducto(producto: Producto): void {
    console.log(`Producto: ${producto.nombre}`);
    console.log(`Precio: ${producto.precio} €`);
    if (producto.descripcion) {
        console.log(`Descripción: ${producto.descripcion}`);
    }
}
```

| Retorno | Úsalo cuando... |
|---------|-----------------|
| `void` | La función **hace** algo (pinta, imprime, guarda) pero no devuelve |
| `number`, `string`, `Libro`... | La función **calcula** algo y lo devuelve |
| `Promise<T>` | La función es `async` (ver 4.6) |
| `never` | La función **siempre** lanza un error |

---

## 4.3 Parámetros opcionales `?` y por defecto `=`

### Ejemplo REAL — opcional (`ejercicios-typescript.ts`, EJERCICIO 3)

```ts
function presentar(nombre: string, apellido?:string): string {
    if (apellido) {
        return `Hola, me llamo ${nombre} ${apellido}.`;
    } else {
        return `Hola, me llamo ${nombre}.`;
    }
}

console.log(presentar("Ana", "Perez"));   // "Hola, me llamo Ana Perez."
console.log(presentar("Ana"));            // "Hola, me llamo Ana."
```

### Ejemplo REAL — por defecto (`ProyectoReproductor/src/app.ts`)

```ts
function formatTime(seconds = 0) {
  const s = Math.floor(seconds);
  return `${Math.floor(s / 60)}:${String(s % 60).padStart(2, '0')}`;
}

function setStatus(message: string, error = false) {
  status.textContent = message;
  status.classList.toggle('error', error);
}
```

> 📌 Con valor por defecto **no hace falta** poner el tipo: `seconds = 0` ya es
> `number`, `error = false` ya es `boolean`. Y además el parámetro pasa a ser opcional.

### Ejemplo REAL — opcional en Nest (`puta/api-books-nest/src/books/books.service.ts`)

```ts
async findAll(author?: string, maxPages?: number): Promise<Book[]> {
```

| Forma | Tipo dentro | ¿Obligatorio al llamar? |
|-------|-------------|------------------------|
| `x: string` | `string` | ✅ Sí |
| `x?: string` | `string \| undefined` | ❌ No |
| `x = "hola"` | `string` | ❌ No (usa `"hola"` si falta) |
| `x: string \| undefined` | `string \| undefined` | ✅ Sí (hay que pasar algo, aunque sea `undefined`) |

> ⚠️ Los opcionales van **al final**: `function f(a?: string, b: number)` ❌.

---

## 4.4 Funciones flecha tipadas

```ts
const sumar = (a: number, b: number): number => a + b;
```

### Ejemplo REAL (`git/Pagina-Web/ts/programacion.ts`)

```ts
const escribir = (): void => {
    if (posicion < texto.length) {
      const contenidoActual: string = elemento.textContent ?? "";
      elemento.textContent = contenidoActual + texto.charAt(posicion);
      posicion++;
      setTimeout(escribir, 10); // Más milisegundos = más lento
    }
  };
```

```
const escribir = ( ): void => { ... }
                 ▲    ▲
          sin params  no devuelve nada
```

### Callbacks: los parámetros se infieren solos

Cuando pasas una función a `map`, `forEach`, `addEventListener`... **no hace falta**
tipar sus parámetros: TS ya sabe qué le van a llegar (*tipado contextual*).

**Ejemplo REAL (`practica-typescript/precio.ts`):**

```ts
const todasLasTareas: Tarea[] = [ /* ... */ ];
 todasLasTareas.forEach((tarea) => mostrarTarea(tarea));
//                       ▲ TS sabe que tarea es Tarea
```

**Ejemplo REAL — aquí sí se anotó (`git/Pagina-Web/ts/programacion.ts`):**

```ts
const observer = new IntersectionObserver(
  (entries: IntersectionObserverEntry[], observer: IntersectionObserver) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add("is-visible");
        observer.unobserve(entry.target);
      }
    });
  },
  { threshold: 0.2 }
);
```

> 📌 Esas anotaciones (`entries: IntersectionObserverEntry[]`) son correctas pero
> **opcionales**: `IntersectionObserver` ya define qué recibe su callback. `(entries, observer) => ...`
> estaría igual de tipado. Fíjate en que `entry` (dentro del `forEach`) no se anotó y
> TS sabe que es `IntersectionObserverEntry`.

---

## 4.5 Tipar una función como parámetro (callbacks propios)

Cuando **tú** escribes una función que recibe otra, describes su forma así:

```ts
(parametro: Tipo) => TipoRetorno
```

```ts
// Ejemplo propio (no está en tus proyectos)
function aplicar(numeros: number[], operacion: (n: number) => number): number[] {
  return numeros.map(operacion);
}

aplicar([1, 2, 3], n => n * 2);        // ✅ [2, 4, 6]
aplicar([1, 2, 3], n => `${n}`);       // ❌ Type 'string' is not assignable to type 'number'.
```

También puedes ponerle nombre al tipo de función:

```ts
type Operacion = (a: number, b: number) => number;

const suma: Operacion = (a, b) => a + b;     // a y b ya son number
const resta: Operacion = (a, b) => a - b;
```

### Ejemplo REAL — callback en un decorador (`puta/api-books-nest/src/books/dto/create-book.dto.ts`)

```ts
@Transform(({ value }: { value: unknown }) =>
    typeof value === 'string' ? value.trim() : value,
)
```

> 📌 La función recibe un objeto, se desestructura `{ value }`, y se anota `value`
> como `unknown` (llega del cliente, puede ser cualquier cosa) → se comprueba con
> `typeof` antes de usar `.trim()`.

---

## 4.6 Funciones `async` → `Promise<T>`

Una función `async` **siempre** devuelve una `Promise`. El tipo se escribe
`Promise<LoQueDevuelve>`.

### Ejemplo REAL (`ejercicios-typescript.ts`, EJERCICIO 20)

```ts
async function obtenerMensaje(): Promise<string> {
    let mejorAlumno = "";
    let mejorNota = -1;

    for (const nombre of Object.keys(puntuaciones)) {
        if (puntuaciones[nombre] > mejorNota) {
            mejorNota = puntuaciones[nombre];
            mejorAlumno = nombre;
        }
    }

    return `El alumno con la puntuación más alta es ${mejorAlumno} con ${mejorNota} puntos.`;
}

obtenerMensaje().then((mensaje) => console.log(mensaje));
//                     ▲ mensaje: string
```

### Ejemplo REAL — servicio Nest (`puta/api-books-nest/src/books/books.service.ts`)

```ts
async findOne(id: number): Promise<Book> {
    const result = await this.pool.query<BookRow>(
      'SELECT id, title, author, pages, created_at FROM books WHERE id = $1',
      [id],
    );
    if (result.rows.length === 0) {
      throw new NotFoundException('Libro no encontrado');
    }
    return toBook(result.rows[0]);
}

async count(): Promise<{ total: number }> {
    const result = await this.pool.query<{ total: string }>(
      'SELECT COUNT(*)::text AS total FROM books',
    );
    return { total: Number(result.rows[0].total) };
}
```

```ts
// ❌ Una función async NO puede devolver Book a secas
async findOne(id: number): Book { ... }
// The return type of an async function or method must be the global Promise<T> type.

// ✅
async findOne(id: number): Promise<Book> { ... }

// Al usarla con await, obtienes el Book:
const libro = await service.findOne(1);   // libro: Book
```

---

## 4.7 Genéricos `<T>` — la idea

Problema: quiero una función `primero` que devuelva el primer elemento de **cualquier** array.

```ts
// ❌ Con any: funciona, pero pierdes el tipo
function primero(elementos: any[]): any {
  return elementos[0];
}
const n = primero([10, 20]);   // n: any  → TS ya no te ayuda
```

```ts
// ✅ Con genérico: T es "el tipo que sea", y TS lo recuerda
function primero<T>(elementos: T[]): T | undefined {
  return elementos[0];
}
```

### Ejemplo REAL copiado (`ejercicios-typescript.ts`, EJERCICIO 19)

```ts
function primero<T>(elementos: T[]): T | undefined {
    return elementos[0];
}

console.log(primero([10, 20, 30]));       // T = number  → devuelve number | undefined
console.log(primero(["Ana", "Luis"]));    // T = string  → devuelve string | undefined
console.log(primero([]));                 // T = never   → undefined
```

**Cómo leerlo:**

```
function primero <T> ( elementos: T[] ): T | undefined
                  ▲               ▲      ▲
       "hay un tipo T"   recibe array de T   devuelve un T (o nada)
```

> 📌 `T` es una **variable de tipo**: como un parámetro, pero para tipos. Se rellena
> **sola** según lo que le pases. El nombre `T` es convención (Type); puedes usar
> `<Item>`, `<K, V>`...

---

## 4.8 Genéricos que ya usas sin darte cuenta

```ts
Array<number>          // = number[]
Promise<string>        // promesa que resolverá un string
Set<string>            // conjunto de strings
Map<string, Track[]>   // diccionario de string → array de Track
Record<string, number> // objeto clave string → valor number
```

### Ejemplo REAL — `Map` y `Set` (`ProyectoReproductor/src/app.ts`)

```ts
let favorites = new Set<string>();

const albumMap = new Map<string, Track[]>();
tracks.forEach(t => {
    const alb = t.album || 'Álbum desconocido';
    if (!albumMap.has(alb)) albumMap.set(alb, []);
    albumMap.get(alb)!.push(t);
});
```

> 📌 `new Set()` sin `<string>` sería `Set<unknown>` y luego `favorites.has(track.id)`
> no te ayudaría. Cuando el contenedor empieza **vacío**, dale el tipo con `<...>`.

### Ejemplo REAL — MongoDB tipado (`api-rastro/api-rastro/src/db.ts`)

```ts
export function usersCollection() {
  return getDb().collection<User>("users");
}

export function listingsCollection() {
  return getDb().collection<Listing>("listings");
}

export function offersCollection() {
  return getDb().collection<Offer>("offers");
}
```

> 📌 `collection<Listing>` → a partir de aquí, `findOne` devuelve un `Listing`,
> `insertOne` exige un `Listing`, etc. **Un solo `<Listing>` tipa todas las consultas**.

### Ejemplo REAL — PostgreSQL tipado (`puta/api-books-nest/src/books/books.service.ts`)

```ts
const result = await this.pool.query<BookRow>(
  `SELECT id, title, author, pages, created_at
   FROM books ...`,
  [author || null, maxPages ?? null],
);
return result.rows.map(toBook);   // result.rows: BookRow[]
```

> ⚠️ Ojo: `query<BookRow>` es una **promesa que le haces a TS**, no una comprobación.
> Si tu `SELECT` no trae `created_at`, TS no lo sabe. Mantén el SQL y la interfaz
> sincronizados.

---

## 4.9 Escribir tus propios genéricos — ejemplos REALES

### `request<T>` — un `fetch` tipado (`ProyectoReproductor/src/login.ts`)

```ts
async function request<T>(url: string, options?: RequestInit): Promise<T> {
  const response = await fetch(url, {
    headers: { 'Content-Type': 'application/json', ...(options?.headers || {}) },
    ...options
  });
  if (!response.ok) {
    const body = await response.json().catch(() => ({}));
    throw new Error(body.error || 'No se pudo completar la solicitud.');
  }
  return response.status === 204 ? (undefined as T) : (response.json() as Promise<T>);
}
```

**Uso (`ProyectoReproductor/src/app.ts`):**

```ts
const tracks = await request<Track[]>(`/api/tracks/search?q=${encodeURIComponent(query)}`);
//    ▲ tracks: Track[]

const user = await request<{ email: string }>('/api/auth/me');
//    ▲ user: { email: string }
```

> 📌 Aquí `T` **no se puede adivinar** (depende de lo que responda el servidor), así
> que lo indicas tú al llamar: `request<Track[]>(...)`. Es el patrón más útil de
> genéricos en el frontend.

### `one<T>` y `many<T>` — consultas SQL (`ProyectoReproductor/server/index.ts`)

```ts
function one<T>(sql: string, params: unknown[] = []): T | undefined { const statement = db.prepare(sql); statement.bind(params as never[]); const result = statement.step() ? statement.getAsObject() as T : undefined; statement.free(); return result; }
function many<T>(sql: string, params: unknown[] = []): T[] { const statement = db.prepare(sql); statement.bind(params as never[]); const results: T[] = []; while (statement.step()) results.push(statement.getAsObject() as T); statement.free(); return results; }
```

Formateado para leer:

```ts
function one<T>(sql: string, params: unknown[] = []): T | undefined {
  const statement = db.prepare(sql);
  statement.bind(params as never[]);
  const result = statement.step() ? statement.getAsObject() as T : undefined;
  statement.free();
  return result;
}
```

**Uso:**

```ts
const user = one<{id:string;email:string;password_hash:string}>('SELECT id,email,password_hash FROM users WHERE email=?',[email]);
```

### `$<T extends Element>` — genérico con restricción (`ProyectoReproductor/src/app.ts`)

```ts
const $ = <T extends Element>(selector: string) => document.querySelector<T>(selector)!;
```

```
<T extends Element>  → T puede ser cualquier tipo... siempre que SEA un Element
                       (HTMLButtonElement, HTMLInputElement, SVGElement...)
```

**Uso:**

```ts
const input = $<HTMLInputElement>('#search-input');
input.focus();                              // ✅ input.value existe

$<HTMLButtonElement>('#queue-close').focus();
$<string>('#x');                            // ❌ string no es un Element
```

> 📌 `extends` en un genérico = **restricción**: "T, pero como mínimo esto". Así
> dentro de la función puedes usar todo lo que tiene un `Element`.

---

## 4.10 Errores comunes con funciones

| Error | Causa | Solución |
|-------|-------|----------|
| `Parameter 'x' implicitly has an 'any' type` | Parámetro sin tipo (con `strict`) | Anótalo: `x: number` |
| `Expected 2 arguments, but got 1` | Falta un argumento | Pásalo o hazlo opcional `x?:` |
| `A required parameter cannot follow an optional parameter` | Opcional antes que obligatorio | Mueve los opcionales al final |
| `Function lacks ending return statement` | Un camino del `if` no devuelve nada | Añade `return` o cambia el tipo |
| `The return type of an async function... must be Promise<T>` | `async f(): Book` | `async f(): Promise<Book>` |
| Todo es `any` tras usar mi función | No usaste genéricos | `function f<T>(x: T): T` |

---

## ✅ Resumen rápido

1. `function f(a: number, b: string): Retorno { ... }`.
2. `void` si no devuelve nada; `Promise<T>` si es `async`.
3. Opcionales `x?:` y por defecto `x = valor` → siempre al final.
4. En callbacks (`map`, `forEach`, `addEventListener`) los parámetros **se infieren**.
5. Tipo de función: `(n: number) => number`.
6. Genéricos `<T>`: una función, muchos tipos, sin perder información.
7. `<T extends Algo>` restringe lo que puede ser `T`.
8. `request<Track[]>()`, `collection<Listing>()`, `query<BookRow>()` → genéricos de tus proyectos.

👉 **Siguiente:** `05-enums-clases-y-utility-types.md`
