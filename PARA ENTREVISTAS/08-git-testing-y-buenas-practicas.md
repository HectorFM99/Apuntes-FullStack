# 08 — 🧰 GIT, TESTING Y BUENAS PRÁCTICAS

> Esto es lo que demuestra que **sabes trabajar en un equipo real**, no solo programar
> sola/o. A un junior se le perdona no saber un algoritmo; no se le perdona romper `main`.

---

## 8.1 Git: conceptos básicos

| Concepto | Qué es |
|----------|--------|
| **Repositorio** | Carpeta con historial de cambios (`.git`) |
| **Commit** | "Foto" de los cambios con un mensaje |
| **Branch (rama)** | Línea de trabajo independiente |
| **Working dir → staging → repo** | `git add` mueve al staging; `git commit` lo guarda |
| **Remote** | Copia en un servidor (GitHub): `origin` |
| **Git vs GitHub** | Git = la herramienta de control de versiones. GitHub = plataforma que aloja repos + PRs, issues, Actions |

```bash
git status                     # qué ha cambiado
git add archivo.ts             # al staging
git commit -m "feat: añade endpoint de ofertas"
git pull                       # = fetch + merge
git push origin mi-rama
git log --oneline --graph      # historial
git checkout -b feat/login     # crear rama y cambiar (o: git switch -c feat/login)
git stash                      # guardar cambios sin commitear para luego
```

| Comando | Para qué |
|---------|----------|
| `git fetch` | Descarga cambios del remoto **sin** tocar tu rama |
| `git pull` | `fetch` + integra en tu rama |
| `git revert <commit>` | Crea un commit que **deshace** otro (seguro en ramas compartidas) |
| `git reset` | Mueve la rama hacia atrás (⚠️ reescribe historial: solo en local) |

---

## 8.2 Flujo de trabajo con ramas y PRs ⭐

```
main ──●────────────────●──────────▶  (siempre desplegable)
        \              /
         ●──●──●──●──●   feat/ofertas  → Pull Request → revisión → merge
```

1. Actualizas `main`: `git pull`.
2. Creas rama: `git switch -c feat/ofertas`.
3. Commits pequeños y con mensajes claros.
4. `git push` y abres una **Pull Request** en GitHub.
5. El **CI** pasa (lint, tipos, tests) y un compañero hace **code review**.
6. Se hace merge a `main` y se borra la rama.

| Estrategia | En una frase |
|------------|--------------|
| **GitHub Flow** | `main` + ramas cortas de feature con PR. La más común y sencilla |
| **Git Flow** | `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`. Más pesada |
| **Trunk-based** | Todos integran en `main` muy a menudo, con ramas de horas |

### Conventional Commits

```
feat: añade filtro por autor
fix: corrige 404 al buscar libro inexistente
refactor: extrae la consulta al servicio
docs: actualiza README
test: añade tests del BooksService
chore: actualiza dependencias
```

```
❌ "cambios"   ❌ "arreglado"   ❌ "asdf"
✅ "fix: devuelve 409 si el email ya está registrado"
```

---

## 8.3 Merge vs rebase ⭐

```
Merge:                          Rebase:
main  A──B──────M                main  A──B
       \       /                             \
feat    C──D──                   feat         C'──D'   (tus commits "recolocados")
```

| | `git merge` | `git rebase` |
|---|---|---|
| Qué hace | Une dos ramas con un **commit de merge** | **Reaplica** tus commits encima de otra rama |
| Historial | Fiel a lo que pasó, con bifurcaciones | **Lineal** y limpio |
| ¿Reescribe historial? | ❌ No | ✅ Sí (cambian los hashes) |
| Peligro | Ninguno | ⚠️ **Nunca rebase de ramas compartidas** ya publicadas |

> 📌 Regla de oro: *rebase* para poner al día **tu** rama local; *merge* para integrar en
> ramas compartidas.

### Conflictos

```
<<<<<<< HEAD
const limite = 20;
=======
const limite = 50;
>>>>>>> feat/paginacion
```

1. Abre el archivo, decide qué código queda (o combina).
2. Borra las marcas `<<<<<<<`, `=======`, `>>>>>>>`.
3. `git add` + `git commit` (o `git rebase --continue`).
4. **Prueba que todo funciona.**

---

## 8.4 Testing ⭐

### La pirámide de tests

```
        /\        E2E         → pocos, lentos, prueban la app entera como un usuario
       /  \
      /----\      Integración → varias piezas juntas (endpoint + BD)
     /      \
    /--------\    Unitarios   → muchos, rápidos, una función/clase aislada
```

