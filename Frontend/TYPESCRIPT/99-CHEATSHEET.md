# 📋 CHEATSHEET — TYPESCRIPT

> Chuleta rápida de comandos y código. Teoría en los documentos 01–08 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal

| Comando | Qué hace |
|---------|----------|
| `npm install -D typescript` | Instala TS como dependencia de desarrollo |
| `npx tsc --init` | Crea `tsconfig.json` |
| `npx tsc` | Compila según el `tsconfig.json` |
| `npx tsc archivo.ts` | Compila solo ese archivo (⚠️ ignora el `tsconfig.json`) |
| `npx tsc --watch` | Recompila al guardar |
| `npx tsc --noEmit` | Solo comprueba tipos, sin generar `.js` |
| `npm install -D tsx` | Ejecutor de TS para desarrollo |
| `npx tsx archivo.ts` | Ejecuta `.ts` directamente (**no** comprueba tipos) |
| `npx tsx watch src/index.ts` | Como nodemon para TS |
| `npm install -D @types/node @types/express` | Tipos de librerías JS |

```json
// package.json (scripts reales de tus proyectos)
"scripts": {
  "dev": "tsx watch --env-file=.env src/index.ts",   // api-rastro
  "build": "tsc",
  "typecheck": "tsc --noEmit",
  "start": "node dist/index.js"
}
```

---

## 2. `tsconfig.json` básico

```jsonc
{
  "compilerOptions": {
    "target": "es2022",          // versión de JS generada
    "module": "nodenext",        // sistema de módulos (Node)
    "rootDir": "src",            // dónde están los .ts
    "outDir": "dist",            // dónde van los .js
    "strict": true,              // ✅ siempre
    "esModuleInterop": true,
    "skipLibCheck": true,
    "noEmit": false              // true si otro (Vite/tsx) genera el JS
  },
  "include": ["src"]
}
```

---

## 3. Tipos básicos

```ts
let nombre: string = "Ana";
let edad: number = 25;
let activo: boolean = true;
let x = 10;                       // inferido: number (no hace falta anotar)
const modo = "oscuro";            // tipo literal "oscuro"

let notas: number[] = [7, 8];     // array
let tags: Array<string> = [];     // igual, con genérico
let punto: [number, number] = [3, 4];          // tupla
let par: [string, number] = ["edad", 25];

let nada: null = null;  let sinValor: undefined;
let cualquiera: any;    // ❌ desactiva TS
let desconocido: unknown; // ✅ obliga a comprobar antes de usar
function log(m: string): void { }             // no devuelve nada
function fallar(): never { throw new Error(); } // nunca termina
let usuario: { nombre: string; edad?: number } = { nombre: "Ana" };  // objeto en línea
```

---

## 4. Uniones, literales, `type` e `interface`

```ts
let id: string | number;                       // unión
type Estado = "pendiente" | "pagado" | "enviado";   // literales
type Usuario = { id: number; nombre: string; email?: string };  // ? opcional

interface Producto {
  readonly id: number;        // no se puede reasignar
  nombre: string;
  precio: number;
  categoria?: string;
}
interface ProductoDigital extends Producto { urlDescarga: string }   // herencia
type Admin = Usuario & { permisos: string[] };                       // intersección

const precios: Record<string, number> = { cafe: 1.5, te: 1.2 };      // diccionario
```

| `type` | `interface` |
|--------|-------------|
| Uniones, literales, tuplas, cualquier tipo | Solo formas de objetos |
| `&` para combinar | `extends`; se puede reabrir (declaration merging) |
| Regla práctica: `type` para todo, `interface` para objetos/contratos de clases |

---

## 5. Funciones y genéricos

```ts
function sumar(a: number, b: number): number { return a + b; }
const doble = (n: number): number => n * 2;
function saludar(nombre: string, saludo = "Hola", titulo?: string) { }   // default y opcional
function total(...nums: number[]) { }                                   // rest

type Callback = (error: Error | null, data?: string) => void;           // tipo función
function procesar(items: string[], fn: (item: string) => void) { }

async function cargar(): Promise<Producto[]> {                          // async
  const res = await fetch("/api/products");
  return res.json();
}

function primero<T>(arr: T[]): T | undefined { return arr[0]; }        // genérico
primero<number>([1, 2]);  primero(["a"]);                               // T inferido
function conId<T extends { id: number }>(x: T) { return x.id; }        // restricción

async function getJson<T>(url: string): Promise<T> {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`Error ${res.status}`);
  return res.json() as Promise<T>;
}
```

