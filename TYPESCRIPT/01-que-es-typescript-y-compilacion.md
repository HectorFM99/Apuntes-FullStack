# 01 — QUÉ ES TYPESCRIPT Y CÓMO SE COMPILA

> **TypeScript (TS)** es **JavaScript + tipos**. Escribes lo mismo que en JS, pero
> le dices a cada variable, parámetro y objeto **qué forma tiene**. Antes de
> ejecutarse, un programa (el **compilador `tsc`**) revisa que todo encaje y lo
> **convierte en JavaScript normal**, que es lo que de verdad ejecuta el navegador
> o Node.

---

## 1.1 La idea en una frase

```
archivo.ts  ──(tsc revisa tipos y quita los tipos)──▶  archivo.js  ──▶ navegador / Node
```

- El navegador **no entiende** TypeScript. Node tampoco (salvo herramientas como `tsx`).
- Los tipos **solo existen mientras programas**. En el `.js` final desaparecen.
- TypeScript **no hace tu código más rápido**: hace que te equivoques **menos**,
  porque los errores salen **en el editor**, antes de ejecutar.

### Ejemplo REAL copiado (`ejercicios-Bikram-Typescript/ejercicios-Bikram-Typescripts/ejercicios-typescript.ts`)

```ts
// EJERCICIO 1
let nombre: string = "Hector";
let edad: number = 27;
let activo: boolean = true;

//edad = "ventisiete"; Error: Type 'string' is not assignable to type 'number'
```

> 📌 Ese comentario es la **razón de ser** de TypeScript. En JS, `edad = "ventisiete"`
> funciona sin quejarse y el fallo aparece mucho después (por ejemplo al hacer
> `edad + 1` → `"ventisiete1"`). En TS, el editor lo subraya en rojo **al escribirlo**.

---

## 1.2 JavaScript vs TypeScript lado a lado

```js
// JavaScript
function calcularDescuento(precio, porcentaje) {
  return precio - (precio * porcentaje / 100);
}

calcularDescuento("100", 20);   // 😱 "100" - 20 → 80 (por suerte), pero "100" + ... sería un desastre
calcularDescuento(100);         // NaN, sin aviso
```

```ts
// TypeScript (EJERCICIO 4 de tus ejercicios Bikram)
function calcularDescuento(precio: number, porcentaje: number): number {
    return precio - (precio * porcentaje / 100);
}

console.log(calcularDescuento(100, 20));   // ✅ 80
// calcularDescuento("100", 20);  ❌ Argument of type 'string' is not assignable to parameter of type 'number'.
// calcularDescuento(100);        ❌ Expected 2 arguments, but got 1.
```

| | JavaScript | TypeScript |
|---|-----------|------------|
| Extensión | `.js` | `.ts` (y `.tsx` con React) |
| ¿Se ejecuta directamente? | ✅ Sí | ❌ Hay que compilarlo (o usar `tsx`) |
| Errores de tipo | En ejecución (cuando ya ha fallado) | En el editor / al compilar |
| Autocompletado | Limitado | Muy bueno (el editor sabe qué propiedades existen) |
| Todo JS válido... | — | ...es casi siempre TS válido |

---

## 1.3 Instalar TypeScript en un proyecto

TypeScript se instala como **dependencia de desarrollo** (solo hace falta para programar,
no en producción):

```bash
npm init -y
npm install -D typescript       # -D = devDependencies
npx tsc --init                  # crea un tsconfig.json con opciones por defecto
```

### Ejemplo REAL copiado (`practica-typescript/package.json`)

```json
{
  "name": "practica-typescript",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "tsc",
    "start": "node index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs",
  "devDependencies": {
    "typescript": "^7.0.2"
  }
}
```

**Cómo se lee:**

```
"build": "tsc"            → npm run build  = compila todos los .ts a .js
"start": "node index.js"  → npm start      = ejecuta el JS YA compilado
devDependencies           → typescript solo se usa al programar
```

> 📌 Flujo de trabajo: **1)** editas `index.ts` → **2)** `npm run build` →
> **3)** `npm start`. Si te saltas el paso 2, ejecutas el `.js` **viejo**.

---

## 1.4 El compilador `tsc` — comandos que vas a usar

