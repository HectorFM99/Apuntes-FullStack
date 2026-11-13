# 01 — QUÉ ES NODE.JS Y CÓMO SE EJECUTA

> **Node.js** es un programa que permite ejecutar **JavaScript fuera del navegador**:
> en tu terminal, en tu ordenador o en un servidor. Gracias a Node, con el mismo
> lenguaje que usas en el frontend puedes hacer **backend** (APIs, servidores,
> scripts, herramientas...).

---

## 1.1 Node NO es un lenguaje: es un *runtime*

Mucha gente dice "programo en Node", pero el lenguaje sigue siendo **JavaScript**
(o TypeScript). Node es el **entorno de ejecución** (*runtime*):

```
┌──────────────────────────────────────────────┐
│                  NODE.JS                     │
│                                              │
│   ┌───────────┐   ┌────────────────────────┐ │
│   │    V8     │   │   libuv (event loop,   │ │
│   │ (motor JS)│   │   disco, red, timers)  │ │
│   └───────────┘   └────────────────────────┘ │
│                                              │
│   + módulos propios: fs, path, http, crypto, │
│     process, os, events...                   │
└──────────────────────────────────────────────┘
```

| Pieza | Qué hace |
|-------|----------|
| **V8** | El motor de JavaScript de Google Chrome. Convierte tu JS en código máquina y lo ejecuta. |
| **libuv** | Biblioteca en C que se encarga del **event loop** y de hablar con el sistema operativo (archivos, red, temporizadores). |
| **Módulos nativos** | Funciones que trae Node de serie: `fs` (archivos), `path` (rutas), `http` (servidores)... |

> 📌 **Resumen en una frase:** Node = **V8** (entiende JS) + **libuv** (habla con el
> sistema operativo) + **módulos** (herramientas listas para usar).

### ¿Qué cambia respecto al navegador?

```js
// ❌ En Node esto FALLA: no hay página web
document.querySelector("h1");
// ReferenceError: document is not defined

// ✅ En Node esto funciona: puedes tocar el disco
const fs = require("fs");
fs.readFileSync("datos.txt", "utf-8");
```

> ⚠️ Fíjate en tus ejercicios de `Async-hector` y `Async-Hector-2`: usan `fetch` (que
> existe en Node y en el navegador) pero también `document.body.insertAdjacentHTML`,
> que **solo** existe en el navegador. Por eso se abren con `index.html` y no con
> `node index.js`.

---

## 1.2 Tu primer programa en Node

### Ejemplo REAL copiado (`node-practica/hola.js`)

```js
console.log("Hola desde Node.js");
console.log("Estamos ejecutando JavaScript fuera del navegador");
```

Para ejecutarlo, abre la terminal **en la carpeta del archivo** y escribe:

```bash
node hola.js
```

Salida:

```
Hola desde Node.js
Estamos ejecutando JavaScript fuera del navegador
```

**Puntos clave:**
- `node` + **nombre del archivo** → Node lee el archivo y lo ejecuta de arriba abajo.
- `console.log` ya **no** sale en la consola del navegador (F12), sale **en la terminal**.
- Cuando el programa no tiene nada más que hacer, **termina solo** y vuelves al prompt.
- La extensión `.js` es opcional al ejecutar: `node hola` también funciona.

### Comprobar que tienes Node instalado

```bash
node -v     # versión de Node  → v22.x.x
npm -v      # versión de npm   → 10.x.x (npm viene con Node)
```

---

## 1.3 El REPL: Node interactivo

Si escribes solo `node` (sin archivo), entras en el **REPL** (*Read-Eval-Print Loop*):
una consola donde pruebas JS línea a línea.

```bash
$ node
> 2 + 2
4
> const nombre = "Hector"
undefined
> `Hola ${nombre}`
'Hola Hector'
> .exit
```

> 📌 Útil para probar algo rápido (un método de array, una fecha...). Sales con
> `.exit` o pulsando **Ctrl + C** dos veces.

---

## 1.4 Un solo hilo (*single thread*)

JavaScript en Node se ejecuta en **un único hilo**: solo hace **una cosa a la vez**
con tu código.

Imagina un **camarero** en un restaurante:

```
❌ Camarero BLOQUEANTE (como otros lenguajes antiguos):
   Toma nota mesa 1 → va a cocina → ESPERA 10 min a que salga el plato → lo sirve
   → solo entonces atiende la mesa 2.

✅ Camarero NO BLOQUEANTE (Node):
   Toma nota mesa 1 → deja la comanda en cocina → toma nota mesa 2 → toma nota mesa 3
   → "¡plato de la mesa 1 listo!" → lo sirve → ...
```

