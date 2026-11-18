# 05 — ENUMS, CLASES E INTERFACES, Y UTILITY TYPES

> Tres herramientas que TypeScript añade sobre JavaScript:
> - **`enum`**: un conjunto de constantes con nombre.
> - **Clases tipadas**: `private`, `readonly`, `implements`...
> - **Utility types**: tipos "fabricados" a partir de otros (`Partial`, `Omit`, `Pick`...).

---

## 5.1 `enum` — constantes con nombre

```ts
enum Direccion {
  Arriba,
  Abajo,
  Izquierda,
  Derecha,
}

let mover: Direccion = Direccion.Arriba;
```

### Enum numérico — Ejemplo REAL (`ejercicios-typescript.ts`, EJERCICIO 18)

```ts
enum EstadoPedido {
    Pendiente,
    Enviado,
    Entregado,
}

function mensajeEstado(estado: EstadoPedido): string {
    switch (estado) {
        case EstadoPedido.Pendiente:
            return "El pedido está pendiente de preparación.";
        case EstadoPedido.Enviado:
            return "El pedido ha sido enviado.";
        case EstadoPedido.Entregado:
            return "El pedido ha sido entregado.";
        default:
            return "Estado desconocido.";
    }
}

console.log(mensajeEstado(EstadoPedido.Pendiente));
console.log(mensajeEstado(EstadoPedido.Enviado));
console.log(mensajeEstado(EstadoPedido.Entregado));
```

**Por debajo, cada miembro es un número:**

```
EstadoPedido.Pendiente  → 0
EstadoPedido.Enviado    → 1
EstadoPedido.Entregado  → 2
```

> ⚠️ Problema de los enums numéricos: si guardas el estado en una base de datos o lo
> imprimes, ves `0`, `1`, `2`... **ilegible**. Y si añades un estado en medio, los
> números se desplazan.

### Enum de texto — Ejemplo REAL (`practica-typescript/precio.ts`)

```ts
//4
enum EstadoTarea2 {
    Pendiente = "pendiente",
    EnProceso = "en-proceso",
    Completada = "completada"
}

function mostrarEstado(estado : EstadoTarea2): void {
    switch (estado) {
        case EstadoTarea2.Pendiente:
            console.log("La tarea está pendiente.");
            break;
        case EstadoTarea2.EnProceso:
            console.log("La tarea está en proceso.");
            break;
        case EstadoTarea2.Completada:
            console.log("La tarea está completada.");
            break;
    }
}
 mostrarEstado(EstadoTarea2.Pendiente);
 mostrarEstado(EstadoTarea2.EnProceso);
 mostrarEstado(EstadoTarea2.Completada);
```

> 📌 Con valores de texto, `console.log(EstadoTarea2.EnProceso)` imprime
> `"en-proceso"`: se entiende y es estable. **Si usas enum, mejor de texto.**

### ¿Qué genera un enum al compilar?

A diferencia de `type` e `interface`, un `enum` **sí existe** en el JavaScript final:

```js
// Aproximadamente lo que genera tsc para EstadoTarea2
var EstadoTarea2;
(function (EstadoTarea2) {
    EstadoTarea2["Pendiente"] = "pendiente";
    EstadoTarea2["EnProceso"] = "en-proceso";
    EstadoTarea2["Completada"] = "completada";
})(EstadoTarea2 || (EstadoTarea2 = {}));
```

### `enum` vs unión de literales

```ts
// Con enum (practica-typescript)
enum EstadoTarea { Pendiente = "pendiente", EnProceso = "en-proceso", Completada = "completada" }
mostrarEstado(EstadoTarea.Pendiente);

// Con unión de literales (ejercicios Bikram, EJERCICIO 8 / api-rastro)
type estadoTarea = "pendiente" | "en progreso" | "completada";
mostrar("pendiente");
```

| | `enum` | Unión de literales |
|---|--------|-------------------|
| Código JS generado | Sí (un objeto) | No (0 bytes) |
| Se escribe | `Estado.Pendiente` | `"pendiente"` |
| Recorrer los valores | `Object.values(Estado)` | Hace falta un array aparte |
| Uso en APIs modernas | Menos | ✅ Más (es lo que usa `api-rastro`) |

> 📌 Ambos son correctos. En tu bootcamp verás los dos; en Express/Mongo suele
> preferirse la unión de literales.

---

