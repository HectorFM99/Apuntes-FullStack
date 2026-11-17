# 🎤 CARPETA PARA ENTREVISTAS — ÍNDICE DE ESTUDIO

Apuntes para preparar tu **primera entrevista como desarrollador FullStack junior**.
Aquí no se trata de saberlo *todo*, sino de saber **explicar con tus palabras** los
conceptos que más se preguntan y **relacionarlos con lo que tú has hecho** en el Bootcamp.

> ⚠️ Estos apuntes **no modifican** ninguno de tus proyectos. Solo los citan como
> ejemplos que puedes mencionar en la entrevista.

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a repasar |
|---|---------|-------------------|
| 1 | `01-fundamentos-web-y-http.md` | Qué pasa al escribir una URL, HTTP, métodos, status codes, cookies, sesiones, `localStorage`, CORS |
| 2 | `02-html-css-preguntas-clave.md` | Semántica, accesibilidad, box model, especificidad, flex vs grid, responsive, `position`, unidades |
| 3 | `03-javascript-preguntas-clave.md` | `var`/`let`/`const`, hoisting, scope, closures, `this`, coerción, prototipos, event loop, promesas, métodos de array |
| 4 | `04-typescript-preguntas-clave.md` | Tipos, `interface` vs `type`, genéricos, `any` vs `unknown`, utility types, narrowing |
| 5 | `05-backend-node-express-nest.md` | Node y su event loop, middleware, REST, arquitectura en capas, MVC, inyección de dependencias, validación, errores |
| 6 | `06-bases-de-datos-sql-vs-nosql.md` | SQL vs NoSQL, normalización, JOINs, índices, transacciones ACID, embebido vs referenciado |
| 7 | `07-seguridad-y-autenticacion.md` | Hash de contraseñas, JWT vs sesiones, XSS, CSRF, inyección SQL, HTTPS, `.env` |
| 8 | `08-git-testing-y-buenas-practicas.md` | Flujo Git, merge vs rebase, PRs, testing, SOLID, DRY/KISS, clean code, CI/CD, Docker |
| 9 | `09-algoritmia-y-ejercicios-de-live-coding.md` | Big O básico + 10 ejercicios típicos resueltos en JS |
| 10 | `10-preguntas-de-entrevista-con-respuestas.md` | ~95 preguntas frecuentes con respuesta modelo (incluye React y despliegue), soft skills, STAR y preguntas para la empresa |
| 11 | `11-react-y-frontend-preguntas-clave.md` | 🆕 React: declarativo, Virtual DOM, props vs estado, `useState`, `useEffect`, `key`, lifting state up, Context, hooks, formularios, consumir APIs, Router, CSR vs SSR |
| 12 | `12-despliegue-y-cloud-render-supabase-atlas.md` | 🆕 PaaS/DBaaS/BaaS, build vs start, variables de entorno, Render, Supabase (pooler, SSL, RLS), MongoDB Atlas (replica set, `+srv`, IP Access List), CORS en producción |

> 🆕 **Actualización:** añadidos `11` (carpeta `../REACT/`) y `12` (carpetas
> `../RENDER/`, `../SUPABASE/` y `../MONGODB-ATLAS/`), y las preguntas nuevas en
> `10` (secciones **L** y **M**), `06` (§6.10) y `08` (§8.7).

---

## 🚀 Orden de estudio recomendado

1. `01` → la base: cómo viaja una petición. Sale en casi **todas** las entrevistas.
2. `03` → **JavaScript** es lo que más te van a preguntar. Prioridad máxima.
3. `05` y `06` → backend y bases de datos (tu bootcamp tuvo mucho de esto: luce).
4. `11` → **React**: en cualquier oferta "FullStack" lo preguntan.
5. `12` → despliegue: haber subido algo a producción te diferencia de otros juniors.
6. `07` → seguridad: preguntas cortas pero eliminatorias.
7. `04` y `02` → TypeScript y HTML/CSS.
8. `08` → Git, testing y buenas prácticas (demuestra que sabes trabajar en equipo).
9. `09` → practica los ejercicios **escribiéndolos tú**, sin mirar.
10. `10` → repaso final en voz alta, como si fuera la entrevista.

