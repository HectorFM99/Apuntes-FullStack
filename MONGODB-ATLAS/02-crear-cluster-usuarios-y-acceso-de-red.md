# 02 — CREAR EL CLUSTER, USUARIOS Y ACCESO DE RED

> Para que tu API llegue a los datos de Atlas tiene que atravesar **dos puertas**,
> y las dos están cerradas por defecto:
>
> 1. 🧱 **La red** (*Network Access*): ¿se acepta una conexión desde esta IP?
> 2. 🔐 **La autenticación** (*Database Access*): ¿usuario y contraseña correctos? ¿qué puede hacer?
>
> Casi todos los fallos de conexión con Atlas son una de estas dos puertas.

```
 Tu API ──▶ 🧱 IP Access List ──▶ 🔐 Usuario + contraseña ──▶ 📜 Rol (permisos) ──▶ datos
            ¿tu IP está?          ¿son correctos?              ¿puedes leer/escribir
            NO → timeout          NO → "bad auth"               en bootcamp_api?
                 (30 s)                                         NO → "not authorized"
```

---

## 2.1 Crear el cluster M0

1. Cuenta en `mongodb.com/cloud/atlas` (con Google/GitHub o email).
2. Crear **organización** y **proyecto** (el asistente lo hace solo la primera vez).
3. **Create / Build a Cluster** → elegir **Free (M0)**.
4. **Proveedor y región**: la más cercana a tu Render (ej. AWS, Frankfurt).
5. **Nombre**: por defecto `Cluster0` → el host será `cluster0.<ID>.mongodb.net` (tu caso).
6. Esperar unos minutos a que se aprovisione.

> 📌 El asistente inicial (*Quickstart*) te pide **a la vez** crear un usuario y
> añadir tu IP. Es justo lo que se explica a continuación; si lo saltaste, se
> hace desde el menú lateral: **Security → Database Access** y **Security → Network Access**.

---

## 2.2 🔐 Database Users: quién entra en la base de datos

### Dos tipos de "usuario" que no hay que confundir

| | Usuario de **Atlas** (panel web) | Usuario de **base de datos** |
|---|---|---|
| Para qué | Entrar en `cloud.mongodb.com`, crear clusters, ver facturas | Que tu **API** / `mongosh` / Compass se conecten |
| Dónde se gestiona | *Organization / Project → Access Manager* | *Security → Database Access* |
| Credenciales | Tu email (+ Google/GitHub, 2FA) | Usuario + contraseña **que van en la URI** |
| Ejemplo | Tu cuenta personal | `<USUARIO>` en `mongodb+srv://<USUARIO>:<PASSWORD>@...` |

> ⚠️ Tu email y la contraseña de tu cuenta de Atlas **nunca** van en la URI. Van
> las de un **Database User**.

### Crear el usuario

**Security → Database Access → Add New Database User**

| Campo | Recomendación |
|-------|---------------|
| Método | **Password** |
| Usuario | Algo descriptivo: `api-render`, `api-local` |
| Contraseña | **Autogenerate Secure Password** (sin caracteres raros que rompan la URI) |
| Rol | Ver tabla de abajo |
| *Restrict access to specific clusters* | Opcional: limitarlo a `cluster0` |
| *Temporary user* | Útil para dar acceso a alguien durante unas horas |

### Roles: qué puede hacer

| Rol (built-in) | Puede | Úsalo para |
|----------------|-------|------------|
| `atlasAdmin` (*Atlas admin*) | Todo: crear usuarios, borrar bases... | Solo administración puntual |
| `readWriteAnyDatabase` (*Read and write to any database*) | Leer/escribir en **todas** las bases | Lo que el asistente pone por defecto |
| `readAnyDatabase` | Solo leer todas | Dashboards, informes |
| **`readWrite` sobre `bootcamp_api`** (*Specific Privileges*) | Leer/escribir **solo** en esa base | ✅ **Tu API en producción** |
| `read` sobre una base | Solo leer esa base | Un analista, Charts |

> 💡 **Principio de mínimo privilegio**: si alguien roba la `MONGODB_URI` de tu
> API, que solo pueda tocar `bootcamp_api`, no el resto del cluster. Además, con
> `readWrite` tu API puede crear índices (lo que hace tu `connectDb()`), así que
> no necesita más.

### Un usuario por "quién se conecta"

```
api-render  → readWrite@bootcamp_api   (va en Render)
api-local   → readWrite@bootcamp_api   (va en tu .env si trabajas contra Atlas)
admin-personal → atlasAdmin             (solo para ti, con Compass/mongosh)
```

Ventaja: si se filtra una URI, **borras ese usuario** y los demás siguen
funcionando. No tienes que cambiar la contraseña en todas partes.

