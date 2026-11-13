# 05 — AUTH Y ROW LEVEL SECURITY (RLS)

> Si el frontend habla directamente con la base de datos, ¿quién impide que un
> usuario lea los datos de otro? En tu API NestJS lo impide **tu código**. En
> Supabase lo impide **Postgres**, con Row Level Security. Es el concepto más
> importante de Supabase y lo que separa un proyecto seguro de uno con todos los
> datos expuestos.

---

## 5.1 Supabase Auth en dos minutos

Supabase Auth gestiona usuarios por ti:

- Métodos: email + contraseña, *magic link* (enlace por email), OTP, OAuth (Google, GitHub, Discord...), anónimos.
- Los usuarios se guardan en la tabla **`auth.users`** (con la contraseña hasheada, como hacías con `argon2`).
- Al hacer login, devuelve un **JWT** (*access token*, caduca, por defecto 1 h) y un *refresh token* para renovarlo.

```ts
// Frontend con supabase-js — NO está en tus proyectos
await supabase.auth.signUp({ email, password });                 // registro (envía email de confirmación)
await supabase.auth.signInWithPassword({ email, password });     // login
await supabase.auth.signInWithOAuth({ provider: 'github' });     // login social
const { data: { user } } = await supabase.auth.getUser();        // ¿quién soy?
await supabase.auth.signOut();

supabase.auth.onAuthStateChange((event, session) => { /* SIGNED_IN, SIGNED_OUT... */ });
```

`supabase-js` guarda la sesión y **añade el JWT automáticamente** a cada petición
(`Authorization: Bearer <jwt>`).

### Comparado con tu Auth hecha a mano (proyectos 04/05 con `JWT_SECRET`)

| Paso | A mano (NestJS) | Supabase Auth |
|------|-----------------|---------------|
| Guardar usuario | `INSERT INTO users (email, password_hash)` con `argon2` | Automático en `auth.users` |
| Login | Comparar hash, firmar JWT con `JWT_SECRET` | `signInWithPassword` |
| Proteger ruta | `AuthGuard` verifica el JWT | El JWT llega a Postgres y **RLS** decide |
| Saber quién es | `req.user.id` | `auth.uid()` en SQL |
| Email de confirmación, reset de contraseña, OAuth | Tendrías que programarlo | Incluido |

---

## 5.2 Qué es Row Level Security

Los permisos clásicos de SQL (`GRANT SELECT ON books TO ...`) son **por tabla**:
o ves la tabla entera o no la ves. **RLS** añade permisos **por fila**:

```
                       SELECT * FROM books;
                               │
              ┌────────────────▼────────────────┐
              │  ¿RLS activado en books?        │
              └───────┬──────────────────┬──────┘
                   NO │                  │ SÍ
                      ▼                  ▼
            todas las filas      Postgres añade en secreto:
                                 WHERE <condición de la política>
                                 (si no hay ninguna política → 0 filas)
```

Es decir, una política es un **`WHERE` que Postgres añade automáticamente** a
cada consulta, según quién la hace.

### Activarlo

```sql
ALTER TABLE books ENABLE ROW LEVEL SECURITY;
```

> 📌 Activar RLS **sin políticas** = **nadie** (salvo roles que se lo saltan:
> `postgres`, `service_role`) puede ver nada. Es el estado "todo cerrado" y es el
> punto de partida correcto: luego abres lo justo.

---

## 5.3 Roles: quién está preguntando

Cada petición a la API de Supabase se ejecuta en Postgres con un **rol**:

| Rol | Cuándo | ¿Le afecta RLS? |
|-----|--------|-----------------|
| `anon` | Petición con la clave pública y **sin** sesión | ✅ Sí |
| `authenticated` | Petición con la clave pública **y** un JWT de usuario válido | ✅ Sí |
| `service_role` | Petición con la clave **secreta** | ❌ No (`BYPASSRLS`) |
| `postgres` | Tu API con `DATABASE_URL`, el SQL Editor | ❌ No (superusuario / dueño) |

Y dentro de una política puedes preguntar **quién** es:

| Función | Devuelve |
|---------|----------|
| `auth.uid()` | El `id` (UUID) del usuario del JWT, o `NULL` si es `anon` |
| `auth.jwt()` | El JWT completo como JSON (para leer `email`, `app_metadata.role`...) |

---

## 5.4 Escribir políticas

Sintaxis:

```sql
CREATE POLICY "nombre descriptivo"
ON tabla
FOR SELECT | INSERT | UPDATE | DELETE | ALL
TO anon | authenticated | ...
USING ( condición para filas EXISTENTES )        -- qué filas puedo ver / tocar
WITH CHECK ( condición para filas NUEVAS );      -- qué filas puedo escribir
```

- **`USING`** filtra las filas que **ya existen** (SELECT, UPDATE, DELETE).
- **`WITH CHECK`** valida las filas que **vas a escribir** (INSERT, UPDATE).

### Ejemplo completo: biblioteca personal (cada usuario sus libros)

