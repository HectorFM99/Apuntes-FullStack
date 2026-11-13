# 10 — 🗣️ PREGUNTAS DE ENTREVISTA CON RESPUESTAS

> Repaso final: **~70 preguntas** frecuentes con una **respuesta corta modelo**.
> 📌 No las memorices palabra por palabra: entiéndelas y **dilas con tus palabras, en voz
> alta**. Si puedes, añade siempre un *"por ejemplo, en mi proyecto…"*.

---

## 🙋 A. Sobre ti

**1. Háblame de ti.**
> Estructura **presente → pasado → futuro** (1 minuto):
> "Soy desarrollador FullStack junior. He terminado un bootcamp donde he trabajado con
> JavaScript y TypeScript, Node, Express y NestJS, y bases de datos PostgreSQL y
> MongoDB. Antes [tu experiencia previa y qué te aporta: trato con clientes, trabajo en
> equipo…]. Lo que más me gusta es el backend / construir APIs, y busco un equipo donde
> seguir aprendiendo y aportar desde el primer día."

**2. ¿Por qué quieres ser desarrollador?**
> Una razón **real y concreta** (qué te enganchó) + qué has hecho para conseguirlo
> (bootcamp, proyectos propios).

**3. ¿Por qué esta empresa?**
> Algo **específico** que hayas investigado: su producto, su stack, su forma de
> trabajar. ❌ "Porque buscáis gente" ✅ "Porque usáis NestJS y PostgreSQL, que es
> justo con lo que más he trabajado, y vuestro producto…"

**4. ¿Cuál es tu mayor punto fuerte?**
> Uno, con un ejemplo: "Soy constante: cuando algo no me funciona lo investigo hasta
> entenderlo; por ejemplo, cuando…"

**5. ¿Y tu mayor debilidad?**
> Una real pero no crítica + **qué haces para mejorarla**: "Al principio me costaba
> pedir ayuda y perdía tiempo; ahora me pongo un límite de tiempo y, si no avanzo,
> pregunto con lo que ya he probado."

**6. ¿Dónde te ves en 2-3 años?**
> "Siendo un desarrollador autónomo en el equipo, con más profundidad en backend /
> arquitectura, y ayudando a los nuevos que entren."

---

## 🌐 B. Web y HTTP

**7. ¿Qué pasa cuando escribes una URL en el navegador?**
> DNS → IP, conexión TCP + TLS, petición HTTP, el servidor procesa y responde con un
> status code y contenido, el navegador construye DOM + CSSOM, ejecuta JS y pinta.

**8. ¿Qué métodos HTTP conoces?**
> GET leer, POST crear, PUT reemplazar, PATCH modificar parcialmente, DELETE borrar.
> GET, PUT y DELETE son idempotentes; POST no.

**9. ¿Qué códigos de estado conoces?**
> 2xx éxito (200, 201, 204), 3xx redirección (301, 304), 4xx error del cliente (400,
> 401, 403, 404, 409), 5xx error del servidor (500, 503).

**10. ¿401 vs 403?**
> 401: no autenticado (no sé quién eres). 403: autenticado pero sin permiso.

**11. ¿Qué es CORS?**
> Mecanismo del navegador que bloquea leer respuestas de otro origen salvo que el
> servidor lo permita con cabeceras `Access-Control-Allow-*`. Se configura en el backend.

**12. ¿Cookies vs localStorage vs sessionStorage?**
> Cookies: pequeñas, se envían solas al servidor, pueden ser HttpOnly. localStorage:
> persiste, solo en el navegador. sessionStorage: igual pero se borra al cerrar la pestaña.

**13. ¿Qué significa que HTTP es stateless?**
> Cada petición es independiente; el servidor no recuerda las anteriores. Para mantener
> la sesión se usan cookies, sesiones o tokens.

---

## 🎨 C. HTML y CSS

**14. ¿Qué es el HTML semántico?**
> Usar etiquetas con significado (`header`, `nav`, `main`, `article`, `button`). Mejora
> accesibilidad, SEO y legibilidad.

**15. Explica el box model.**
> Contenido, padding, borde y margen. Con `box-sizing: border-box` el ancho incluye
> padding y borde.

**16. ¿Flexbox vs Grid?**
> Flex es 1D (fila o columna), para alinear componentes. Grid es 2D, para layouts.

**17. ¿Cómo centras un div?**
> `display: flex; justify-content: center; align-items: center;` en el padre (o
> `display: grid; place-items: center;`).