### Sobre tu `.env` actual

La **forma** de tu `.env` de `express-mongo-atlas-render` (sin valores):

```
MONGODB_USERNAME=...
MONGODB_PASSWORD=...
MONGODB_URI="mongodb+srv://<USUARIO>:<PASSWORD>@cluster0.<ID>.mongodb.net"
```

- `MONGODB_USERNAME` y `MONGODB_PASSWORD` **no las lee el código** (`db.ts` solo usa `MONGODB_URI` y `DB_NAME`): son una copia de algo que ya está dentro de la URI. Duplicar secretos = más sitios por donde se pueden filtrar. Se pueden quitar.
- `DB_NAME` **no está**: funciona porque `db.ts` usa `process.env.DB_NAME ?? "bootcamp_api"`.

---

## 2.3 🧱 Network Access: desde dónde se puede entrar

Atlas tiene un cortafuegos: **IP Access List**. Solo acepta conexiones desde las
IPs (o rangos) de la lista. Por defecto está **vacía** → nadie entra.

**Security → Network Access → Add IP Address**

| Opción | Qué añade | Cuándo |
|--------|-----------|--------|
| **Add Current IP Address** | La IP pública de tu conexión actual (ej. `83.45.12.7/32`) | Trabajar desde tu PC |
| Una IP / rango concreto | `x.x.x.x/32` o un rango CIDR | Servidores con IP fija |
| **Allow Access from Anywhere** | `0.0.0.0/0` | Render Free (IPs de salida variables) |
| Temporal | Cualquiera de las anteriores con caducidad | Dar acceso puntual |

### Notación CIDR en 20 segundos

```
83.45.12.7/32   → exactamente esa IP (los 32 bits fijos)
83.45.12.0/24   → de 83.45.12.0 a 83.45.12.255 (256 IPs)
0.0.0.0/0       → TODAS las IPs de Internet
```

### Tu IP de casa **cambia**

La mayoría de conexiones domésticas tienen IP **dinámica**. Si ayer funcionaba y
hoy tu API local da *timeout* contra Atlas, lo más probable es que tu IP haya
cambiado. Atlas lo detecta: en *Network Access* te avisa "Your current IP
address is not added".

### El caso de Render

Tu API en Render se conecta **desde las IPs de salida de Render**, no desde la tuya:

```
Tu PC (83.45.12.7) ──▶ Atlas   ✅ si 83.45.12.7/32 está en la lista
Render (???)       ──▶ Atlas   ❌ timeout si no está su IP
```

Opciones:

1. **`0.0.0.0/0`** (lo habitual en el bootcamp): cualquier IP puede *intentar* conectar. La seguridad queda en **usuario + contraseña fuertes** y **TLS** (que Atlas siempre exige).
2. **IPs de salida de Render**: Render muestra los rangos de su región en *Settings → Outbound IPs* de tu servicio. Añadirlos a Atlas es más seguro, pero esos rangos son compartidos con otros clientes de Render y pueden cambiar.
3. **Red privada** (*VPC Peering / Private Endpoint*): solo en clusters dedicados y planes de pago de Render. Lo que se hace en empresa.

> ⚠️ `0.0.0.0/0` **no** es "sin seguridad", pero sí quita una capa. Por eso, si
> lo usas: contraseña autogenerada larga, usuario con rol **solo** sobre tu base,
> y nunca subir la URI a GitHub.

---

## 2.4 Las dos puertas, comparadas con Supabase

| | MongoDB Atlas | Supabase |
|---|---|---|
| Cortafuegos por IP | ✅ **Cerrado por defecto** (IP Access List) | Abierto por defecto (*Network Restrictions* opcionales) |
| Usuario de BD | Los que crees en *Database Access* | `postgres` (superusuario) + roles que crees |
| Roles por base de datos | `readWrite@bootcamp_api` | `GRANT` de Postgres |
| Permisos por fila | ❌ (lo hace tu API) | ✅ RLS |
| TLS | Siempre obligatorio | Opcional (*Enforce SSL*) |
| Problema típico desde Render | Olvidar `0.0.0.0/0` → timeout | IPv6 → usar el pooler |

---

## ✅ Checklist

- [ ] Cluster M0 creado en la región de tu Render.
- [ ] Usuario de BD (no tu cuenta de Atlas) con contraseña autogenerada.
- [ ] Rol mínimo: `readWrite` sobre `bootcamp_api`.
- [ ] Tu IP actual en Network Access (para trabajar en local).
- [ ] `0.0.0.0/0` (o las IPs de Render) para el despliegue.
- [ ] Secretos solo en la URI, no duplicados en otras variables.