```sql
-- 1. La tabla necesita saber de quién es cada libro
ALTER TABLE books
  ADD COLUMN user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE
  DEFAULT auth.uid();          -- al insertar, se rellena con el usuario actual

ALTER TABLE books ENABLE ROW LEVEL SECURITY;

-- 2. Cualquiera (incluso sin login) puede LEER el catálogo
CREATE POLICY "catálogo público"
ON books FOR SELECT
TO anon, authenticated
USING (true);

-- 3. Solo usuarios logueados pueden CREAR, y solo a su nombre
CREATE POLICY "crear mis libros"
ON books FOR INSERT
TO authenticated
WITH CHECK (user_id = auth.uid());

-- 4. Solo el dueño puede EDITAR (y no puede "regalar" el libro a otro)
CREATE POLICY "editar mis libros"
ON books FOR UPDATE
TO authenticated
USING (user_id = auth.uid())
WITH CHECK (user_id = auth.uid());

-- 5. Solo el dueño puede BORRAR
CREATE POLICY "borrar mis libros"
ON books FOR DELETE
TO authenticated
USING (user_id = auth.uid());
```

### Qué pasa en la práctica

```
Usuario Ana (uid = aaa...) hace:  supabase.from('books').delete().eq('id', 7)
Postgres ejecuta realmente:      DELETE FROM books WHERE id = 7 AND user_id = 'aaa...';
Si el libro 7 es de Luis:        0 filas borradas. Sin error. Simplemente no pasa nada.
```

> ⚠️ RLS **no da error** cuando filtra: devuelve menos filas o afecta a 0 filas.
> Si "no se borra / no se ve nada" y no hay error, sospecha primero de las políticas.

### La misma lógica en tu API NestJS (comparación)

```ts
// Equivalente a mano de la política "borrar mis libros"
async remove(id: number, userId: string) {
  const book = await this.findOne(id);                          // 404 si no existe
  if (book.userId !== userId) throw new ForbiddenException();  // 403 si no es suyo
  await this.pool.query('DELETE FROM books WHERE id = $1', [id]);
  return book;
}
```

| | Permisos en la API (forma A) | RLS (forma B) |
|---|---|---|
| Dónde vive la regla | En TS, en cada método | En SQL, una vez por tabla |
| Si olvidas la comprobación en un endpoint | Agujero en **ese** endpoint | Imposible: Postgres la aplica **siempre** |
| Mensajes de error claros (403 vs 404) | ✅ Tú decides | ❌ Filtra en silencio |
| Probar | Tests de la API | Consultas SQL con distintos roles |

---

## 5.5 Políticas habituales (recetario)

```sql
-- Perfil: cada usuario solo ve y edita su fila
CREATE POLICY "mi perfil" ON profiles FOR ALL TO authenticated
USING (id = auth.uid()) WITH CHECK (id = auth.uid());

-- Solo admins (rol guardado en app_metadata del JWT, que el usuario NO puede editar)
CREATE POLICY "admins todo" ON books FOR ALL TO authenticated
USING ((auth.jwt() -> 'app_metadata' ->> 'role') = 'admin');

-- Datos de equipo: ver filas de los equipos a los que pertenezco
CREATE POLICY "mis equipos" ON projects FOR SELECT TO authenticated
USING (team_id IN (SELECT team_id FROM team_members WHERE user_id = auth.uid()));
```

> ⚠️ Nunca bases permisos en `user_metadata`: el propio usuario puede modificarlo.
> Usa `app_metadata` (solo editable con la clave secreta) o una tabla de roles.

### Rendimiento

Una política se evalúa **por cada fila**. Dos trucos estándar:

```sql
-- 1. Índice en la columna de la política
CREATE INDEX books_user_id_idx ON books (user_id);

-- 2. Envolver auth.uid() en un SELECT para que se calcule una vez, no por fila
USING (user_id = (SELECT auth.uid()))
```

---

## 5.6 Probar políticas desde SQL

En el SQL Editor puedes simular ser un usuario:

```sql
BEGIN;
  SET LOCAL ROLE authenticated;
  SET LOCAL request.jwt.claims = '{"sub": "aaaaaaaa-0000-0000-0000-000000000000"}';
  SELECT * FROM books;              -- ¿qué vería ese usuario?
  DELETE FROM books WHERE id = 7;   -- ¿le dejaría?
ROLLBACK;                           -- no deja rastro
```

> 💡 *Dashboard → Advisors → Security Advisor* revisa tu proyecto y avisa de
> tablas en `public` sin RLS, políticas demasiado abiertas, etc. Pásalo antes de
> enseñar un proyecto.

---

## 5.7 ¿Y en tu proyecto (forma A)?

- Tu API entra como `postgres` → **RLS no le afecta**: la seguridad la ponen tus DTO, guards y servicios.
- Pero la API automática de Supabase **sí** existe para `books` → activa RLS **sin políticas** en las tablas que solo usa tu API. Resultado: cerrada para el mundo, abierta para tu NestJS.
- Si algún día quieres usar Supabase Auth con tu API propia: el frontend hace login con `supabase-js`, envía el JWT a tu API, y tu API lo verifica (con la clave pública/JWKS del proyecto) en un guard, en lugar de firmar tus propios tokens con `JWT_SECRET`.

---

## ✅ Resumen

- Supabase Auth = usuarios en `auth.users` + JWT; `auth.uid()` te dice quién pregunta.
- RLS = un `WHERE` automático por fila. Activado y sin políticas = todo cerrado.
- `USING` filtra filas existentes; `WITH CHECK` valida filas nuevas.
- `anon`/`authenticated` respetan RLS; `service_role` y `postgres` se lo saltan.
- RLS filtra **en silencio**. Indexa las columnas de las políticas.
