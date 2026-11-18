# 08 — EJEMPLOS COMPLETOS COPIADOS DE TUS PROYECTOS

> 📁 **Los archivos de los proyectos NO se han modificado.** Todo lo de aquí abajo son
> **copias** íntegras de su contenido original, con notas para estudiar.
> Los archivos `.env` **no** se copian (contienen secretos).

---

## EJEMPLO 1 — Interfaces y clases (el más corto)

**Origen:** `practica-typescript/index.ts` *(copiado íntegro)*

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

**Qué enseña:**
- `interface` con un **método** → un contrato (`03`, `05`).
- `implements` → cada clase **está obligada** a tener `enviar(mensaje: string): void`.
- `constructor(private x: string) {}` → declara y asigna la propiedad en un paso (`05`).
- `Notificable[]` → array de objetos de clases **distintas** que cumplen lo mismo (polimorfismo).

**Cómo ejecutarlo:**

```bash
cd practica-typescript
npm run build     # tsc → genera index.js, index.d.ts y los .map
npm start         # node index.js
```

---

## EJEMPLO 2 — Funciones, `type`, enums y `extends`

**Origen:** `practica-typescript/precio.ts` *(copiado íntegro)*

```ts
//1
function calcularDescuento(precio: number, porcentaje: number): number {
const descuento = precio * (porcentaje / 100);
return precio - descuento;
}
console.log(calcularDescuento(100, 20));

//2
type libro = {
    titulo: string | null;
    autor: string | null;
    pagina: number | null;
    disponible: boolean | null;
}
const libro1: libro = {
    titulo: "El Principito",
    autor: "Antoine de Saint-Exupéry",
    pagina: 96,
    disponible: true
};

const libro2: libro = {
    titulo: "1984",
    autor: "George Orwell",
    pagina: 328,
    disponible: false
};

function mostrarLibro(libro: libro): void {
  console.log(`${libro.titulo} - ${libro.autor}`);
}

mostrarLibro(libro1);
mostrarLibro(libro2);

//3
interface Notificable {enviar(mensaje: string): void;}

class NotficacionMail implements Notificable {
    enviar(mensaje: string): void {
        console.log(`Enviando correo con el mensaje: ${mensaje}`);
    }}

class NotificacionSMS implements Notificable {
    enviar(mensaje: string): void {
        console.log(`Enviando SMS por con el mensaje: ${mensaje}`);
    }}

const notificaciones: Notificable[] = 
[new NotficacionMail(), 
new NotificacionSMS(),
];

notificaciones.forEach((notificacion) => {
    notificacion.enviar("Tu pedido está listo.");
});

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

//5

interface identificable {
    id: number;
}

interface Tarea extends identificable{
    titulo: string;
    estado: EstadoTarea;
} 

enum EstadoTarea {
    Pendiente = "pendiente",
    EnProceso = "en-proceso",
    Completada = "completada"
}

function crearTarea(titulo: string, estado: EstadoTarea, id: number): Tarea {
    return {
        titulo,
        estado,
        id: Date.now() // Asignar un ID único
    } as Tarea;
}

function mostrarTarea(tarea: Tarea): void {
    console.log(`Título: ${tarea.titulo}`);
    console.log(`ID: ${tarea.id}`);
    console.log(`Estado: ${tarea.estado}`);
}
const todasLasTareas: Tarea[] = [
    crearTarea("Aprender TypeScript", EstadoTarea.Pendiente, 1),
    crearTarea("Practicar TypeScript", EstadoTarea.EnProceso, 2),
    crearTarea("Crear un proyecto con TypeScript", EstadoTarea.Completada, 3)
];
 todasLasTareas.forEach((tarea) => mostrarTarea(tarea));

```

**Qué enseña, ejercicio a ejercicio:**

| # | Concepto | Archivo de teoría |
|---|----------|-------------------|
| 1 | Función con parámetros y retorno `number` | `04` |
| 2 | `type` de objeto con uniones `\| null`, función `void` | `02`, `03` |
| 3 | `interface` + `implements` + `forEach` sobre `Notificable[]` | `05` |
| 4 | `enum` de texto + `switch` | `05` |
| 5 | `interface ... extends`, `enum`, función que devuelve `Tarea`, `Tarea[]` | `03`, `05` |

