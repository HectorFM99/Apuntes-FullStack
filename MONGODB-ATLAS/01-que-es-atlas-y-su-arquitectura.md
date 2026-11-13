# 01 — QUÉ ES MONGODB ATLAS Y SU ARQUITECTURA

> Atlas es el servicio oficial de **MongoDB en la nube**, hecho por la propia
> empresa MongoDB. Tú no instalas ni mantienes servidores: pides un **cluster**,
> eliges dónde y de qué tamaño, y recibes una cadena de conexión. Por debajo hay
> bastante más de lo que parece: tres servidores, copias automáticas de los
> datos, cifrado y un cortafuegos.

---

## 1.1 Qué es un DBaaS

**DBaaS** (*Database as a Service*): una base de datos gestionada. Es la misma
idea que Supabase usado como "Postgres en la nube" (forma A), pero para MongoDB.

| Tarea | Mongo en Docker (tu PC) | Mongo en un servidor propio | **Atlas** |
|-------|-------------------------|-----------------------------|-----------|
| Instalar MongoDB | `docker compose up -d` | Instalar y configurar a mano | ✅ Automático |
| Siempre encendido / accesible por Internet | ❌ Solo tu PC | Tú (y abrir puertos) | ✅ |
| Réplicas (si un servidor cae, otro sigue) | ❌ 1 servidor | Montar un *replica set* a mano | ✅ 3 nodos siempre |
| Cifrado TLS | ❌ | Configurar certificados | ✅ Obligatorio |
| Actualizaciones de versión | Tú | Tú | ✅ Automáticas |
| Backups | Tú (`mongodump`) | Tú | ✅ En planes de pago |
| Monitorización | ❌ | Montar herramientas | ✅ Gráficas y alertas |
| Coste | Gratis | Servidor + tu tiempo | Gratis (M0) → de pago |

> 📌 La razón práctica en el bootcamp: **Render no puede conectarse a tu Docker**.
> Tu API desplegada necesita una base de datos que también esté en Internet, y
> Atlas te da una gratis.

---

## 1.2 La jerarquía: organización → proyecto → cluster → base de datos

```
Organización  (tu cuenta / empresa; facturación)
└── Proyecto  "Bootcamp"   ← usuarios de BD y Network Access se configuran AQUÍ
    ├── Cluster  "cluster0"   (M0, AWS, Frankfurt)
    │   ├── Base de datos  bootcamp_api      ← DB_NAME
    │   │   ├── Colección  products
    │   │   └── Colección  categories
    │   ├── Base de datos  admin            (interna)
    │   └── Base de datos  local            (interna, replicación)
    └── Cluster  "cluster-pruebas" ...
```

| Nivel | Qué contiene | Ojo con |
|-------|--------------|---------|
| **Organización** | Proyectos, facturación, miembros | Los "miembros" son personas que entran **al panel web**, no a la BD |
| **Proyecto** | Clusters, **Database Users**, **Network Access**, alertas | Usuarios y reglas de red valen para **todos** los clusters del proyecto |
| **Cluster** | Los servidores de MongoDB | Su nombre forma el host: `cluster0.<ID>.mongodb.net` |
| **Base de datos / colección** | Tus datos | Se crean **solas** al insertar el primer documento (como en Docker) |

> 💡 Buena práctica: **un proyecto por entorno o por aplicación** (ej.
> `bootcamp-dev`, `bootcamp-prod`). Así un usuario o una regla de red de pruebas
> no da acceso a producción.

---

## 1.3 El replica set: por qué tu "cluster" son 3 servidores

Incluso el cluster gratuito es un **replica set** de 3 nodos:

```
                    escrituras y lecturas (por defecto)
   tu API ─────────────────────────┐
                                   ▼
                          ┌─────────────────┐
                          │    PRIMARY      │  ← el único que acepta escrituras
                          └───┬─────────┬───┘
                 replica (oplog)│         │replica (oplog)
                          ┌─────▼───┐ ┌───▼──────┐
                          │SECONDARY│ │SECONDARY │  ← copias casi en tiempo real
                          └─────────┘ └──────────┘

   Si el PRIMARY cae → los otros dos VOTAN → uno pasa a PRIMARY (en segundos)
```

- **Primary**: recibe todas las escrituras.
- **Secondaries**: copian los cambios del primary leyendo su **oplog** (registro de operaciones).
- **Elección automática**: si el primary falla (o Atlas lo reinicia para actualizarlo), un secondary toma el relevo.

### Consecuencias que notas como programador

