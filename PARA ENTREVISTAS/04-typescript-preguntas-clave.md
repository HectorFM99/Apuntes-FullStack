# 04 — 🔷 TYPESCRIPT: PREGUNTAS CLAVE

> **TypeScript = JavaScript + tipos.** Se compila (transpila) a JS. Los tipos solo
> existen **mientras programas**: en ejecución desaparecen.

---

## 4.1 ¿Por qué TypeScript?

| Ventaja | Ejemplo |
|---------|---------|
| 🐛 Detecta errores **antes** de ejecutar | Pasar un `string` donde se espera `number` |
| 💡 Autocompletado en el editor | Sabe qué propiedades tiene cada objeto |
| 📖 El código se documenta solo | La firma de una función dice qué recibe y devuelve |
| 🔧 Refactors seguros | Cambias un nombre y te marca todos los sitios rotos |

```ts
// JS: el error aparece en producción
function total(precio, cantidad) { return precio * cantidad; }
total("10", 2); // 20... o un bug raro

// TS: el error aparece en el editor
function total(precio: number, cantidad: number): number {
  return precio * cantidad;
}
total("10", 2); // ❌ Argument of type 'string' is not assignable to 'number'
```

> ⚠️ TS **no valida datos en tiempo de ejecución**. Lo que llega de una petición HTTP
> puede ser cualquier cosa → hay que validarlo (DTOs con `class-validator` en Nest,
> Zod, etc.). Pregunta típica.

> 📌 En `api-rastro` tienes un script `"typecheck": "tsc --noEmit"`, que comprueba los
> tipos sin generar JS, y tu CI lo ejecuta en cada push.

---

## 4.2 Tipos básicos

```ts
let nombre: string = "Héctor";
let edad: number = 27;
let activo: boolean = true;
let etiquetas: string[] = ["js", "ts"];
let par: [string, number] = ["edad", 27];   // tupla
let nada: null = null;

function log(msg: string): void { console.log(msg); }      // no devuelve nada
function fallar(msg: string): never { throw new Error(msg); } // nunca termina
```

> 📌 **Inferencia**: no hace falta tipar todo. `const x = 5` ya se sabe que es `number`.
> Tipa sobre todo **parámetros de funciones** y **estructuras de datos**.

---

## 4.3 `interface` vs `type` ⭐

```ts
interface Libro {
  id: number;
  titulo: string;
  paginas?: number;          // opcional
  readonly isbn: string;     // solo lectura
}

type Estado = "AVAILABLE" | "RESERVED" | "SOLD";   // unión de literales
type Id = string | number;
```

| | `interface` | `type` |
|---|---|---|
| Objetos | ✅ | ✅ |
| Uniones (`A \| B`), primitivos, tuplas | ❌ | ✅ |
| Extender | `extends` | Intersección `&` |
| Declaration merging (declararla 2 veces y se juntan) | ✅ | ❌ |
| `implements` en clases | ✅ | ✅ (si es un objeto) |

> 📌 Regla práctica: `interface` para la **forma de objetos/contratos**; `type` para
> **uniones** y tipos compuestos. Lo importante es ser **consistente**.

**Ejemplo real (`api-rastro/src/types.ts`):**

```ts
export type ListingStatus = "AVAILABLE" | "RESERVED" | "SOLD";

export type Listing = {
  title: string;
  price: number;
  status: ListingStatus;
  seller: { id: ObjectId; name: string };
  photos: { url: string; order: number }[];
  attributes: Record<string, unknown>;
};
```

---

## 4.4 `any` vs `unknown` vs `never` ⭐

| Tipo | Significado | ¿Seguro? |
|------|-------------|----------|
| `any` | "Desactiva TypeScript aquí" | ❌ Evítalo |
| `unknown` | "No sé qué es, **compruébalo antes de usarlo**" | ✅ |
| `never` | "Esto no puede ocurrir nunca" | ✅ |

```ts
function procesar(valor: unknown) {
  valor.toUpperCase();          // ❌ error: primero comprueba
  if (typeof valor === "string") {
    valor.toUpperCase();        // ✅ aquí TS sabe que es string (narrowing)
  }
}
```

---

## 4.5 Uniones, intersecciones y narrowing

```ts
type Resultado = { ok: true; datos: string[] } | { ok: false; error: string };

function mostrar(r: Resultado) {
  if (r.ok) {
    console.log(r.datos);   // TS sabe que aquí existe "datos"
  } else {
    console.log(r.error);   // y aquí "error"
  }
}
```

Formas de **estrechar** (narrowing) un tipo: `typeof`, `instanceof`, `in`,
comparar con un literal (unión discriminada, como arriba), `Array.isArray`.