**18. ¿Cómo funciona la especificidad?**
> Inline > id > clase/atributo/pseudo-clase > etiqueta. Si empatan, gana la última.

**19. ¿Qué es mobile first?**
> Diseñar primero para móvil y ampliar con media queries `min-width`.

---

## ⚡ D. JavaScript

**20. ¿`var`, `let`, `const`?**
> `var`: ámbito de función, hoisting con `undefined`. `let`/`const`: ámbito de bloque,
> TDZ. `const` no se reasigna (pero un objeto `const` sí se puede modificar).

**21. ¿Qué es el hoisting?**
> JS mueve las declaraciones al inicio de su ámbito. Las `function` se pueden usar
> antes de declararlas; `var` vale `undefined`; `let`/`const` dan error (TDZ).

**22. ¿Qué es una closure?**
> Una función que recuerda las variables del ámbito donde se creó. Ej.: un contador
> con variable privada, o un debounce.

**23. ¿Cómo funciona `this`?**
> Depende de cómo se llame la función. Las arrow functions no tienen `this` propio.

**24. ¿`==` vs `===`?**
> `==` convierte tipos antes de comparar; `===` compara valor y tipo. Uso siempre `===`.

**25. ¿`null` vs `undefined`?**
> `undefined`: sin valor asignado (lo pone JS). `null`: vacío intencionado (lo pones tú).

**26. ¿Tipos primitivos vs por referencia?**
> Primitivos (string, number, boolean, null, undefined, symbol, bigint) se copian por
> valor; objetos y arrays, por referencia.

**27. ¿Qué es el event loop?**
> JS es monohilo: lo síncrono va al call stack; lo asíncrono se delega y su callback
> espera en una cola. Con el stack vacío se ejecutan primero las microtareas (promesas)
> y luego las macrotareas (setTimeout).

**28. ¿Qué es una promesa?**
> Un objeto que representa un valor futuro: pending, fulfilled o rejected. Se consume
> con `.then/.catch` o con `async/await`.

**29. ¿`Promise.all` vs `Promise.allSettled`?**
> `all` falla en cuanto una falla; `allSettled` espera a todas y te da el resultado de
> cada una.

**30. ¿`map` vs `forEach`?**
> `map` devuelve un array nuevo transformado; `forEach` no devuelve nada, es solo para
> efectos.

**31. ¿Qué es la inmutabilidad y por qué importa?**
> No modificar datos existentes sino crear copias nuevas (spread, `map`, `filter`). Evita
> efectos secundarios inesperados y es clave en React.

**32. ¿Qué son los prototipos?**
> Cada objeto enlaza con otro (su prototipo) del que hereda propiedades. Las clases ES6
> son azúcar sintáctico sobre esto.

**33. ¿Qué es la delegación de eventos?**
> Un único listener en el padre que aprovecha el bubbling y `event.target`. Funciona con
> elementos añadidos dinámicamente.

---

## 🔷 E. TypeScript

**34. ¿Por qué TypeScript?**
> Detecta errores antes de ejecutar, autocompletado, código autodocumentado y refactors
> seguros.

**35. ¿`interface` vs `type`?**
> Similares para objetos. `type` permite uniones y primitivos; `interface` se extiende
> y admite declaration merging.

**36. ¿`any` vs `unknown`?**
> `any` desactiva el tipado; `unknown` obliga a comprobar el tipo antes de usarlo.

**37. ¿Qué son los genéricos?**
> Tipos como parámetro para código reutilizable sin perder tipado: `Promise<Libro[]>`.

---

## 🖥️ F. Backend

**38. ¿Qué es Node.js?**
> Entorno de ejecución de JS en el servidor (V8), monohilo con I/O no bloqueante.

**39. ¿Qué es un middleware?**
> Función `(req, res, next)` entre la petición y la respuesta: parsear, autenticar,
> loguear, manejar errores.

**40. ¿Qué es una API REST?**
> API basada en recursos con URLs y métodos HTTP, status codes adecuados y sin estado.

**41. ¿Cómo estructuras un backend?**
> En capas: controller (HTTP), service (lógica de negocio), acceso a datos.

**42. ¿Qué es la inyección de dependencias?**
> Una clase recibe sus dependencias desde fuera en lugar de crearlas. Menos
> acoplamiento y tests más fáciles. NestJS la trae integrada.

**43. ¿Express vs NestJS?**
> Express: minimalista y libre. Nest: estructura, TypeScript, DI, módulos; mejor para
> proyectos grandes.