**Cosas a mejorar (buen ejercicio de repaso):**

```ts
// 1. Nombres de tipos en PascalCase
type libro = {...}             →  type Libro = {...}
interface identificable {...}  →  interface Identificable {...}

// 2. crearTarea ignora el parámetro `id` y usa `as` sin necesidad
function crearTarea(titulo: string, estado: EstadoTarea, id: number): Tarea {
    return { titulo, estado, id };
}

// 3. EstadoTarea2 y EstadoTarea son idénticos → sobra uno
```

> ⚠️ Recuerda (`01`): el `precio.js` que hay en la carpeta es de una **versión
> anterior** de este archivo. Ejecuta `npm run build` para regenerarlo.

---

## EJEMPLO 3 — Los 20 ejercicios de TypeScript (Bikram)

**Origen:** `ejercicios-Bikram-Typescript/ejercicios-Bikram-Typescripts/ejercicios-typescript.ts` *(copiado íntegro)*

```ts

// EJERCICIO 1 
let nombre: string = "Hector";
let edad: number = 27;
let activo: boolean = true;

//edad = "ventisiete"; Error: Type 'string' is not assignable to type 'number'

// ---------------------------------------------------------------------------
// EJERCICIO 2 
let precios: number[] = [12.5, 30, 8.99, 45];

function sumarPrecios(arr: number[]): number {
    let suma = 0;
    for (let i = 0; i < arr.length; i++) {
        suma += arr[i];
    }
    return suma;
}

console.log(sumarPrecios(precios));


// ---------------------------------------------------------------------------
// EJERCICIO 3

function presentar(nombre: string, apellido?:string): string {
    if (apellido) {
        return `Hola, me llamo ${nombre} ${apellido}.`;
    } else {
        return `Hola, me llamo ${nombre}.`;
    }
}

console.log(presentar("Ana", "Perez"));
console.log(presentar("Ana"));

// ---------------------------------------------------------------------------
// EJERCICIO 4 

function calcularDescuento(precio: number, porcentaje: number): number {
    return precio - (precio * porcentaje / 100);
}

console.log(calcularDescuento(100, 20));


// ---------------------------------------------------------------------------
// EJERCICIO 5

function mostrarID(id: number | string): void {
    if (typeof id === "string") {
        console.log(id.toUpperCase());
    }else{
        console.log(id.toFixed(2));
    }
}

console.log(mostrarID(123));
console.log(mostrarID("abc"));


// ---------------------------------------------------------------------------
// EJERCICIO 6

type Producto = {
    nombre: string;
    precio: number;
    descripcion?: string;
};

console.log({nombre: "Camiseta", precio: 19.99});
console.log({nombre: "Camiseta", precio: 19.99, descripcion: "Algodón 100%"});


// ---------------------------------------------------------------------------
// EJERCICIO 7

function mostrarProducto(producto: Producto): void {
    console.log(`Producto: ${producto.nombre}`);
    console.log(`Precio: ${producto.precio} €`);
    if (producto.descripcion) {
        console.log(`Descripción: ${producto.descripcion}`);
    }
}

mostrarProducto({nombre: "Teclado", precio: 35});
mostrarProducto({nombre: "Teclado", precio: 35, descripcion: "Teclado mecánico"});


// ---------------------------------------------------------------------------
// EJERCICIO 8
type estadoTarea = "pendiente" | "en progreso" | "completada";

// miTarea = "cancelada"; Type '"cancelada"' is not assignable to type 'estadoTarea'.


// ---------------------------------------------------------------------------
// EJERCICIO 9 
interface Persona {
    nombre: string;
    edad: number;
    ciudad?: string;
}

console.log({nombre: "Hector", edad: 27, ciudad: "Valencia" });
console.log({nombre: "Hector", edad: 27});
// ---------------------------------------------------------------------------
// EJERCICIO 10 
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

function estaDisponible(libro: Libro): boolean {
    return libro.Disponible;
}

for (const libro of libros) {
    if (estaDisponible(libro)) {
        console.log(`Disponible: ${libro.Titulo}`);
    }
}


// ---------------------------------------------------------------------------
// EJERCICIO 11 — Datos que pueden faltar
// ---------------------------------------------------------------------------
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

console.log(buscarUsuario(2));


// ---------------------------------------------------------------------------
// EJERCICIO 12 — Comprobar un unknown
// ---------------------------------------------------------------------------
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


// ---------------------------------------------------------------------------
// EJERCICIO 13 — Actualizar parcialmente un usuario
// ---------------------------------------------------------------------------
type Usuario = {
    id: number;
    nombre: string;
    email: string;
    edad: number;
};

const cambios: Partial<Usuario> = { email: "ana@correo.com" };

const resumen: Pick<Usuario, "id" | "nombre"> = { id: 1, nombre: "Ana" };


// ---------------------------------------------------------------------------
// EJERCICIO 14 — Usuario público
// ---------------------------------------------------------------------------
type UsuarioPublico = Omit<Usuario, "email">;

const usuarioPublico: UsuarioPublico = {
    id: 1,
    nombre: "Ana",
    edad: 28,
};


// ---------------------------------------------------------------------------
// EJERCICIO 15 — Configuración obligatoria y de solo lectura
// ---------------------------------------------------------------------------
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


// ---------------------------------------------------------------------------
// EJERCICIO 16 — Cuenta bancaria
// ---------------------------------------------------------------------------
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
console.log(cuenta.consultarSaldo());


// ---------------------------------------------------------------------------
// EJERCICIO 17 — Elementos reproducibles
// ---------------------------------------------------------------------------
interface Reproducible {
    reproducir(): void;
}

class Cancion implements Reproducible {
    titulo: string;

    constructor(titulo: string) {
        this.titulo = titulo;
    }

    reproducir(): void {
        console.log(`♪ Reproduciendo canción: ${this.titulo}`);
    }
}

class Video implements Reproducible {
    titulo: string;

    constructor(titulo: string) {
        this.titulo = titulo;
    }

    reproducir(): void {
        console.log(`▶ Reproduciendo vídeo: ${this.titulo}`);
    }
}

const elementos: Reproducible[] = [
    new Cancion("Bohemian Rhapsody"),
    new Video("Tutorial de TypeScript"),
];

for (const elemento of elementos) {
    elemento.reproducir();
}


// ---------------------------------------------------------------------------
// EJERCICIO 18 — Estado de un pedido
// ---------------------------------------------------------------------------
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


// ---------------------------------------------------------------------------
// EJERCICIO 19 — Primer elemento genérico
// ---------------------------------------------------------------------------
function primero<T>(elementos: T[]): T | undefined {
    return elementos[0];
}

console.log(primero([10, 20, 30]));
console.log(primero(["Ana", "Luis"]));
console.log(primero([]));


// ---------------------------------------------------------------------------
// EJERCICIO 20 — Registro de puntuaciones y promesa
// ---------------------------------------------------------------------------
type Puntuaciones = Record<string, number>;

const puntuaciones: Puntuaciones = {
    Ana: 85,
    Luis: 92,
    Marta: 78,
};

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
```

