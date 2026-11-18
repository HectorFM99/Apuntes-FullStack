# 02 — NPM, `package.json` Y DEPENDENCIAS

> **npm** (*Node Package Manager*) es el gestor de paquetes que viene con Node.
> Sirve para **instalar librerías de otros** (Express, dayjs, MongoDB...) y para
> **ejecutar los comandos** de tu proyecto (`npm run dev`).
> Toda la información del proyecto vive en un archivo: **`package.json`**.

---

## 2.1 Crear un proyecto: `npm init`

```bash
mkdir prueba
cd prueba
npm init -y      # -y = "sí a todo", crea package.json con valores por defecto
```

Esto genera un `package.json` mínimo. Después instalas lo que necesites:

```bash
npm install dayjs                  # dependencia normal
npm install -D typescript          # dependencia de desarrollo (-D = --save-dev)
```

### Ejemplo REAL copiado (`node-practica/prueba/package.json`)

```json
{
  "name": "prueba",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
 
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs",
  "dependencies": {
    "dayjs": "^1.11.23"
  },
  "devDependencies": {
    "typescript": "^7.0.2"
  }
}
```

**Desglose campo a campo:**

| Campo | Qué significa |
|-------|---------------|
| `name` | Nombre del proyecto (minúsculas, sin espacios) |
| `version` | Versión de tu proyecto (`1.0.0`) |
| `description`, `keywords`, `author` | Info descriptiva (opcional en proyectos privados) |
| `license` | Licencia. `ISC` es la que pone npm por defecto |
| `main` | Archivo de entrada si otro proyecto importara el tuyo como librería |
| `scripts` | **Comandos** que ejecutas con `npm run <nombre>` |
| `type` | `"commonjs"` (usa `require`) o `"module"` (usa `import`) → tema 03 |
| `dependencies` | Librerías que tu app **necesita para funcionar** |
| `devDependencies` | Librerías que solo necesitas **mientras desarrollas** |

---

## 2.2 Usar una dependencia instalada

Una vez instalada con `npm install dayjs`, la usas en tu código.

### Ejemplo REAL copiado (`node-practica/prueba/index.js`)

```js
const dayjs = require("dayjs");

console.log("Hoy es:", dayjs().format("DD/MM/YYYY HH:mm:ss"));
```

```bash
node index.js
# Hoy es: 09/11/2026 10:35:12
```

> 📌 Fíjate: `require("dayjs")` **sin** `./` delante. Cuando el nombre no empieza por
> `./` ni `../`, Node lo busca en la carpeta **`node_modules`**.

```js
require("dayjs");      // 📦 paquete de node_modules
require("./utils");    // 📄 archivo TUYO (utils.js en la misma carpeta)
require("fs");         // 🟢 módulo nativo de Node
```

---

## 2.3 `scripts`: los comandos del proyecto

En `scripts` defines **atajos** para comandos largos. Se ejecutan con `npm run <nombre>`.

### Ejemplo REAL copiado (`api-rastro/api-rastro/package.json`)

```json
"scripts": {
  "dev": "tsx watch --env-file=.env src/index.ts",
  "lint": "eslint .",
  "typecheck": "tsc --noEmit"
},
```

| Comando que escribes | Lo que ejecuta realmente |
|----------------------|--------------------------|
| `npm run dev` | `tsx watch --env-file=.env src/index.ts` → arranca el servidor y lo reinicia al guardar |
| `npm run lint` | `eslint .` → revisa el estilo y errores del código |
| `npm run typecheck` | `tsc --noEmit` → comprueba los tipos de TypeScript sin generar archivos |

### Ejemplo REAL copiado (`ProyectoReproductor/package.json`)

```json
"scripts": {"dev":"tsx watch server/index.ts", "build":"tsc && node scripts/copy-client.mjs", "check":"tsc --noEmit",  "start":"node dist/server/index.js"},
```

| Script | Qué hace |
|--------|----------|
| `dev` | Desarrollo: ejecuta el `.ts` directamente y se reinicia al guardar |
| `build` | Compila TypeScript a JavaScript (`tsc`) **y después** (`&&`) copia archivos del cliente |
| `check` | Solo comprueba tipos |
| `start` | Producción: ejecuta el **JavaScript ya compilado** con `node` |

> 📌 `&&` = "si lo primero sale bien, ejecuta lo segundo". Si `tsc` falla, no se copia nada.

### Scripts especiales: `start` y `test`

```bash
npm run dev      # scripts normales → necesitan "run"
npm start        # "start" y "test" se pueden lanzar SIN "run"
npm test
```