## 5.2 Clases tipadas: propiedades y constructor

En TS las **propiedades** de una clase se declaran con su tipo **antes** de usarlas.

### Ejemplo REAL copiado (`ejercicios-typescript.ts`, EJERCICIO 17)

```ts
class Cancion implements Reproducible {
    titulo: string;                 // ← declaración de la propiedad

    constructor(titulo: string) {
        this.titulo = titulo;       // ← asignación
    }

    reproducir(): void {
        console.log(`♪ Reproduciendo canción: ${this.titulo}`);
    }
}
```

```ts
// ❌ En JS se hacía sin declarar; en TS:
class Cancion {
  constructor(titulo: string) {
    this.titulo = titulo;   // Property 'titulo' does not exist on type 'Cancion'.
  }
}
```

---

## 5.3 Modificadores de acceso: `public`, `private`, `protected`, `readonly`

| Modificador | ¿Quién puede usarlo? |
|-------------|---------------------|
| `public` (por defecto) | Todo el mundo |
| `private` | Solo la propia clase |
| `protected` | La clase y sus hijas (`extends`) |
| `readonly` | Se asigna una vez (declaración o constructor) y no se cambia |

### Ejemplo REAL copiado (`ejercicios-typescript.ts`, EJERCICIO 16)

```ts
class CuentaBancaria {
    private saldo: number;

    constructor(saldoInicial: number) {
        this.saldo = saldoInicial;
    }

    ingresar(cantidad: number): void {
        if (cantidad > 0) {
            this.saldo += cantidad;
        }
    }

    consultarSaldo(): number {
        return this.saldo;
    }
}

const cuenta = new CuentaBancaria(100);
cuenta.ingresar(50);
cuenta.ingresar(-20);
console.log(cuenta.consultarSaldo());   // 150
```

```ts
cuenta.saldo = 1000000;
// ❌ Property 'saldo' is private and only accessible within class 'CuentaBancaria'.
```

> 📌 Esto es **encapsulación**: nadie de fuera puede tocar el saldo directamente;
> solo a través de `ingresar()`, que valida que la cantidad sea positiva (por eso
> `ingresar(-20)` no hace nada).
>
> ⚠️ `private` de TS solo protege **al compilar**. Si quieres privacidad real en
> ejecución, JS moderno tiene `#saldo`.

---

## 5.4 Atajo: propiedades en el constructor (*parameter properties*)

Poner un modificador (`private`, `public`, `readonly`) **delante de un parámetro del
constructor** declara y asigna la propiedad **de un golpe**.

### Ejemplo REAL copiado (`practica-typescript/index.ts`)

```ts
class NotificacionEmail implements Notificable {
  constructor(private destinatario: string) {}

  enviar(mensaje: string): void {
    console.log(`📧 Enviando email a ${this.destinatario}: "${mensaje}"`);
  }
}
```

**Es exactamente lo mismo que:**

```ts
class NotificacionEmail implements Notificable {
  private destinatario: string;

  constructor(destinatario: string) {
    this.destinatario = destinatario;
  }
  // ...
}
```

### Ejemplo REAL — el patrón de NestJS (`puta/api-books-nest/src/books/books.controller.ts`)

```ts
@Controller('books') // todas las rutas de esta clase: /api/books...
export class BooksController {
  constructor(private readonly booksService: BooksService) {} // pide el servicio
```

> 📌 En Nest verás este atajo **en todas partes**: el constructor pide sus
> dependencias y el `private readonly` las guarda como propiedades.

---

## 5.5 `implements` — una clase cumple un contrato

`implements Interfaz` **obliga** a la clase a tener todo lo que pide la interfaz.

### Ejemplo REAL copiado completo (`practica-typescript/index.ts`)