**Mapa de ejercicios → teoría:**

| Ejercicios | Tema | Dónde se explica |
|------------|------|------------------|
| 1, 2 | Tipos primitivos, `number[]`, inferencia | `02` |
| 3, 4 | Parámetros opcionales, tipos de retorno | `04` |
| 5 | Unión `number \| string` + `typeof` | `03`, `06` |
| 6, 7 | `type` con propiedad opcional | `03` |
| 8 | Unión de literales | `03` |
| 9, 10 | `interface`, array de interfaces | `03` |
| 11 | `string \| undefined` y comprobación | `02`, `06` |
| 12 | `unknown` + `typeof` | `06` |
| 13, 14, 15 | `Partial`, `Pick`, `Omit`, `Required`, `Readonly` | `05` |
| 16 | Clase con `private` | `05` |
| 17 | `interface` + `implements` (polimorfismo) | `05` |
| 18 | `enum` numérico + `switch` | `05` |
| 19 | Genérico `primero<T>` | `04` |
| 20 | `Record<string, number>` + `async` / `Promise<string>` | `03`, `04` |

**Detalles para pulir:**
- EJERCICIO 5: `console.log(mostrarID(123))` imprime también `undefined` (la función es `void`).
- EJERCICIOS 6 y 9: los objetos del `console.log` **no** están tipados con `Producto`/`Persona`;
  para que TS los compruebe, guárdalos en una variable: `const p: Producto = {...}`.