---

## 🗓️ Plan de estudio (2 semanas)

| Día | Qué hacer | Tiempo |
|-----|-----------|--------|
| 1 | `01` Web y HTTP + dibujar en papel "qué pasa al escribir una URL" | 2 h |
| 2 | `03` JavaScript (1ª mitad: variables, scope, closures, `this`) | 2 h |
| 3 | `03` JavaScript (2ª mitad: event loop, promesas, arrays) | 2 h |
| 4 | `09` ejercicios 1–5 a mano + `04` TypeScript | 2 h |
| 5 | `05` Backend: repasa `api-rastro` y `api-books-nest` | 2 h |
| 6 | `06` Bases de datos: repasa `Proyecto-SQL` (JOINs) | 2 h |
| 7 | 🧘 Descanso / repaso ligero de lo que peor llevas | 1 h |
| 8 | `11` React: repasa `Vite/academia` y explica en voz alta props vs estado y `useEffect` | 2 h |
| 9 | `12` Despliegue (Render + Supabase + Atlas) + `07` Seguridad + `02` HTML/CSS + `08` Git y testing | 2,5 h |
| 10 | `09` ejercicios 6–10 a mano + rehacer los de la semana 1 | 2 h |
| 11 | Prepara **tu pitch** (presentación de 1 min) y 2 historias STAR (`10`) | 1,5 h |
| 12 | `10` preguntas técnicas en voz alta (grábate) | 2 h |
| 13 | Simulacro: que alguien te pregunte 10 al azar + 1 ejercicio de live coding | 1,5 h |
| 14 | Repaso suave, investiga la empresa, prepara tus preguntas. Duerme bien | 1 h |

> 📌 Mejor **2 horas cada día** que 10 horas el domingo. La memoria funciona por repetición.

---

## 🎯 Consejos para el día de la entrevista

### Antes
- ✅ Investiga la empresa: producto, tecnologías que usan (mira sus ofertas y su GitHub).
- ✅ Ten abierto tu **GitHub** y tus proyectos funcionando (o con capturas).
- ✅ Si es online: prueba cámara, micro, conexión y compartir pantalla **antes**.
- ✅ Ten agua, papel y boli (para dibujar o apuntar la pregunta).

### Durante
- 🗣️ **Piensa en voz alta.** En el live coding importa más cómo razonas que acertar a la primera.
- ❓ Si no entiendes la pregunta, **pide que la repitan o pon un ejemplo**. Es buena señal, no mala.
- 🙋 Si no sabes algo: *"No lo he usado todavía, pero entiendo que sirve para X; lo buscaría en la documentación"*. **Nunca te lo inventes.**
- 🧩 En los ejercicios: 1) entiende el problema, 2) pon ejemplos de entrada/salida, 3) solución sencilla que funcione, 4) luego mejora.
- 🔗 Relaciona cada respuesta con **algo que hayas hecho**: *"En mi API de libros con NestJS usé…"*.

### Después
- ✉️ Manda un mensaje corto de agradecimiento.
- 📝 Apunta las preguntas que te hicieron y las que fallaste → estúdialas para la siguiente.

---

## 💡 Lo que de verdad buscan en un junior

| Buscan | Cómo lo demuestras |
|--------|--------------------|
| Bases sólidas | Explicas bien lo básico (HTTP, JS, SQL) aunque no sepas lo avanzado |
| Ganas de aprender | Hablas de lo que estás aprendiendo ahora por tu cuenta |
| Saber trabajar en equipo | Git, PRs, pedir ayuda, aceptar feedback |
| Honestidad | Dices "no lo sé, pero lo averiguaría así…" |
| Comunicación | Explicas algo técnico de forma sencilla |
