# 04 — LA API AUTOMÁTICA, LAS CLAVES Y `supabase-js`

> Esta es la "forma B" de usar Supabase (ver `01`, 1.4). En el momento en que
> creaste la tabla `books`, Supabase **ya generó** una API REST para ella, sin que
> escribieras un controller ni un service. Entenderla es importante incluso si no
> la usas: **está encendida**, y si no la proteges, cualquiera puede usarla.

---

## 4.1 PostgREST: de tabla a endpoint

PostgREST lee el esquema de Postgres y publica cada tabla/vista del schema
`public` como un recurso REST:

| Lo que harías en tu API NestJS | Lo que ya existe en Supabase |
|--------------------------------|------------------------------|
| `GET /api/books` | `GET https://<REF>.supabase.co/rest/v1/books?select=*` |
| `GET /api/books/1` | `GET .../rest/v1/books?id=eq.1` |
| `GET /api/books?author=garcia` (`ILIKE '%' \|\| $1 \|\| '%'`) | `GET .../rest/v1/books?author=ilike.*garcia*` |
| `POST /api/books` | `POST .../rest/v1/books` con body JSON |
| `PATCH /api/books/1` | `PATCH .../rest/v1/books?id=eq.1` |
| `DELETE /api/books/1` | `DELETE .../rest/v1/books?id=eq.1` |

### Probarlo con curl

```bash
curl "https://<REF>.supabase.co/rest/v1/books?select=id,title&order=id" \
  -H "apikey: <CLAVE_PUBLICA>"
```

### Operadores de filtro (en la query string)

| PostgREST | SQL |
|-----------|-----|
| `eq` / `neq` | `=` / `<>` |
| `gt`, `gte`, `lt`, `lte` | `>`, `>=`, `<`, `<=` |
| `like`, `ilike` (`*` = `%`) | `LIKE`, `ILIKE` |
| `in.(1,2,3)` | `IN (1,2,3)` |
| `is.null` | `IS NULL` |
| `order=pages.desc` | `ORDER BY pages DESC` |
| `limit=10&offset=20` | `LIMIT 10 OFFSET 20` |
| `select=title,reviews(*)` | `JOIN` usando las FK |

> 📌 Por dentro, cada petición se traduce a **un `SELECT`/`INSERT`/... en
> Postgres**, ejecutado **con el rol del usuario** (`anon` o `authenticated`). Y
> ahí es donde RLS decide qué filas se ven (archivo `05`).

---

## 4.2 Las claves de API

En *Project Settings → API Keys* hay dos tipos de clave. Supabase ha tenido dos
generaciones de nombres; verás ambas:

| Clave | Nombre clásico | Nombre nuevo | ¿Pública? | Rol en Postgres | ¿Respeta RLS? |
|-------|----------------|--------------|-----------|-----------------|---------------|
| Pública | `anon` key | **publishable** key (`sb_publishable_...`) | ✅ Puede ir en el frontend | `anon` (o `authenticated` si hay sesión) | ✅ Sí |
| Secreta | `service_role` key | **secret** key (`sb_secret_...`) | ❌ **NUNCA** en el frontend | `service_role` | ❌ **Se la salta** |

Y, aparte, la **contraseña de la base de datos** (la de tu `DATABASE_URL`), que
es otra cosa distinta: no sirve para la API REST, sirve para conectar con `pg`.

```
                   ¿Quién la usa?            ¿Dónde vive?                     Poder
anon/publishable → navegador (supabase-js) → código del frontend (público)  → lo que permita RLS
service_role     → tu servidor             → variable de entorno (secreta)  → TODO, ignora RLS
DATABASE_URL     → tu API con pg           → variable de entorno (secreta)  → TODO (es superusuario)
```

> ⚠️ **La clave pública es pública de verdad**: cualquiera puede sacarla del JS de
> tu web en 10 segundos. Su seguridad **no** consiste en esconderla, sino en que
> **RLS** limite lo que se puede hacer con ella. Si una tabla de `public` no tiene
> RLS, esa clave permite leer, modificar y **borrar** la tabla entera.

### Comprobación de seguridad para TU proyecto

Tu API usa la forma A, pero la tabla `books` está en `public`, así que **también**
es accesible por la API automática. Prueba:

```bash
curl "https://<REF>.supabase.co/rest/v1/books" -H "apikey: <CLAVE_PUBLICA>"
```

- Si devuelve los libros → cualquiera puede saltarse tu API NestJS (y sus validaciones) y escribir directamente. Activa RLS (`ALTER TABLE books ENABLE ROW LEVEL SECURITY;`) sin crear políticas: la API automática verá **cero filas**, y tu API (que entra como `postgres`) seguirá funcionando igual.
- Si devuelve `[]` → RLS está activo y sin políticas: correcto para la forma A.

---

## 4.3 `@supabase/supabase-js`: el cliente oficial

En vez de escribir las URLs de PostgREST a mano, se usa la librería:

```bash
npm install @supabase/supabase-js
```

```ts
// supabase.ts (frontend, ej. React + Vite) — NO está en tus proyectos
import { createClient } from '@supabase/supabase-js';

export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,          // https://<REF>.supabase.co
  import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY, // clave PÚBLICA, nunca la secreta
);
```

### El CRUD de `books`, comparado con tu `books.service.ts`

```ts
// findAll(author?)
let query = supabase.from('books').select('id, title, author, pages, created_at').order('id');
if (author) query = query.ilike('author', `%${author}%`);
const { data, error } = await query;

// findOne(id)
const { data, error } = await supabase.from('books').select('*').eq('id', id).single();

// create(dto)
const { data, error } = await supabase
  .from('books')
  .insert({ title, author, pages })
  .select()          // ≈ RETURNING *
  .single();

// update(id, dto)
const { data, error } = await supabase.from('books').update({ pages: 500 }).eq('id', id).select().single();

// remove(id)
const { error } = await supabase.from('books').delete().eq('id', id);
```

### Diferencias de comportamiento importantes

| | Tu API NestJS (`pg`) | `supabase-js` |
|---|---|---|
| Errores | Lanza excepción (`NotFoundException`...) | **No lanza**: devuelve `{ data, error }` → tienes que comprobar `error` |
| Inyección SQL | La evitas con `$1, $2` | No escribes SQL: la librería construye la petición |
| Validación de entrada | DTO + `class-validator` | Solo restricciones de la BD (`NOT NULL`, `CHECK`, tipos) |
| Nombres de columnas | Conviertes `created_at` → `createdAt` con `toBook()` | Recibes `created_at` tal cual (o lo mapeas tú) |
| "Not found" | `404` explícito | `.single()` devuelve `error` con código `PGRST116` |
| Filas sin permiso | No aplica (eres `postgres`) | Simplemente **no aparecen** (RLS filtra en silencio) |

```ts
// El patrón obligatorio con supabase-js
const { data, error } = await supabase.from('books').select('*');
if (error) throw error;          // ¡si no lo compruebas, data es null y nadie te avisa!
```

> 📌 `toBook()` de tu proyecto (`created_at` → `createdAt`) es un ejemplo de
> **capa de traducción** entre BD y API. Con `supabase-js` en el frontend esa capa
> desaparece: el frontend ve la forma exacta de tus tablas. Más cómodo, pero
> cambiar una columna rompe el frontend directamente.

---

## 4.4 Tipos de TypeScript generados

La CLI puede generar los tipos a partir del esquema real:

```bash
npx supabase gen types typescript --project-id <REF> > src/database.types.ts
```

```ts
import type { Database } from './database.types';
const supabase = createClient<Database>(url, key);

const { data } = await supabase.from('books').select('title, pages');
//      ^? { title: string; pages: number }[] | null   ← tipado automático
```

> 💡 Es la versión automática de tu interfaz `BookRow` escrita a mano en `book.ts`.

---

## 4.5 Funciones SQL como endpoints (RPC)

Si una operación no es un CRUD simple, puedes escribirla como **función de
Postgres** y llamarla:

```sql
CREATE FUNCTION books_by_author_count()
RETURNS TABLE(author VARCHAR, total BIGINT)
LANGUAGE sql STABLE AS $$
  SELECT author, COUNT(*) FROM books GROUP BY author ORDER BY 2 DESC;
$$;
```

```ts
const { data } = await supabase.rpc('books_by_author_count');
// POST https://<REF>.supabase.co/rest/v1/rpc/books_by_author_count
```

> 📌 Es la forma de "versionar la API" o encapsular lógica sin backend propio:
> si cambias la tabla, ajustas la función y los clientes no se enteran.

---

## 4.6 `supabase-js` en el servidor

También se puede usar desde Node (por ejemplo dentro de una API NestJS) con la
clave **secreta**, para tareas administrativas:

```ts
// SOLO en servidor, con variable de entorno secreta
const supabaseAdmin = createClient(process.env.SUPABASE_URL!, process.env.SUPABASE_SECRET_KEY!, {
  auth: { persistSession: false },
});
await supabaseAdmin.auth.admin.deleteUser(userId);   // operaciones que el usuario no puede hacer
```

> ⚠️ Con la clave secreta **no hay RLS**: tu código es el único responsable de los
> permisos, igual que con `pg` + `DATABASE_URL`.

---

## ✅ Resumen

- Cada tabla de `public` tiene **ya** una API REST (PostgREST) en `/rest/v1/<tabla>`.
- Clave **pública** (anon/publishable): va en el frontend, la protege **RLS**.
- Clave **secreta** (service_role/secret) y **DATABASE_URL**: solo en servidor, ignoran RLS.
- `supabase-js` devuelve `{ data, error }`: **siempre** comprueba `error`.
- Aunque uses la forma A, la API automática existe → activa RLS en tus tablas.
