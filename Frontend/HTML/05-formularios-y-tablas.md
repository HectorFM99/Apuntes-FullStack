# 05 — FORMULARIOS Y TABLAS

---

## 5.1 Estructura de un formulario: `<form>`

```html
<form action="/enviar" method="POST">
    <label for="nombre">Nombre:</label>
    <input type="text" id="nombre" name="nombre" placeholder="Tu nombre" required>

    <button type="submit">Enviar</button>
</form>
```

| Atributo de `<form>` | Qué hace |
|----------------------|----------|
| `action` | **URL** a la que se envían los datos |
| `method` | `GET` (los datos van en la URL) o `POST` (van en el cuerpo) |
| `novalidate` | Desactiva la validación nativa del navegador |
| `target` | Dónde se muestra la respuesta (`_blank`, `_self`...) |

> 📌 **`name` es clave:** sin `name`, el campo **no se envía**. El `name` es la clave
> con la que el servidor recibe el dato (`nombre=Hector`).

---

## 5.2 `label` + `input`: el matrimonio básico

```html
<label for="email">Correo electrónico</label>
<input type="email" id="email" name="email" required>
```

- `label for="X"` → se **asocia** al campo con `id="X"".
- Al clicar en la etiqueta, **el foco va al campo**.
- Los lectores de pantalla anuncian el nombre correcto del campo.
- **Siempre** usa `<label>`; el `placeholder` **no** lo sustituye (desaparece al escribir).

### Todos los `type` importantes de `<input>`

| `type` | Qué muestra | Ventaja extra |
|--------|-------------|----------------|
| `text` | Texto libre | |
| `password` | Puntos •••• | oculta lo que escribes |
| `email` | Correo | valida formato `@` |
| `number` | Número | `min`, `max`, `step` |
| `tel` | Teléfono | teclado numérico en móvil |
| `url` | URL | valida `https://` |
| `date` / `time` / `datetime-local` | Selector de fecha/hora | calendario nativo |
| `color` | Selector de color | |
| `range` | Deslizador | `min`, `max`, `step` |
| `checkbox` | Casilla ✔ | `checked` |
| `radio` | Botón redondo ◉ | grupos por `name` |
| `file` | Subir archivo | `accept`, `multiple` |
| `hidden` | Invisible | envía datos fijos |
| `search` | Campo de búsqueda | botón de borrar |
| `submit` / `button` / `reset` | Botones | |

### Atributos de validación nativos

```html
<input type="text"   required>                      <!-- obligatorio -->
<input type="email"  required placeholder="a@b.com">
<input type="text"   minlength="3" maxlength="20">   <!-- longitud -->
<input type="number" min="0" max="120" step="1">
<input type="text"   pattern="[A-Za-z]{3,8}">       <!-- expresión regular -->
<input type="text"   value="Valor por defecto">
<input type="text"   disabled>                      <!-- no editable, no se envía -->
<input type="text"   readonly>                      <!-- no editable, SÍ se envía -->
```

---

## 5.3 Ejemplo real copiado de un proyecto

**Archivo:** `Ejercicio github/cagada-de-repositorio-nano/contacto/contacto.html` *(copiado tal cual)*

```html
<body>
    <img src="https://media1.tenor.com/m/ltMwVmL-Y8oAAAAC/cat-middle-finger.gif" alt="Imagen de contacto">
    <h1>Contacto</h1>
    <p>¡Contáctanos!</p>
    <label for="respuesta">Elige tu respuesta</label>

  <select id="respuesta" name="respuesta">
    <option value="" selected disabled>Elige una respuesta</option>
    <option value="rojo">Si</option>
    <option value="azul">No</option>
  </select>
</body>
```

### 🧠 Análisis etiqueta por etiqueta

| Fragmento | Qué hace |
|-----------|----------|
| `<label for="respuesta">` | Etiqueta visible; `for` coincide con el `id` del `<select>` → al clicar enfoca el desplegable |
| `<select id="respuesta" name="respuesta">` | Desplegable. `name` es la clave de envío, `id` es la que usa el `label` y CSS/JS |
| `<option value="">` | Cada opción; `value` es lo que **se envía**, lo de dentro es lo que **se ve** |
| `selected disabled` | Está preseleccionado y **no se puede elegir** → obliga a elegir una opción real |
| `value="rojo"` / `value="azul"` | **Ojo:** el valor enviado es "rojo"/"azul", aunque se vea "Si"/"No" |

