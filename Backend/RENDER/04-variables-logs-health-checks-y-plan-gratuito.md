# 04 — VARIABLES, LOGS, HEALTH CHECKS Y PLAN GRATUITO

> Una vez desplegado, el servicio hay que **operarlo**: cambiar configuración,
> ver qué está pasando, saber si está sano y entender por qué a veces tarda
> 50 segundos en responder.

---

## 4.1 Variables de entorno (pestaña **Environment**)

### Cómo llegan a tu código

```
Panel de Render                         Tu proceso Node
┌───────────────────────────┐           ┌──────────────────────────────────────┐
│ DATABASE_URL = postgres...│ ────────▶ │ process.env.DATABASE_URL             │
│ NODE_VERSION = 22         │  al       │ process.env.PORT  (la pone Render)   │
│                           │  arrancar │ process.env.RENDER = 'true'          │
└───────────────────────────┘           │ process.env.RENDER_EXTERNAL_URL ...  │
                                        └──────────────────────────────────────┘
```

- Se leen **al arrancar**. Cambiar una variable ⇒ hay que redesplegar/reiniciar.
- Son **texto**: `process.env.MAX_ITEMS` es `"20"`, no `20` → `Number(process.env.MAX_ITEMS)`.
- Render añade algunas propias, útiles para saber dónde estás: `RENDER=true`, `RENDER_EXTERNAL_URL`, `RENDER_GIT_COMMIT`...

```ts
// Ejemplo: comportamiento distinto en Render (NO está en tus proyectos)
if (process.env.RENDER) console.log('Desplegado en', process.env.RENDER_EXTERNAL_URL);
```

### Variables que usan tus proyectos

| Proyecto | Variable | Valor en Render | De dónde sale |
|----------|----------|-----------------|---------------|
| NestJS + Supabase | `DATABASE_URL` | `postgresql://postgres.<REF>:<PASSWORD>@aws-0-eu-central-1.pooler.supabase.com:5432/postgres?sslmode=no-verify` | Supabase → *Connect* → **Session pooler** |
| NestJS (proyectos 04/05 con login) | `JWT_SECRET` | Cadena larga aleatoria | La generas tú (ver abajo) |
| Express + Mongo | `MONGODB_URI` | `mongodb+srv://<user>:<PASSWORD>@<cluster>.mongodb.net/...` | Atlas → *Connect* → *Drivers* |
| Express + Mongo | `DB_NAME` | `bootcamp_api` | Lo eliges tú |

```bash
# Generar un JWT_SECRET decente
node -e "console.log(require('crypto').randomBytes(48).toString('base64url'))"
```

> ⚠️ El `JWT_SECRET` de producción **no** debe ser el mismo que el de tu `.env`
> local, ni algo como `secreto123`. Quien lo conozca puede fabricar tokens
> válidos y hacerse pasar por cualquier usuario.

### Environment Groups y Secret Files

- **Environment Group**: un conjunto de variables compartido entre varios servicios (ej. la misma `DATABASE_URL` para la API y un Cron Job).
- **Secret Files**: archivos completos (ej. un certificado `.pem`) que Render coloca en `/etc/secrets/<nombre>` y en la raíz del servicio. Útil para el certificado CA de Supabase si quieres verificación SSL completa (ver `Supabase/02`).

---

## 4.2 Logs

La pestaña **Logs** muestra en tiempo real **todo lo que tu app escribe por
consola** (`console.log`, `console.error`, el logger de Nest) más los mensajes de
Render (los que empiezan por `==>`).

### Qué loguean tus proyectos

### Ejemplo REAL (`express-mongo-atlas-render/src/index.ts`)

```ts
// 1. Registrar cada petición en la terminal
app.use((req, _res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
...
console.error(err);   // en el middleware de errores
res.status(500).json({ error: "Error interno del servidor" });
```

### Ejemplo REAL (`nestjs-postgresql-supabase-render/src/database/database.module.ts`)

```ts
// Si PostgreSQL se para, cierra las conexiones que no se estaban usando.
// Sin esta línea, ese aviso tumbaría la API entera.
pool.on('error', (error) => console.error('PostgreSQL:', error.message));
```

> 📌 Fíjate en el patrón: el **cliente** recibe un mensaje genérico (`"Error
> interno del servidor"`) y el **detalle** va al log. Así no filtras información
> interna (nombres de tablas, rutas, contraseñas en cadenas de conexión) y tú
> sigues pudiendo depurar en la pestaña Logs.

### `pool.on('error')` explicado a fondo

El `Pool` de `pg` mantiene conexiones abiertas aunque no las uses. Supabase (o su
*pooler*) puede **cerrar** una conexión inactiva en cualquier momento (reinicio,
mantenimiento, *timeout*). `pg` emite entonces un evento `'error'` en el Pool.
En Node, un `EventEmitter` que emite `'error'` **sin nadie escuchando lanza una
excepción que mata el proceso**. Con el `pool.on('error', ...)`:

```
Sin listener:  Supabase cierra conexión → 'error' sin manejar → 💥 proceso muerto → Render reinicia
Con listener:  Supabase cierra conexión → se loguea → el Pool abre otra cuando haga falta ✅
```

En un servidor que está horas encendido, esta línea es **obligatoria**.

---

## 4.3 Health Check

**Settings → Health Check Path**: una ruta que Render llama periódicamente.

- Si responde **2xx/3xx** → el servicio está sano.
- Si falla durante un rato → Render lo marca como caído y **lo reinicia**.
- En cada despliegue, Render **no** manda tráfico a la versión nueva hasta que el health check responde.

Tus proyectos ya tienen una ruta válida:

| Proyecto | Ruta | Respuesta |
|----------|------|-----------|
| NestJS | `/api` | `Hello World!` (el `AppController` + prefijo global `api`) |
| Express | `/` | `Mi primera API con Express` |

### Un health check que comprueba la base de datos (mejora, NO está en tus proyectos)

```ts
// health.controller.ts (NestJS)
@Controller('health')
export class HealthController {
  constructor(private readonly pool: Pool) {}

  @Get()
  async check() {
    await this.pool.query('SELECT 1');   // si la BD no responde, lanza → 500
    return { status: 'ok' };
  }
}
```

> 💡 Debate habitual: si la BD cae y el health check la incluye, Render reiniciará
> tu API en bucle aunque el fallo no sea suyo. Muchos equipos tienen dos rutas:
> `/health` (¿está vivo el proceso?) para Render y `/health/db` para monitorizar.

---

## 4.4 El plan gratuito: qué esperar

> ⚠️ Los límites cambian con el tiempo. Compruébalos en `render.com/pricing` y
> `render.com/docs/free`. Estos son los habituales:

| Característica | Plan Free (Web Service) |
|----------------|-------------------------|
| RAM / CPU | 512 MB / 0,1 CPU |
| Horas de instancia | ~750 h/mes por workspace (suficiente para 1 servicio 24/7) |
| **Suspensión por inactividad** | Se **duerme** tras **15 min** sin peticiones |
| Despertar (*cold start*) | ~30-60 s en la primera petición |
| Disco persistente | ❌ No |
| Shell / SSH al contenedor | ❌ No |
| Postgres Free de Render | Expira a los **30 días** (por eso usas **Supabase**) |

### El *cold start*, visualmente

```
10:00  GET /api/books  →  200 en 80 ms
10:01  ...nadie usa la API durante 15 minutos...
10:16  💤 Render apaga la instancia
11:30  GET /api/books  →  ⏳ Render arranca el contenedor (node dist/main, conectar a BD...)
                       →  200 en 45 s   ← el usuario cree que "no funciona"
11:31  GET /api/books  →  200 en 80 ms  ← ya está despierta
```

Cómo convivir con ello:

- En una demo o entrevista, **abre la URL un minuto antes** para despertarla.
- En el frontend, muestra un "Cargando…" en vez de dar error por *timeout*.
- Los "pingers" externos que la llaman cada 10 min la mantienen despierta, pero consumen tus 750 h y van contra el espíritu del plan gratuito.
- Para algo real: plan de pago (sin suspensión).

> 📌 Los **Static Sites** no se duermen: son archivos servidos por CDN.

---

## 4.5 Dominio propio y HTTPS

- Cada servicio tiene gratis `https://<name>.onrender.com` con certificado TLS automático.
- **Settings → Custom Domains** → añades `api.tudominio.com` → Render te dice qué registro DNS crear (`CNAME`) → emite el certificado solo.

> 📌 HTTPS no es opcional: los navegadores bloquean que una web `https://` llame a
> una API `http://` (*mixed content*). Render lo resuelve por ti.

---

## 4.6 Redes: IPs de salida y bases de datos externas

Cuando tu API conecta con Atlas o Supabase, la conexión **sale** de los
servidores de Render:

- **MongoDB Atlas** tiene lista blanca de IPs (*Network Access*). Las IPs de salida de Render en el plan Free son compartidas y pueden cambiar, por eso en el bootcamp se suele permitir `0.0.0.0/0` (cualquier IP). La seguridad recae entonces en **usuario + contraseña fuertes**. Render publica sus rangos de IP de salida por región en *Settings → Outbound IPs* si quieres restringirlo.
- **Supabase**: Render **no tiene salida IPv6**, y la conexión *directa* de Supabase es solo IPv6 → por eso hay que usar el **Session pooler** (IPv4). Explicado a fondo en `Supabase/02`.

---

## ✅ Resumen

- Variables: se leen al arrancar, son texto, cambiar una = redesplegar.
- Logs: el detalle del error va al log; al cliente, un mensaje genérico.
- `pool.on('error')` evita que una conexión cortada mate la API.
- Health check: una ruta rápida que devuelve 200.
- Plan Free: se **duerme** a los 15 min → *cold start* de ~1 min.
- Render no tiene IPv6 de salida → Supabase vía **pooler**.
