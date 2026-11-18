# 📋 CHEATSHEET — SUPABASE

> Chuleta rápida de comandos y código. Teoría en los documentos 01–07 de esta carpeta.

---

## 1. ⌨️ Comandos de terminal

### Probar la conexión

```bash
# psql contra el Session pooler (cadena entre comillas)
psql "postgresql://postgres.<REF>:<PASSWORD>@aws-0-eu-central-1.pooler.supabase.com:5432/postgres?sslmode=require"

# Sin instalar psql: usando el contenedor de clase
docker exec -it postgres-sql psql "postgresql://postgres.<REF>:<PASSWORD>@aws-0-eu-central-1.pooler.supabase.com:5432/postgres?sslmode=require"

# Script Node que lee el .env
node --env-file=.env probar-conexion.mjs
```

```sql
SELECT current_user, version();   -- ¿a dónde me he conectado?
\dt                               -- ¿están mis tablas?
```

### CLI de Supabase (`npx supabase ...`)

| Comando | Qué hace |
|---------|----------|
| `npx supabase login` | Autenticarte (abre el navegador) |
| `npx supabase init` | Crea la carpeta `supabase/` en el proyecto |
| `npx supabase link --project-ref <REF>` | Enlaza con el proyecto remoto |
| `npx supabase migration new create_books` | Crea `supabase/migrations/<fecha>_create_books.sql` |
| `npx supabase db push` | Aplica las migraciones pendientes en Supabase |
| `npx supabase db pull` | Genera una migración con el estado remoto actual |
| `npx supabase start` | Supabase completo en local (Docker) |
| `npx supabase db reset` | Recrea la BD local con migraciones + seed |
| `npx supabase db dump -f backup.sql` | Copia de seguridad |
| `npx supabase gen types typescript --project-id <REF> > src/database.types.ts` | Tipos TS de tus tablas |
| `npx supabase functions new hello` / `functions deploy hello` | Crear / desplegar Edge Function |
| `npx supabase secrets set CLAVE=valor` | Secretos de las Edge Functions |

```bash
# Backup con pg_dump (solo tu schema public)
pg_dump "postgresql://postgres.<REF>:<PASSWORD>@aws-0-eu-central-1.pooler.supabase.com:5432/postgres" \
  --schema=public --no-owner > backup.sql
```

---

## 2. Cadenas de conexión

```
postgresql://postgres.<REF>:<PASSWORD>@aws-0-eu-central-1.pooler.supabase.com:5432/postgres?sslmode=no-verify
             └─usuario+ref─┘            └──────── host del pooler (Frankfurt) ──┘ └puerto┘ └base┘ └─SSL─┘
```

| Tipo | Host | Puerto | Red | Para |
|------|------|:------:|-----|------|
| Direct connection | `db.<REF>.supabase.co` | 5432 | Solo IPv6 | Migraciones, `pg_dump` con IPv6 |
| **Session pooler** ✅ | `aws-0-<region>.pooler.supabase.com` | **5432** | IPv4 + IPv6 | **Servidor persistente (tu API en Render)** |
| Transaction pooler | `aws-0-<region>.pooler.supabase.com` | 6543 | IPv4 + IPv6 | Serverless (sin prepared statements) |

| `sslmode` | Cifra | Verifica certificado | Con `pg` |
|-----------|:-----:|:--------------------:|----------|
| *(nada)* / `disable` | ❌ | ❌ | Conecta en claro ⚠️ |
| `no-verify` | ✅ | ❌ | ✅ El de tu README |
| `require` | ✅ | ⚠️ como `verify-full` en `pg` reciente | ❌ `self-signed certificate` |
| `verify-full` | ✅ | ✅ | Necesita el CA de Supabase |

| Entorno | Dónde | `DATABASE_URL` |
|---------|-------|----------------|
| PC con Docker | `.env` | `postgres://alumno:<PASSWORD>@localhost:5432/bootcamp_api` |
| PC contra Supabase | `.env` | Session pooler + `?sslmode=no-verify` |
| Render | Panel *Environment* | Session pooler + `?sslmode=no-verify` |
| GitHub | — | Nada (solo `.env.example`) |

