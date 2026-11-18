# 📚 CARPETA TYPESCRIPT — ÍNDICE DE ESTUDIO

Teoría de **TypeScript** explicada con ejemplos reales **copiados y pegados de tus
proyectos** del Bootcamp.

> ⚠️ Los proyectos **NO** se han modificado. Solo se han copiado fragmentos de su código.
> Los archivos `.env` **no** se copian (tienen secretos); como mucho se muestra el `.env.example`.

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a aprender |
|---|---------|--------------------|
| 1 | `01-que-es-typescript-y-compilacion.md` | Qué es TS, `tsc`, `tsx`, `tsconfig.json`, qué genera la compilación, `strict` |
| 2 | `02-tipos-basicos-inferencia-arrays-tuplas.md` | `string`/`number`/`boolean`, inferencia, arrays, tuplas, `null`/`undefined`, `void` |
| 3 | `03-union-literal-type-e-interface.md` | Uniones `\|`, tipos literales, `type` vs `interface`, `?` opcional, `readonly`, `Record` |
| 4 | `04-funciones-tipadas-y-genericos.md` | Parámetros y retorno, opcionales, callbacks, `Promise<T>`, genéricos `<T>` |
| 5 | `05-enums-clases-y-utility-types.md` | `enum`, clases con `private`, `implements`, `Partial`, `Pick`, `Omit`, `Readonly`... |
| 6 | `06-narrowing-any-unknown-y-aserciones.md` | `typeof`, `instanceof`, `?.`, `??`, `value is T`, `any` vs `unknown`, `as`, `!`, `as const` |
| 7 | `07-modulos-y-typescript-con-node-express.md` | `import`/`export`, `import type`, `@types`, `process.env`, Express y NestJS con TS |
| 8 | `08-ejemplos-completos-de-proyectos.md` | Archivos TS enteros copiados de tus proyectos |
| 📋 | `99-CHEATSHEET.md` | Chuleta de comandos y código para consultar rápido |

---

## 🗂️ Proyectos de los que se han extraído los ejemplos

| Proyecto | Archivo TS | Temas que cubre |
|----------|-----------|-----------------|
| `practica-typescript/` | `index.ts`, `precio.ts`, `tsconfig.json` | Compilación con `tsc`, `interface` + `implements`, `type`, `enum`, `extends` |
| `ejercicios-Bikram-Typescript/ejercicios-Bikram-Typescripts/` | `ejercicios-typescript.ts` | **20 ejercicios**: tipos, uniones, literales, `unknown`, utility types, clases, enums, genéricos |
| `git/Pagina-Web/` | `ts/programacion.ts` + `tsconfig.json` | TS en el **navegador**: DOM, `instanceof`, `?.`, `??`, `rootDir`/`outDir` |
| `node-practica/prueba/` | `index.ts` | `import` de una librería (`dayjs`) en Node |
| `ProyectoReproductor/` | `server/index.ts`, `src/app.ts`, `src/login.ts` | Genéricos (`request<T>`, `one<T>`, `$<T extends Element>`), `Map`/`Set`, aserciones, ESM |
| `api-rastro/api-rastro/` | `src/types.ts`, `src/db.ts`, `src/index.ts`, `src/routes/*.ts` | **Express + MongoDB** tipados, `collection<T>`, `Filter<T>`, `value is T`, `unknown` |
| `puta/api-books-nest/` | `src/books/*.ts`, `src/books/dto/*.ts` | **NestJS**: interfaces, contratos, DTOs con decoradores, `Promise<T>`, `query<T>` |

---

## 🚀 Orden de estudio recomendado

1. `01` → entiende **qué es** TypeScript y cómo se convierte en JS (sin esto te pierdes).
2. `02` → tipos básicos e inferencia: la base de todo.
3. `03` → `type`, `interface` y uniones: el 80% del TS del día a día.
4. `04` → funciones tipadas y genéricos.
5. `05` → enums, clases y utility types.
6. `06` → narrowing y `unknown`: cómo trabajar **seguro** con datos inciertos.
7. `07` → TS en proyectos reales de Node, Express y Nest.
8. `08` → código real completo de tus proyectos + ejercicio final.

> 📌 Antes de esta carpeta conviene tener claro `JAVASCRIPT/` (sobre todo funciones,
> objetos y arrays): TypeScript **es** JavaScript con tipos encima.

---

## 💡 Diferencia clave con JavaScript

```js
// JavaScript: nadie te avisa del error hasta que lo ejecutas
function calcularDescuento(precio, porcentaje) {
  return precio - (precio * porcentaje / 100);
}
calcularDescuento("100");   // NaN, sin ningún aviso
```

```ts
// TypeScript: el editor lo marca en rojo al escribirlo
function calcularDescuento(precio: number, porcentaje: number): number {
    return precio - (precio * porcentaje / 100);
}
calcularDescuento("100");
// ❌ Expected 2 arguments, but got 1.
calcularDescuento("100", 20);
// ❌ Argument of type 'string' is not assignable to parameter of type 'number'.
```

```bash
# Y para ejecutarlo, primero se compila
npx tsc          # archivo.ts → archivo.js
node archivo.js
```