| Comando | Qué hace |
|---------|----------|
| `npx tsc` | Compila todo lo que diga el `tsconfig.json` |
| `npx tsc archivo.ts` | Compila solo ese archivo (⚠️ **ignora** el `tsconfig.json`) |
| `npx tsc --watch` | Se queda vigilando y recompila al guardar |
| `npx tsc --noEmit` | Solo **revisa** tipos, **no** genera `.js` |
| `npx tsc --init` | Crea un `tsconfig.json` |

### Ejemplos REALES de scripts en tus proyectos

**`git/Pagina-Web/package.json`** (web personal, TS para el navegador):

```json
"scripts": {
  "build": "tsc",
  "watch": "tsc --watch",
  "start": "node index.js"
},
```

**`api-rastro/api-rastro/package.json`** (API con Express + MongoDB):

```json
"scripts": {
  "dev": "tsx watch --env-file=.env src/index.ts",
  "lint": "eslint .",
  "typecheck": "tsc --noEmit"
},
```

**`ProyectoReproductor/package.json`**:

```json
"scripts": {"dev":"tsx watch server/index.ts", "build":"tsc && node scripts/copy-client.mjs", "check":"tsc --noEmit",  "start":"node dist/server/index.js"},
```

### 🧠 `tsc` vs `tsx`

| Herramienta | Qué hace | Cuándo |
|-------------|----------|--------|
| `tsc` | Revisa tipos **y** genera `.js` | Build para producción, o comprobar tipos (`--noEmit`) |
| `tsx` | Ejecuta `.ts` **directamente** (quita los tipos al vuelo) **sin revisarlos** | Desarrollo rápido (`tsx watch` = como `nodemon`) |

> ⚠️ `tsx` **no comprueba tipos**: si hay un error de tipos, el servidor arranca igual.
> Por eso en `api-rastro` y en `ProyectoReproductor` hay un script aparte
> (`typecheck` / `check`) con `tsc --noEmit`. Lánzalo antes de hacer commit.

---

## 1.5 Qué genera la compilación (los `.js`, `.d.ts`, `.map`)

En `practica-typescript/` tienes, junto a cada `.ts`, varios archivos generados:

```
practica-typescript/
├── index.ts          ← TU código (lo único que editas)
├── index.js          ← generado: JS que se ejecuta
├── index.js.map      ← generado: "mapa" .js ↔ .ts para depurar
├── index.d.ts        ← generado: solo los TIPOS (declaraciones)
├── index.d.ts.map    ← generado: mapa de las declaraciones
├── precio.ts / precio.js / precio.d.ts / ...
└── tsconfig.json
```

Se generan porque el `tsconfig.json` lo pide:

```jsonc
"sourceMap": true,        // → .js.map
"declaration": true,      // → .d.ts
"declarationMap": true,   // → .d.ts.map
```

### Lo que entra vs. lo que sale

**Entra** — `practica-typescript/index.ts` (fragmento real):

```ts
interface Notificable {
  enviar(mensaje: string): void;
}

class NotificacionEmail implements Notificable {
  constructor(private destinatario: string) {}

  enviar(mensaje: string): void {
    console.log(`📧 Enviando email a ${this.destinatario}: "${mensaje}"`);
  }
}
```

**Sale** — `practica-typescript/index.js` (generado por `tsc`, fragmento real):

```js
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
// 2. Creamos clases que "implementan" esa interfaz (cada una notifica a su manera)
class NotificacionEmail {
    destinatario;
    constructor(destinatario) {
        this.destinatario = destinatario;
    }
    enviar(mensaje) {
        console.log(`📧 Enviando email a ${this.destinatario}: "${mensaje}"`);
    }
}
```

**Fíjate en lo que ha pasado:**

```
interface Notificable {...}        → DESAPARECE (solo existía para revisar)
implements Notificable             → DESAPARECE
: string, : void                   → DESAPARECEN
constructor(private destinatario)  → se convierte en una propiedad + this.destinatario = ...
"use strict" + exports             → porque el proyecto es "type": "commonjs"
```

**Y el `.d.ts`** (`practica-typescript/index.d.ts`, real):

```ts
export {};
//# sourceMappingURL=index.d.ts.map
```

> 📌 Está casi vacío porque `index.ts` **no exporta nada**. El `.d.ts` describe
> lo que un archivo **ofrece a otros**; si no exportas nada, no hay nada que describir.
> Los `.d.ts` son lo que instalas con `@types/express`, `@types/node`... (ver `07`).

### ⚠️ Cuidado: el `.js` puede estar DESACTUALIZADO

