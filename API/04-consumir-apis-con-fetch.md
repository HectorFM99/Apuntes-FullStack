# 04 — CONSUMIR APIs CON `fetch` (GET)

> `fetch` es la función que trae el navegador (y Node 18+) para hacer **peticiones
> HTTP desde JavaScript**. Devuelve una **promesa**: la respuesta no llega al
> instante, así que hay que **esperarla** con `.then()` o con `await`.

---

## 4.1 Lo mínimo

```js
fetch("https://dog.ceo/api/breeds/image/random");   // ← hace un GET
```

Esto **lanza** la petición, pero no guarda nada. `fetch` devuelve una **promesa
de un objeto `Response`**. Hay que esperar **dos** veces:

```
fetch(url)  ──espera──▶  Response (status, headers...)
response.json()  ──espera──▶  los datos (objeto JS)
```

> 📌 ¿Por qué dos esperas? Primero llegan las **cabeceras** (con el código de
> estado) y después se descarga y se lee el **cuerpo**. Por eso `response.json()`
> **también** devuelve una promesa.

---

## 4.2 Con `.then()` (estilo promesas)

```js
fetch("https://dog.ceo/api/breeds/image/random")
  .then((response) => response.json())    // 1ª espera: leer el body como JSON
  .then((data) => {
    console.log(data.message);            // 2ª: ya tienes los datos
  })
  .catch((error) => {
    console.error("Algo falló:", error.message);
  });
```

### Ejemplo REAL copiado (`Async-hector/Advanced-JS-Asynchronous-code/index.js`, ejercicio 9)

Aquí `.then` se usa dentro de un `map` para convertir cada respuesta a JSON:

```js
// 9.
async function fetchGithubUsers(userNames) {
    const promises = userNames.map(name =>
        fetch(`https://api.github.com/users/${name}`).then(res => res.json())
    );
    const users = await Promise.all(promises);

    users.forEach(user => {
        console.log('Repo url:', user.html_url);
        console.log('Name:', user.name);
    });

    return users;
}
```

### Ejemplo REAL copiado (`ProyectoReproductor/public/app.js`)

El `.catch` de una cadena de promesas que carga las canciones destacadas:

```js
            renderFilteredResults();
            setStatus(tracks.length ? 'Mostrando canciones populares y destacadas.' : 'No hay canciones destacadas disponibles.');
        })
            .catch(err => {
            currentTracks = [];
            list.innerHTML = '';
            setStatus(err.message || 'No se pudieron cargar las canciones destacadas.', true);
        });
```

---

## 4.3 Con `async` / `await` (lo que más usas)

```js
async function obtenerPerro() {
  const response = await fetch("https://dog.ceo/api/breeds/image/random");
  const data = await response.json();
  return data.message;
}
```

**Reglas:**
- `await` solo funciona **dentro** de una función `async` (o en módulos).
- Una función `async` **siempre devuelve una promesa**. Para usar su valor:
  `const url = await obtenerPerro();` o `obtenerPerro().then(url => ...)`.

### Ejemplo REAL copiado (`Async-hector/Advanced-JS-Asynchronous-code/index.js`)

```js
// 1.
async function getAllBreeds() {
    const res = await fetch('https://dog.ceo/api/breeds/list/all');
    const data = await res.json();
    return Object.keys(data.message);
}

// 2.
async function getRandomDog() {
    const res = await fetch('https://dog.ceo/api/breeds/image/random');
    const data = await res.json();
    return data.message;
}
```

> 📌 **Lee siempre la forma del JSON** antes de programar. La Dog API devuelve
> `{ "message": ..., "status": "success" }`, por eso usas `data.message`. En
> `getAllBreeds`, `message` es un **objeto** `{ affenpinscher: [], african: [], ... }`
> y por eso haces `Object.keys(...)` para quedarte con los nombres.

### `.then` vs `async/await` — lo mismo escrito distinto

```js
// .then
function getRandomDog() {
  return fetch('https://dog.ceo/api/breeds/image/random')
    .then(res => res.json())
    .then(data => data.message);
}

// async/await
async function getRandomDog() {
  const res = await fetch('https://dog.ceo/api/breeds/image/random');
  const data = await res.json();
  return data.message;
}
```

---

## 4.4 URLs dinámicas con template strings

### Ejemplo REAL copiado (`Async-hector/.../index.js`)

```js
// 3.
async function getAllImagesByBreed(breed = 'komondor') {
    const res = await fetch(`https://dog.ceo/api/breed/${breed}/images`);
    const data = await res.json();
    return data.message;
}

