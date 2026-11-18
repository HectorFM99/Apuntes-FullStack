# 03 — UNIONES, TIPOS LITERALES, `type` E `interface`

> Aquí está el 80% del TypeScript que vas a escribir en el día a día: **describir la
> forma de tus datos** (usuarios, libros, anuncios, canciones...) y decir **qué
> valores concretos** se permiten.

---

## 3.1 Tipos unión `|` — "esto O aquello"

```ts
let id: number | string;

id = 123;      // ✅
id = "abc";    // ✅
id = true;     // ❌ Type 'boolean' is not assignable to type 'string | number'.
```

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

> 📌 Con una unión **solo puedes usar lo que tienen en común** los dos tipos. `id.toUpperCase()`
> fuera del `if` daría error, porque un `number` no tiene `toUpperCase`. Dentro del
> `if (typeof id === "string")` TS **ya sabe** que es `string`. Eso se llama
> **narrowing** (estrechamiento) y tiene su propio archivo: `06`.

### Uniones con `null` / `undefined` (las más comunes)

```ts
function buscarUsuario(identificador: number): string | undefined { ... }   // EJERCICIO 11
let current: Track | undefined;                                              // ProyectoReproductor/src/app.ts
```

---

## 3.2 Tipos literales — valores exactos

Un tipo literal es un **valor concreto** usado como tipo:

```ts
let direccion: "izquierda" = "izquierda";
direccion = "derecha";   // ❌ solo admite "izquierda"
```

Solo, no sirve de mucho. **Combinado con `|`** es buenísimo: una lista cerrada de
opciones válidas.

### Ejemplo REAL copiado (`ejercicios-typescript.ts`, EJERCICIO 8)

```ts
type estadoTarea = "pendiente" | "en progreso" | "completada";

// miTarea = "cancelada"; Type '"cancelada"' is not assignable to type 'estadoTarea'.
```

> 📌 Para que el error salga de verdad, la variable tiene que estar declarada con el tipo:
> ```ts
> let miTarea: estadoTarea = "pendiente";   // ✅
> miTarea = "cancelada";                    // ❌ Type '"cancelada"' is not assignable...
> ```

### Ejemplo REAL — estados de una API (`api-rastro/api-rastro/src/types.ts`)

```ts
export type ListingStatus = "AVAILABLE" | "RESERVED" | "SOLD";
export type OfferStatus = "PENDING" | "ACCEPTED" | "REJECTED";
```

Y se usan al crear datos (`api-rastro/api-rastro/src/routes/offers.ts`):

```ts
const newOffer: Offer = {
    amount,
    status: "PENDING",          // ✅ autocompletado con las 3 opciones
    // status: "PENDIENTE",     // ❌ no es uno de los valores permitidos
    createdAt: new Date(),
    listing: { id: listing._id, title: listing.title, price: listing.price },
    buyer: { id: buyer._id, name: buyer.name },
};
```

> 📌 Con un literal, una **errata** (`"AVAILABE"`) es un **error de compilación**, no
> un bug que encuentras a las 3 de la mañana en la base de datos.

### Ejemplo REAL — parámetro con literales (`ProyectoReproductor/src/login.ts` y `app.ts`)

```ts
function selectTab(name: 'login' | 'register') {
  // ...
}

$('#login-tab').addEventListener('click', () => selectTab('login'));
$('#register-tab').addEventListener('click', () => selectTab('register'));
```

```ts
async function library(kind: 'favorites' | 'history') {
  // ...
  const tracks = await request<Track[]>(`/api/library/${kind}`);
}
```

| Unión de literales | vs | `enum` |
|---|---|---|
| `type Estado = "A" \| "B"` | | `enum Estado { A = "A", B = "B" }` |
| Desaparece al compilar (0 bytes) | | Genera un objeto en el JS |
| Usas el texto directamente: `"A"` | | Usas `Estado.A` |
| ✅ Lo más usado hoy en día | | Lo verás en `05` |

---

## 3.3 Alias de tipo con `type`

`type` le pone **nombre** a un tipo para reutilizarlo:

```ts
type ID = number | string;
type Punto = { x: number; y: number };
```

### Ejemplo REAL copiado (`ejercicios-typescript.ts`, EJERCICIOS 6 y 7)