- EJERCICIO 8: `miTarea` no está declarada; para ver el error: `let miTarea: estadoTarea = "pendiente";`.

---

## EJEMPLO 4 — TypeScript en el navegador (DOM)

**Origen:** `git/Pagina-Web/ts/programacion.ts` *(copiado íntegro)*

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

// Animación "reveal": los elementos con la clase .reveal aparecen al hacer scroll
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

document.querySelectorAll(".reveal").forEach((element) => {
  observer.observe(element);
});

// Efecto de máquina de escribir en el texto de la página "Sobre mí"
const elemento = document.getElementById("texto-escrito");

if (elemento) {
  const texto: string = elemento.textContent?.trim() ?? "";
  elemento.textContent = "";

  let posicion = 0;

  const escribir = (): void => {
    if (posicion < texto.length) {
      const contenidoActual: string = elemento.textContent ?? "";
      elemento.textContent = contenidoActual + texto.charAt(posicion);
      posicion++;
      setTimeout(escribir, 10); // Más milisegundos = más lento
    }
  };

  escribir();
}

// Efecto Cyberpunk interactivo: feedback visual al hacer clic en enlaces o botones
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

**Qué enseña:**
- `instanceof HTMLElement` / `instanceof Node` → narrowing de elementos del DOM (`06`).
- Callbacks tipados de `IntersectionObserver` (`04`).
- `textContent?.trim() ?? ""` → `?.` y `??` para manejar `null` (`06`).
- `const escribir = (): void => {...}` → función flecha con retorno `void` (`04`).

**Cómo se compila** (`git/Pagina-Web/tsconfig.json`: `rootDir: "ts"`, `outDir: "js"`):

```bash
cd git/Pagina-Web
npm run watch     # tsc --watch → cada vez que guardas, regenera js/programacion.js
```

El `index.html` carga el resultado compilado:

```html
<script src="js/programacion.js"></script>
```

---

## EJEMPLO 5 — API REST con Express + MongoDB + TypeScript

### 5.1 Los tipos del dominio

**Origen:** `api-rastro/api-rastro/src/types.ts` *(copiado íntegro)*

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

### 5.2 La conexión, con colecciones tipadas

**Origen:** `api-rastro/api-rastro/src/db.ts` *(copiado íntegro)*

```ts
import { Db, MongoClient } from "mongodb";
import type { Listing, Offer, User } from "./types";

const uri = process.env.MONGODB_URI;
if (!uri) {
  throw new Error("Falta MONGODB_URI en el archivo .env");
}

const client = new MongoClient(uri, { serverSelectionTimeoutMS: 5000 });

let db: Db;

/**
 * Conecta a la base de datos MongoDB y asigna la instancia de Db.
 */
export async function connectDb() {
  await client.connect();
  db = client.db(process.env.DB_NAME ?? "rastro");
  console.log(`Conectado a MongoDB (base de datos ${db.databaseName})`);
}

export function getDb() {
  return db;
}

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

### 5.3 El servidor

**Origen:** `api-rastro/api-rastro/src/index.ts` *(copiado íntegro)*

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import { connectDb, getDb } from "./db";
import listingsRouter from "./routes/listings";
import offersRouter from "./routes/offers";

const app = express();
const PORT = 3000;

app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

app.use(express.json());

app.get("/api/health", async (_req, res) => {
  await getDb().command({ ping: 1 });
  res.json({ status: "ok", db: getDb().databaseName });
});

app.use("/api/listings", listingsRouter);
app.use("/api/offers", offersRouter);

app.use((req, res) => {
  res.status(404).json({ error: `No existe la ruta ${req.method} ${req.url}` });
});

app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  if (err instanceof SyntaxError) {
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }

  console.error(err);
  res.status(500).json({ error: "Error interno del servidor" });
});

async function start() {
  await connectDb();

  app.listen(PORT, () => {
    console.log(`Servidor escuchando en http://localhost:${PORT}`);
  });
}