### Ejemplo REAL: un script que lanza comandos de Docker

**Archivo:** `mongo-api-libros/02b-express-mongodb/02b-express-mongodb/package.json`
*(la contraseña se ha ocultado con `****`)*

```json
"scripts": {
  "dev": "tsx watch --env-file=.env src/index.ts",
  "typecheck": "tsc --noEmit",
  "lint": "eslint .",
  "seed": "docker cp mongo/seed.js mongodb-practica:/tmp/seed.js && docker exec mongodb-practica mongosh --quiet -u admin -p **** --authenticationDatabase admin /tmp/seed.js"
},
```

> 📌 Un script puede ejecutar **cualquier comando** de terminal, no solo Node. Aquí
> `npm run seed` copia un archivo al contenedor de MongoDB y lo ejecuta para llenar
> la base de datos con datos de prueba.
> ⚠️ Escribir contraseñas en `package.json` solo es aceptable para contraseñas de
> **desarrollo local**. Lo correcto es leerlas de variables de entorno (tema 05).

---

## 2.4 `dependencies` vs. `devDependencies`

| | `dependencies` | `devDependencies` |
|---|----------------|-------------------|
| Se instala con | `npm install paquete` | `npm install -D paquete` |
| ¿Cuándo se usa? | Cuando la app **está funcionando** | Solo mientras **programas / compilas** |
| Ejemplos | `express`, `mongodb`, `dotenv`, `dayjs` | `typescript`, `tsx`, `eslint`, `@types/...` |
| ¿Va a producción? | ✅ Sí | ❌ No hace falta (`npm install --omit=dev`) |

### Ejemplo REAL copiado (`ProyectoReproductor/package.json`)

```json
"dependencies": {"cookie-parser":"^1.4.7", "dotenv":"^16.4.7", "express":"^4.21.2", "sql.js":"^1.13.0"},
"devDependencies": {"@types/cookie-parser":"^1.4.8", "@types/express":"^5.0.0", "@types/node":"^22.10.5", "@types/sql.js":"^1.4.9", "tsx":"^4.19.2", "typescript":"^5.7.2"}
```

**Razonamiento:**
- `express`, `cookie-parser`, `dotenv`, `sql.js` → el servidor los usa **al ejecutarse** → `dependencies`.
- `typescript`, `tsx` → solo sirven para **compilar/ejecutar en desarrollo** → `devDependencies`.
- `@types/...` → solo dan **tipos** a TypeScript; desaparecen al compilar → `devDependencies`.

### Ejemplo REAL copiado (`API_Godines/api-express/api-books-claude/package.json`)

```json
{
  "name": "api-books",
  "version": "1.0.0",
  "main": "src/index.ts",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "typecheck": "tsc --noEmit"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": "",
  "dependencies": {
    "express": "^5.3.0"
  },
  "devDependencies": {
    "@types/express": "^5.0.6",
    "@types/node": "^26.6.6",
    "tsx": "^4.23.15",
    "typescript": "^7.0.2"
  }
}
```

> 📌 Es el `package.json` **mínimo** de una API con Express + TypeScript: **una**
> dependencia real (`express`) y las herramientas de desarrollo.

```bash
# ❌ Mal: TypeScript como dependencia normal (no hace falta en producción)
npm install typescript

# ✅ Bien
npm install -D typescript
```

---

## 2.5 Versiones: `^`, `~` y versión exacta

Las versiones siguen **SemVer** (*Semantic Versioning*): `MAYOR.MENOR.PARCHE`

```
   5 . 3 . 0
   │   │   └── PARCHE: arreglos de bugs (no rompe nada)
   │   └────── MENOR:  funciones nuevas (no rompe nada)
   └────────── MAYOR:  cambios que PUEDEN romper tu código
```

| Escrito | Significa | Ejemplo de tus proyectos |
|---------|-----------|--------------------------|
| `^5.3.0` | Acepta `5.x.x` ≥ 5.3.0 (no sube la MAYOR) | `"express": "^5.3.0"` |
| `~6.0` | Acepta `6.0.x` (solo parches) | `"typescript": "~6.0"` en `api-rastro` |
| `5.3.0` | Exactamente esa versión | — |

> 📌 Fíjate en un detalle real: `ProyectoReproductor` usa `"express": "^4.21.2"` y
> `api-rastro` usa `"express": "^5.3.0"`. Son **versiones mayores distintas** y
> tienen pequeñas diferencias. Con `^` nunca saltarías de la 4 a la 5 sin querer.

