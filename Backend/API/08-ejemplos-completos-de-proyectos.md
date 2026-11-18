# 08 — EJEMPLOS COMPLETOS DE PROYECTOS

> Archivos **enteros** (o funciones completas) copiados **tal cual** de tus proyectos
> del Bootcamp, ordenados de más sencillo a más completo. Léelos de arriba abajo
> después de haber estudiado los temas `01`–`07`.

---

## 8.1 `Prueba-Api/prueba.js` — GET con `response.ok`

**Temas:** `fetch`, `async/await`, `try/catch`, `response.ok`, URL que no existe (404).

```js

async function obtenerPublicacion() {
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

obtenerPublicacion();
```

> 📌 Pide el post `9999`, que no existe → `response.ok` es `false` → se lanza el
> error y el `catch` muestra `Error HTTP: 404`.

---

## 8.2 `Prueba-Api/02-obtener-publicacion-id2.js` — cambiar el recurso en la URL

**Temas:** el id del recurso va en la ruta (`/posts/2`).

```js
// PASO 2 y 3: Cambia el 1 por 2 en la URL.
// Ya no se trae el post 1 sino el post 2: cambian el title, el body y el id.
// (El código ya muestra el title y el body, como pedía el paso 3).

async function obtenerPublicacion() {
  try {
    const response = await fetch(
      "https://jsonplaceholder.typicode.com/posts/2"
    );

    if (!response.ok) {
      throw new Error(`Error HTTP: ${response.status}`);
    }

    const data = await response.json();

    console.log("Title:", data.title);
    console.log("Body:", data.body);
  } catch (error) {
    console.error("No se pudo obtener la publicación:", error.message);
  }
}

obtenerPublicacion();
```

---

## 8.3 `Prueba-Api/prueba2.js` — POST con JSON

**Temas:** `method`, `headers`, `body`, `JSON.stringify`, respuesta `201 Created`.

```js
// PASO 5: Ejecuta crearPublicacion() y localiza el objeto devuelto.
// Es un POST: la "página" del recurso lleva el body (title, body, userId).
// JSONPlaceholder responde 201 Created y devuelve el objeto creado
// con un id asignado (101). Ojo: es una API de prueba, NO guarda nada.

async function crearPublicacion() {
  try {
    const response = await fetch(
      "https://jsonplaceholder.typicode.com/posts",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          title: "Aprendiendo JavaScript",
          body: "Mi primera petición POST",
          userId: 1
        })
      }
    );

    if (!response.ok) {
      throw new Error(`Error HTTP: ${response.status}`);
    }

    const data = await response.json();
    console.log("Publicación creada:", data);
  } catch (error) {
    console.error("No se pudo crear la publicación:", error.message);
  }
}

crearPublicacion();
```

---

## 8.4 `Prueba-Api/prueba3.js` — dos peticiones en paralelo

**Temas:** `Promise.all`, destructuring de arrays, comprobar `ok` de cada respuesta.

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

mostrarAutorYPublicacion();
```

Y el `index.html` que lo carga (`Prueba-Api/index.html`):

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=<device-width>, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script src="prueba3.js"></script>

</body>
</html>
```

> 📌 El `README.md` de esta carpeta nombra los archivos `01-...` a `05-...`, pero en
> disco algunos se llaman `prueba.js`, `prueba2.js` y `prueba3.js`. Aquí se usan los
> nombres reales.

---

## 8.5 `Async-hector/Advanced-JS-Asynchronous-code/index.js` — Dog API + GitHub API

**Temas:** `async/await`, URLs dinámicas, leer JSON anidado, pintar en el DOM,
eventos, `Promise.all` con `map`.

```js
//RESUELVE TUS EJERCICIOS AQUI

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

// 3.
async function getAllImagesByBreed(breed = 'komondor') {
    const res = await fetch(`https://dog.ceo/api/breed/${breed}/images`);
    const data = await res.json();
    return data.message;
}