```ts
type Producto = {
    nombre: string;
    precio: number;
    descripcion?: string;
};

function mostrarProducto(producto: Producto): void {
    console.log(`Producto: ${producto.nombre}`);
    console.log(`Precio: ${producto.precio} €`);
    if (producto.descripcion) {
        console.log(`Descripción: ${producto.descripcion}`);
    }
}

mostrarProducto({nombre: "Teclado", precio: 35});
mostrarProducto({nombre: "Teclado", precio: 35, descripcion: "Teclado mecánico"});
```

> 📌 En el EJERCICIO 6 haces `console.log({nombre: "Camiseta", precio: 19.99})` — eso
> **no** usa el tipo `Producto` para nada. Para que TS compruebe el objeto hay que
> "atarlo" al tipo:
> ```ts
> const camiseta: Producto = { nombre: "Camiseta", precio: 19.99 };
> ```

### Ejemplo REAL — un tipo de una sola línea (`ProyectoReproductor/src/app.ts`)

```ts
type Track = { id:string; name:string; artist:string; album?:string; image?:string; audio:string; duration?:number };
```

Mismo tipo, formateado para leerlo mejor:

```ts
type Track = {
  id: string;
  name: string;
  artist: string;
  album?: string;      // opcional
  image?: string;      // opcional
  audio: string;
  duration?: number;   // opcional
};
```

---

## 3.4 Interfaces con `interface`

`interface` también describe **la forma de un objeto**:

```ts
interface Persona {
  nombre: string;
  edad: number;
}
```

### Ejemplo REAL copiado (`ejercicios-typescript.ts`, EJERCICIO 9)

```ts
interface Persona {
    nombre: string;
    edad: number;
    ciudad?: string;
}

console.log({nombre: "Hector", edad: 27, ciudad: "Valencia" });
console.log({nombre: "Hector", edad: 27});
```

### Ejemplo REAL — interfaces de una API (`puta/api-books-nest/src/books/book.ts`)

```ts
// Así llega un libro desde PostgreSQL (una fila de la tabla)
export interface BookRow {
  id: number;
  title: string;
  author: string;
  pages: number;
  created_at: Date; // en SQL, con guion bajo
}

// Así lo devuelve nuestra API
export interface Book {
  id: number;
  title: string;
  author: string;
  pages: number;
  createdAt: Date; // en JavaScript, en camelCase
}

// Convierte una fila de la base de datos en un libro de la API
export function toBook(row: BookRow): Book {
  return {
    id: row.id,
    title: row.title,
    author: row.author,
    pages: row.pages,
    createdAt: row.created_at,
  };
}
```

> 📌 **Patrón muy profesional:** dos interfaces para el mismo dato, una con la forma
> de la **base de datos** (`created_at`) y otra con la forma de la **API**
> (`createdAt`), y una función que traduce. Si te equivocas en un nombre, TS lo marca.

### Interfaces con métodos (contratos)

Una interfaz puede exigir **métodos**, no solo datos.

**Ejemplo REAL (`practica-typescript/index.ts`):**

```ts
// 1. Definimos la interfaz: todo lo "Notificable" DEBE poder enviar un mensaje
interface Notificable {
  enviar(mensaje: string): void;
}
```

**Ejemplo REAL (`puta/api-books-nest/src/books/books.contract.ts`):**

```ts
// El contrato: qué tiene que saber hacer un servicio de libros
export interface BooksContract {
  findAll(author?: string): Promise<Book[]>;
  findOne(id: number): Promise<Book>;
  create(dto: CreateBookDto): Promise<Book>;
  update(id: number, dto: UpdateBookDto): Promise<Book>;
  remove(id: number): Promise<Book>;
}
```

> 📌 Esto se completa con `class X implements Interfaz` (ver `05`): la clase está
> **obligada** a tener todos esos métodos con esas firmas.

---

## 3.5 Herencia de interfaces: `extends`

### Ejemplo REAL copiado (`practica-typescript/precio.ts`, ejercicio 5)

```ts
interface identificable {
    id: number;
}

interface Tarea extends identificable{
    titulo: string;
    estado: EstadoTarea;
}
```

`Tarea` tiene **todo** lo de `identificable` + lo suyo:

```ts
// Tarea equivale a:
{
  id: number;
  titulo: string;
  estado: EstadoTarea;
}
```

> 📌 Convención: los nombres de tipos e interfaces van en **PascalCase**
> (`Identificable`, `Libro`, `EstadoTarea`). En tu `precio.ts` hay `identificable` y
> `type libro` en minúscula: funciona, pero cuesta distinguir un tipo de una variable
> (`libro: libro` 😵).

