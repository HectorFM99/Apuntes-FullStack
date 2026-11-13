# 07 — 🔐 SEGURIDAD Y AUTENTICACIÓN

> Las preguntas de seguridad suelen ser cortas, pero **fallarlas preocupa mucho** a
> quien te entrevista. No te piden ser experto: te piden **no cometer los errores
> clásicos**.

---

## 7.1 Autenticación vs autorización

| | Autenticación (AuthN) | Autorización (AuthZ) |
|---|---|---|
| Pregunta | **¿Quién eres?** | **¿Qué puedes hacer?** |
| Ejemplo | Login con email y contraseña | Solo el admin puede borrar |
| Error HTTP | `401 Unauthorized` | `403 Forbidden` |

---

## 7.2 Contraseñas: hash, nunca en texto plano ⭐

| Concepto | Qué es |
|----------|--------|
| **Cifrado** | Reversible con una clave (sirve para datos que necesitas leer luego) |
| **Hash** | **Irreversible**: de la contraseña sale una "huella" y no se puede volver atrás |
| **Salt** | Valor aleatorio que se añade a cada contraseña antes del hash → dos usuarios con la misma contraseña tienen hashes distintos (y frena las *rainbow tables*) |

```ts
// ❌ Nunca
users.insert({ email, password });                 // texto plano
users.insert({ email, password: md5(password) });  // MD5/SHA-1: demasiado RÁPIDOS

// ✅ Algoritmos LENTOS a propósito: bcrypt, scrypt o argon2
import bcrypt from "bcrypt";
const hash = await bcrypt.hash(password, 10);       // 10 = coste (salt rounds)
const ok = await bcrypt.compare(passwordIntroducida, hash);
```

> 📌 "Lento a propósito" significa que a un atacante con la base de datos robada le
> cuesta muchísimo probar millones de contraseñas.

**Ejemplo real (`ProyectoReproductor/server/index.ts`):** usas `scryptSync` de
`node:crypto` con un **salt aleatorio** por usuario (`randomBytes(16)`), guardas
`salt:hash`, y al comparar usas `timingSafeEqual` (comparación en tiempo constante, que
evita ataques de temporización). Además, el login responde lo mismo si falla el email o
la contraseña ("Correo o contraseña incorrectos") → no revela qué emails existen. 👌

---

## 7.3 Sesiones vs JWT ⭐

### Sesión (stateful)

```
1. Login OK → el servidor crea una sesión (en BD/memoria) con un id aleatorio
2. Envía el id en una cookie HttpOnly
3. Cada petición trae la cookie → el servidor busca la sesión → sabe quién eres
4. Logout → se borra la sesión del servidor
```

### JWT — JSON Web Token (stateless)

```
1. Login OK → el servidor crea un TOKEN firmado con un secreto
2. El cliente lo guarda y lo envía:  Authorization: Bearer <token>
3. El servidor verifica la FIRMA (no consulta la BD) → lee el userId del token
```

Un JWT tiene 3 partes en Base64URL separadas por puntos: `header.payload.signature`

```ts
import jwt from "jsonwebtoken";
const token = jwt.sign({ sub: user.id, role: "user" }, process.env.JWT_SECRET, { expiresIn: "15m" });
const payload = jwt.verify(token, process.env.JWT_SECRET); // lanza error si es falso o caducado
```

> ⚠️ El payload de un JWT **NO está cifrado**, solo **firmado**: cualquiera puede leerlo
> (base64). **Nunca metas contraseñas ni datos sensibles** en él.

| | Sesiones | JWT |
|---|---|---|
| Dónde está el estado | Servidor | En el propio token (cliente) |
| Cerrar sesión / revocar | ✅ Fácil: borras la sesión | ❌ Difícil: es válido hasta que caduca (necesitas lista negra o tokens cortos + *refresh token*) |
| Escalar varios servidores | Necesitan compartir el almacén de sesiones | ✅ Cualquier servidor con el secreto lo verifica |
| Uso típico | Web tradicional, mismo dominio | APIs, apps móviles, microservicios |