// 4.
async function getAllImagesByBreed2(breed) {
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

// 6.
async function printGithubUserProfile(username) {
    const profile = await getGitHubUserProfile(username);
    const user = { img: profile.avatar_url, name: profile.name };

    const section = document.createElement('section');
    section.innerHTML = `
        <img src="${user.img}" alt="${user.name}">
        <h1>${user.name}</h1>
    `;
    document.body.appendChild(section);

    return user;
}

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

> ⚠️ Recuerda: estas funciones **no** comprueban `response.ok`. Si pides un usuario
> de GitHub que no existe, `profile.name` será `undefined` en vez de dar un error
> claro (ver `04`, apartado 4.5).

---

## 8.6 `Async-Hector-2/advanced-js-bikram-async-await/index.js` — PokéAPI, Dog API y Rick and Morty

**Temas:** ids aleatorios en la URL, JSON muy anidado (`sprites.other['official-artwork']`),
peticiones **encadenadas** (personaje → primer episodio), combinar dos APIs.

```js
//DESARROLLA AQUI TUS SOLUCIONES

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

//3
async function printImageAndName(pokemon) {
    let name = pokemon;
    if (!name) {
        const randomPokemon = await getRandomPokemon();
        name = randomPokemon.name;
    }

    const data = await getImageAndName(name);

    return `<section>
    <img src="${data.img}" alt="${data.name}">
    <h1>${data.name}</h1>
    </section>`;
}


//4
async function getRandomDogImage() {
    const response = await fetch('https://dog.ceo/api/breeds/image/random');
    const data = await response.json();
    return data.message;
}

//5
async function getRandomPokemonImage() {
    const pokemon = await getRandomPokemon();
    const img = pokemon.sprites.front_default ||
        pokemon.sprites.other['official-artwork'].front_default;
    return img;
}

//6
async function printPugVsPikachu() {
    const dogResponse = await fetch('https://dog.ceo/api/breed/pug/images/random');
    const dogData = await dogResponse.json();
    const pugImg = dogData.message;

    const pikachu = await getImageAndName('pikachu');

    const battle = `<section>
    <h2>Pug VS Pikachu</h2>
    <div>
        <img src="${pugImg}" alt="Pug">
        <h1>Pug</h1>
    </div>
    <div>
        <img src="${pikachu.img}" alt="${pikachu.name}">
        <h1>${pikachu.name}</h1>
    </div>
</section>`;

    document.body.insertAdjacentHTML('beforeend', battle);
    return battle;
}

//7
async function getRandomCharacter() {
    const id = Math.floor(Math.random() * 826) + 1;
    const response = await fetch(`https://rickandmortyapi.com/api/character/${id}`);
    const data = await response.json();
    return data;
}

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

//9
async function printCharacterInfo() {
    const info = await getRandomCharacterInfo();

    const character = `<section>
    <img src="${info.img}" alt="${info.name}">
    <h1>${info.name}</h1>
    <p>Episodios en los que aparece: ${info.episodes.length}</p>
    <p>Primer episodio: ${info.firstEpisode} - ${info.dateEpisode}</p>
    </section>`;

    document.body.insertAdjacentHTML('beforeend', character);
    return character;
}