- El **camarero** es el hilo de JavaScript (solo uno).
- La **cocina** es el sistema operativo / libuv (lee disco, hace peticiones de red...).
- El camarero **nunca se queda parado esperando**: deja el trabajo lento a la cocina
  y le avisan cuando termina.

> 📌 Por eso Node es muy bueno para **servidores y APIs**: la mayor parte del tiempo
> el servidor está **esperando** (a la base de datos, a otra API, al disco). Mientras
> espera, puede atender a otros usuarios.

---

## 1.5 I/O no bloqueante

**I/O** = *Input/Output* (entrada/salida): leer un archivo, consultar una base de
datos, hacer un `fetch`, recibir una petición HTTP...

```js
// Ejemplo propio (no es de tus proyectos)
const fs = require("fs");

console.log("1. Antes de leer");

// readFile es NO BLOQUEANTE: pide el archivo y sigue adelante
fs.readFile("datos.txt", "utf-8", (err, contenido) => {
  console.log("3. Archivo leído:", contenido);
});

console.log("2. Después de pedir la lectura");
```

Salida:

```
1. Antes de leer
2. Después de pedir la lectura
3. Archivo leído: ...
```

> ⚠️ El orden **no** es el del código. El `3` sale el último porque la lectura del
> disco tarda, y Node **no se queda esperando**: ejecuta el `2` mientras tanto.

### Bloqueante vs. no bloqueante

```js
// ❌ BLOQUEANTE (Sync): el programa se para hasta que termina de leer
const texto = fs.readFileSync("datos.txt", "utf-8");

// ✅ NO BLOQUEANTE: Node sigue con lo demás y te avisa al terminar
fs.readFile("datos.txt", "utf-8", (err, texto) => { /* ... */ });

// ✅ NO BLOQUEANTE moderno (promesas + await) → lo verás en el tema 06
const texto2 = await fs.promises.readFile("datos.txt", "utf-8");
```

> 📌 Las versiones `...Sync` están bien para **scripts pequeños** o para **arrancar**
> el servidor (por ejemplo, leer un archivo una vez al inicio). Dentro de una ruta
> que atiende peticiones, **evítalas**: bloquearías a todos los usuarios.

En tu `ProyectoReproductor/server/index.ts` se usan funciones `Sync` **al arrancar**
el servidor (y para guardar la base de datos en disco):

```ts
mkdirSync(join(dbPath, '..'), { recursive: true });
// ...
try { db = new SQL.Database(readFileSync(dbPath)); } catch { db = new SQL.Database(); }
```

---

## 1.6 El *event loop* (bucle de eventos)

El **event loop** es el "jefe de sala" que decide **qué se ejecuta después**.
Funciona así:

```
         ┌──────────────────────────────┐
         │  1. Ejecuta TODO tu código    │
         │     síncrono (de arriba abajo)│
         └──────────────┬───────────────┘
                        ▼
         ┌──────────────────────────────┐
   ┌───► │  2. ¿Hay tareas terminadas?   │
   │     │  (timers, archivos leídos,    │
   │     │   peticiones HTTP, promesas)  │
   │     └──────────────┬───────────────┘
   │                    ▼
   │     ┌──────────────────────────────┐
   │     │  3. Ejecuta su callback       │
   │     └──────────────┬───────────────┘
   │                    │
   └────────────────────┘   ← y vuelta a empezar mientras quede trabajo pendiente
```

**Reglas prácticas:**
1. Primero se ejecuta **todo el código síncrono**.
2. Después, las **promesas** resueltas (*microtareas*).
3. Después, los **timers** (`setTimeout`), lecturas de disco, red... (*macrotareas*).
4. Si no queda nada pendiente (ni timers, ni servidor escuchando), **Node termina**.

```js
// Ejemplo propio para ver el orden
console.log("A");

setTimeout(() => console.log("B (timeout 0)"), 0);

Promise.resolve().then(() => console.log("C (promesa)"));

console.log("D");
```

Salida:

```
A
D
C (promesa)
B (timeout 0)
```

> 📌 Aunque el `setTimeout` sea de `0` ms, **nunca** se ejecuta antes que el código
> síncrono, y las promesas tienen prioridad sobre los timers.

### ¿Por qué un servidor NO termina?