### El equivalente con `type`: intersección `&`

```ts
type Identificable = { id: number };

type Tarea = Identificable & {
  titulo: string;
  estado: EstadoTarea;
};
```

---

## 3.6 `type` vs `interface` — ¿cuál uso?

| | `type` | `interface` |
|---|--------|-------------|
| Objetos | ✅ | ✅ |
| Uniones (`"A" \| "B"`) | ✅ | ❌ No puede |
| Primitivos / tuplas (`type ID = string`) | ✅ | ❌ |
| Heredar | `&` (intersección) | `extends` |
| `class X implements ...` | ✅ (si es un objeto) | ✅ |
| Declararla dos veces se **fusiona** | ❌ Error | ✅ (declaration merging) |

**Regla práctica (y la que siguen tus proyectos):**

- **`type`** para uniones, literales y "datos" (`api-rastro/src/types.ts` usa solo `type`).
- **`interface`** para **contratos** que implementan clases (`Notificable`,
  `BooksContract`) y modelos de objetos (`Book`, `BookRow`).
- Lo importante es **ser consistente** dentro de un proyecto.

```ts
// Esto SOLO se puede con type:
type ListingStatus = "AVAILABLE" | "RESERVED" | "SOLD";

// ❌ No existe:
// interface ListingStatus = "AVAILABLE" | "RESERVED";
```

---

## 3.7 Propiedades opcionales `?`

`?` = la propiedad **puede no estar**. Su tipo pasa a ser `T | undefined`.

```ts
type Producto = {
    nombre: string;
    precio: number;
    descripcion?: string;   // string | undefined
};
```

```ts
const p1: Producto = { nombre: "Teclado", precio: 35 };                           // ✅
const p2: Producto = { nombre: "Teclado", precio: 35, descripcion: "Mecánico" };  // ✅
const p3: Producto = { nombre: "Teclado" };                                       // ❌ falta precio
```

### Antes de usar una opcional, compruébala

```ts
// ❌
console.log(producto.descripcion.toUpperCase());
// 'producto.descripcion' is possibly 'undefined'.

// ✅ (así lo haces en tu EJERCICIO 7)
if (producto.descripcion) {
    console.log(`Descripción: ${producto.descripcion}`);
}

// ✅ Alternativa con ?. y ??
console.log(producto.descripcion?.toUpperCase() ?? "Sin descripción");
```

### Ejemplo REAL — valores por defecto para opcionales (`ProyectoReproductor/src/app.ts`)

```ts
row.innerHTML = `<img class="track-art" src="${track.image || ''}" ...>
  ... ${escape(track.album || 'Sin álbum')} ...`;
```

> 📌 `track.image` es `string | undefined` (porque `image?:`). `|| ''` garantiza que
> siempre haya un `string`.

### Parámetros opcionales (mismo `?`)

```ts
// EJERCICIO 3 (real)
function presentar(nombre: string, apellido?:string): string {
    if (apellido) {
        return `Hola, me llamo ${nombre} ${apellido}.`;
    } else {
        return `Hola, me llamo ${nombre}.`;
    }
}
```

Más en `04`.

---

## 3.8 `readonly` — no se puede reasignar

```ts
type Config = {
  readonly apiUrl: string;
  timeout: number;
};

const config: Config = { apiUrl: "https://api.com", timeout: 5000 };
config.timeout = 3000;          // ✅
config.apiUrl = "otra";         // ❌ Cannot assign to 'apiUrl' because it is a read-only property.
```

### Ejemplo REAL — `readonly` en el constructor (`puta/api-books-nest/src/books/books.service.ts`)

```ts
@Injectable()
export class BooksService implements BooksContract  {
  constructor(private readonly pool: Pool) {}
```

> 📌 `private readonly pool` = "guárdame el `pool` como propiedad, que solo use la
> clase (`private`) y que nadie lo reemplace (`readonly`)". En Nest **todas** las
> dependencias inyectadas se declaran así.

### Arrays de solo lectura

```ts
const colores: readonly string[] = ["rojo", "verde"];
colores.push("azul");    // ❌ Property 'push' does not exist on type 'readonly string[]'.
```

> 📌 `readonly` es **solo en compilación**. En el `.js` final se puede modificar.
> Para todo un objeto de golpe: `Readonly<Usuario>` (ver `05`).

---

## 3.9 Objetos anidados y `Record`

### Ejemplo REAL completo (`api-rastro/api-rastro/src/types.ts`)