```

---

## 8.7 `ProyectoReproductor/public/login.js` — frontend que habla con tu propio backend

**Temas:** función `request` reutilizable, POST desde formularios con `FormData`,
leer `{ error }` del servidor, comprobar la sesión con `GET /api/auth/me`, 204.

```js
const $ = (selector) => document.querySelector(selector);
async function request(url, options) {
    const response = await fetch(url, {
        headers: { 'Content-Type': 'application/json', ...(options?.headers || {}) },
        ...options
    });
    if (!response.ok) {
        const body = await response.json().catch(() => ({}));
        throw new Error(body.error || 'No se pudo completar la solicitud.');
    }
    return response.status === 204 ? undefined : response.json();
}
function feedback(form, message = '', error = false) {
    const target = form.querySelector('.form-feedback');
    target.textContent = message;
    target.classList.toggle('error', error);
}
function selectTab(name) {
    for (const id of ['login', 'register']) {
        const selected = id === name;
        $(`#${id}-form`).toggleAttribute('hidden', !selected);
        const tab = $(`#${id}-tab`);
        tab.classList.toggle('is-active', selected);
        tab.setAttribute('aria-selected', String(selected));
    }
}
$('#login-tab').addEventListener('click', () => selectTab('login'));
$('#register-tab').addEventListener('click', () => selectTab('register'));
for (const [id, endpoint] of [['login-form', 'login'], ['register-form', 'register']]) {
    $(`#${id}`).addEventListener('submit', async (event) => {
        event.preventDefault();
        const form = event.currentTarget;
        const data = new FormData(form);
        feedback(form);
        try {
            await request(`/api/auth/${endpoint}`, {
                method: 'POST',
                body: JSON.stringify({ email: data.get('email'), password: data.get('password') })
            });
            window.location.href = 'app.html';
        }
        catch (error) {
            feedback(form, error.message, true);
        }
    });
}
(async () => {
    try {
        await request('/api/auth/me');
        window.location.href = 'app.html';
    }
    catch {
        // No active session, stay on landing/login page
    }
})();
export {};
```

---

## 8.8 `ProyectoReproductor/public/app.js` — las partes que llaman a la API

El archivo tiene ~550 líneas (reproductor, cola, filtros...). Aquí van **completas**
las funciones relacionadas con la API.

### La función `request` (líneas 11–30)

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

### Guardar / quitar favoritos y registrar el historial

```js
async function toggleFavorite(track, button) {
    try {
        if (favorites.has(track.id)) {
            await request(`/api/library/favorites/${track.id}`, { method: 'DELETE' });
            favorites.delete(track.id);
            button.classList.remove('is-saved');
            button.setAttribute('aria-label', `Guardar ${track.name}`);
            if (activeView === 'favorites') {
                currentTracks = currentTracks.filter(t => t.id !== track.id);
                renderFilteredResults();
            }
        }
        else {
            await request(`/api/library/favorites/${track.id}`, { method: 'POST', body: JSON.stringify({ track }) });
            favorites.add(track.id);
            button.classList.add('is-saved');
            button.setAttribute('aria-label', `Quitar ${track.name}`);
        }
    }
    catch (error) {
        setStatus(error.message, true);
    }
}
async function startTrack(track) {
    const playbackId = ++currentTrackId;
    current = track;
    $('#player-title').textContent = track.name;
    $('#player-artist').textContent = track.artist || 'Artista sin nombre';
    const image = $('#player-image');
    image.src = track.image || '';
    image.hidden = !track.image;
    audio.src = track.audio;
    play.disabled = false;
    try {
        await audio.play();
    }
    catch {
        setStatus('La vista previa no pudo reproducirse. Prueba otra canción.', true);
        return;
    }
    try {
        await request('/api/library/history', { method: 'POST', body: JSON.stringify({ track }) });
        if (playbackId !== currentTrackId)
            return;
        if (activeView === 'history')
            library('history');
    }
```

### Logout y carga inicial (final del archivo)

```js
window.addEventListener('hashchange', renderRoute);
$('#logout-button').addEventListener('click', async () => {
    await request('/api/auth/logout', { method: 'POST' });
    audio.pause();
    window.location.href = 'index.html';
});
(async () => {
    try {
        const user = await request('/api/auth/me');
        $('#account-email').textContent = user.email;
        const saved = await request('/api/library/favorites');
        favorites = new Set(saved.map(track => track.id));
        renderRoute();
    }
    catch {
        window.location.href = 'index.html';
    }
})();
export {};
```

### Las rutas del servidor a las que llama (`ProyectoReproductor/dist/server/index.js`)

```js
app.post('/api/auth/register', (req, res) => { const email = String(req.body?.email ?? '').trim().toLowerCase(), password = String(req.body?.password ?? ''); if (!/^\S+@\S+\.\S+$/.test(email) || password.length < 10 || password.length > 128)
    return res.status(400).json({ error: 'Introduce un correo válido y una contraseña de 10 a 128 caracteres.' }); const id = randomBytes(16).toString('hex'); try {
    run('INSERT INTO users(id,email,password_hash) VALUES(?,?,?)', [id, email, hashPassword(password)]);
}
catch {
    return res.status(409).json({ error: 'Ya existe una cuenta con ese correo.' });
} createSession(id, res); res.status(201).json({ id, email }); });
app.post('/api/auth/login', (req, res) => { const email = String(req.body?.email ?? '').trim().toLowerCase(), password = String(req.body?.password ?? ''), user = one('SELECT id,email,password_hash FROM users WHERE email=?', [email]); if (!user || !passwordMatches(password, user.password_hash))
    return res.status(401).json({ error: 'Correo o contraseña incorrectos.' }); createSession(user.id, res); res.json({ id: user.id, email: user.email }); });
app.post('/api/auth/logout', (req, res) => { const token = req.cookies?.nightjar_session; if (token)
    run('DELETE FROM sessions WHERE token=?', [token]); res.clearCookie('nightjar_session'); res.status(204).end(); });
app.get('/api/auth/me', (req, res) => { const user = userFor(req); user ? res.json(user) : res.status(401).json({ error: 'Sesión no iniciada.' }); });
app.get('/api/library/:kind', session, (req, res) => { const kind = String(req.params.kind); if (!['favorites', 'history'].includes(kind))
    return res.status(404).json({ error: 'Biblioteca no encontrada.' }); const orderColumn = kind === 'history' ? 'played_at' : 'created_at'; const rows = many(`SELECT track_json FROM ${kind} WHERE user_id=? ORDER BY ${orderColumn} DESC LIMIT 100`, [res.locals.user.id]); res.json(rows.map(row => JSON.parse(row.track_json))); });
app.post('/api/library/favorites/:trackId', session, (req, res) => { const track = req.body?.track; if (!track || String(track.id) !== String(req.params.trackId) || !track.audio)
    return res.status(400).json({ error: 'La canción no tiene datos válidos.' }); run('INSERT OR REPLACE INTO favorites(user_id,track_id,track_json) VALUES(?,?,?)', [res.locals.user.id, String(req.params.trackId), JSON.stringify(track)]); res.status(201).json({ saved: true }); });
app.delete('/api/library/favorites/:trackId', session, (req, res) => { run('DELETE FROM favorites WHERE user_id=? AND track_id=?', [res.locals.user.id, String(req.params.trackId)]); res.status(204).end(); });
app.post('/api/library/history', session, (req, res) => { const track = req.body?.track; if (!track || !track.id || !track.audio)
    return res.status(400).json({ error: 'La canción no tiene datos válidos.' }); run('INSERT INTO history(user_id,track_id,track_json) VALUES(?,?,?)', [res.locals.user.id, String(track.id), JSON.stringify(track)]); res.status(201).json({ saved: true }); });
```

> 📌 Pon las dos piezas una al lado de la otra: cada `request(...)` del cliente
> tiene su `app.get/post/delete(...)` en el servidor con la **misma ruta y el mismo
> método**. Eso es el "contrato" de una API.

---

## 8.9 `api-rastro/api-rastro/src/index.ts` — la "carcasa" de una API REST

**Temas:** logger de peticiones, `express.json()`, `/api/health`, montar routers por
recurso, 404 en JSON para rutas inexistentes, 400 para JSON mal formado, 500 genérico.

```ts
import express from "express";
import type { NextFunction, Request, Response } from "express";
import { connectDb, getDb } from "./db";
import listingsRouter from "./routes/listings";
import offersRouter from "./routes/offers";

const app = express();
const PORT = 3000;

app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

app.use(express.json());

app.get("/api/health", async (_req, res) => {
  await getDb().command({ ping: 1 });
  res.json({ status: "ok", db: getDb().databaseName });
});

app.use("/api/listings", listingsRouter);
app.use("/api/offers", offersRouter);

app.use((req, res) => {
  res.status(404).json({ error: `No existe la ruta ${req.method} ${req.url}` });
});

app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  if (err instanceof SyntaxError) {
    res.status(400).json({ error: "El body no es un JSON válido" });
    return;
  }

  console.error(err);
  res.status(500).json({ error: "Error interno del servidor" });
});