**Ejemplo real:** `ProyectoReproductor` usa **sesiones**: token aleatorio de 32 bytes
guardado en una tabla `sessions` con caducidad, y enviado en una cookie
`httpOnly`, `sameSite: 'lax'` y `secure` en producción. El logout borra la sesión de la
BD.

### ¿Dónde guardo el token en el frontend?

| Sitio | Riesgo |
|-------|--------|
| `localStorage` | ❌ Lo puede leer cualquier JS → vulnerable a **XSS** |
| Cookie `HttpOnly` + `Secure` + `SameSite` | ✅ JS no la lee. Hay que cuidar **CSRF** (lo mitiga `SameSite`) |
| Memoria (variable) | ✅ Seguro, pero se pierde al recargar (se combina con *refresh token* en cookie) |

---

## 7.4 OWASP: los ataques que debes conocer

### 💉 Inyección SQL / NoSQL ⭐

```ts
// ❌ Concatenar input del usuario
db.query(`SELECT * FROM users WHERE email = '${email}'`);
// email = "' OR '1'='1"  → devuelve TODOS los usuarios

// ✅ Consultas parametrizadas / ORM
db.query("SELECT * FROM users WHERE email = $1", [email]);
```

```js
// NoSQL injection en Mongo
// body: { "email": { "$ne": null }, "password": { "$ne": null } }  😱
users.findOne({ email: req.body.email, password: req.body.password });

// ✅ Validar que son strings (DTOs, Zod...) o forzar String(...)
```

> 📌 `ProyectoReproductor` hace `String(req.body?.email ?? '')` antes de usarlo y usa
> parámetros `?` en todas las consultas. Ojo: en `/api/library/:kind` el nombre de la
> tabla va en la SQL, pero **solo después de comprobar que es `favorites` o `history`**
> (lista blanca). Esa es la forma correcta cuando no puedes parametrizar.

### 🦠 XSS (Cross-Site Scripting) ⭐

Un atacante consigue que **su JavaScript se ejecute en la página de otro usuario**
(p. ej. escribiendo `<script>` en un comentario).

```js
// ❌ Inserta HTML sin escapar
div.innerHTML = comentario.texto;

// ✅ Inserta como texto
div.textContent = comentario.texto;
```

| Defensa | |
|---------|---|
| Escapar/sanear lo que se pinta | `textContent`, frameworks como React escapan por defecto |
| Cookies `HttpOnly` | Aunque haya XSS, no pueden robar la cookie de sesión |
| **CSP** (*Content Security Policy*) | Cabecera que limita de dónde se puede cargar/ejecutar JS |

### 🎣 CSRF (Cross-Site Request Forgery)

Una web maliciosa hace que **tu navegador** envíe una petición a otra web donde estás
logueado, aprovechando que **las cookies se envían solas**.

```html
<!-- en web-malvada.com -->
<form action="https://banco.com/transferir" method="POST">
  <input name="a" value="atacante"><input name="cantidad" value="1000">
</form>
<script>document.forms[0].submit()</script>
```

| Defensa |
|---------|
| Cookies con `SameSite=Lax` o `Strict` |
| Token anti-CSRF en formularios |
| No usar `GET` para acciones que cambian datos |
| Comprobar cabecera `Origin` |

> 📌 Diferencia clave: **XSS** = el atacante ejecuta código **en** tu web.
> **CSRF** = el atacante hace peticiones **a** tu web desde otra, usando tu sesión.

### Otros

| Riesgo | Defensa |
|--------|---------|
| Fuerza bruta en el login | *Rate limiting* (`express-rate-limit`), bloqueo temporal |
| Exponer datos sensibles | No devolver el `password_hash`, usar `Omit`/DTOs de salida |
| Dependencias vulnerables | `npm audit`, Dependabot |
| Control de acceso roto | Comprobar en el backend que el recurso es **del usuario** (`WHERE user_id = ?`) |
| Mensajes de error detallados | No enviar *stack traces* al cliente |
| Cabeceras inseguras | `helmet` en Express |
| Payloads enormes | Limitar el body: `express.json({ limit: '32kb' })` (como en `ProyectoReproductor`) |

