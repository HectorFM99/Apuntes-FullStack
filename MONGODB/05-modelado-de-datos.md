# 05 — MODELADO DE DATOS: EMBEBER vs REFERENCIAR

> En SQL la regla es **normalizar** (cada dato en una sola tabla y relacionarlas con
> claves ajenas). En MongoDB la pregunta es otra: **¿cómo voy a LEER estos datos?**
> Lo que se lee junto, se guarda junto.

---

## 5.1 Las dos opciones

### Opción A — Embeber (*embedding*): el dato va DENTRO del documento

```js
// anuncio
{
  _id: ObjectId("..."),
  titulo: "Google Pixel 7",
  fotos: [                                       // ← embebido
    { url: "https://img.rastro.es/pixel-1.jpg", orden: 1 },
    { url: "https://img.rastro.es/pixel-2.jpg", orden: 2 }
  ],
  atributos: { marca: "Google", almacenamientoGB: 128 }   // ← embebido
}
```

### Opción B — Referenciar (*referencing*): se guarda solo el `_id` del otro documento

```js
// oferta (colección "ofertas")
{
  _id: ObjectId("680000000000000000000003"),
  anuncio: { id: ObjectId("670000000000000000000001") },  // ← referencia al anuncio
  importe: 340,
  estado: "PENDIENTE"
}
```

| | Embeber | Referenciar |
|---|---|---|
| Lectura | ✅ **Una sola consulta** | ❌ Varias consultas o `$lookup` |
| Actualizar el dato | ❌ Si está copiado en muchos sitios, hay que cambiarlo en todos | ✅ Se cambia en un solo sitio |
| Crecimiento | ❌ Un documento tiene un **límite de 16 MB** | ✅ Sin límite |
| Consultar el dato "suelto" | ❌ Difícil | ✅ Fácil (tiene su colección) |

---

## 5.2 Reglas para decidir

**Embebe** cuando:
- La relación es **"tiene unos pocos"** (1 a pocos): fotos de un anuncio, líneas de un pedido.
- Los datos **se leen siempre juntos**.
- El dato **no tiene vida propia** fuera del padre.
- Quieres una **foto histórica** que no cambie (dirección de envío de un pedido).

**Separa (colección aparte + referencia)** cuando:
- La relación es **"tiene muchísimos"** o **crece sin límite**: ofertas, reseñas, mensajes.
- El dato tiene **vida propia** y se consulta solo (usuarios, categorías).
- El dato es **compartido** por muchos documentos y cambia a menudo.

> 📌 Pregunta mágica: *"¿Este array puede crecer sin límite?"* → si sí, **aparte**.

---

## 5.3 Ejemplo REAL: tus respuestas del Rastro

**Archivo:** `mongo-rastro/mongo-rastro/respuestas.md` *(copiado tal cual)*

```md
## A6. ¿Dentro o aparte?
1. **Mensajes de chat:** Aparte (historial creciente y paginable).

2. **Dirección de envío:** Embebida en la compra (copia histórica inalterable).

3. **Etiquetas:** Embebidas en el anuncio (array pequeño, lectura conjunta).

4. **Categorías:** Aparte (compartidas e independientes).

5. **Historial de precios:** Aparte (creciente y consultado por fechas).

6. **Anuncios del usuario:** Aparte (vida propia, con referencia al vendedor).

7. **Líneas de pedido:** Embebidas (parte indivisible del pedido).

8. **Reseñas de producto:** Aparte (crecen indefinidamente).
```

Y tu resumen con tus palabras (B8.3):

```md
3. **Embebido vs Aparte:** Embeber datos estables o pequeños que se leen juntos (fotos, líneas); separar colecciones si crecen indefinidamente o tienen vida propia (ofertas, reseñas).
```

---

## 5.4 El modelo del Rastro completo

```
usuarios                     anuncios                         ofertas
┌───────────────────┐        ┌────────────────────────────┐   ┌──────────────────────────────┐
│ _id               │◄───┐   │ _id                        │◄──│ anuncio: { id, titulo,precio}│
│ nombre            │    └───│ vendedor: { id, nombre }   │   │ comprador: { id, nombre }    │
│ email (único)     │        │ fotos: [ {url, orden} ]    │   │ importe, estado, fecha       │
│ favoritos: [ids]──┼──────► │ atributos: { ... }         │   └──────────────────────────────┘
│ valoracionMedia   │        │ titulo, precio, estado...  │
│ numValoraciones   │        └────────────────────────────┘   valoraciones
└───────────────────┘                                         ┌──────────────────────────────┐
                                                              │ anuncioId, autor: {id,nombre}│
                                                              │ valoradoId, estrellas        │
                                                              └──────────────────────────────┘
```

Tu apartado 9 (`respuestas.md`) lo resume:

```md
Coincide con el modelo base: colecciones separadas para usuarios, anuncios, ofertas y valoraciones; fotos y atributos embebidos; nombre de vendedor desnormalizado; favoritos en array de usuario.
```