async function start() {
  await connectDb();

  app.listen(PORT, () => {
    console.log(`Servidor escuchando en http://localhost:${PORT}`);
  });
}

start();
```

---

## 8.10 `api-rastro/api-rastro/src/routes/offers.ts` — una ruta con TODOS los códigos

**Temas:** validar el body (400), recurso inexistente (404), conflicto de estado
(409), crear (201), PATCH que cambia el estado de un recurso (200).

```ts
import { Router } from "express";
import { ObjectId } from "mongodb";
import { listingsCollection, offersCollection, usersCollection } from "../db";
import type { Offer } from "../types";

const router = Router();

/**
 * Crea una nueva oferta para un anuncio.
 */
router.post("/", async (req, res) => {
  const { listingId, buyerId, amount } = req.body ?? {};

  if (typeof listingId !== "string" || !ObjectId.isValid(listingId)) {
    res.status(400).json({ error: "listingId no tiene un formato válido" });
    return;
  }
  if (typeof buyerId !== "string" || !ObjectId.isValid(buyerId)) {
    res.status(400).json({ error: "buyerId no tiene un formato válido" });
    return;
  }

  if (typeof amount !== "number" || !Number.isFinite(amount) || amount <= 0) {
    res.status(400).json({ error: "amount tiene que ser un número mayor que 0" });
    return;
  }

  const listing = await listingsCollection().findOne({ _id: new ObjectId(listingId) });
  if (!listing) {
    res.status(400).json({ error: "No existe ningún anuncio con ese listingId" });
    return;
  }

  const buyer = await usersCollection().findOne({ _id: new ObjectId(buyerId) });
  if (!buyer) {
    res.status(400).json({ error: "No existe ningún usuario con ese buyerId" });
    return;
  }

  // Validación: el comprador no puede ser el vendedor del anuncio
  if (listing.seller.id.equals(buyer._id)) {
    res.status(400).json({ error: "No puedes ofertar por tu propio anuncio" });
    return;
  }

  if (listing.status !== "AVAILABLE") {
    res.status(409).json({ error: `No se puede ofertar: el anuncio está ${listing.status}` });
    return;
  }

  const newOffer: Offer = {
    amount,
    status: "PENDING",
    createdAt: new Date(),
    listing: { id: listing._id, title: listing.title, price: listing.price },
    buyer: { id: buyer._id, name: buyer.name },
  };

  const result = await offersCollection().insertOne(newOffer);

  res.status(201).json({ _id: result.insertedId, ...newOffer });
});