start();
```

### 5.4 Las rutas de ofertas

**Origen:** `api-rastro/api-rastro/src/routes/offers.ts` *(copiado íntegro)*

```ts
import { Router } from "express";
import { ObjectId } from "mongodb";
import { listingsCollection, offersCollection, usersCollection } from "../db";
import type { Offer } from "../types";

const router = Router();

/**
 * Crea una nueva oferta para un anuncio.
 */
router.post("/", async (req, res) => {
  const { listingId, buyerId, amount } = req.body ?? {};

  if (typeof listingId !== "string" || !ObjectId.isValid(listingId)) {
    res.status(400).json({ error: "listingId no tiene un formato válido" });
    return;
  }
  if (typeof buyerId !== "string" || !ObjectId.isValid(buyerId)) {
    res.status(400).json({ error: "buyerId no tiene un formato válido" });
    return;
  }

  if (typeof amount !== "number" || !Number.isFinite(amount) || amount <= 0) {
    res.status(400).json({ error: "amount tiene que ser un número mayor que 0" });
    return;
  }

  const listing = await listingsCollection().findOne({ _id: new ObjectId(listingId) });
  if (!listing) {
    res.status(400).json({ error: "No existe ningún anuncio con ese listingId" });
    return;
  }

  const buyer = await usersCollection().findOne({ _id: new ObjectId(buyerId) });
  if (!buyer) {
    res.status(400).json({ error: "No existe ningún usuario con ese buyerId" });
    return;
  }

  // Validación: el comprador no puede ser el vendedor del anuncio
  if (listing.seller.id.equals(buyer._id)) {
    res.status(400).json({ error: "No puedes ofertar por tu propio anuncio" });
    return;
  }

  if (listing.status !== "AVAILABLE") {
    res.status(409).json({ error: `No se puede ofertar: el anuncio está ${listing.status}` });
    return;
  }

  const newOffer: Offer = {
    amount,
    status: "PENDING",
    createdAt: new Date(),
    listing: { id: listing._id, title: listing.title, price: listing.price },
    buyer: { id: buyer._id, name: buyer.name },
  };

  const result = await offersCollection().insertOne(newOffer);

  res.status(201).json({ _id: result.insertedId, ...newOffer });
});

/**
 * Acepta una oferta pendiente y reserva el anuncio.
 */
router.patch("/:id", async (req, res) => {
  const { id } = req.params;

  if (!ObjectId.isValid(id)) {
    res.status(400).json({ error: "El id no tiene un formato válido" });
    return;
  }

  if (req.body?.status !== "ACCEPTED") {
    res.status(400).json({ error: 'status tiene que ser "ACCEPTED"' });
    return;
  }

  const offerId = new ObjectId(id);

  const offer = await offersCollection().findOne({ _id: offerId });
  if (!offer) {
    res.status(404).json({ error: "No existe ninguna oferta con ese id" });
    return;
  }

  if (offer.status !== "PENDING") {
    res.status(409).json({ error: `No se puede aceptar: la oferta está ${offer.status}` });
    return;
  }

  const listing = await listingsCollection().findOne({ _id: offer.listing.id });
  if (!listing || listing.status !== "AVAILABLE") {
    res.status(409).json({ error: "No se puede aceptar: el anuncio ya no está disponible" });
    return;
  }

  const acceptedOffer = await offersCollection().findOneAndUpdate(
    { _id: offerId, status: "PENDING" },
    { $set: { status: "ACCEPTED" } },
    { returnDocument: "after" },
  );
  if (!acceptedOffer) {
    res.status(409).json({ error: "No se puede aceptar: la oferta ya no está PENDING" });
    return;
  }

  await listingsCollection().updateOne({ _id: offer.listing.id }, { $set: { status: "RESERVED" } });

  res.json(acceptedOffer);
});

export default router;
```

### 5.5 Configuración

**Origen:** `api-rastro/api-rastro/tsconfig.json` *(copiado íntegro)*

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "dist"
  },
  "include": ["src/**/*.ts"]
}
```

**Origen:** `api-rastro/api-rastro/.env.example` *(copiado íntegro — el `.env` real no se copia)*