```ts
import type { ObjectId } from "mongodb";

export type ListingStatus = "AVAILABLE" | "RESERVED" | "SOLD";
export type OfferStatus = "PENDING" | "ACCEPTED" | "REJECTED";

export type User = {
  name: string;
  email: string;
  city: string;
  joinedAt: Date;
  favorites: ObjectId[];
  ratingAverage: number;
  ratingCount: number;
};

export type Listing = {
  title: string;
  description: string;
  price: number;
  category: string;
  status: ListingStatus;
  city: string;
  publishedAt: Date;
  views: number;
  seller: { id: ObjectId; name: string };
  photos: { url: string; order: number }[];
  attributes: Record<string, unknown>;
};

export type Offer = {
  amount: number;
  status: OfferStatus;
  createdAt: Date;
  listing: { id: ObjectId; title: string; price: number };
  buyer: { id: ObjectId; name: string };
};
```

**Qué tipo tiene cada cosa:**

| Propiedad | Tipo | Significa |
|-----------|------|-----------|
| `joinedAt` | `Date` | Un objeto fecha de JS |
| `favorites` | `ObjectId[]` | Array de ids de MongoDB (tipo que viene de la librería) |
| `status` | `ListingStatus` | Uno de los 3 textos permitidos |
| `seller` | `{ id; name }` | Objeto **anidado** (copia "desnormalizada" del vendedor) |
| `photos` | `{ url; order }[]` | Array de objetos |
| `attributes` | `Record<string, unknown>` | Objeto con **claves libres** y valores a comprobar |

### `Record<Clave, Valor>` — diccionarios

```ts
Record<string, number>   // { [cualquier texto]: number }
```

**Ejemplo REAL (`ejercicios-typescript.ts`, EJERCICIO 20):**

```ts
type Puntuaciones = Record<string, number>;

const puntuaciones: Puntuaciones = {
    Ana: 85,
    Luis: 92,
    Marta: 78,
};
```

**Ejemplo REAL (`ProyectoReproductor/src/app.ts`):**

```ts
const metadata = {
    discover: ['Escucha recomendada', 'Canciones destacadas del catálogo independiente con sus carátulas, listas para reproducir o guardar.'],
    favorites: ['Tus favoritos', 'Canciones, álbumes y artistas que decidiste guardar.'],
    history: ['Tu historial', 'Las últimas canciones que escuchaste en Nightjar.']
} as Record<string, string[]>;
```

> 📌 Con claves **literales** el `Record` es aún más útil, porque obliga a que estén todas:
> ```ts
> type Vista = "discover" | "favorites" | "history";
> const titulos: Record<Vista, string> = {
>   discover: "Escucha recomendada",
>   favorites: "Tus favoritos",
>   // ❌ Property 'history' is missing...
> };
> ```

---

## 3.10 Errores comunes

| Error | Causa | Solución |
|-------|-------|----------|
| `Property 'x' is missing in type...` | Falta una propiedad obligatoria | Añádela o márcala `x?:` |
| `Object literal may only specify known properties` | Sobra una propiedad (o hay una errata) | Revisa el nombre |
| `Type '"abc"' is not assignable to type '"A" \| "B"'` | Valor fuera de la unión de literales | Usa uno de los permitidos |
| `'x' is possibly 'undefined'` | Usas una opcional sin comprobar | `if (x)`, `x?.`, `x ?? valor` |
| `Cannot assign to 'x' because it is a read-only property` | Intentas cambiar algo `readonly` | Crea un objeto nuevo `{ ...obj, x: nuevo }` |

```ts
// ❌ Errata detectada por TS
const p: Producto = { nombre: "Ratón", precio: 10, descripcon: "Inalámbrico" };
//                                                ~~~~~~~~~~ Did you mean 'descripcion'?
```

---

## ✅ Resumen rápido

1. `A | B` = unión: "puede ser A **o** B". Solo puedes usar lo común hasta que compruebes cuál es.
2. Literales `"pendiente" | "completada"` = lista cerrada de valores.
3. `type` → nombres para cualquier tipo (uniones incluidas).
4. `interface` → forma de objetos y **contratos** para clases; se hereda con `extends`.
5. `?` = opcional (`T | undefined`). Compruébalo antes de usarlo.
6. `readonly` = no reasignable. `private readonly` en constructores de Nest.
7. `Record<K, V>` = diccionario.

👉 **Siguiente:** `04-funciones-tipadas-y-genericos.md`
