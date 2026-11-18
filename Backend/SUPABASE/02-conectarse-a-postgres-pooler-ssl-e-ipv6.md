# 02 — CONECTARSE A POSTGRES: POOLER, SSL E IPv6

> Es el archivo más práctico: lo que necesitas para que tu API NestJS (en tu PC o
> en Render) hable con el Postgres de Supabase. Detrás de la cadena "rara" que
> pegas en `DATABASE_URL` hay tres decisiones: **qué tipo de conexión**, **por qué
> red** (IPv4/IPv6) y **con qué cifrado** (SSL).

---

## 2.1 Anatomía de la cadena de conexión

### La forma REAL de tu `DATABASE_URL` de Supabase (contraseña y ref ocultos)

```
postgresql://postgres.<REF>:<PASSWORD>@aws-0-eu-central-1.pooler.supabase.com:5432/postgres
```

Desmontada:

```
postgresql://  postgres.<REF>  :  <PASSWORD>  @  aws-0-eu-central-1.pooler.supabase.com  :  5432  /  postgres  ?sslmode=no-verify
└─protocolo─┘  └──usuario────┘    └contraseña┘    └──────────── host ──────────────────┘    └puerto┘   └─base──┘  └─opciones─┘
```

| Parte | Valor | Por qué |
|-------|-------|---------|
| Usuario | `postgres.<REF>` | El pooler atiende a **miles de proyectos**: el `.<REF>` le dice a cuál vas |
| Host | `aws-0-eu-central-1.pooler.supabase.com` | Servidor del **pooler** (Supavisor) en Frankfurt |
| Puerto | `5432` | **Session mode** (con `6543` sería *transaction mode*) |
| Base | `postgres` | La base de datos por defecto de Supabase; tus tablas van en su schema `public` |
| `sslmode` | `no-verify` | Cifra la conexión sin validar el certificado (ver 2.5) |

Compárala con la de desarrollo:

### Ejemplo REAL (`nestjs-postgresql-supabase-render/.env.example`)

```
DATABASE_URL=postgres://alumno:TU_PASSWORD@localhost:5432/bootcamp_api
```

> 📌 **El código es el mismo.** Solo cambia la cadena. Eso es exactamente lo que
> permite que el mismo `database.module.ts` sirva en local y en producción.

> ⚠️ Si tu contraseña tiene caracteres especiales (`@`, `#`, `/`, `:`, `%`...),
> rompen la URL. Codifícalos (`@` → `%40`, `#` → `%23`...) con
> `encodeURIComponent('mi@pass')`, o usa una contraseña solo alfanumérica larga.

---

## 2.2 Los tres tipos de conexión de Supabase

En el panel: botón **Connect** → pestaña de cadenas de conexión.

| Tipo | Host | Puerto | Red | Para qué |
|------|------|--------|-----|----------|
| **Direct connection** | `db.<REF>.supabase.co` | 5432 | **Solo IPv6** (IPv4 es un *add-on* de pago) | Servidores con IPv6, migraciones, `pg_dump` |
| **Session pooler** | `aws-0-<region>.pooler.supabase.com` | 5432 | IPv4 + IPv6 | **Servidores persistentes en redes IPv4 → Render** ✅ |
| **Transaction pooler** | `aws-0-<region>.pooler.supabase.com` | 6543 | IPv4 + IPv6 | Serverless (Vercel, Lambda, Edge): muchas conexiones cortas |

### ¿Qué es un *connection pooler*?

Cada conexión a Postgres es un **proceso** en el servidor que consume memoria
(varios MB). El plan Free admite pocas conexiones reales simultáneas. Un pooler
se pone en medio:

```
Sin pooler:                           Con pooler (Supavisor):
API 1 ─conexión─┐                     API 1 ─┐
API 2 ─conexión─┼──▶ Postgres         API 2 ─┼──▶ Supavisor ══ pocas conexiones ══▶ Postgres
API 3 ─conexión─┤   (cada una un      API 3 ─┤   (reparte)      reales
...  (100)     ─┘    proceso: 💥)     ... (100)
```

### Session mode vs Transaction mode

```
SESSION MODE (5432)                         TRANSACTION MODE (6543)
cliente A ════════ conexión real 1          cliente A ── BEGIN..COMMIT ──▶ conexión real 1
(la tiene mientras esté conectado)          cliente A ── siguiente query ─▶ conexión real 2 (¡otra!)
                                            cliente B ── query ──────────▶ conexión real 1
```