### Por qué cada decisión

| Dato | Decisión | Motivo (de tus respuestas) |
|---|---|---|
| Fotos | Embebidas en el anuncio | Pocas, se leen siempre con el anuncio |
| Atributos | Embebidos (objeto libre) | Cambian por categoría → esquema flexible |
| Vendedor | Referencia `id` + **copia del `nombre`** | El usuario tiene vida propia; el nombre se copia para pintar la ficha sin otra consulta |
| Ofertas | Colección aparte | *"Un anuncio recibe muchas ofertas y el comprador consulta las suyas"* |
| Valoraciones | Colección aparte | *"Evita arrays en crecimiento ilimitado dentro de usuarios"* |
| Favoritos | Array de ids **en el usuario** | Un usuario tiene pocos; un anuncio popular tendría miles |

---

## 5.5 Desnormalizar: copiar datos a propósito

**Desnormalizar** = guardar una **copia** de un dato de otra colección para no tener
que hacer `$lookup` al leer.

### Ejemplo REAL (`mongo-rastro/mongo-rastro/respuestas.md`, A2)

```md
## A2. Anuncio y vendedor
1. **No se embebe al usuario completo:** Tiene entidad propia y se relaciona con múltiples anuncios; duplicarla dificultaría las actualizaciones.

2. **Nombre copiado:** Se almacena el nombre del vendedor en el anuncio para mostrar la ficha en una sola lectura, sin datos personales mutables (email/ciudad).

3. **Actualización:** Al cambiar el nombre del vendedor, debe actualizarse en sus anuncios y ofertas.

4. **Comparativa SQL:** En SQL requiere un JOIN; en MongoDB se lee directamente con la copia desnormalizada.
```

```js
// ❌ Embeber el usuario ENTERO en cada anuncio
vendedor: { id, nombre, email, ciudad, fechaAlta, favoritos: [...], ... }

// ✅ Referencia + SOLO lo que se muestra en el anuncio
vendedor: { id: ObjectId("660000000000000000000004"), nombre: "Hugo Sanz" }
```

### El precio: actualizar todas las copias

Cuando Luis cambia su nombre, tu ejercicio B7.6 (`rastro.js`) tiene que tocar
**cuatro** colecciones (ver `02`, apartado 2.6, donde está copiado tal cual; aquí
lo pongo en una línea por consulta para que se vea de un vistazo):

```js
db.usuarios.updateOne(      { _id: ObjectId("660000000000000000000002") },           { $set: { nombre: "Luis M. Martín" } })
db.anuncios.updateMany(     { "vendedor.id": ObjectId("660000000000000000000002") },  { $set: { "vendedor.nombre": "Luis M. Martín" } })
db.ofertas.updateMany(      { "comprador.id": ObjectId("660000000000000000000002") }, { $set: { "comprador.nombre": "Luis M. Martín" } })
db.valoraciones.updateMany( { "autor.id": ObjectId("660000000000000000000002") },     { $set: { "autor.nombre": "Luis M. Martín" } })
```

> 📌 **Copia solo datos que cambian poco** (un nombre) y que lees mucho. Nunca copies
> datos que cambian a menudo (stock, precio actual...).

---

## 5.6 Copias históricas: "la foto del momento"

A veces copiar no es un truco de rendimiento, sino que **debe** ser una copia: el
dato tiene que quedar congelado tal y como estaba.

### Ejemplo REAL (`mongo-rastro/mongo-rastro/respuestas.md`, A3)

```md
3. **Precio histórico:** Guarda el precio del anuncio al momento de la oferta para conservar el contexto histórico frente a cambios posteriores.
```

### En la API (`api-rastro/api-rastro/src/routes/offers.ts`)

```ts
  const newOffer: Offer = {
    amount,
    status: "PENDING",
    createdAt: new Date(),
    listing: { id: listing._id, title: listing.title, price: listing.price },
    buyer: { id: buyer._id, name: buyer.name },
  };
```

Y tu explicación (`api-rastro/api-rastro/respuestas.md`, pregunta 4):

```md
No. La copia `listing.price: 380` de la oferta es una **foto del momento** en que Sara ofertó: indica el precio que tenía el anuncio cuando hizo su oferta de 340 €, y ese dato histórico debe conservarse. El precio actual siempre se puede consultar en el propio anuncio; si el `PATCH` cambiara la copia, perderíamos el contexto de la oferta.
```

> 📌 Otro ejemplo clásico: la **dirección de envío** y el **precio** dentro de un
> pedido. Si mañana cambias de casa o el producto sube, el pedido antiguo no debe cambiar.

---

## 5.7 Resúmenes precalculados

### Ejemplo REAL (`mongo-rastro/mongo-rastro/respuestas.md`, A4)