| Tipo | Qué prueba | Herramientas |
|------|-----------|--------------|
| **Unitario** | Una función o clase **aislada** (dependencias simuladas con *mocks*) | Jest, Vitest |
| **Integración** | Varias piezas juntas (ruta + servicio + BD de test) | Jest + Supertest |
| **E2E** | Flujo completo como un usuario real | Playwright, Cypress, Supertest |

```ts
// Unitario (Jest/Vitest) — patrón AAA: Arrange, Act, Assert
import { esPalindromo } from "./utils";

describe("esPalindromo", () => {
  it("detecta un palíndromo ignorando mayúsculas y espacios", () => {
    // Arrange
    const texto = "Anita lava la tina";
    // Act
    const resultado = esPalindromo(texto);
    // Assert
    expect(resultado).toBe(true);
  });
});
```

```ts
// Integración/E2E de una API con Supertest
it("GET /api/books/999 devuelve 404", async () => {
  await request(app.getHttpServer()).get("/api/books/999").expect(404);
});
```

> 📌 `api-books-nest` trae de serie `src/app.controller.spec.ts` (unitario) y
> `test/app.e2e-spec.ts` (e2e) que genera el CLI de Nest. Es buen punto de partida
> para decir "conozco la estructura y lo siguiente que haría es añadir tests al
> `BooksService` con un mock del `Pool`".

| Concepto | Qué es |
|----------|--------|
| **Mock** | Objeto falso que sustituye una dependencia (BD, API externa) |
| **Cobertura** | % del código que ejecutan los tests (útil, pero 100% ≠ sin bugs) |
| **TDD** | Escribir el test **antes** que el código: rojo → verde → refactor |

---

## 8.5 Principios: SOLID, DRY, KISS, YAGNI

### SOLID ⭐

| Letra | Principio | En cristiano | Ejemplo |
|-------|-----------|--------------|---------|
| **S** | Single Responsibility | Una clase/función = **una** responsabilidad | Controller solo HTTP, service solo lógica |
| **O** | Open/Closed | Abierto a **extender**, cerrado a **modificar** | Añadir un método de pago nuevo sin tocar los existentes |
| **L** | Liskov Substitution | Una subclase debe poder usarse donde se usa el padre sin romper nada | Un `Pinguino extends Pajaro` que lanza error en `volar()` ❌ |
| **I** | Interface Segregation | Interfaces pequeñas y específicas, no una gigante | `Leible`, `Escribible` en vez de `TodoEnUno` |
| **D** | Dependency Inversion | Depender de **abstracciones**, no de clases concretas | Inyección de dependencias en Nest |

### Otros