/**
 * Acepta una oferta pendiente y reserva el anuncio.
 */
router.patch("/:id", async (req, res) => {
  const { id } = req.params;

  if (!ObjectId.isValid(id)) {
    res.status(400).json({ error: "El id no tiene un formato válido" });
    return;
  }

  if (req.body?.status !== "ACCEPTED") {
    res.status(400).json({ error: 'status tiene que ser "ACCEPTED"' });
    return;
  }

  const offerId = new ObjectId(id);

  const offer = await offersCollection().findOne({ _id: offerId });
  if (!offer) {
    res.status(404).json({ error: "No existe ninguna oferta con ese id" });
    return;
  }

  if (offer.status !== "PENDING") {
    res.status(409).json({ error: `No se puede aceptar: la oferta está ${offer.status}` });
    return;
  }

  const listing = await listingsCollection().findOne({ _id: offer.listing.id });
  if (!listing || listing.status !== "AVAILABLE") {
    res.status(409).json({ error: "No se puede aceptar: el anuncio ya no está disponible" });
    return;
  }

  const acceptedOffer = await offersCollection().findOneAndUpdate(
    { _id: offerId, status: "PENDING" },
    { $set: { status: "ACCEPTED" } },
    { returnDocument: "after" },
  );
  if (!acceptedOffer) {
    res.status(409).json({ error: "No se puede aceptar: la oferta ya no está PENDING" });
    return;
  }

  await listingsCollection().updateOne({ _id: offer.listing.id }, { $set: { status: "RESERVED" } });

  res.json(acceptedOffer);
});

export default router;

```

### Tabla resumen de `offers.ts`

| Situación | Código |
|-----------|--------|
| `listingId`/`buyerId` con formato inválido, `amount` no numérico o ≤ 0 | `400` |
| El anuncio o el comprador no existen (referencia en el body) | `400` |
| Ofertar por tu propio anuncio | `400` |
| El anuncio no está `AVAILABLE` | `409` |
| Oferta creada | `201` |
| `PATCH` con `status` distinto de `"ACCEPTED"` | `400` |
| La oferta a aceptar no existe (id en la URL) | `404` |
| La oferta ya no está `PENDING` o el anuncio ya no está disponible | `409` |
| Oferta aceptada | `200` |

> 📌 Fíjate en la diferencia: si el id **de la URL** no existe → `404` (el recurso
> que pides no está). Si un id **del body** apunta a algo que no existe → `400` (los
> datos que mandas son incorrectos).

---

## 8.11 `API_Godines/api-express/api-books-claude/docs/prd-books.md` — diseñar antes de programar

```md
# PRD: API de libros