```ts
// 1. Definimos la interfaz: todo lo "Notificable" DEBE poder enviar un mensaje
interface Notificable {
  enviar(mensaje: string): void;
}

// 2. Creamos clases que "implementan" esa interfaz (cada una notifica a su manera)
class NotificacionEmail implements Notificable {
  constructor(private destinatario: string) {}

  enviar(mensaje: string): void {
    console.log(`📧 Enviando email a ${this.destinatario}: "${mensaje}"`);
  }
}

class NotificacionSMS implements Notificable {
  constructor(private telefono: string) {}

  enviar(mensaje: string): void {
    console.log(`📱 Enviando SMS al ${this.telefono}: "${mensaje}"`);
  }
}

class NotificacionPush implements Notificable {
  constructor(private dispositivo: string) {}

  enviar(mensaje: string): void {
    console.log(`🔔 Enviando push al dispositivo ${this.dispositivo}: "${mensaje}"`);
  }
}

// 3. Creamos el array de elementos Notificable (pueden ser de clases distintas)
const notificaciones: Notificable[] = [
  new NotificacionEmail("ana@correo.com"),
  new NotificacionSMS("+54 9 11 5555-5555"),
  new NotificacionPush("iPhone de Ana"),
];

// 4. Recorremos el array llamando a enviar() en cada uno
for (const notificacion of notificaciones) {
  notificacion.enviar("Tu pedido está listo");
}
```

**Salida:**

```
📧 Enviando email a ana@correo.com: "Tu pedido está listo"
📱 Enviando SMS al +54 9 11 5555-5555: "Tu pedido está listo"
🔔 Enviando push al dispositivo iPhone de Ana: "Tu pedido está listo"
```

### 🧠 Por qué esto es importante (polimorfismo)

```ts
const notificaciones: Notificable[] = [ ... ];
```

El array **no sabe** si cada elemento es email, SMS o push. Solo sabe que **todos
tienen `enviar()`**. Puedes añadir `NotificacionWhatsApp` mañana sin tocar el bucle.
Es la misma idea que el EJERCICIO 17 (`Reproducible` con `Cancion` y `Video`).

### Si olvidas un método

```ts
class NotificacionFax implements Notificable {
  // sin enviar()
}
// ❌ Class 'NotificacionFax' incorrectly implements interface 'Notificable'.
//    Property 'enviar' is missing in type 'NotificacionFax' but required in type 'Notificable'.
```

### Ejemplo REAL — servicio que implementa un contrato (`puta/api-books-nest/src/books/books.service.ts`)

```ts
@Injectable()
export class BooksService implements BooksContract  {
  constructor(private readonly pool: Pool) {}

  async findAll(author?: string, maxPages?: number): Promise<Book[]> { /* ... */ }
  async count(): Promise<{ total: number }> { /* ... */ }
  async findOne(id: number): Promise<Book> { /* ... */ }
  async create(dto: CreateBookDto): Promise<Book> { /* ... */ }
  async update(id: number, dto: UpdateBookDto): Promise<Book> { /* ... */ }
  async remove(id: number): Promise<Book> { /* ... */ }
}
```

> 📌 La clase puede tener **más** métodos que el contrato (`count()` no está en
> `BooksContract`), pero **nunca menos**. Y `findAll` añade un parámetro opcional
> `maxPages?` → sigue siendo compatible porque es opcional.

---

## 5.6 Utility types — tipos fabricados a partir de otros

TypeScript trae "funciones de tipos" que transforman un tipo existente. Así **no
repites** la definición.

Partimos de este tipo **real** (`ejercicios-typescript.ts`, EJERCICIO 13):

```ts
type Usuario = {
    id: number;
    nombre: string;
    email: string;
    edad: number;
};
```

| Utility | Qué hace | Resultado sobre `Usuario` |
|---------|----------|---------------------------|
| `Partial<T>` | **Todo opcional** | `{ id?; nombre?; email?; edad? }` |
| `Required<T>` | **Todo obligatorio** (quita los `?`) | — |
| `Readonly<T>` | **Todo de solo lectura** | `{ readonly id; readonly nombre; ... }` |
| `Pick<T, K>` | **Solo** las propiedades `K` | `Pick<Usuario, "id" \| "nombre">` → `{ id; nombre }` |
| `Omit<T, K>` | **Todas menos** `K` | `Omit<Usuario, "email">` → `{ id; nombre; edad }` |
| `Record<K, V>` | Objeto con claves `K` y valores `V` | `Record<string, number>` |
| `ReturnType<F>` | Tipo que devuelve una función | `ReturnType<typeof toBook>` → `Book` |

### `Partial` y `Pick` — Ejemplo REAL (EJERCICIO 13)

```ts
const cambios: Partial<Usuario> = { email: "ana@correo.com" };

const resumen: Pick<Usuario, "id" | "nombre"> = { id: 1, nombre: "Ana" };
```

> 📌 `Partial` es **el tipo perfecto para un PATCH**: el cliente manda solo lo que
> cambia.