---

## 2.6 `node_modules` y `package-lock.json`

```
mi-proyecto/
├── node_modules/        ← 📦 el código de TODAS las librerías (¡miles de archivos!)
├── package.json         ← 📝 lo que TÚ pides ("quiero express ^5.3.0")
├── package-lock.json    ← 🔒 lo que se instaló EXACTAMENTE (5.3.0 y sus sub-dependencias)
└── src/
```

| Archivo/carpeta | ¿Se sube a Git? | Por qué |
|-----------------|-----------------|---------|
| `package.json` | ✅ Sí | Es la "lista de la compra" |
| `package-lock.json` | ✅ Sí | Garantiza que todos instalan las **mismas** versiones |
| `node_modules/` | ❌ **NO** | Pesa muchísimo y se regenera con `npm install` |

### Ejemplo REAL copiado (`api-rastro/api-rastro/.gitignore`)

```
node_modules/
.env
dist
```

### Ejemplo REAL copiado (`ProyectoReproductor/.gitignore`)

```
.env
node_modules/
dist/
data/
```

> 📌 En todos tus proyectos se ignoran: `node_modules` (se reinstala), `.env`
> (secretos), `dist` (se regenera compilando). Si clonas un proyecto, **lo primero**
> es `npm install`.

```bash
git clone https://github.com/HectorFM99/api-rastro.git
cd api-rastro
npm install          # lee package.json + package-lock.json y crea node_modules
npm run dev
```

---

## 2.7 Comandos de npm que más vas a usar

| Comando | Qué hace |
|---------|----------|
| `npm init -y` | Crea `package.json` |
| `npm install` (o `npm i`) | Instala **todo** lo que dice `package.json` |
| `npm install express` | Instala y añade a `dependencies` |
| `npm install -D tsx` | Instala y añade a `devDependencies` |
| `npm uninstall dayjs` | Desinstala y lo quita de `package.json` |
| `npm run dev` | Ejecuta el script `dev` |
| `npm start` / `npm test` | Ejecuta `start` / `test` (sin `run`) |
| `npm ci` | Instalación **limpia y exacta** desde `package-lock.json` (CI, servidores) |
| `npm outdated` | Muestra qué paquetes tienen versiones nuevas |
| `npx <herramienta>` | Ejecuta una herramienta sin instalarla globalmente (`npx tsc --init`) |

### `npx` vs. scripts

```bash
# Dentro de un script, npm ya busca en node_modules/.bin:
"typecheck": "tsc --noEmit"        # ✅ funciona aunque tsc no esté instalado globalmente

# En la terminal directamente, necesitas npx:
tsc --noEmit                        # ❌ 'tsc' no se reconoce (si no es global)
npx tsc --noEmit                    # ✅ usa el de node_modules
```

---

## 2.8 Campo `private`

### Ejemplo REAL (`ProyectoReproductor/package.json` y `mongo-api-libros/.../package.json`)

```json
"private": true,
```

> 📌 Evita que publiques el proyecto **por accidente** en el registro público de npm
> con `npm publish`. Para aplicaciones (no librerías) es buena práctica ponerlo.

---

## 2.9 Errores comunes

| Error | Causa | Solución |
|-------|-------|----------|
| `Cannot find module 'express'` | No has hecho `npm install` | `npm install` |
| `npm ERR! Missing script: "dev"` | El script no existe en ese `package.json` | Revisa `scripts` o estás en otra carpeta |
| `npm ERR! enoent ... package.json` | No estás en la carpeta del proyecto | `cd` a la carpeta que tiene `package.json` |
| Subiste `node_modules` a GitHub | Falta en `.gitignore` | Añádelo y `git rm -r --cached node_modules` |
| A tu compañero le falla y a ti no | Versiones distintas | Sube `package-lock.json` y usad `npm ci` |

---

## ✅ Resumen rápido

1. `npm init -y` crea `package.json`; `npm install` crea `node_modules`.
2. `scripts` = atajos → `npm run dev`, `npm run build`, `npm start`.
3. `dependencies` = lo que necesita la app al ejecutarse; `devDependencies` (`-D`) = herramientas de desarrollo.
4. `^5.3.0` acepta mejoras de la misma versión mayor; `~6.0` solo parches.
5. `node_modules` **nunca** a Git; `package-lock.json` **sí**.
6. `require("paquete")` busca en `node_modules`; `require("./archivo")` busca tus archivos.

👉 **Siguiente:** `03-modulos-commonjs-vs-es-modules.md`