| | Session | Transaction |
|---|---|---|
| Una conexión real por... | cliente conectado | transacción / consulta |
| Prepared statements, `SET`, `LISTEN`, tablas temporales | ✅ Funcionan | ❌ No fiables (cambias de conexión) |
| Escala a miles de clientes efímeros | Peor | ✅ Mejor |
| Ideal para | **Un servidor con un `Pool`** (tu API en Render) | Funciones serverless |

> 📌 Tu API NestJS en Render es un **servidor persistente** con **un `Pool`** que
> reutiliza conexiones → el **Session pooler** es la elección correcta (y es la que
> indica tu README).

---

## 2.3 IPv4 vs IPv6: por qué no sirve la conexión directa

- **IPv4**: direcciones tipo `18.192.10.5`. Se agotaron hace años y cuestan dinero.
- **IPv6**: direcciones tipo `2a05:d014:...`. Abundantes.

Supabase da a la conexión **directa** solo una dirección **IPv6**. Muchas redes
**no** pueden salir por IPv6, entre ellas **Render** (y muchas redes domésticas):

```
Render ──IPv4──▶ db.<REF>.supabase.co (solo IPv6)    ❌ ENETUNREACH / getaddrinfo ENOTFOUND
Render ──IPv4──▶ aws-0-eu-central-1.pooler.supabase.com (IPv4)   ✅
```

> 📌 Por eso el README de tu proyecto insiste en "la del **Session pooler**": no
> es una preferencia, es la única opción gratuita que funciona desde Render.

---

## 2.4 Tu código de conexión, explicado

### Ejemplo REAL (`nestjs-postgresql-supabase-render/src/database/database.module.ts`)

```ts
import { Module } from '@nestjs/common';
import { Pool } from 'pg';

// Crea el Pool y avisa (sin caerse) si PostgreSQL corta una conexión
function createPool() {
  const pool = new Pool({ connectionString: process.env.DATABASE_URL });

  // Si PostgreSQL se para, cierra las conexiones que no se estaban usando.
  // Sin esta línea, ese aviso tumbaría la API entera.
  pool.on('error', (error) => console.error('PostgreSQL:', error.message));

  return pool;
}

@Module({
  providers: [
    {
      provide: Pool, // cuando alguien pida un Pool...
      useFactory: createPool, // ...NestJS ejecuta esta función una vez
    },
  ],
  exports: [Pool], // otros módulos pueden pedir el Pool
})
export class DatabaseModule {}
```

Puntos clave **en relación con Supabase**:

1. **Un solo `Pool`** para toda la app (`useFactory` se ejecuta una vez). Cada `new Pool()` extra multiplica las conexiones al pooler. En el Free, el pooler limita las conexiones por proyecto; con el `max: 10` por defecto de `pg` vas sobrado.
2. **`connectionString`**: `pg` entiende la URL completa, incluido `?sslmode=...`.
3. **Conexión perezosa**: `new Pool()` **no** conecta todavía. La primera conexión se abre con la primera `pool.query(...)`. Por eso la API de Nest **arranca** aunque Supabase esté caído y falla después, en las peticiones.
4. **`pool.on('error')`**: Supavisor cierra conexiones inactivas tras un tiempo; sin este listener, ese cierre mataría el proceso (ver `Render/04`, 4.2).

### Opciones del `Pool` útiles con Supabase (NO están en tu proyecto)

```ts
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 5,                         // conexiones como máximo (sé prudente en el Free)
  idleTimeoutMillis: 30_000,      // cierra conexiones ociosas a los 30 s
  connectionTimeoutMillis: 10_000 // no esperes eternamente si Supabase no responde
});
```

---

## 2.5 SSL: cifrar la conexión (y el famoso `sslmode=no-verify`)

Tu API y Supabase se comunican **por Internet**. Sin SSL/TLS, la contraseña y
los datos viajan **en claro**. SSL hace dos cosas distintas:

1. **Cifrar** el canal (nadie en medio puede leerlo).
2. **Verificar** que el servidor es quien dice ser (comprobando su certificado contra una autoridad de confianza, CA).

| `sslmode` | ¿Cifra? | ¿Verifica certificado? | Con `pg` y Supabase |
|-----------|---------|------------------------|---------------------|
| *(ninguno)* | ❌ (en `pg`, sin SSL) | ❌ | Conecta, pero **en claro** ⚠️ |
| `disable` | ❌ | ❌ | Ídem |
| `no-verify` | ✅ | ❌ | ✅ Funciona — lo que usa tu README |
| `require` | ✅ | ⚠️ En versiones recientes de `pg` se trata **como `verify-full`** | ❌ `self-signed certificate in certificate chain` |
| `verify-full` | ✅ | ✅ (y comprueba el nombre del host) | Necesita el **certificado CA de Supabase** |