### `Omit` — Ejemplo REAL (EJERCICIO 14)

```ts
type UsuarioPublico = Omit<Usuario, "email">;

const usuarioPublico: UsuarioPublico = {
    id: 1,
    nombre: "Ana",
    edad: 28,
};
```

> 📌 Caso típico: el usuario de la base de datos tiene `passwordHash`, y el que
> devuelves en la API es `Omit<Usuario, "passwordHash">`.

### `Required` y `Readonly` — Ejemplo REAL (EJERCICIO 15)

```ts
interface Configuracion {
    tema?: string;
    idioma?: string;
}

const configuracionCompleta: Required<Configuracion> = {
    tema: "oscuro",
    idioma: "es",
};

const usuarioSoloLectura: Readonly<Usuario> = { id: 1, nombre: "Ana", email: "ana@correo.com", edad: 28 };

// usuarioSoloLectura.nombre = "Luis";  // Error: Cannot assign to 'nombre' because it is a read-only property.
```

### `Record` — Ejemplo REAL (EJERCICIO 20)

```ts
type Puntuaciones = Record<string, number>;

const puntuaciones: Puntuaciones = {
    Ana: 85,
    Luis: 92,
    Marta: 78,
};
```

### Combinarlos

```ts
// "Para crear un usuario manda todo menos el id"
type CrearUsuario = Omit<Usuario, "id">;

// "Para actualizarlo manda lo que quieras, menos el id"
type ActualizarUsuario = Partial<Omit<Usuario, "id">>;
```

### 🧠 Comparación con tus DTOs de Nest

En `puta/api-books-nest` el DTO de actualización se escribió **a mano**, repitiendo
los campos con `?` (copiado tal cual de `dto/update-book.dto.ts`):

```ts
// PATCH: los campos pueden faltar, pero si llegan tienen que ser válidos
export class UpdateBookDto {
  @ValidateIf((_, value) => value !== undefined) // valida solo si ha llegado
  @IsString()
  @MinLength(1)
  @MaxLength(200)
  title?: string;

  @ValidateIf((_, value) => value !== undefined)
  @IsString()
  @MinLength(1)
  @MaxLength(100)
  author?: string;

  @ValidateIf((_, value) => value !== undefined)
  @IsInt()
  @Min(1)
  pages?: number;
}
```

> 📌 A nivel de **tipos** es `Partial<CreateBookDto>`. Pero un DTO de Nest es una
> **clase con decoradores** que se ejecutan al validar, y los utility types de TS
> **desaparecen** al compilar. Por eso Nest tiene su propia versión que sí copia
> los decoradores: `PartialType(CreateBookDto)` de `@nestjs/mapped-types`. Más en la
> carpeta `NESTJS`.

---

## 5.7 `as` en un objeto de retorno — cuidado

### Ejemplo REAL (`practica-typescript/precio.ts`, ejercicio 5)

```ts
function crearTarea(titulo: string, estado: EstadoTarea, id: number): Tarea {
    return {
        titulo,
        estado,
        id: Date.now() // Asignar un ID único
    } as Tarea;
}
```

Dos cosas a mejorar:

```ts
// 1. El parámetro `id` se recibe pero NO se usa (se usa Date.now()).
// 2. El `as Tarea` sobra: el objeto ya cumple Tarea y el `: Tarea` del retorno lo comprueba.
//    Peor aún: `as` puede TAPAR errores (si faltara `titulo`, el `as` lo dejaría pasar).

// ✅ Versión limpia
function crearTarea(titulo: string, estado: EstadoTarea, id: number): Tarea {
    return { titulo, estado, id };
}
```

> 📌 Regla: **no uses `as` si no hace falta**. Más sobre `as` en `06`.

---

## ✅ Resumen rápido

1. `enum` crea constantes con nombre; mejor de **texto** (`= "pendiente"`) que numérico.
2. Unión de literales es la alternativa ligera al enum (no genera código).
3. Clases: declara las propiedades con su tipo; `private` / `readonly` / `protected`.
4. `constructor(private x: string) {}` declara y asigna en un paso (patrón Nest).
5. `implements` obliga a cumplir una interfaz → polimorfismo (`Notificable[]`).
6. Utility types: `Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record`.
7. No abuses de `as`: tapa errores.

👉 **Siguiente:** `06-narrowing-any-unknown-y-aserciones.md`