> ⚠️ **Detalle real:** el `name` y el `id` tienen el mismo nombre (`respuesta`).
> Eso es lo habitual, pero cumplen funciones distintas: `name` = servidor, `id` = HTML/CSS/JS.

---

## 5.4 `<select>`, `<optgroup>` y `<textarea>`

```html
<select name="pais" multiple size="4">   <!-- multiple = selección múltiple -->
    <optgroup label="Europa">
        <option value="es" selected>España</option>
        <option value="pt">Portugal</option>
        <option value="fr">Francia</option>
    </optgroup>
    <optgroup label="América">
        <option value="mx">México</option>
        <option value="ar">Argentina</option>
    </optgroup>
</select>

<textarea name="mensaje" rows="6" cols="40"
          placeholder="Escribe tu mensaje aquí"
          maxlength="500"></textarea>
```

> 📌 `<textarea>` **se cierra** y el texto por defecto va **entre** las etiquetas.
> Las dimensiones se ponen con `rows`/`cols`, no con `width`/`height`.

---

## 5.5 Agrupaciones: `<fieldset>` y `<legend>`

```html
<fieldset>
    <legend>Datos de contacto</legend>

    <label for="tel">Teléfono</label>
    <input type="tel" id="tel" name="tel">

    <label for="mail">Email</label>
    <input type="email" id="mail" name="mail">
</fieldset>
```

---

## 5.6 Botones

```html
<button type="submit">Enviar</button>   <!-- envía el formulario -->
<button type="reset">Vaciar</button>    <!-- limpia los campos -->
<button type="button">Solo botón</button> <!-- no hace nada por sí solo: lo mueve JS -->
<input type="submit" value="Enviar">    <!-- versión antigua -->
```

> 📌 Dentro de un `<form>`, si pones `<button>` **sin `type`**, por defecto es `submit`.
> Esa es una causa muy común de "me envía el formulario cuando no quiero".

---

## 5.7 Ejemplo real: formulario con validación y CSS

**HTML (así quedaría montado):**

```html
<form action="/suscribirse" method="POST">
    <fieldset>
        <legend>Suscríbete a la newsletter</legend>

        <label for="nombre">Nombre</label>
        <input type="text" id="nombre" name="nombre"
               minlength="2" maxlength="50" required
               placeholder="Tu nombre">

        <label for="email">Correo</label>
        <input type="email" id="email" name="email" required
               placeholder="tucorreo@ejemplo.com">

        <label for="edad">Edad</label>
        <input type="number" id="edad" name="edad" min="0" max="120">

        <label for="pais">País</label>
        <select id="pais" name="pais" required>
            <option value="" selected disabled>Elige una opción</option>
            <option value="es">España</option>
            <option value="mx">México</option>
            <option value="ar">Argentina</option>
        </select>

        <label>
            <input type="checkbox" name="acepto" value="si" required>
            Acepto la política de privacidad
        </label>

        <button type="submit">Suscribirme</button>
        <button type="reset">Vaciar</button>
    </fieldset>
</form>
```

> 📌 Fíjate en la **2ª forma de usar `<label>`**: envolviendo el input.
> Es válida y muy cómoda con checkboxes y radios.

### Estilo CSS que le iría bien (patrón real de tus proyectos)

Basado en la convención de `git/Pagina-Web/css/cuerpo.css`:

```css
.enlace-proyecto {          /* patrón "botón" usado en tus proyectos */
  display: inline-block;
  padding: 9px 14px;
  background: #e63946;
  color: white;
  text-decoration: none;
  border-radius: 6px;
}

.enlace-proyecto:hover {
  background: #c92f3b;
}
```

---

## 5.8 Botones de radio y checkboxes en grupo

```html
<!-- Radios: solo UNO se puede elegir (mismo name) -->
<fieldset>
    <legend>Género</legend>
    <label><input type="radio" name="genero" value="m" checked> Masculino</label>
    <label><input type="radio" name="genero" value="f"> Femenino</label>
    <label><input type="radio" name="genero" value="x"> Otro</label>
</fieldset>

<!-- Checkboxes: varios a la vez (name distinto o array []) -->
<fieldset>
    <legend>Idiomas</legend>
    <label><input type="checkbox" name="idiomas[]" value="es" checked> Español</label>
    <label><input type="checkbox" name="idiomas[]" value="en"> Inglés</label>
    <label><input type="checkbox" name="idiomas[]" value="fr"> Francés</label>
</fieldset>
```