---

## 3. Conectar desde tu API (`pg`)

```ts
// database.module.ts (NestJS) — un Pool que no se cae si Postgres corta
function createPool() {
  const pool = new Pool({ connectionString: process.env.DATABASE_URL });
  pool.on('error', (error) => console.error('PostgreSQL:', error.message));   // sin esto, crash
  return pool;
}

@Module({
  providers: [{ provide: Pool, useFactory: createPool }],
  exports: [Pool],
})
export class DatabaseModule {}
```

```ts
// Opciones prudentes para el plan Free
new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 5,                          // máximo de conexiones
  idleTimeoutMillis: 30_000,       // cierra ociosas a los 30 s
  connectionTimeoutMillis: 10_000, // no esperar eternamente
});

// SSL verificado con el CA (URL SIN ?sslmode)
new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { ca: readFileSync('certs/supabase-ca.crt', 'utf8'), rejectUnauthorized: true },
});
```

```js
// probar-conexion.mjs
import pg from 'pg';
const pool = new pg.Pool({ connectionString: process.env.DATABASE_URL });
console.log((await pool.query('SELECT now(), current_database()')).rows);
await pool.end();
```

---

## 4. Esquema y SQL Editor

```sql
CREATE TABLE books (
  id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  author VARCHAR(100) NOT NULL,
  pages INTEGER NOT NULL CHECK (pages > 0),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

SELECT table_schema, table_name FROM information_schema.tables
WHERE table_name = 'books';                    -- debe estar en "public"

ALTER TABLE books ADD COLUMN user_id UUID
  REFERENCES auth.users(id) ON DELETE CASCADE; -- FK a usuarios de Supabase Auth
```

```yaml
# Docker local: cargar schema.sql la primera vez
volumes:
  - ./database/schema.sql:/docker-entrypoint-initdb.d/schema.sql:ro
```

---

## 5. API automática y claves

```bash
curl "https://<REF>.supabase.co/rest/v1/books?select=id,title&order=id" -H "apikey: <CLAVE_PUBLICA>"
```

| Clave | Quién | Dónde | Poder |
|-------|-------|-------|-------|
| `anon` / publishable | Navegador | Frontend (pública) | Lo que permita RLS |
| `service_role` / secret | Tu servidor | Variable de entorno secreta | TODO, ignora RLS |
| `DATABASE_URL` | Tu API con `pg` | Variable de entorno secreta | TODO (superusuario) |

---

## 6. `supabase-js`

```bash
npm install @supabase/supabase-js
```

```ts
import { createClient } from '@supabase/supabase-js';
export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,              // https://<REF>.supabase.co
  import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY,  // clave PÚBLICA
);
```

```ts
const { data, error } = await supabase.from('books').select('*');
if (error) throw error;                                            // ⚠️ siempre comprobar

supabase.from('books').select('id, title').order('id');            // SELECT ... ORDER BY
supabase.from('books').select('*').ilike('author', `%${author}%`); // filtro ILIKE
supabase.from('books').select('*').eq('id', id).single();          // uno (WHERE id =)
supabase.from('books').insert({ title, author, pages }).select().single();   // INSERT ... RETURNING
supabase.from('books').update({ pages: 500 }).eq('id', id).select().single(); // UPDATE
supabase.from('books').delete().eq('id', id);                      // DELETE
supabase.rpc('books_by_author_count');                             // llamar función SQL
```

```ts
// Tipado con los tipos generados
import type { Database } from './database.types';
const supabase = createClient<Database>(url, key);

// Servidor con clave secreta (nunca en el front)
const supabaseAdmin = createClient(process.env.SUPABASE_URL!, process.env.SUPABASE_SECRET_KEY!, {
  auth: { persistSession: false },
});
```

---

## 7. Auth