### ¿Por qué falla `require`?

El certificado de Supabase está firmado por **la CA propia de Supabase**, no por
una autoridad pública que Node conozca. Si `pg` intenta verificarlo, no encuentra
de quién fiarse → `self-signed certificate in certificate chain`.
`no-verify` dice: "cifra, pero no compruebes el certificado".

> 📌 `no-verify` protege contra alguien que **escucha** la red, pero no contra
> alguien que **suplanta** al servidor (*man-in-the-middle*). Para un proyecto de
> bootcamp es el equilibrio razonable. Para producción seria, verificación completa:

### Verificación completa (NO está en tu proyecto)

1. Supabase → *Database Settings* → **SSL Configuration** → descargar el certificado (`prod-ca-2021.crt` o similar).
2. Guardarlo en el repo (no es secreto: es un certificado **público**) o como *Secret File* en Render.
3. Configurar `pg`:

```ts
import { readFileSync } from 'node:fs';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,   // SIN ?sslmode en la URL
  ssl: {
    ca: readFileSync('certs/supabase-ca.crt', 'utf8'),
    rejectUnauthorized: true,
  },
});
```

> ⚠️ Gotcha de `pg`: si la URL lleva `?sslmode=...` **y** además pasas un objeto
> `ssl`, los parámetros de la URL pueden **sobrescribir** tu objeto. Usa una de las
> dos formas, no ambas.

> 💡 Supabase permite activar **Enforce SSL** en *Database Settings*: a partir de
> ahí rechaza conexiones sin cifrar. Tu `.env` local de Supabase no lleva
> `sslmode`; funciona porque esa opción viene desactivada, pero viaja en claro.
> Añadir `?sslmode=no-verify` también en local es lo coherente.

---

## 2.6 Probar la conexión fuera de tu API

Antes de culpar al código, prueba la cadena directamente:

```bash
# Con psql (si lo tienes) — pon la cadena entre comillas
psql "postgresql://postgres.<REF>:<PASSWORD>@aws-0-eu-central-1.pooler.supabase.com:5432/postgres?sslmode=require"

# Con el contenedor Docker de clase, sin instalar nada
docker exec -it postgres-sql psql "postgresql://postgres.<REF>:<PASSWORD>@aws-0-eu-central-1.pooler.supabase.com:5432/postgres?sslmode=require"
```

```sql
SELECT current_user, version();   -- ¿a dónde me he conectado?
\dt                               -- ¿están mis tablas?
```

> 📌 `psql` es de `libpq`, donde `sslmode=require` **sí** significa "cifrar sin
> verificar". La diferencia de significado de `require` entre `psql` y `pg` (Node)
> es justo lo que confunde a todo el mundo.

O con un script de Node de tres líneas:

```js
// probar-conexion.mjs   →   node --env-file=.env probar-conexion.mjs
import pg from 'pg';
const pool = new pg.Pool({ connectionString: process.env.DATABASE_URL });
console.log((await pool.query('SELECT now(), current_database()')).rows);
await pool.end();
```

---

## 2.7 Dónde va cada cadena

| Entorno | Variable | Valor |
|---------|----------|-------|
| Tu PC con Docker | `.env` → `DATABASE_URL` | `postgres://alumno:...@localhost:5432/bootcamp_api` |
| Tu PC contra Supabase | `.env` → `DATABASE_URL` | Session pooler + `?sslmode=no-verify` |
| Render | Panel *Environment* → `DATABASE_URL` | Session pooler + `?sslmode=no-verify` |
| GitHub | **Nada** | `.env` está en `.gitignore`; solo se sube `.env.example` |

> ⚠️ La `DATABASE_URL` de Supabase usa el usuario `postgres`, que es
> **superusuario** de tu proyecto: se salta RLS y puede borrar cualquier tabla.
> Trátala como la contraseña más importante del proyecto. Si se filtra (por
> ejemplo, la subes a GitHub por error): Supabase → *Database Settings* →
> **Reset database password**, y actualiza la variable en Render.

---

## ✅ Resumen

- La cadena de Supabase = usuario `postgres.<REF>` + host del **pooler** + puerto + `sslmode`.
- **Direct** (IPv6) no funciona desde Render → usa **Session pooler** (5432).
- **Transaction pooler** (6543) es para serverless; rompe prepared statements.
- `pg` + `sslmode=require` = verifica certificado → falla; `no-verify` cifra sin verificar.
- Un solo `Pool`, `pool.on('error')`, y la contraseña de `postgres` como secreto máximo.