```md
3. **Resumen en usuario:** Almacena valoracionMedia y numValoraciones para consultarlas instantáneamente.
```

```js
// En el usuario se guarda el resumen...
{ nombre: "Ana García", valoracionMedia: 4.5, numValoraciones: 12 }

// ...y las valoraciones completas, aparte (colección "valoraciones")
```

Así, para mostrar "⭐ 4.5 (12)" en el perfil no hace falta un `aggregate` cada vez.
A cambio, cada valoración nueva debe **recalcular** esos dos campos (ejemplo mío):

```js
db.usuarios.updateOne(
  { _id: ObjectId("660000000000000000000003") },
  { $set: { valoracionMedia: 4.2 }, $inc: { numValoraciones: 1 } }
)
```

---

## 5.8 Sin claves ajenas: la integridad la pones tú

MongoDB **no comprueba** que un id de referencia exista. Tu respuesta B4.7:

```md
- **B4.7 (Integridad referencial):** A diferencia de SQL, MongoDB no aplica restricciones de clave ajena por defecto; la aplicación debe validarlas.
```

Por eso en la API se comprueba **antes** de insertar
(`api-rastro/api-rastro/src/routes/listings.ts`):

```ts
  const seller = await usersCollection().findOne({ _id: new ObjectId(body.sellerId) });
  if (!seller) {
    res.status(400).json({ error: "No existe ningún usuario con ese sellerId" });
    return;
  }
```

Y tu respuesta a la pregunta 6 de `api-rastro/respuestas.md` reparte el trabajo:

```md
MongoDB comprueba lo que le pedimos con **índices**, por ejemplo el índice único de `users.email` (no deja dos usuarios con el mismo email) y que cada `_id` sea único. Todo lo demás lo comprueba la API en el código: que una referencia existe (que el `sellerId` o el `listingId` estén de verdad en su colección), que `price`/`amount` sean números (y no el texto `"75"`), que los campos obligatorios lleguen o que el anuncio esté `AVAILABLE` antes de aceptar una oferta.
```

### Validación de esquema en Mongo (opcional, ejemplo mío)

Si quieres que **la propia BD** rechace datos mal tipados, existe `$jsonSchema`:

```js
db.createCollection("anuncios", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["titulo", "precio"],
      properties: {
        titulo: { bsonType: "string" },
        precio: { bsonType: ["double", "int"], minimum: 0 }
      }
    }
  }
})
```

> 📌 Con esto, el `precio: "75"` del ejercicio B4.4 daría error al insertarse.

---

## 5.9 Atomicidad y transacciones

- Una escritura sobre **un** documento es siempre **atómica** (todo o nada), aunque
  cambie varios campos o arrays. → Otro motivo para embeber lo que va junto.
- Si necesitas cambiar **dos documentos** a la vez, necesitas una **transacción**.

### Ejemplo REAL (`api-rastro/api-rastro/respuestas.md`, reto 1)

```md
**¿Qué pasa si el servidor se apaga entre una escritura y otra?** Los datos quedan incoherentes: la oferta queda `ACCEPTED` pero el anuncio sigue `AVAILABLE`, y otros compradores podrían seguir ofertando por algo que ya está reservado. MongoDB solo garantiza que cada escritura sobre **un** documento es atómica; para que las dos se hagan juntas o ninguna haría falta una **transacción** (que necesita MongoDB en modo replica set).
```

---

## 5.10 Errores comunes de modelado

| Error | Por qué está mal | Mejor |
|---|---|---|
| Embeber **todas** las ofertas en el anuncio | Array sin límite → documento enorme (16 MB) | Colección `ofertas` con `anuncio.id` |
| Embeber el usuario completo en cada anuncio | Al cambiar el email hay que tocar miles de docs | Referencia + copia del nombre |
| Una colección por categoría (`moviles`, `coches`...) | Imposible buscar en todas a la vez | Una colección `anuncios` con `categoria` y `atributos` libres |
| Modelar como en SQL (todo separado + `$lookup` siempre) | Pierdes la ventaja de Mongo | Embebe lo que se lee junto |
| Guardar ids como texto | No coinciden con los `ObjectId` | Guarda `ObjectId` |

---

## ✅ Resumen rápido

1. Modela pensando en **cómo vas a leer**: lo que se lee junto, se guarda junto.
2. **Embeber**: pocos elementos, sin vida propia, se leen juntos, o copia histórica.
3. **Aparte + referencia**: crece sin límite, vida propia, compartido.
4. **Desnormalizar** = copiar campos poco cambiantes (nombre) → lectura en una consulta, pero hay que actualizar todas las copias.
5. Las copias "foto del momento" (precio en la oferta) **no se actualizan nunca**.
6. Mongo **no** tiene claves ajenas: la API comprueba que las referencias existen.
7. Atómico por documento; para varios documentos → transacción.

👉 **Siguiente:** `06-mongodb-con-node-express-typescript.md`