```ts
type ConFechas = { createdAt: Date };
type LibroConFechas = Libro & ConFechas;  // intersección: tiene TODO
```

---

## 4.6 Genéricos ⭐

Permiten escribir código **reutilizable** que funciona con cualquier tipo **sin perder
el tipado**.

```ts
// ❌ Con any perdemos la información
function primero(arr: any[]): any { return arr[0]; }

// ✅ Con un genérico, el tipo "viaja"
function primero<T>(arr: T[]): T | undefined {
  return arr[0];
}
primero([1, 2, 3]);      // number
primero(["a", "b"]);     // string
```

Ya los usas sin darte cuenta:

```ts
const libros: Array<Libro> = [];
async function getLibros(): Promise<Libro[]> { ... }
this.pool.query<BookRow>("SELECT ...");    // api-books-nest: tipa las filas devueltas
```

---

## 4.7 Utility types

| Utility | Qué hace | Ejemplo de uso |
|---------|----------|----------------|
| `Partial<T>` | Todo opcional | Datos de un `PATCH` |
| `Required<T>` | Todo obligatorio | — |
| `Readonly<T>` | Todo solo lectura | Configuración |
| `Pick<T, "a" \| "b">` | Solo esas propiedades | Vista pública de un usuario |
| `Omit<T, "a">` | Todas menos esas | `Omit<User, "password">` |
| `Record<K, V>` | Objeto con claves K y valores V | `Record<string, number>` |
| `ReturnType<typeof fn>` | Tipo que devuelve una función | — |

```ts
type CrearLibro = Omit<Libro, "id">;        // al crear aún no hay id
type ActualizarLibro = Partial<CrearLibro>; // en PATCH todo es opcional
```

> 📌 En NestJS se suele hacer lo mismo con `PartialType(CreateBookDto)` para el DTO de
> actualización.

---

## 4.8 Clases en TS

```ts
class BooksService {
  constructor(private readonly pool: Pool) {}  // crea y asigna this.pool automáticamente
}
```

| Modificador | Accesible desde |
|-------------|-----------------|
| `public` (por defecto) | Cualquier sitio |
| `private` | Solo la propia clase |
| `protected` | La clase y sus hijas |
| `readonly` | Se asigna una vez y no cambia |

**Ejemplo real (`api-books-nest`):** tu `BooksService` hace
`implements BooksContract`, es decir, la clase se compromete a cumplir una interfaz.

---

## 4.9 Otras preguntas rápidas

| Pregunta | Respuesta corta |
|----------|-----------------|
| ¿Qué es `tsconfig.json`? | Configuración del compilador (target, `strict`, rutas, etc.) |
| ¿Qué hace `strict: true`? | Activa las comprobaciones más estrictas (p. ej. `strictNullChecks`) |
| ¿Qué es un `enum`? | Conjunto de constantes con nombre. Muchos prefieren uniones de literales |
| ¿Type assertion (`as`)? | "Confía en mí, es de este tipo". Úsalo poco: puede mentir |
| ¿Qué es `!` (non-null assertion)? | "Esto no es null". Igual de peligroso que `as` |
| ¿`@types/...`? | Paquetes con los tipos de librerías escritas en JS (`@types/express`) |
| ¿`keyof` y `typeof`? | `keyof T` = unión de claves; `typeof x` = tipo de una variable |

---

## 🎯 Cómo responderlo en la entrevista

**¿Por qué usar TypeScript?**
> "Porque detecta errores de tipos mientras programo, antes de ejecutar, me da
> autocompletado y hace el código más fácil de mantener y refactorizar. En el bootcamp
> hice las APIs con TypeScript y en el CI comprobaba los tipos con `tsc --noEmit`."

**¿`interface` o `type`?**
> "Son muy parecidos para objetos. `type` además permite uniones y tipos primitivos;
> `interface` permite declaration merging y se extiende con `extends`. Suelo usar
> `interface` para contratos de objetos y `type` para uniones, como los estados de un
> anuncio: `'AVAILABLE' | 'RESERVED' | 'SOLD'`."

**¿`any` vs `unknown`?**
> "`any` desactiva el tipado; `unknown` también acepta cualquier cosa pero obliga a
> comprobar el tipo antes de usarlo, así que es la opción segura."

**¿Qué son los genéricos?**
> "Tipos como parámetro, para escribir funciones o clases reutilizables sin perder el
> tipado. Por ejemplo `Promise<Libro[]>` o una función `primero<T>(arr: T[]): T`."

**¿TypeScript valida los datos que llegan a la API?**
> "No, los tipos desaparecen al compilar. Por eso en NestJS valido el body con DTOs y
> `class-validator` y un `ValidationPipe` global."