**44. ¿Cómo manejas los errores en una API?**
> Status codes correctos, formato de error consistente, un manejador centralizado, sin
> exponer el stack trace, y logueando en el servidor.

---

## 🗄️ G. Bases de datos

**45. ¿SQL vs NoSQL?**
> SQL: tablas, esquema fijo, relaciones y JOINs, ACID. NoSQL (Mongo): documentos
> flexibles, escalado horizontal. Elijo según cómo sean los datos.

**46. ¿INNER JOIN vs LEFT JOIN?**
> INNER: solo coincidencias. LEFT: todas las de la izquierda, con NULL si no coincide.

**47. ¿Qué es un índice?**
> Estructura que acelera las búsquedas a costa de espacio y escrituras algo más lentas.

**48. ¿Qué es una transacción? ¿ACID?**
> Conjunto de operaciones que se hacen todas o ninguna. Atomicidad, Consistencia,
> Aislamiento, Durabilidad.

**49. ¿Qué es la normalización?**
> Organizar tablas para no duplicar datos, relacionándolas con claves foráneas.

**50. ¿Embeber o referenciar en MongoDB?**
> Embebo datos pequeños que se leen juntos; referencio relaciones grandes o que se
> consultan por separado.

---

## 🔐 H. Seguridad

**51. ¿Cómo guardas las contraseñas?**
> Hash lento con salt (bcrypt, scrypt, argon2). Nunca texto plano ni MD5.

**52. ¿JWT vs sesiones?**
> Sesión: estado en el servidor, fácil de revocar. JWT: token firmado autocontenido,
> escala bien pero difícil de revocar. El payload del JWT no está cifrado.

**53. ¿XSS vs CSRF?**
> XSS: el atacante ejecuta su JS en tu web (se evita escapando, HttpOnly, CSP). CSRF:
> otra web envía peticiones a la tuya con tu sesión (se evita con SameSite y tokens).

**54. ¿Cómo evitas la inyección SQL?**
> Consultas parametrizadas u ORM, nunca concatenar el input del usuario.

**55. ¿Dónde guardas los secretos?**
> Variables de entorno en un `.env` ignorado por Git, con un `.env.example` sin valores.

---

## 🧰 I. Git, testing y buenas prácticas

**56. ¿Merge vs rebase?**
> Merge une con un commit de merge sin reescribir; rebase reaplica commits para un
> historial lineal. Rebase nunca en ramas compartidas.

**57. ¿Qué es una Pull Request?**
> Propuesta para integrar una rama, con revisión de código y CI antes del merge.

**58. ¿Qué tipos de tests hay?**
> Unitarios (aislados, con mocks), integración (varias piezas), E2E (flujo completo).

**59. ¿Qué es SOLID?**
> Responsabilidad única, abierto/cerrado, sustitución de Liskov, segregación de
> interfaces, inversión de dependencias.

**60. ¿Qué es Docker?**
> Empaqueta la app y sus dependencias en una imagen que se ejecuta como contenedor
> aislado igual en cualquier máquina. Compose levanta varios servicios juntos.

---

## 🧩 J. Preguntas de comportamiento (soft skills)

### ⭐ El método STAR

| Letra | Qué contar | Peso |
|-------|-----------|------|
| **S**ituación | Contexto breve: dónde, cuándo | 10% |
| **T**area | Cuál era tu responsabilidad / el reto | 10% |
| **A**cción | **Qué hiciste TÚ** (en primera persona, "yo") | 60% |
| **R**esultado | Qué salió (con datos si puedes) + qué aprendiste | 20% |

**61. "Háblame de un proyecto del que estés orgulloso."** *(Ejemplo con STAR)*

> **S:** "En el bootcamp construí una API REST de libros con NestJS y PostgreSQL."
> **T:** "Tenía que permitir listar con filtros, crear y actualizar libros, validando
> bien los datos."
> **A:** "La organicé en módulos con controller y service separados, usé inyección de
> dependencias para el pool de PostgreSQL, hice todas las consultas parametrizadas
> para evitar inyección SQL, validé la entrada con DTOs y `class-validator` con un
> `ValidationPipe` global, y levanté la base de datos con Docker Compose."
> **R:** "Quedó una API limpia y fácil de ampliar; por ejemplo, añadir un endpoint de
> conteo fue trivial. Aprendí lo útil que es separar responsabilidades, y lo siguiente
> que añadiría son tests unitarios del servicio."