```ts
await supabase.auth.signUp({ email, password });               // registro
await supabase.auth.signInWithPassword({ email, password });   // login
await supabase.auth.signInWithOAuth({ provider: 'github' });   // login social
const { data: { user } } = await supabase.auth.getUser();      // ¿quién soy?
await supabase.auth.signOut();
supabase.auth.onAuthStateChange((event, session) => { /* SIGNED_IN, SIGNED_OUT */ });
```

---

## 8. Row Level Security (RLS)

```sql
ALTER TABLE books ENABLE ROW LEVEL SECURITY;      -- sin políticas = nadie (salvo postgres/service_role)

CREATE POLICY "catálogo público" ON books FOR SELECT
TO anon, authenticated USING (true);

CREATE POLICY "crear mis libros" ON books FOR INSERT
TO authenticated WITH CHECK (user_id = auth.uid());

CREATE POLICY "editar mis libros" ON books FOR UPDATE
TO authenticated USING (user_id = auth.uid()) WITH CHECK (user_id = auth.uid());

CREATE POLICY "borrar mis libros" ON books FOR DELETE
TO authenticated USING (user_id = auth.uid());

CREATE POLICY "admins todo" ON books FOR ALL TO authenticated
USING ((auth.jwt() -> 'app_metadata' ->> 'role') = 'admin');
```

| Cláusula | Se aplica a |
|----------|-------------|
| `USING (...)` | Filas **existentes** que puedo ver/tocar (SELECT, UPDATE, DELETE) |
| `WITH CHECK (...)` | Filas **nuevas** que puedo escribir (INSERT, UPDATE) |
| `auth.uid()` | Id del usuario del JWT |
| `TO anon` / `TO authenticated` | Sin login / con login |

```sql
CREATE INDEX books_user_id_idx ON books (user_id);    -- rendimiento
USING (user_id = (SELECT auth.uid()))                 -- se calcula una vez, no por fila

BEGIN;                                                -- probar como un usuario
  SET LOCAL ROLE authenticated;
  SET LOCAL request.jwt.claims = '{"sub": "aaaaaaaa-0000-0000-0000-000000000000"}';
  SELECT * FROM books;
ROLLBACK;
```

---

## 9. Storage, Realtime y Edge Functions

```ts
await supabase.storage.from('covers').upload(`${user.id}/${bookId}.jpg`, file, { upsert: true });
supabase.storage.from('covers').getPublicUrl(`${user.id}/${bookId}.jpg`);       // bucket público
await supabase.storage.from('invoices').createSignedUrl('2026/11/f-001.pdf', 60); // privado, 60 s
```

```sql
ALTER PUBLICATION supabase_realtime ADD TABLE books;   -- activar Realtime en la tabla
```

```ts
const channel = supabase
  .channel('books-changes')
  .on('postgres_changes', { event: '*', schema: 'public', table: 'books' },
      (payload) => console.log(payload.eventType, payload.new))
  .subscribe();
supabase.removeChannel(channel);                        // al desmontar

await supabase.functions.invoke('hello', { body: { name: 'Ana' } });   // llamar Edge Function
```

---

## ⚠️ Errores típicos

| Síntoma | Arreglo |
|---------|---------|
| `ENETUNREACH` / dirección `2a05:...` desde Render | Conexión directa (IPv6) → usar el **Session pooler** |
| `self-signed certificate in certificate chain` | `?sslmode=no-verify` (o CA + `verify-full`) |
| `Tenant or user not found` | Usuario sin `.<REF>` o cadena de otro proyecto |
| `relation "books" does not exist` | Ejecutar `schema.sql` en el SQL Editor / `db push` |
| `supabase-js` devuelve `[]` sin error | RLS activo sin política que lo permita |
| `data` es `null` y nadie avisa | Comprobar `error` tras cada llamada |
| Todo falla tras ~1 semana sin uso | Proyecto **pausado** → restaurarlo en el panel |
| La API automática expone tus tablas | `ENABLE ROW LEVEL SECURITY` en todas las de `public` |