// 5.
async function getGitHubUserProfile(username) {
    const res = await fetch(`https://api.github.com/users/${username}`);
    const profile = await res.json();
    return profile;
}
```

### Ejemplo REAL copiado (`Async-Hector-2/advanced-js-bikram-async-await/index.js`)

```js
//1
async function getRandomPokemon() {
    const id = Math.floor(Math.random() * 1025) + 1;
    const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`);
    const data = await response.json();
    return data;
}

//2
async function getImageAndName(pokemon) {

    let response = await fetch(`https://pokeapi.co/api/v2/pokemon/${pokemon}`)
    let data = await response.json();
    let name = data.name;
    let img = data.sprites.front_default;
    return { name, img }

}
```

> 📌 `data.sprites.front_default` → los JSON de las APIs reales suelen estar
> **muy anidados**. Ábrelo primero en el navegador (pega la URL en la barra) o en la
> pestaña **Network** de las DevTools para ver dónde está lo que buscas.

---

## 4.5 ⚠️ `fetch` NO falla con un 404 o un 500 → `response.ok`

Esta es **la** trampa de `fetch`. Tu `Prueba-Api/README.md` lo explica:

> Porque **`fetch` NO lanza error cuando el servidor responde con un 4xx o 5xx**:
> solo falla de forma "grave" si no hay red o se corta la conexión. Sin la
> comprobación, el código seguiría adelante y `response.json()` intentaría parsear
> el JSON de error de la API, lanzando un error confuso.

| Qué pasa | ¿`fetch` lanza error (va al `catch`)? | `response.ok` |
|----------|----------------------------------------|---------------|
| 200, 201, 204 | No | `true` |
| 404, 400, 401, 409 | **No** 😱 | `false` |
| 500, 502, 503 | **No** 😱 | `false` |
| Sin internet / DNS falla / CORS bloqueado | **Sí** (`TypeError: Failed to fetch`) | — |

`response.ok` es `true` solo si el estado está entre **200 y 299**.

### Ejemplo REAL copiado (`Prueba-Api/03-url-que-no-existe.js`)

```js
// PASO 4: URL que NO existe (el post 9999 no está).
// La API responde con estado 404 y response.ok es false,
// así que el if lanza el error y el catch lo muestra.

async function obtenerPublicacionInexistente() {
  try {
    const response = await fetch(
      "https://jsonplaceholder.typicode.com/posts/9999"
    );

    if (!response.ok) {
      throw new Error(`Error HTTP: ${response.status}`);
    }

    const data = await response.json();

    console.log(data.title);
    console.log(data.body);
  } catch (error) {
    console.error("No se pudo obtener la publicación:", error.message);
  }
}

obtenerPublicacionInexistente();
```

Por consola: `No se pudo obtener la publicación: Error HTTP: 404`

```js
// ❌ Sin comprobar ok: con un 404 data es {} y data.title es undefined (sin avisar)
const response = await fetch(url);
const data = await response.json();
console.log(data.title);   // undefined... ¿por qué? 🤔