> 📌 Todos los `radio` de un mismo grupo deben llevar **el mismo `name`**.
> Si llevan nombres distintos, se pueden marcar varios a la vez (y eso está mal).

---

## 5.9 TABLAS: `<table>`

```html
<table>
    <caption>Proyectos del bootcamp</caption>
    <thead>
        <tr>
            <th>Nombre</th>      <!-- th = celda de CABECERA -->
            <th>Tecnología</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Página Web</td>  <!-- td = celda de DATOS -->
            <td>HTML, CSS, JS</td>
        </tr>
        <tr>
            <td>Bichomon</td>
            <td>HTML, CSS, JS</td>
        </tr>
    </tbody>
    <tfoot>
        <tr><td colspan="2">Total: 2 proyectos</td></tr>
    </tfoot>
</table>
```

| Etiqueta | Qué es |
|----------|--------|
| `<table>` | La tabla |
| `<caption>` | Título accesible de la tabla |
| `<tr>` | **T**able **R**ow → una fila |
| `<th>` | Celda de cabecera (negrita + centrada por defecto) |
| `<td>` | Celda de datos |
| `<thead>`, `<tbody>`, `<tfoot>` | Agrupación semántica de filas |
| `colspan="n"` | La celda ocupa **n columnas** |
| `rowspan="n"` | La celda ocupa **n filas** |

> ⚠️ **No uses tablas para maquetar.** Para eso está **CSS Flexbox / Grid**.
> Las tablas son para **datos tabulares**.

### Ejemplo real copiado de un proyecto

**Archivo:** `varita-magica/varita-magica-hector/index.html` (fragmento simplificado)

```html
<table style="border-spacing:8px">
    ...
</table>
```

> 📌 `border-spacing` es la separación **entre** celdas.
> Nota: aquí el estilo va **en línea** (`style="..."`), que es una forma válida pero
> menos recomendable que usar un archivo CSS aparte.

### CSS básico de tablas

```css
table {
  width: 100%;
  border-collapse: collapse;   /* une los bordes de las celdas */
}

th, td {
  border: 1px solid #ddd;
  padding: 10px 12px;
  text-align: left;
}

th {
  background-color: #f6f6f6;
  color: #111;
}

tr:nth-child(even) {
  background-color: #fafafa;   /* filas zebradas */
}
```

---

## 5.10 Formularios de contacto — maquetación con CSS real

Del proyecto `git/Pagina-Web/css/cuerpo.css` (tarjetas y botones que ya usas):

```css
.contactos-sociales {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 28px;
  margin: 36px auto;
}

.contacto-social {
  display: flex;
  width: 130px;
  min-height: 120px;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  border: 1px solid rgb(255 255 255 / 30%);
  border-radius: 18px;
  background: rgb(20 20 20 / 75%);
  color: white;
  font-size: 17px;
  font-weight: 600;
  text-decoration: none;
  box-shadow: 0 8px 20px rgb(0 0 0 / 25%);
  transition: transform 180ms ease, box-shadow 180ms ease;
}

.contacto-social:hover,
.contacto-social:focus-visible {
  transform: translateY(-7px) scale(1.06);
  box-shadow: 0 14px 26px rgb(0 0 0 / 40%);
}
```

---

## ✅ Resumen rápido

1. `<form action method>` → `action` es **dónde va**, `method` es **cómo va**.
2. Todo campo necesita **`name`** para enviarse y **`id`** si tiene `<label for>`.
3. `required`, `type="email"`, `minlength`, `pattern` → validación **gratis y sin JS**.
4. `<select>` + `<option>` → desplegable; el `value` es lo que se envía.
5. `<button type="submit">` envía; dentro de `<form>`, un `<button>` sin `type` también.
6. Tablas: `<table>`, `<tr>`, `<th>`, `<td>`, `colspan`/`rowspan`. **Nunca para maquetar.**

👉 **Siguiente:** `06-atributos-accesibilidad-y-buenas-practicas.md`