```
MONGODB_URI=mongodb://usuario:contraseña@localhost:27017/?authSource=admin
DB_NAME=rastro
```

**Qué enseña todo el proyecto:**
- `types.ts` centraliza los tipos; los demás archivos hacen `import type` (`07`).
- `collection<Listing>(...)` → **un genérico** tipa todas las consultas (`04`).
- `process.env.MONGODB_URI` + `if (!uri) throw` → narrowing de `string | undefined` (`06`).
- Middleware de errores con `err: unknown` + `instanceof SyntaxError` (`06`, `07`).
- `req.body` es `any` → se valida **cada campo** antes de crear `const newOffer: Offer` (`07`).
- `status: "PENDING"` → solo acepta los literales de `OfferStatus` (`03`).

**Cómo ejecutarlo:**

```bash
cd api-rastro/api-rastro
cp .env.example .env      # y rellena tus datos
npm install
npm run dev               # tsx watch: arranca sin compilar (no revisa tipos)
npm run typecheck         # tsc --noEmit: revisa tipos sin generar archivos
```

> 📌 Las rutas de anuncios (`src/routes/listings.ts`) tienen los ejemplos de
> `value is string`, `Filter<Listing>` y `as const`; están copiadas por partes en
> `06` y `07`.

---

## EJEMPLO 6 — NestJS: interfaces, contrato y DTOs

### 6.1 Modelo de la base de datos y de la API

**Origen:** `puta/api-books-nest/src/books/book.ts` *(copiado íntegro)*

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

### 6.2 El contrato del servicio

**Origen:** `puta/api-books-nest/src/books/books.contract.ts` *(copiado íntegro)*

```ts
import { Book } from './book';
import { CreateBookDto } from './dto/create-book.dto';
import { UpdateBookDto } from './dto/update-book.dto';

// El contrato: qué tiene que saber hacer un servicio de libros
export interface BooksContract {
  findAll(author?: string): Promise<Book[]>;
  findOne(id: number): Promise<Book>;
  create(dto: CreateBookDto): Promise<Book>;
  update(id: number, dto: UpdateBookDto): Promise<Book>;
  remove(id: number): Promise<Book>;
}
```

### 6.3 DTO para crear

**Origen:** `puta/api-books-nest/src/books/dto/create-book.dto.ts` *(copiado íntegro)*

```ts
import { Transform } from 'class-transformer';
import { IsInt, IsString, Max, MaxLength, Min, MinLength } from 'class-validator';

// Lo que tiene que enviar el cliente para crear un libro
export class CreateBookDto {
  // Si llega un texto, le quita los espacios del principio y del final
  @Transform(({ value }: { value: unknown }) =>
    typeof value === 'string' ? value.trim() : value,
  )
  @IsString() // tiene que ser texto
  @MinLength(1) // no puede estar vacío
  @MaxLength(200) // como mucho 200 caracteres
  title: string;

  @IsString()
  @MinLength(1)
  @MaxLength(100)
  author: string;

  @IsInt() // número entero
  @Min(1, { message: 'El libro tiene que tener al menos 1 página' }) // al menos 1 página
  @Max(2000) // como mucho 2000 páginas
  pages: number;
}
```

### 6.4 DTO para actualizar

**Origen:** `puta/api-books-nest/src/books/dto/update-book.dto.ts` *(copiado íntegro)*

```ts
import { IsInt, IsString, MaxLength, Min, MinLength, ValidateIf } from 'class-validator';

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

### 6.5 El servicio que implementa el contrato

**Origen:** `puta/api-books-nest/src/books/books.service.ts` *(copiado íntegro)*

```ts
import { BadRequestException, Injectable, NotFoundException } from '@nestjs/common';
import { Pool } from 'pg';
import { Book, BookRow, toBook } from './book';
import { CreateBookDto } from './dto/create-book.dto';
import { UpdateBookDto } from './dto/update-book.dto';
import { BooksContract } from './books.contract';

@Injectable()
export class BooksService implements BooksContract  {
  constructor(private readonly pool: Pool) {}