---

## 7.5 HTTPS

HTTP + **TLS**: cifra el tráfico entre el navegador y el servidor.

| Protege de… | Cómo |
|-------------|------|
| Que alguien **lea** los datos (contraseñas, cookies) en una wifi pública | Cifrado |
| Que alguien los **modifique** por el camino | Integridad |
| Hablar con un servidor **falso** | Certificado firmado por una autoridad (CA) |

> 📌 Por eso las cookies de sesión llevan `Secure`: solo viajan por HTTPS.

---

## 7.6 Variables de entorno y secretos ⭐

```bash
# .env  → NUNCA se sube al repositorio
DATABASE_URL=postgres://usuario:CONTRASEÑA@host:5432/db
JWT_SECRET=una-cadena-larga-y-aleatoria
```

```bash
# .env.example  → SÍ se sube: nombres de variables, sin valores reales
DATABASE_URL=
JWT_SECRET=
```

```gitignore
# .gitignore
.env
node_modules
```

```ts
// Uso
const uri = process.env.MONGODB_URI;
if (!uri) throw new Error("Falta MONGODB_URI en el archivo .env"); // ✅ falla pronto (api-rastro)
```

| Regla | |
|-------|---|
| ✅ `.env` en `.gitignore` y un `.env.example` de plantilla | `api-rastro`, `mongo-api-libros`, `api-books-nest` y `ProyectoReproductor` lo hacen así 👍 |
| ✅ Secretos distintos en desarrollo y producción | |
| ❌ Nunca secretos en el código ni en el frontend | El código del navegador lo ve cualquiera |
| 🚨 Si subes un secreto por error | **Rótalo** (cámbialo). Borrar el commit no basta: queda en el historial |

---

## 🎯 Cómo responderlo en la entrevista

**¿Cómo guardas las contraseñas?**
> "Nunca en texto plano ni con MD5. Uso un algoritmo de hash lento con salt, como
> bcrypt, scrypt o argon2, y al hacer login comparo el hash. En mi reproductor de
> música usé scrypt de `node:crypto` con un salt aleatorio por usuario y comparación
> en tiempo constante."

**¿JWT o sesiones?**
> "Con sesiones el estado está en el servidor y el cliente solo tiene un id en una
> cookie; es fácil revocar. Con JWT el token firmado lleva la información y el servidor
> solo verifica la firma, lo que escala bien pero es difícil de revocar, así que se usan
> tokens cortos con refresh token. En mi proyecto usé sesiones con cookie HttpOnly,
> SameSite y Secure."

**¿Qué es XSS y cómo lo evitas?**
> "Es cuando un atacante consigue ejecutar su JavaScript en la página de otro usuario,
> por ejemplo con un comentario que contiene un script. Se evita escapando lo que se
> pinta, usando `textContent` en vez de `innerHTML`, cookies HttpOnly y una CSP."

**¿Qué es CSRF?**
> "Es cuando otra web hace que tu navegador envíe una petición a una web en la que
> estás logueado, aprovechando que las cookies se envían solas. Se mitiga con
> `SameSite` en las cookies, tokens anti-CSRF y no usando GET para modificar datos."

**¿Cómo evitas la inyección SQL?**
> "Con consultas parametrizadas, nunca concatenando lo que manda el usuario. En mi API
> de NestJS con PostgreSQL todas las consultas usan `$1`, `$2` y el array de valores."

**¿Dónde guardas las claves y contraseñas de la base de datos?**
> "En variables de entorno, en un `.env` que está en el `.gitignore`, y subo un
> `.env.example` sin valores. Si se filtrara un secreto, lo rotaría."