En tu `practica-typescript/` el `precio.js` **no coincide** con el `precio.ts` actual.
El `.ts` tiene:

```ts
//1
function calcularDescuento(precio: number, porcentaje: number): number {
const descuento = precio * (porcentaje / 100);
return precio - descuento;
}
```

Pero el `precio.js` generado tiene una versión anterior:

```js
function calcularDescuento(precio, descuento) {
    if (descuento < 0 || descuento > 100) {
        throw new Error("El descuento debe estar entre 0 y 100");
    }
    return precio - (precio * descuento / 100);
}
```

> 📌 Eso pasa cuando editas el `.ts` y **no vuelves a compilar**. `node precio.js`
> ejecutaría la versión vieja. Regla: **el `.ts` es la verdad; el `.js` se regenera**.
> Por eso en proyectos serios los `.js` generados van a una carpeta `dist/` y se
> meten en `.gitignore`.

---

## 1.6 `tsconfig.json` — el archivo de configuración

Le dice a `tsc` **qué compilar, a qué versión de JS, dónde dejar el resultado y lo
estricto que debe ser**.

### Opciones más importantes

| Opción | Para qué sirve | Ejemplo |
|--------|----------------|---------|
| `target` | Versión de JS que genera | `"ES2020"`, `"ES2022"`, `"esnext"` |
| `module` | Sistema de módulos de salida | `"NodeNext"`, `"ES2020"`, `"commonjs"` |
| `rootDir` | Carpeta donde están tus `.ts` | `"ts"`, `"src"` |
| `outDir` | Carpeta donde deja los `.js` | `"js"`, `"dist"` |
| `include` | Qué archivos entran | `["src/**/*.ts"]` |
| `strict` | Activa TODAS las comprobaciones estrictas | `true` ✅ siempre |
| `lib` | Qué APIs conoce (DOM, ES2020...) | `["ES2020", "DOM"]` |
| `esModuleInterop` | Permite `import express from "express"` con librerías CommonJS | `true` |
| `skipLibCheck` | No revisa los `.d.ts` de `node_modules` (más rápido) | `true` |
| `sourceMap` | Genera `.js.map` | `true` |
| `noEmit` | No genera archivos (solo revisa) | — |