## Objetivo
Una API para consultar y gestionar el catálogo de libros de una librería.

## Usuarios
- Clientes: buscan libros.
- Empleados: añaden, editan y borran libros.

## Funcionalidades
1. Listar libros, con filtros por autor, género y precio máximo.
2. Ver un libro por su id.
3. Crear un libro (título, autor y precio obligatorios).
4. Editar y borrar un libro:
   - `PUT /api/books/:id`: sustituye el libro entero (todos los campos obligatorios).
   - `PATCH /api/books/:id`: modifica solo los campos enviados (al menos uno).
   - `DELETE /api/books/:id`: borra el libro y lo devuelve (200).

## Fuera de alcance (por ahora)
- Login y usuarios.
- Pagos.

## Hecho cuando...
- Todos los endpoints responden con los códigos HTTP correctos (200, 201, 400, 404).
- Se puede cambiar de memoria a MongoDB sin tocar las rutas.
```

---

## 8.12 `api-rastro/api-rastro/respuestas.md` — tus respuestas sobre diseño de APIs

(Solo las preguntas relacionadas con HTTP/REST; el Ejercicio 0 de Git y los retos de
MongoDB están en sus apuntes correspondientes.)

```md
## Pregunta 1

Con MongoDB parado, `GET /api/health` tarda unos 5 segundos (el `serverSelectionTimeoutMS`) y devuelve **`500`** con `{ "error": "Error interno del servidor" }`. Ha respondido el **manejador de errores** (el paso 5 de `index.ts`): el `ping` lanza un error, Express 5 lo pasa al middleware de 4 parámetros y este responde un mensaje genérico sin detalles internos.

## Pregunta 2

Porque todo lo que viene en la URL llega como **texto**: `req.query` es `{ price: "380" }`, con el `"380"` como string. En MongoDB el precio está guardado como número (`380`), y `"380"` y `380` son tipos distintos, así que el filtro no coincide con ningún documento.

## Pregunta 3

Porque esos campos los decide el servidor, no el cliente: un anuncio nuevo siempre empieza `AVAILABLE` y con 0 visitas. Si la API los aceptara, cualquiera podría inflar sus visitas (`views: 9999`) para salir como anuncio popular, o publicarlo directamente como `SOLD`/`RESERVED`, falseando los datos que ven los demás usuarios.

## Pregunta 4

No. La copia `listing.price: 380` de la oferta es una **foto del momento** en que Sara ofertó: indica el precio que tenía el anuncio cuando hizo su oferta de 340 €, y ese dato histórico debe conservarse. El precio actual siempre se puede consultar en el propio anuncio; si el `PATCH` cambiara la copia, perderíamos el contexto de la oferta.

## Pregunta 5

Porque la petición está bien hecha: los ids tienen buen formato, existen y la cantidad es válida. El problema no es lo que envía el cliente, sino que choca con el **estado actual** del anuncio (ya está `SOLD`). `400` es para peticiones mal formadas; `409` es para peticiones correctas que entran en conflicto con el estado del recurso.

## Pregunta 6

MongoDB comprueba lo que le pedimos con **índices**, por ejemplo el índice único de `users.email` (no deja dos usuarios con el mismo email) y que cada `_id` sea único. Todo lo demás lo comprueba la API en el código: que una referencia existe (que el `sellerId` o el `listingId` estén de verdad en su colección), que `price`/`amount` sean números (y no el texto `"75"`), que los campos obligatorios lleguen o que el anuncio esté `AVAILABLE` antes de aceptar una oferta.

## Reto 3: estadísticas

Escrita **después** de `router.get("/:id", ...)`, `GET /api/listings/stats` devuelve **`400`** (`El id no tiene un formato válido`). Express prueba las rutas en el orden en que se registran, y `/:id` encaja con cualquier texto, así que toma `"stats"` como id y, como no es un ObjectId válido, responde 400 sin llegar nunca a la ruta de estadísticas. Se arregla registrando `/stats` **antes** de `/:id`: las rutas fijas tienen que ir antes que las que tienen parámetros.
```