  async findAll(author?: string, maxPages?: number): Promise<Book[]> {
    const result = await this.pool.query<BookRow>(
      `SELECT id, title, author, pages, created_at
       FROM books
       WHERE ($1::text IS NULL OR author ILIKE '%' || $1 || '%')
         AND ($2::int IS NULL OR pages <= $2)
       ORDER BY id`,
      [author || null, maxPages ?? null],
    );
    return result.rows.map(toBook);
  }

  async count(): Promise<{ total: number }> {
    const result = await this.pool.query<{ total: string }>(
      'SELECT COUNT(*)::text AS total FROM books',
    );
    return { total: Number(result.rows[0].total) };
  }

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

  async create(dto: CreateBookDto): Promise<Book> {
    const result = await this.pool.query<BookRow>(
      `INSERT INTO books (title, author, pages)
       VALUES ($1, $2, $3)
       RETURNING id, title, author, pages, created_at`,
      [dto.title, dto.author, dto.pages],
    );
    return toBook(result.rows[0]);
  }

  async update(id: number, dto: UpdateBookDto): Promise<Book> {
    if (dto.title === undefined && dto.author === undefined && dto.pages === undefined) {
      throw new BadRequestException('Envía al menos un campo');
    }
    const book = await this.findOne(id);

    const result = await this.pool.query<BookRow>(
      `UPDATE books
       SET title = COALESCE($1, title),
           author = COALESCE($2, author),
           pages = COALESCE($3, pages)
       WHERE id = $4
       RETURNING id, title, author, pages, created_at`,
      [dto.title ?? null, dto.author ?? null, dto.pages ?? null, id],
    );
    return toBook(result.rows[0]);
  }

  async remove(id: number): Promise<Book> {
    const book = await this.findOne(id);
    await this.pool.query('DELETE FROM books WHERE id = $1', [id]);
    return book;
  }
}
```

**Qué enseña:**
- Dos interfaces para el mismo dato (`BookRow` de SQL → `Book` de la API) y una función
  `toBook` que traduce (`03`).
- `interface BooksContract` + `class BooksService implements BooksContract` (`05`).
- `Promise<Book[]>`, `Promise<{ total: number }>` → retornos `async` (`04`).
- `pool.query<BookRow>(...)` → genérico de la librería `pg` (`04`).
- DTOs como **clases** con decoradores: validan en ejecución, cosa que los tipos no hacen (`07`).
- `dto.title ?? null` → `??` para pasar `NULL` a SQL cuando falta un campo (`06`).

> 📌 Detalle: en `update()` la constante `const book = await this.findOne(id);`
> **no se usa** después (solo sirve para lanzar el 404 si el libro no existe). Con
> `noUnusedLocals: true` TS lo marcaría; bastaría con `await this.findOne(id);`.
> En `remove()`, en cambio, `book` sí se usa: se devuelve el libro borrado.

---

## EJEMPLO 7 — Frontend tipado con `fetch` genérico

**Origen:** `ProyectoReproductor/src/login.ts` *(copiado íntegro)*

```ts
type User = { email: string };
const $ = <T extends Element>(selector: string) => document.querySelector<T>(selector)!;

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

function feedback(form: HTMLFormElement, message = '', error = false) {
  const target = form.querySelector('.form-feedback')!;
  target.textContent = message;
  target.classList.toggle('error', error);
}

function selectTab(name: 'login' | 'register') {
  for (const id of ['login', 'register'] as const) {
    const selected = id === name;
    $(`#${id}-form`).toggleAttribute('hidden', !selected);
    const tab = $<HTMLButtonElement>(`#${id}-tab`);
    tab.classList.toggle('is-active', selected);
    tab.setAttribute('aria-selected', String(selected));
  }
}

$('#login-tab').addEventListener('click', () => selectTab('login'));
$('#register-tab').addEventListener('click', () => selectTab('register'));

for (const [id, endpoint] of [['login-form', 'login'], ['register-form', 'register']] as const) {
  $<HTMLFormElement>(`#${id}`).addEventListener('submit', async event => {
    event.preventDefault();
    const form = event.currentTarget as HTMLFormElement;
    const data = new FormData(form);
    feedback(form);
    try {
      await request<User>(`/api/auth/${endpoint}`, {
        method: 'POST',
        body: JSON.stringify({ email: data.get('email'), password: data.get('password') })
      });
      window.location.href = 'app.html';
    } catch (error) {
      feedback(form, (error as Error).message, true);
    }
  });
}