| Concepto | Qué significa | Dónde aparece |
|----------|---------------|---------------|
| **Alta disponibilidad** | Atlas puede actualizar/reiniciar nodos sin que tu app se caiga | Mantenimientos sin cortes |
| **`retryWrites=true`** | Si una escritura falla por un cambio de primary, el driver la **reintenta una vez** sola | Viene en la URI que te da Atlas (y es el valor por defecto del driver) |
| **`w=majority`** | Una escritura se confirma cuando la tienen **la mayoría** de nodos (2 de 3): no se pierde si cae el primary | En la URI de Atlas |
| **Transacciones multi-documento** | Solo funcionan en replica sets | En Atlas sí; en tu Docker de 1 nodo **no** |
| **Change Streams** | Escuchar cambios en tiempo real (≈ Realtime de Supabase) | Requieren replica set → Atlas sí, tu Docker no |

> 📌 Esta es la diferencia de fondo entre tu `compose.yaml` y Atlas: en local
> tienes **un** servidor suelto (*standalone*); en Atlas, un **replica set**. El
> CRUD funciona igual, pero transacciones y change streams solo en Atlas.

---

## 1.4 Tipos de cluster (tiers)

> ⚠️ Nombres, precios y límites cambian; compruébalos en `mongodb.com/pricing`.

| Tier | Para qué | Características (aprox.) |
|------|----------|--------------------------|
| **M0 (Free)** | Aprender, bootcamp, portfolio | **Gratis para siempre**, 512 MB de almacenamiento, recursos compartidos, 1 por proyecto, sin backups en la nube, rendimiento limitado |
| **Flex** | Proyectos pequeños / prototipos | Pago por uso con tope mensual, algo más de capacidad que M0 |
| **Dedicated (M10, M20...)** | Producción | Servidores dedicados, backups continuos, escalado, red privada, más métricas |

### ¿Qué significa "compartido" en M0?

Tu cluster M0 corre en máquinas **compartidas** con otros usuarios. Por eso:

- Hay límites de **conexiones** simultáneas (del orden de 500) y de **operaciones por segundo**.
- Algunas funciones avanzadas no están disponibles o están limitadas (ciertas métricas, *Performance Advisor*, backups).
- No eliges la versión exacta de MongoDB.

Para una API de bootcamp con unos pocos usuarios, **sobra**.

---

## 1.5 Proveedor y región

Al crear el cluster eliges **proveedor cloud** (AWS, Google Cloud, Azure) y
**región**. Atlas monta el cluster en la infraestructura de ese proveedor.

La regla es la misma que con Supabase (ver `RENDER/03`, apartado 3.4):

```
API en Render (Frankfurt) ──1-2 ms──▶ Atlas en AWS eu-central-1 (Frankfurt)  ✅
API en Render (Frankfurt) ──~150 ms─▶ Atlas en AWS us-east-1 (Virginia)       ❌ cada consulta
```

> 📌 Elige para Atlas **la misma zona** que para tu servicio de Render. Con M0 no
> todas las regiones están disponibles; elige la más cercana de las que ofrezca.

---

## 1.6 Más allá de la base de datos

Atlas también ofrece servicios alrededor (no los usa tu proyecto, pero conviene
saber que existen):

| Servicio | Qué hace | Equivalente en Supabase |
|----------|----------|-------------------------|
| **Atlas Search** | Búsqueda de texto completo (relevancia, autocompletado, errores de tecleo) basada en Lucene | Full-text search de Postgres |
| **Vector Search** | Búsquedas por similitud para IA | `pgvector` |
| **Charts** | Gráficas y dashboards sobre tus colecciones | — |
| **Triggers** | Ejecutar funciones al cambiar datos o por horario | Triggers / `pg_cron` |
| **Online Archive / Data Federation** | Mover datos antiguos a almacenamiento barato y consultarlos | — |

> 💡 Atlas **no** trae autenticación de usuarios finales ni una API REST
> automática para el frontend (los antiguos *App Services / Data API* se han
> retirado). Atlas es una **base de datos gestionada**: el backend lo pones tú
> (tu API Express en Render). Esa es la gran diferencia con Supabase usado como BaaS.

---

## ✅ Resumen

- Atlas = MongoDB gestionado en la nube (DBaaS), accesible desde Render.
- Jerarquía: organización → **proyecto** (usuarios + red) → **cluster** → BD → colecciones.
- Todo cluster es un **replica set** de 3 nodos: alta disponibilidad, `retryWrites`, `w=majority`, transacciones.
- M0: gratis, 512 MB, compartido; suficiente para aprender.
- Región: la misma que tu API.