| Principio | Significa |
|-----------|-----------|
| **DRY** (*Don't Repeat Yourself*) | No dupliques lógica: extráela a una función |
| **KISS** (*Keep It Simple, Stupid*) | La solución más simple que funcione |
| **YAGNI** (*You Aren't Gonna Need It*) | No programes cosas "por si acaso" |
| **Separation of concerns** | Cada parte del código se ocupa de una cosa |

---

## 8.6 Clean code

```js
// ❌
function p(d) {
  let r = 0;
  for (let i = 0; i < d.length; i++) if (d[i].a) r += d[i].p * 1.21;
  return r;
}

// ✅
const IVA = 1.21;

function calcularTotalConIva(productos) {
  return productos
    .filter(producto => producto.activo)
    .reduce((total, producto) => total + producto.precio * IVA, 0);
}
```

| Regla | |
|-------|---|
| Nombres **descriptivos** | `usuariosActivos`, no `arr2` |
| Funciones **pequeñas** que hacen una cosa | Si necesitas "y" para describirla, divídela |
| Sin **números mágicos** | Constantes con nombre (`IVA`, `MAX_PAGINAS`) |
| *Early return* | Menos `if` anidados |
| Comentarios: el **porqué**, no el qué | El código ya dice el qué |
| Formato automático y linter | Prettier + ESLint (los usas en tus proyectos) |

```js
// ❌ Anidado
function procesar(user) {
  if (user) {
    if (user.activo) {
      // ...
    }
  }
}

// ✅ Early return
function procesar(user) {
  if (!user || !user.activo) return;
  // ...
}
```

---

## 8.7 CI/CD

| Término | Qué es |
|---------|--------|
| **CI** (Integración Continua) | En cada push/PR se ejecutan **automáticamente** lint, tipos y tests |
| **CD** (Entrega/Despliegue Continuo) | Si todo pasa, se despliega automáticamente (o queda listo para desplegar) |

**Ejemplo real (`api-rastro/.github/workflows/ci.yml`)** — GitHub Actions:

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-node@v7
        with: { node-version: 24, cache: npm }
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
```

> 📌 Esto lo puedes contar en la entrevista: "mis APIs tienen un pipeline de CI con
> GitHub Actions que pasa ESLint y la comprobación de tipos en cada push y PR".

### CD en tus proyectos: Render 🆕

Render está conectado a GitHub: cada `git push` a `main` dispara **build** (`npm install
&& npm run build`) y **despliegue** automáticos. Si el build falla, sigue en marcha la
versión anterior.

```
git push → GitHub Actions (CI: lint + tipos) → Render (CD: build + start) → producción
```

> 📌 Frase para la entrevista: "Tengo CI con GitHub Actions y despliegue continuo con
> Render: lo que llega a `main` se despliega solo, con las credenciales en variables de
> entorno del panel" (más en `12`).

### Testing en el frontend 🆕

| Herramienta | Para qué |
|-------------|----------|
| **Vitest** | Tests unitarios en proyectos Vite (API igual que Jest) |
| **React Testing Library** | Testear componentes como un usuario: buscar por rol/texto, hacer clic, comprobar lo que se ve |
| **Playwright** / Cypress | E2E en un navegador real |

> 💡 Ejemplo de test del `Counter` de `Vite/academia` en `../REACT/09`.

---

## 8.8 Docker básico ⭐

| Concepto | Qué es |
|----------|--------|
| **Imagen** | Plantilla de solo lectura (SO + dependencias + app). Ej: `postgres:17` |
| **Contenedor** | Una imagen **en ejecución** (proceso aislado) |
| **Dockerfile** | Receta para construir **tu** imagen |
| **Volumen** | Almacenamiento que **persiste** aunque borres el contenedor |
| **Docker Compose** | Archivo YAML para levantar **varios servicios** juntos (API + BD) |

**Docker vs máquina virtual:** el contenedor comparte el kernel del sistema anfitrión →
mucho **más ligero** y rápido de arrancar que una VM, que virtualiza un SO entero.

**Ejemplo (basado en el `compose.yaml` de `api-books-nest`):**

```yaml
services:
  postgres:
    image: postgres:17
    environment:
      POSTGRES_DB: academia
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "5432:5432"            # puerto_tu_PC:puerto_contenedor
    volumes:
      - postgres_data:/var/lib/postgresql/data   # los datos sobreviven
    restart: unless-stopped

volumes:
  postgres_data:
```

```bash
docker compose up -d       # levantar en segundo plano
docker compose ps          # ver qué está corriendo
docker compose logs -f     # ver logs
docker compose down        # parar (los datos del volumen se mantienen)
docker compose down -v     # parar Y borrar volúmenes (⚠️ se pierden datos)
```

```dockerfile
# Dockerfile mínimo para una API Node
FROM node:24-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

> 📌 ¿Por qué `COPY package*.json` antes que el resto? Por la **caché de capas**: si no
> cambian las dependencias, Docker no vuelve a ejecutar `npm ci`.

---

## 🎯 Cómo responderlo en la entrevista

**¿Cómo trabajas con Git en equipo?**
> "Parto de `main` actualizado, creo una rama por funcionalidad, hago commits pequeños
> con mensajes claros siguiendo Conventional Commits y abro una Pull Request. El CI
> pasa lint y tipos, un compañero la revisa y luego se hace merge."

**¿Merge o rebase?**
> "Merge une las ramas con un commit de merge y conserva el historial tal cual. Rebase
> reaplica mis commits sobre la otra rama y deja un historial lineal, pero reescribe
> commits, así que solo lo uso en mis ramas locales, nunca en ramas compartidas."

**¿Qué tipos de tests conoces?**
> "Unitarios, que prueban una función aislada con mocks; de integración, que prueban
> varias piezas juntas, como un endpoint con la base de datos; y E2E, que prueban el
> flujo completo como un usuario. Siguiendo la pirámide, muchos unitarios y pocos E2E.
> He usado la estructura de Jest que trae NestJS."

**¿Qué es SOLID?**
> "Cinco principios de diseño. El que más aplico es la responsabilidad única: en mis
> APIs el controller solo gestiona HTTP y el servicio la lógica. Y la inversión de
> dependencias, con la inyección de dependencias de NestJS."

**¿Qué es CI/CD?**
> "CI es ejecutar automáticamente lint, tipos y tests en cada push o PR para detectar
> errores pronto. CD es desplegar automáticamente si todo pasa. En mis APIs tengo un
> workflow de GitHub Actions que hace `npm ci`, ESLint y `tsc --noEmit`."

**¿Qué es Docker?**
> "Una herramienta para empaquetar una aplicación con todas sus dependencias en una
> imagen y ejecutarla como contenedor aislado, igual en cualquier máquina. Con Docker
> Compose levanto varios servicios; en el bootcamp lo usé para levantar PostgreSQL y
> MongoDB con un volumen para que los datos persistan."