(async () => {
  try {
    await request<User>('/api/auth/me');
    window.location.href = 'app.html';
  } catch {
    // No active session, stay on landing/login page
  }
})();
```

**Qué enseña:**
- `type User = { email: string }` → tipo pequeño para la respuesta de la API (`03`).
- `const $ = <T extends Element>(...) => document.querySelector<T>(selector)!` →
  genérico con restricción + `!` (`04`, `06`).
- `async function request<T>(...): Promise<T>` → el `fetch` tipado que reutiliza todo el
  proyecto (`04`).
- `name: 'login' | 'register'` → parámetro con literales (`03`).
- `[...] as const` → tuplas de literales para el bucle (`02`, `06`).
- `(error as Error).message` → aserción en el `catch` (`06`).
- `(async () => { ... })();` → función autoinvocada asíncrona.

> 📌 El archivo hermano `ProyectoReproductor/src/app.ts` (557 líneas) usa el mismo
> `request<T>` con `type Track`, `Map<string, Track[]>`, `Set<string>` y muchas
> aserciones `as HTMLButtonElement`. Sus fragmentos más útiles están en `02`, `04` y `06`.

**Variables de entorno del proyecto** (`ProyectoReproductor/.env.example`, copiado íntegro —
el `.env` real no se copia):

```
JAMENDO_CLIENT_ID=
SESSION_SECRET=
PORT=3000
DATABASE_URL=file:./data/nightjar.sqlite
```

**Cómo ejecutarlo:**

```bash
cd ProyectoReproductor
npm install
npm run dev       # tsx watch server/index.ts
npm run check     # tsc --noEmit
npm run build     # tsc + copia del cliente compilado
```

---

## 🧪 Ejercicio final para practicar (ejemplo propio)

Crea `practica.ts` en `practica-typescript/` y resuélvelo usando lo aprendido:

```ts
// 1. Tipo literal para la categoría de un producto
type Categoria = "ropa" | "electronica" | "hogar";

// 2. Interfaz Producto (stock opcional)
interface Producto {
  id: number;
  nombre: string;
  precio: number;
  categoria: Categoria;
  stock?: number;
}

// 3. Array tipado con 3 productos
const productos: Producto[] = [
  { id: 1, nombre: "Camiseta", precio: 19.99, categoria: "ropa", stock: 10 },
  { id: 2, nombre: "Auriculares", precio: 59, categoria: "electronica" },
  { id: 3, nombre: "Lámpara", precio: 25, categoria: "hogar", stock: 0 },
];

// 4. Función genérica que filtra por una propiedad
function filtrarPor<T, K extends keyof T>(items: T[], clave: K, valor: T[K]): T[] {
  return items.filter(item => item[clave] === valor);
}

console.log(filtrarPor(productos, "categoria", "ropa"));
// filtrarPor(productos, "categoria", "comida");   // ❌ "comida" no es una Categoria

// 5. Tipo para actualizar (id obligatorio, el resto opcional)
type ActualizarProducto = Pick<Producto, "id"> & Partial<Omit<Producto, "id">>;

const cambio: ActualizarProducto = { id: 2, precio: 49 };

// 6. Narrowing con el stock opcional
function hayStock(p: Producto): boolean {
  return p.stock !== undefined && p.stock > 0;
}
```

> 📌 `K extends keyof T` = "K tiene que ser **una de las claves** de T"
> (`"id" | "nombre" | "precio" | "categoria" | "stock"`), y `T[K]` = "el tipo de esa
> propiedad". Así `valor` tiene que encajar con la clave elegida.

**Pruebas a hacer:**
1. `npx tsc --noEmit` → no debe haber errores.
2. Quita el comentario de la línea `"comida"` → mira el error en el editor.
3. Cambia `p.stock !== undefined && p.stock > 0` por `p.stock > 0` → lee el error.
4. `npm run build` y `node practica.js`.

👉 **Vuelve al índice:** `00-INDICE.md`