---

## 6. Enums y clases

```ts
enum Rol { Admin = "ADMIN", User = "USER" }   // enum de strings
const r: Rol = Rol.Admin;
// Alternativa moderna: const ROLES = ["admin", "user"] as const; type Rol = typeof ROLES[number];

interface Repositorio { buscar(id: number): Producto | undefined }

class Tienda implements Repositorio {
  private productos: Producto[] = [];          // solo dentro de la clase
  constructor(public nombre: string, private readonly iva = 0.21) {}  // parameter properties
  buscar(id: number) { return this.productos.find(p => p.id === id); }
  protected log() {}                            // clase y subclases
  static crear() { return new Tienda("Mi tienda"); }
}
```

| Modificador | Acceso |
|-------------|--------|
| `public` (defecto) | Desde cualquier sitio |
| `private` | Solo la clase |
| `protected` | La clase y sus hijas |
| `readonly` | No se reasigna tras el constructor |

---

## 7. Utility types

| Tipo | Resultado | Uso típico |
|------|-----------|-----------|
| `Partial<T>` | Todo opcional | Body de un PATCH |
| `Required<T>` | Todo obligatorio | |
| `Readonly<T>` | Todo `readonly` | |
| `Pick<T, "a" \| "b">` | Solo esas claves | Vista pública |
| `Omit<T, "id">` | Todo menos esas | Body de un POST (`Omit<Producto, "id">`) |
| `Record<K, V>` | Objeto clave → valor | Diccionarios |
| `ReturnType<typeof fn>` | Tipo que devuelve `fn` | |
| `Awaited<Promise<T>>` | `T` | |
| `NonNullable<T>` | Sin `null`/`undefined` | |

```ts
type NuevoProducto = Omit<Producto, "id">;
type CambiosProducto = Partial<NuevoProducto>;
```

---

## 8. Narrowing y aserciones

```ts
function formatear(v: string | number) {
  if (typeof v === "string") return v.toUpperCase();   // aquí v: string
  return v.toFixed(2);                                 // aquí v: number
}
if (err instanceof Error) err.message;                // clases / Error / DOM
if ("urlDescarga" in producto) { }                    // propiedad
if (valor !== undefined) { }  if (valor) { }          // truthiness

// Unión discriminada
type Res = { ok: true; data: string } | { ok: false; error: string };
if (res.ok) res.data; else res.error;

// Type predicate
function esProducto(x: unknown): x is Producto {
  return typeof x === "object" && x !== null && "precio" in x;
}

try { } catch (error) {                                // error: unknown
  const msg = error instanceof Error ? error.message : String(error);
}

user?.direccion?.calle;          // ?. encadenamiento opcional
const puerto = process.env.PORT ?? 3000;   // ?? valor por defecto
const el = document.querySelector("#root")!;          // ! "no es null"
const input = document.querySelector("#email") as HTMLInputElement;  // as
const config = { modo: "dark" } as const;             // readonly + literales
```

---

## 9. Módulos y Node/Express con TS

```ts
export function sumar() {}           // export con nombre
export default class App {}          // export por defecto
import { sumar } from "./utils.js";  // ESM en Node: extensión .js
import type { Producto } from "./types";   // solo tipos (se borra al compilar)

// process.env siempre es string | undefined
const uri = process.env.MONGODB_URI;
if (!uri) throw new Error("Falta MONGODB_URI");

// Express tipado
import express from "express";
import type { Request, Response, NextFunction } from "express";
app.get("/api/products/:id", (req: Request<{ id: string }>, res: Response) => {
  res.json({ id: req.params.id });
});
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  res.status(500).json({ error: "Error interno del servidor" });
});
```

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `Type 'string' is not assignable to type 'number'` | Convierte (`Number(x)`) o corrige el tipo |
| `Object is possibly 'undefined'/'null'` | Comprueba antes (`if`, `?.`, `??`) o `!` si estás seguro |
| `Property 'x' does not exist on type` | Falta en la interface o necesitas narrowing |
| `Parameter 'x' implicitly has an 'any' type` | Anota el parámetro (`strict` activo) |
| `Could not find a declaration file for module` | `npm i -D @types/<paquete>` |
| `Cannot find module './x'` con ESM | Importa con `.js` (`nodenext`) |
| Ejecuto el `.js` viejo | `npm run build` antes de `npm start` |
| `tsx` arranca pero hay errores de tipos | `tsx` no comprueba: `npm run typecheck` (`tsc --noEmit`) |
