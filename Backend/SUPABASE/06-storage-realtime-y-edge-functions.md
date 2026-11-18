# 06 — STORAGE, REALTIME Y EDGE FUNCTIONS

> Las tres piezas que convierten "una base de datos con API" en "un backend
> completo". Ninguna la usan tus proyectos todavía, pero son las que más se
> preguntan y las que resuelven problemas que con Render solo no tienes resueltos
> (por ejemplo: dónde guardar archivos si el disco de Render es efímero).

---

## 6.1 Storage: archivos

### El problema

En Render el disco **se borra** en cada despliegue (`Render/01`, 1.3). Si tu API
guarda la portada de un libro en `uploads/`, desaparecerá. Los archivos tienen
que ir a un **almacenamiento de objetos** externo (tipo Amazon S3).

### Conceptos

| Concepto | Qué es | Ejemplo |
|----------|--------|---------|
| **Bucket** | "Carpeta raíz" con su propia configuración | `covers` |
| **Objeto** | Un archivo dentro del bucket, identificado por su ruta | `covers/user-aaa/7.jpg` |
| **Bucket público** | Cualquiera puede **descargar** con la URL | Portadas, avatares |
| **Bucket privado** | Descargar requiere permiso o una **URL firmada** temporal | Facturas, documentos |

Los metadatos de cada archivo son filas de la tabla `storage.objects` → los
permisos se escriben **con políticas RLS**, igual que en `05`.

```ts
// Subir (frontend, usuario logueado) — NO está en tus proyectos
const { data, error } = await supabase.storage
  .from('covers')
  .upload(`${user.id}/${bookId}.jpg`, file, { upsert: true });

// URL pública (bucket público)
const { data: { publicUrl } } = supabase.storage.from('covers').getPublicUrl(`${user.id}/${bookId}.jpg`);

// URL temporal (bucket privado), válida 60 s
const { data: signed } = await supabase.storage.from('invoices').createSignedUrl('2026/11/f-001.pdf', 60);
```

```sql
-- Política: cada usuario solo puede subir dentro de SU carpeta
CREATE POLICY "subir a mi carpeta"
ON storage.objects FOR INSERT TO authenticated
WITH CHECK (
  bucket_id = 'covers'
  AND (storage.foldername(name))[1] = auth.uid()::text
);
```

> 📌 Patrón habitual: en la tabla `books` guardas **solo la ruta** (`cover_path`)
> y el archivo vive en Storage. Nunca guardes imágenes en columnas de Postgres
> (`BYTEA`): engordan la BD (500 MB en el Free) y las consultas.

---

## 6.2 Realtime: la base de datos te avisa

### El problema

Sin tiempo real, para ver libros nuevos el frontend tiene que **preguntar**
cada X segundos (*polling*). Con Realtime, el servidor **empuja** el cambio por
un WebSocket en cuanto ocurre.

```
POLLING                                  REALTIME
cliente ─ ¿hay algo nuevo? ─▶ API        cliente ══ WebSocket abierto ══ Supabase Realtime
cliente ◀── no ──────────────            ...                                  ▲
cliente ─ ¿hay algo nuevo? ─▶ API        INSERT INTO books ─▶ Postgres ───────┘ (lee el WAL)
cliente ◀── no ──────────────            cliente ◀══ {"eventType":"INSERT","new":{...}}
(peticiones constantes)                  (solo hay tráfico cuando hay cambios)
```

### Tres modos

| Modo | Para qué | Pasa por la BD |
|------|----------|----------------|
| **Postgres Changes** | Escuchar `INSERT`/`UPDATE`/`DELETE` de una tabla | ✅ (y respeta RLS) |
| **Broadcast** | Mensajes efímeros entre clientes (chat, cursores) | ❌ |
| **Presence** | Quién está conectado ahora | ❌ |

```sql
-- 1. Permitir que Realtime emita los cambios de la tabla
ALTER PUBLICATION supabase_realtime ADD TABLE books;
```

```ts
// 2. Suscribirse desde el frontend — NO está en tus proyectos
const channel = supabase
  .channel('books-changes')
  .on(
    'postgres_changes',
    { event: '*', schema: 'public', table: 'books' },
    (payload) => {
      console.log(payload.eventType, payload.new, payload.old);
      // actualizar el estado de React aquí
    },
  )
  .subscribe();

// 3. Al desmontar el componente
supabase.removeChannel(channel);
```