### Ejemplo REAL 1 — Web para el navegador (`git/Pagina-Web/tsconfig.json`)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "rootDir": "ts",
    "outDir": "js",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,
    "removeComments": false,
    "sourceMap": false,
    "newLine": "crlf"
  },
  "include": ["ts/**/*.ts"]
}
```

**Cómo se lee:**

```
rootDir "ts" + outDir "js"   → ts/programacion.ts  ──▶  js/programacion.js
lib ["DOM", ...]             → conoce document, HTMLElement, IntersectionObserver...
noUnusedLocals               → error si declaras una variable y no la usas
noFallthroughCasesInSwitch   → error si un case de switch no tiene break/return
```

> 📌 El HTML sigue enlazando `js/programacion.js`. Tú editas `ts/programacion.ts`
> y `npm run watch` mantiene el `.js` al día.

### Ejemplo REAL 2 — API con Node (`api-rastro/api-rastro/tsconfig.json`)

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

> 📌 Aquí **no** hay `"DOM"`: es un servidor, no hay `document`. Los tipos de Node
> (`process`, `Buffer`...) vienen del paquete `@types/node` instalado.

### Ejemplo REAL 3 — el generado por `tsc --init` (`practica-typescript/tsconfig.json`)

```jsonc
{
  // Visit https://aka.ms/tsconfig to read more about this file
  "compilerOptions": {
    // File Layout
    // "rootDir": "./src",
    // "outDir": "./dist",

    // Environment Settings
    // See also https://aka.ms/tsconfig/module
    "module": "nodenext",
    "target": "esnext",
    "types": [],
    // For nodejs:
    // "lib": ["esnext"],
    // "types": ["node"],
    // and npm install -D @types/node

    // Other Outputs
    "sourceMap": true,
    "declaration": true,
    "declarationMap": true,

    // Stricter Typechecking Options
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,

    // Style Options
    // "noImplicitReturns": true,
    // "noImplicitOverride": true,
    // "noUnusedLocals": true,
    // "noUnusedParameters": true,
    // "noFallthroughCasesInSwitch": true,
    // "noPropertyAccessFromIndexSignature": true,

    // Recommended Options
    "strict": true,
    "jsx": "react-jsx",
    "verbatimModuleSyntax": true,
    "isolatedModules": true,
    "noUncheckedSideEffectImports": true,
    "moduleDetection": "force",
    "skipLibCheck": true,
  }
}
```

**Las opciones "raras" explicadas:**

| Opción | Qué significa en cristiano |
|--------|---------------------------|
| `"rootDir"`/`"outDir"` comentados | Los `.js` salen **al lado** de los `.ts` (por eso los ves mezclados) |
| `noUncheckedIndexedAccess` | `arr[0]` es `T \| undefined` (puede no existir) → te obliga a comprobarlo |
| `exactOptionalPropertyTypes` | Una propiedad `x?: string` puede **faltar**, pero no puedes ponerle `undefined` a mano |
| `moduleDetection: "force"` | Cada archivo es un **módulo** aislado (sus variables no chocan con las de otros archivos) |
| `verbatimModuleSyntax` | Obliga a escribir `import type` cuando solo importas tipos |
| `"types": []` | No carga ningún `@types/*` automáticamente (por eso `console` viene de la lib y no de Node) |

> 🧠 **Curiosidad de tu proyecto:** `index.ts` y `precio.ts` declaran **los dos**
> `interface Notificable` y `class NotificacionSMS`. Sin `moduleDetection: "force"`
> serían scripts globales y TS se quejaría de "Duplicate identifier". Con `force`,
> cada archivo es su propio módulo y no se pisan.

---

## 1.7 `strict: true` — por qué siempre activado

`strict` enciende un paquete de comprobaciones. Las dos que más notarás:

```ts
// noImplicitAny: no puedes dejar parámetros sin tipo
function doble(n) { return n * 2; }
//            ❌ Parameter 'n' implicitly has an 'any' type.

function doble(n: number) { return n * 2; }   // ✅
```

```ts
// strictNullChecks: null/undefined NO caben en cualquier tipo
let nombre: string = null;          // ❌ Type 'null' is not assignable to type 'string'.
let nombre: string | null = null;   // ✅ lo dices explícitamente
```

### Comparación REAL: tu proyecto NestJS es menos estricto

**`puta/api-books-nest/tsconfig.json`** (fragmento real, generado por Nest CLI):

```json
"strictNullChecks": true,
"forceConsistentCasingInFileNames": true,
"noImplicitAny": false,
"strictBindCallApply": false,
"noFallthroughCasesInSwitch": false
```

> 📌 No tiene `strict: true`; solo activa `strictNullChecks` y **desactiva**
> `noImplicitAny`. Por eso en Nest puedes declarar un DTO con `title: string;`
> sin inicializarlo (con `strict` completo te pediría `title!: string;`). Para
> aprender, mejor `strict: true` desde el principio.

---

## 1.8 Errores típicos al empezar

| Error / síntoma | Causa | Solución |
|-----------------|-------|----------|
| Cambio el `.ts` y no pasa nada | Ejecutas el `.js` viejo | `npm run build` o `tsc --watch` |
| `tsc archivo.ts` no respeta mi config | Con un archivo concreto se ignora `tsconfig.json` | Ejecuta `npx tsc` a secas |
| `Cannot find name 'document'` | Falta `"DOM"` en `lib` | Añádelo (solo si es para el navegador) |
| `Cannot find name 'process'` | Faltan los tipos de Node | `npm i -D @types/node` |
| `Could not find a declaration file for module 'x'` | La librería no trae tipos | `npm i -D @types/x` |
| El servidor con `tsx` arranca pero hay errores rojos | `tsx` no comprueba tipos | `tsc --noEmit` |

---

## ✅ Resumen rápido

1. TS = JS + tipos. Los tipos **se borran** al compilar.
2. `npm i -D typescript` → `npx tsc --init` → `npx tsc`.
3. `tsc` revisa y genera `.js`; `tsx` ejecuta sin revisar; `tsc --noEmit` solo revisa.
4. `tsconfig.json`: `target`, `module`, `rootDir`, `outDir`, `include`, **`strict: true`**.
5. Navegador → `lib: ["DOM"]`. Node → `@types/node`.
6. El `.ts` es la fuente de verdad; el `.js` se **regenera** siempre.

👉 **Siguiente:** `02-tipos-basicos-inferencia-arrays-tuplas.md`