`hola.js` termina en cuanto imprime los dos mensajes. Pero un servidor se queda
"encendido". Es porque `listen` deja una tarea pendiente para siempre: **esperar
peticiones**.

**Ejemplo REAL** (`api-rastro/api-rastro/src/index.ts`):

```ts
app.listen(PORT, () => {
  console.log(`Servidor escuchando en http://localhost:${PORT}`);
});
```

> 📌 Para parar un servidor en la terminal: **Ctrl + C**.

---

## 1.7 ❌ Bloquear el event loop

Como solo hay **un hilo**, si haces un cálculo pesado síncrono, **todo** se para.

```js
// ❌ Ejemplo propio: mientras este bucle gira, el servidor NO responde a nadie
app.get("/lento", (req, res) => {
  let total = 0;
  for (let i = 0; i < 5_000_000_000; i++) total += i;
  res.json({ total });
});
```

| Tarea | ¿Bloquea? | Comentario |
|-------|-----------|------------|
| `await fetch(...)` | ❌ No | La red la gestiona libuv |
| `await collection.find()` (MongoDB) | ❌ No | Igual: espera a la BD sin bloquear |
| `fs.readFile(...)` / `fs.promises.readFile` | ❌ No | |
| `fs.readFileSync(...)` | ✅ Sí | Úsalo solo al arrancar o en scripts |
| Bucle `for` gigante, cálculos pesados | ✅ Sí | Node no es ideal para cálculo intensivo |

---

## 1.8 Objetos globales de Node

En Node hay cosas disponibles **sin importar nada**:

| Global | Para qué |
|--------|----------|
| `console` | `console.log`, `console.error`, `console.table`... |
| `process` | Info del proceso: variables de entorno, argumentos, carpeta actual (tema 05) |
| `setTimeout` / `setInterval` | Temporizadores, igual que en el navegador |
| `fetch` | Peticiones HTTP (Node 18+) |
| `Buffer` | Datos binarios (bytes) |
| `globalThis` | El objeto global (equivale a `window` del navegador) |
| `__dirname`, `__filename` | Carpeta y archivo actual (**solo en CommonJS**, ver tema 03) |

**Ejemplo REAL** de `Buffer` y `process` sin importar nada
(`ProyectoReproductor/server/index.ts`):

```ts
const dbUrl = process.env.DATABASE_URL ?? 'file:./data/nightjar.sqlite';
// ...
function persist() { writeFileSync(dbPath, Buffer.from(db.export())); }
```

---

## 1.9 ¿Dónde encaja Express?

Node trae un módulo `http` para crear servidores, pero es muy "a mano" (lo verás en
el tema 04). **Express** es una librería que se instala con npm y hace lo mismo
de forma mucho más cómoda:

```
Tu API  ──usa──►  Express  ──usa──►  módulo http de Node  ──usa──►  sistema operativo
```

> 📌 Express tiene su **propia carpeta de apuntes** (`Apuntes/EXPRESS/`). Aquí solo
> necesitas saber que **Express funciona encima de Node**.

---

## 1.10 Errores comunes al empezar

| Error | Causa | Solución |
|-------|-------|----------|
| `'node' no se reconoce como un comando` | Node no instalado o no está en el PATH | Instalar Node (LTS) y reabrir la terminal |
| `Error: Cannot find module '.../hola.js'` | Estás en otra carpeta | `cd` a la carpeta correcta o pon la ruta: `node node-practica/hola.js` |
| `document is not defined` | Código de navegador en Node | El DOM no existe en Node |
| El programa "no termina" | Hay un servidor o un `setInterval` activo | **Ctrl + C** |
| `Error: listen EADDRINUSE :::3000` | El puerto ya está ocupado (otro servidor encendido) | Cierra el otro proceso o cambia el puerto |

---

## ✅ Resumen rápido

1. **Node.js** = runtime para ejecutar JavaScript fuera del navegador (V8 + libuv).
2. Se ejecuta con `node archivo.js`; `console.log` sale en la **terminal**.
3. **Un solo hilo** para tu código, pero el I/O (disco, red, BD) es **no bloqueante**.
4. El **event loop** ejecuta primero lo síncrono, luego promesas y luego timers/I/O.
5. Las funciones `...Sync` bloquean: solo para scripts o al arrancar.
6. No hay `document` ni `window`; sí hay `process`, `Buffer`, `fetch`, `globalThis`.
7. Un servidor no termina porque `listen` deja una tarea pendiente → Ctrl + C.

👉 **Siguiente:** `02-npm-package-json-y-dependencias.md`