> 📌 Otras opciones para contar:
> - `ProyectoReproductor`: app con login propio (registro, contraseñas con scrypt y
>   sal, sesiones con cookie HttpOnly), favoritos e historial, y consumo de una API
>   externa (Jamendo) con manejo de errores y timeouts.
> - `api-rastro`: API tipo mercado de segunda mano con Express 5, TypeScript y MongoDB,
>   modelando anuncios y ofertas, con CI en GitHub Actions (ESLint + typecheck).
> - `Proyecto-SQL`: diseño de una base de datos de gimnasio con relaciones 1:1, 1:N y
>   N:M y restricciones (`CHECK`, `UNIQUE`, FKs).

**62. Cuéntame un bug o problema difícil y cómo lo resolviste.**
> STAR: qué fallaba → cómo lo **acotaste** (logs, reproducirlo, aislarlo, leer el
> error, documentación) → la solución → qué aprendiste para la próxima.

**63. Cuéntame un conflicto con un compañero.**
> Céntrate en **cómo lo hablasteis** y el acuerdo: escuchar, argumentar con datos,
> buscar lo mejor para el proyecto. ❌ Nunca critiques al compañero.

**64. ¿Qué haces cuando no sabes algo?**
> "Primero intento entenderlo: documentación oficial, buscar el error, probar en
> pequeño. Me pongo un tiempo límite y, si no avanzo, pregunto explicando lo que ya
> he intentado."

**65. ¿Cómo recibes el feedback / las críticas en code review?**
> "Bien: es la forma más rápida de aprender. Si no entiendo un comentario pregunto el
> porqué, y aplico lo aprendido en los siguientes PRs."

**66. ¿Cómo te mantienes al día?**
> Algo concreto: documentación, un curso, un proyecto personal que estés haciendo, un
> canal o newsletter, retos de código.

**67. Cuéntame un error que hayas cometido.**
> Uno real, asumido sin excusas, **cómo lo arreglaste** y qué cambiaste para que no se
> repita.

**68. ¿Cómo organizas tu trabajo / priorizas?**
> Dividir tareas grandes en pequeñas, priorizar por impacto/urgencia, comunicar pronto
> si algo se retrasa.

**69. ¿Cuál es tu expectativa salarial?**
> Infórmate antes del rango para un junior en tu ciudad/modalidad. Da un **rango**
> razonable o pregunta primero el rango de la posición.

---

## ❓ K. Preguntas que TÚ debes hacer a la empresa

> 📌 Hacer preguntas demuestra interés. **Lleva 3-4 preparadas** y elige según cómo vaya.

| Tema | Pregunta |
|------|----------|
| 👥 Equipo | "¿Cómo es el equipo con el que trabajaría? ¿Cuántas personas y qué perfiles?" |
| 🧑‍🏫 Onboarding | "¿Cómo son las primeras semanas de alguien junior? ¿Tendría un mentor?" |
| 🔄 Forma de trabajar | "¿Cómo es vuestro flujo: sprints, code reviews, cómo se despliega?" |
| 🧪 Calidad | "¿Qué peso tienen los tests y el CI en vuestro día a día?" |
| 🛠️ Stack | "¿Qué tecnologías usáis y hay planes de cambiar algo?" |
| 📈 Crecimiento | "¿Cómo se mide el progreso de un junior? ¿Qué esperáis de mí a los 6 meses?" |
| 🎯 Reto | "¿Cuál es el mayor reto técnico del equipo ahora mismo?" |
| ⏭️ Proceso | "¿Cuáles son los siguientes pasos del proceso?" |

```
❌ Primera pregunta: "¿Cuántos días de vacaciones hay?"
✅ Pregunta primero por el trabajo y el equipo; las condiciones, al final o con RR. HH.
```

---

## 🎯 Cómo responderlo en la entrevista

| Situación | Qué hacer |
|-----------|-----------|
| No sabes la respuesta | "No lo he usado todavía. Lo que sé es… y lo buscaría en…" — **nunca inventes** |
| Sabes un poco | Di lo que sabes con seguridad y reconoce el límite |
| Te piden profundizar | Pon un ejemplo de código o de tu proyecto |
| Respuesta demasiado larga | Respuesta corta primero, y "¿quieres que entre en detalle?" |
| Te equivocas | "Perdona, me corrijo:…" — corregirse es buena señal |

> 📌 **Fórmula para cualquier pregunta técnica:**
> **1. Definición corta** → **2. Para qué sirve / ejemplo** → **3. Dónde lo he usado yo.**