> 📌 ¿Funciona si los cambios los hace **tu API NestJS** con `pg`? **Sí**: Realtime
> escucha el registro de cambios de Postgres (WAL), no la API. Da igual quién
> haga el `INSERT`.

> ⚠️ Realtime respeta RLS: un usuario solo recibe los cambios de las filas que
> podría ver con un `SELECT`. Si no llega nada, revisa la publicación y las políticas.

---

## 6.3 Edge Functions: tu código en Supabase

### El problema

Hay cosas que **no pueden** hacerse en el frontend aunque tengas RLS:

- Usar una clave secreta de otro servicio (Stripe, OpenAI, SendGrid...).
- Recibir *webhooks* de terceros (Stripe avisa de un pago).
- Lógica compleja o que no debe ser manipulable (calcular un precio).

Opción 1: tu API en Render (forma A). Opción 2: una **Edge Function**.

### Qué son

Funciones TypeScript que se despliegan en Supabase y responden a una URL:
`https://<REF>.supabase.co/functions/v1/<nombre>`.

| | Tu API en Render | Edge Function |
|---|---|---|
| Runtime | **Node.js** | **Deno** (TS nativo, `import` por URL o `npm:`) |
| Modelo | Proceso siempre encendido (salvo *sleep* del Free) | Se ejecuta **por petición** (serverless) |
| Estado en memoria | Persiste entre peticiones | No: cada ejecución puede ser nueva |
| Límite de tiempo | Ninguno práctico | Segundos (tiempo de CPU/ejecución limitado) |
| Framework | NestJS, Express... | Un `handler(req) => Response` |

```ts
// supabase/functions/hello/index.ts — NO está en tus proyectos
import { createClient } from 'npm:@supabase/supabase-js@2';

Deno.serve(async (req) => {
  const { name } = await req.json();

  // Cliente que actúa COMO el usuario que llama (respeta RLS)
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_ANON_KEY')!,
    { global: { headers: { Authorization: req.headers.get('Authorization')! } } },
  );

  const { data: books } = await supabase.from('books').select('title').limit(3);

  return new Response(JSON.stringify({ message: `Hola ${name}`, books }), {
    headers: { 'Content-Type': 'application/json' },
  });
});
```

```bash
npx supabase functions new hello
npx supabase functions deploy hello
npx supabase secrets set STRIPE_SECRET_KEY=sk_...   # variables secretas (≈ Environment de Render)
```

```ts
// Llamarla desde el frontend (envía el JWT del usuario automáticamente)
const { data, error } = await supabase.functions.invoke('hello', { body: { name: 'Ana' } });
```

> 📌 Comparación clave: una Edge Function es como **un solo endpoint** de tu API
> NestJS, pero sin servidor que mantener. Para una API grande y estructurada
> (módulos, DTOs, guards), NestJS en Render sigue siendo más ordenado.

---

## 6.4 Otras piezas que conviene conocer

| Pieza | Qué hace |
|-------|----------|
| **Database Functions + Triggers** | Lógica en SQL que se ejecuta sola (ej. al registrarse un usuario, crear su fila en `profiles`) |
| **Database Webhooks** | Llamar a una URL (tu API en Render, una Edge Function) cuando cambia una tabla |
| **Cron** (`pg_cron`) | Tareas programadas dentro de Postgres (≈ Cron Job de Render) |
| **Queues** (`pgmq`) | Colas de mensajes en Postgres |
| **Vault** | Guardar secretos cifrados dentro de la BD |
| **pgvector** | Búsquedas por similitud (IA, *embeddings*) |
| **GraphQL** (`pg_graphql`) | API GraphQL generada, igual que la REST |

```sql
-- Ejemplo clásico de trigger: crear el perfil al registrarse
CREATE FUNCTION public.handle_new_user()
RETURNS trigger LANGUAGE plpgsql SECURITY DEFINER SET search_path = '' AS $$
BEGIN
  INSERT INTO public.profiles (id, email) VALUES (NEW.id, NEW.email);
  RETURN NEW;
END;
$$;

CREATE TRIGGER on_auth_user_created
AFTER INSERT ON auth.users
FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

---

## ✅ Resumen

- **Storage**: archivos fuera del disco efímero de Render; permisos con RLS sobre `storage.objects`; en tus tablas guardas solo la ruta.
- **Realtime**: el WAL de Postgres empujado por WebSocket; añade la tabla a `supabase_realtime`; respeta RLS.
- **Edge Functions**: endpoints serverless en Deno para secretos de terceros y webhooks.
- Triggers, cron, webhooks y vectores: todo **dentro de Postgres**.