// ✅ Comprobando ok: error claro
const response = await fetch(url);
if (!response.ok) throw new Error(`Error HTTP: ${response.status}`);
const data = await response.json();
```

> 📌 Tus ejercicios de `Async-hector` y `Async-Hector-2` **no** comprueban
> `response.ok` (los tests no lo pedían). Para código real, añade siempre el `if`.

---

## 4.6 Propiedades y métodos útiles de `Response`

| Propiedad / método | Qué da |
|--------------------|--------|
| `response.ok` | `true` si 200–299 |
| `response.status` | El número: `200`, `404`... |
| `response.statusText` | `"OK"`, `"Not Found"`... |
| `response.headers.get("content-type")` | Leer un header |
| `await response.json()` | Body → objeto JS |
| `await response.text()` | Body → string (HTML, texto plano) |
| `await response.blob()` | Body → archivo binario (imagen, audio) |

> ⚠️ El body **solo se puede leer UNA vez**. Si haces `response.json()` y después
> `response.text()`, el segundo falla (`body used already`).

---

## 4.7 Manejo de errores completo con `try/catch`

```js
async function cargarLibro(id) {
  try {
    const response = await fetch(`http://localhost:3000/api/books/${id}`);

    if (!response.ok) {
      // Intentamos leer el mensaje de error que manda TU API: { error: "..." }
      const body = await response.json().catch(() => ({}));
      throw new Error(body.error || `Error HTTP: ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.error("No se pudo cargar el libro:", error.message);
    return null;
  }
}

await cargarLibro("abc");   // "El id debe ser un número entero"   (400)
await cargarLibro(9999);    // "No existe ningún libro con id 9999" (404)
await cargarLibro(1);       // { id: 1, title: "Things Fall Apart", ... }
```

*(Ejemplo propio, pero los mensajes son exactamente los que devuelve tu
`books-routes.ts`.)*

### Ejemplo REAL copiado — la función `request` de tu reproductor (`ProyectoReproductor/public/app.js`)

Es la versión "profesional" de todo lo anterior, en **una sola función** que usan
todas las llamadas de la app:

```js
async function request(url, options) {
    const response = await fetch(url, {
        headers: { 'Content-Type': 'application/json', ...(options?.headers || {}) },
        ...options
    });
    if (response.status === 401) {
        window.location.href = 'index.html';
        throw new Error('Sesión expirada');
    }
    const contentType = response.headers.get('content-type') || '';
    if (!contentType.includes('application/json')) {
        await response.text().catch(() => '');
        throw new Error(response.ok ? 'Respuesta no válida del servidor.' : `Error del servidor (${response.status})`);
    }
    if (!response.ok) {
        const body = await response.json().catch(() => ({}));
        throw new Error(body.error || 'No se pudo completar la solicitud.');
    }
    return response.status === 204 ? undefined : response.json();
}
```

**Paso a paso:**

1. Añade `Content-Type: application/json` a todas las peticiones.
2. Si es **401** (sesión caducada) → manda al usuario a la página de login.
3. Si la respuesta **no es JSON** → error claro en vez de un `Unexpected token <`.
4. Si **no es ok** → lee el `{ error }` que manda tu servidor y lo lanza.
5. Si es **204** (sin body) → devuelve `undefined` (hacer `.json()` de un body
   vacío daría error).
6. Si todo va bien → devuelve el JSON.

Y usarla queda limpísimo:

```js
const tracks = await request(`/api/library/${kind}`);
```

> 📌 **Patrón recomendado:** en cualquier proyecto con frontend, crea **una**
> función `request`/`api` así y no repitas `fetch + ok + json` en cada sitio.

> ⚠️ **Detalle fino** de esa función: como `...options` va **después** de
> `headers`, si algún día pasas `options.headers`, ese objeto **sustituye** al
> combinado y se pierde el `Content-Type`. El orden más seguro sería
> `{ ...options, headers: { 'Content-Type': 'application/json', ...(options?.headers || {}) } }`.
> Hoy no te afecta porque ninguna llamada de tu app pasa `headers`.

---

## 4.8 Varias peticiones a la vez: `Promise.all`

```js
// ❌ En serie: la 2ª espera a que acabe la 1ª (más lento)
const usuario = await (await fetch(".../users/1")).json();
const post    = await (await fetch(".../posts/1")).json();

// ✅ En paralelo: las dos salen a la vez
const [resUsuario, resPost] = await Promise.all([
  fetch(".../users/1"),
  fetch(".../posts/1"),
]);
```

### Ejemplo REAL copiado (`Prueba-Api/prueba3.js`)

```js
// RETO OPCIONAL: Pide los datos de un usuario y muestra su nombre
// junto al título de una publicación.
// El usuario 1 (Leanne Graham) es el autor del post 1, así que
// los dos datos encajan. Usamos Promise.all para pedirlos a la vez.

async function mostrarAutorYPublicacion() {
  try {
    const [respuestaUsuario, respuestaPost] = await Promise.all([
      fetch("https://jsonplaceholder.typicode.com/users/1"),
      fetch("https://jsonplaceholder.typicode.com/posts/1")
    ]);

    if (!respuestaUsuario.ok) {
      throw new Error(`Error HTTP usuarios: ${respuestaUsuario.status}`);
    }

    if (!respuestaPost.ok) {
      throw new Error(`Error HTTP posts: ${respuestaPost.status}`);
    }

    const usuario = await respuestaUsuario.json();
    const publicacion = await respuestaPost.json();

    console.log(`Autor: ${usuario.name} (${usuario.email})`);
    console.log(`Título de su publicación: ${publicacion.title}`);
    console.log(`Contenido: ${publicacion.body}`);
  } catch (error) {
    console.error("No se pudo completar la petición:", error.message);
  }
}
```

> 📌 `Promise.all` falla **entera** si **una** promesa falla. Si quieres los
> resultados aunque alguna falle, usa `Promise.allSettled`.

---

## 4.9 Peticiones encadenadas (la 2ª necesita datos de la 1ª)

### Ejemplo REAL copiado (`Async-Hector-2/.../index.js`, ejercicio 8)

Rick and Morty te da los episodios como **URLs**; hay que hacer **otro fetch**:

```js
//8
async function getRandomCharacterInfo() {
    const character = await getRandomCharacter();

    const img = character.image;
    const name = character.name;
    const episodes = character.episode;


    const episodeResponse = await fetch(episodes[0]);
    const episodeData = await episodeResponse.json();

    return {
        img,
        name,
        episodes,
        firstEpisode: episodeData.name,
        dateEpisode: episodeData.air_date
    };
}
```

> 📌 Aquí **no** se puede usar `Promise.all`: no sabes la URL del episodio hasta
> que llega el personaje. **En serie cuando dependen, en paralelo cuando no.**

---

## 4.10 Pintar los datos en el DOM

### Ejemplo REAL copiado (`Async-hector/.../index.js`, ejercicios 7 y 8)

```js
// 7.
async function getAndPrintGitHubUserProfile(username) {
    const profile = await getGitHubUserProfile(username);

    const card = `<section>
    <img src="${profile.avatar_url}" alt="${profile.name}">
    <h1>${profile.name}</h1>
    <p>Public repos: ${profile.public_repos}</p>
</section>`;

    document.body.insertAdjacentHTML('beforeend', card);

    return card;
}

// 8.
window.addEventListener('DOMContentLoaded', () => {
    const searchInput = document.createElement('input');
    searchInput.type = 'text';
    searchInput.placeholder = 'GitHub username';

    const searchButton = document.createElement('button');
    searchButton.textContent = 'Buscar';

    searchButton.addEventListener('click', () => {
        const username = searchInput.value.trim();
        if (username) getAndPrintGitHubUserProfile(username);
    });

    document.body.appendChild(searchInput);
    document.body.appendChild(searchButton);
});
```

> ⚠️ **Seguridad (XSS):** meter datos de una API dentro de `innerHTML` /
> `insertAdjacentHTML` es peligroso si el dato puede contener HTML (p. ej. un nombre
> `<img src=x onerror=alert(1)>`). Tu reproductor lo evita con una función `escape`
> (copiada de `ProyectoReproductor/public/app.js`):
>
> ```js
> function escape(value) {
>     const element = document.createElement('span');
>     element.textContent = value;
>     return element.innerHTML;
> }
> ```
>
> O directamente usa `textContent` en vez de `innerHTML` cuando solo pintas texto.

---

## 4.11 Estados de carga en la interfaz

Una petición tarda. Avisa al usuario de los 3 estados: **cargando**, **éxito**,
**error**.

```js
async function buscar(q) {
  setStatus("Buscando...");                       // 1. cargando
  try {
    const tracks = await request(`/api/tracks/search?q=${encodeURIComponent(q)}`);
    setStatus(`${tracks.length} resultados`);     // 2. éxito
    return tracks;
  } catch (error) {
    setStatus(error.message, true);               // 3. error
    return [];
  }
}
```

*(Ejemplo propio inspirado en tu reproductor; `setStatus` es real y está en
`ProyectoReproductor/public/app.js`:)*

```js
function setStatus(message, error = false) {
    status.textContent = message;
    status.classList.toggle('error', error);
}
```

---

## 4.12 Resumen

```js
async function getJSON(url) {
  const response = await fetch(url);            // 1. pedir
  if (!response.ok) {                           // 2. ¿ha ido bien?
    throw new Error(`Error HTTP: ${response.status}`);
  }
  return response.json();                       // 3. leer el JSON
}

try {
  const data = await getJSON("https://jsonplaceholder.typicode.com/posts/1");
  console.log(data.title);                      // 4. usar
} catch (error) {
  console.error(error.message);                 // 5. avisar del error
}
```
