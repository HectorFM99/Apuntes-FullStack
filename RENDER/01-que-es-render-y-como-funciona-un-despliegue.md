# 01 — QUÉ ES RENDER Y CÓMO FUNCIONA UN DESPLIEGUE

> Mientras programas, tu API vive en `http://localhost:3000`: solo existe **en tu
> ordenador** y solo mientras tengas la terminal abierta. **Desplegar** (*deploy*)
> es poner ese mismo código a funcionar en un ordenador que está **siempre
> encendido**, **conectado a Internet** y con una **URL pública** que cualquiera
> puede usar. Render es un servicio que hace eso por ti.

---

## 1.1 ¿Qué hace falta para que una API esté "en Internet"?

Si lo hicieras tú a mano, necesitarías:

| Necesidad | A mano (servidor propio / VPS) | Con Render |
|-----------|-------------------------------|------------|
| Un ordenador encendido 24/7 | Alquilar una máquina (AWS EC2, DigitalOcean...) | ✅ Lo pone Render |
| Sistema operativo y Node instalados | Instalar y actualizar Linux, Node, npm | ✅ Lo pone Render |
| Descargar tu código | `ssh` + `git clone` + `git pull` en cada cambio | ✅ Conecta con GitHub |
| Compilar e instalar dependencias | `npm install && npm run build` a mano | ✅ *Build Command* |
| Mantener la app viva si se cae | `pm2`, `systemd`... | ✅ Reinicia solo |
| Un dominio y HTTPS | Comprar dominio, configurar DNS, certificados (Let's Encrypt) | ✅ `xxx.onrender.com` con HTTPS gratis |
| Ver errores | Leer archivos de log por `ssh` | ✅ Pestaña **Logs** |

> 📌 Render se encarga de la **infraestructura**; tú solo te ocupas del **código**
> y de **configurarlo** (comandos y variables).

---

## 1.2 IaaS, PaaS, SaaS... ¿y BaaS?

Son niveles de "cuánto te dan hecho":

```
           Tú gestionas ▼                              Te lo gestionan ▲
┌──────────────┬──────────────┬──────────────┬──────────────┐
│     IaaS     │     PaaS     │     BaaS     │     SaaS     │
│ AWS EC2      │ RENDER       │ SUPABASE     │ Gmail        │
│ DigitalOcean │ Heroku       │ Firebase     │ Notion       │
│              │ Railway      │              │              │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ Te dan una   │ Te dan donde │ Te dan el    │ Te dan la    │
│ máquina      │ ejecutar TU  │ backend YA   │ aplicación   │
│ vacía        │ código       │ HECHO (BD,   │ terminada    │
│              │              │ auth, API)   │              │
└──────────────┴──────────────┴──────────────┴──────────────┘
```

- **IaaS** (*Infrastructure as a Service*): una máquina virtual. Máximo control, máximo trabajo.
- **PaaS** (*Platform as a Service*): **Render**. Tú subes código, ellos lo ejecutan.
- **BaaS** (*Backend as a Service*): **Supabase**. Ni siquiera escribes el backend: ya hay base de datos, login y API.
- **SaaS** (*Software as a Service*): un producto terminado para usuarios finales.

> 📌 En tu proyecto `nestjs-postgresql-supabase-render` combinas **dos**: Render
> (PaaS) ejecuta **tu** API NestJS, y Supabase (usado como Postgres gestionado)
> guarda los datos. Es la arquitectura más típica de un proyecto de bootcamp.

---

## 1.3 Qué hace Render por dentro cuando despliega

Cada despliegue pasa por estas fases (las verás en la pestaña **Events / Logs**):

```
 ┌─────────────┐   ┌─────────────┐   ┌──────────────┐   ┌──────────────┐   ┌────────────┐
 │ 1. CLONE    │──▶│ 2. BUILD    │──▶│ 3. DEPLOY    │──▶│ 4. HEALTH    │──▶│ 5. LIVE    │
 │ git clone   │   │ Build       │   │ Start        │   │ ¿responde en │   │ el tráfico │
 │ del commit  │   │ Command     │   │ Command      │   │ el PORT?     │   │ pasa a la  │
 │             │   │ (npm i,     │   │ (node dist/  │   │              │   │ versión    │
 │             │   │  build)     │   │  main)       │   │              │   │ nueva      │
 └─────────────┘   └─────────────┘   └──────────────┘   └──────────────┘   └────────────┘
       ✗ rama/carpeta    ✗ error de TS,     ✗ crash al         ✗ no escucha en
         incorrecta        falta script       arrancar (BD,      PORT → "No open
                                              variable)          ports detected"
```

1. **Clone**: descarga el commit de la rama configurada.
2. **Build**: ejecuta el *Build Command* (ej. `npm install && npm run build`). El resultado (`node_modules`, `dist/`) se empaqueta.
3. **Deploy / Start**: arranca un contenedor con ese paquete y ejecuta el *Start Command* (ej. `npm run start:prod`).
4. **Comprobación**: Render espera a que tu app **escuche en el puerto** que le dijo (variable `PORT`) y, si lo configuraste, que el *Health Check* responda 2xx/3xx.
5. **Live**: solo entonces redirige el tráfico a la versión nueva. La anterior se apaga.

> 💡 Gracias al paso 4, un despliegue roto **no tumba** la versión que ya
> funcionaba: si la nueva no arranca, Render la marca como *failed* y sigue
> sirviendo la anterior (*zero-downtime deploy*). En el plan Free esto es más
> limitado, pero la idea es la misma.

### El contenedor es **efímero**

Tu app corre dentro de un contenedor (como Docker). Cada despliegue o reinicio
crea uno **nuevo desde cero**:

- ❌ Si tu API guarda archivos en disco (`fs.writeFile('uploads/foto.png')`), **se pierden** al redesplegar.
- ❌ Si guardas datos en una variable en memoria (`const books = []`), **se pierden** al reiniciar.
- ✅ Los datos tienen que ir a un servicio **externo y persistente**: una base de datos (Supabase, Atlas) o un almacenamiento de archivos (Supabase Storage, S3).

> 📌 Por eso tus proyectos de despliegue usan Supabase o Atlas y no un array en memoria.

---

## 1.4 Tipos de servicio en Render

| Tipo | Para qué | ¿Tiene URL pública? | Ejemplo |
|------|----------|---------------------|---------|
| **Web Service** | Servidores HTTP que escuchan en un puerto | ✅ | Tu API NestJS / Express |
| **Static Site** | HTML/CSS/JS ya generados (sin servidor) | ✅ (CDN) | Tu web en Vite (`npm run build` → `dist/`) |
| **Private Service** | Servidor accesible solo desde otros servicios de Render | ❌ | Un microservicio interno |
| **Background Worker** | Proceso que no recibe HTTP, consume colas | ❌ | Enviar emails en segundo plano |
| **Cron Job** | Comando que se ejecuta según un horario | ❌ | Limpiar la BD cada noche |
| **Postgres** | Base de datos PostgreSQL gestionada | Conexión, no web | Alternativa a Supabase |
| **Key Value** | Almacén clave-valor compatible con Redis | Conexión | Caché, sesiones |

> 📌 Tus dos APIs son **Web Services**. Un frontend React/Vite sería un **Static
> Site** (gratis y sin *cold start*, porque solo sirve archivos).

---

## 1.5 Web Service vs Static Site: la diferencia clave

```
STATIC SITE                                   WEB SERVICE
navegador ──▶ CDN ──▶ index.html, app.js      navegador ──▶ tu proceso Node ──▶ BD
(archivos fijos, nada se ejecuta en           (se ejecuta TU código en cada petición,
 el servidor)                                  puede leer BD, validar, autenticar)
```

- Un **Static Site** no puede tener secretos: todo lo que contiene lo descarga el navegador.
- Un **Web Service** sí: `DATABASE_URL` vive en el servidor y el navegador **nunca** la ve.

> ⚠️ Por eso **nunca** se conecta un frontend directamente a Postgres con la
> `DATABASE_URL`: se la estarías regalando a cualquiera que abra las DevTools.
> El frontend habla con **tu API** (o con la API de Supabase protegida por RLS, ver
> carpeta `Supabase`).

---

## 1.6 Alternativas a Render (para que te suenen)

| Servicio | Tipo | Comentario |
|----------|------|------------|
| **Railway** | PaaS | Muy parecido a Render, cobra por uso |
| **Fly.io** | PaaS | Despliega contenedores Docker cerca del usuario |
| **Vercel / Netlify** | PaaS (frontend + funciones) | Ideales para frontends y Next.js; las APIs son *serverless* |
| **Heroku** | PaaS | El "original"; ya no tiene plan gratis |
| **AWS / GCP / Azure** | IaaS + todo | Lo que usan las empresas grandes; mucho más complejo |

> 📌 Lo que aprendes en Render (build/start, `PORT`, variables de entorno, logs,
> health checks) se aplica **igual** en todos ellos: cambian los nombres de los
> botones, no los conceptos.

---

## ✅ Resumen

- Desplegar = ejecutar tu código en una máquina ajena, siempre encendida, con URL pública.
- Render es un **PaaS**: clona de GitHub → **Build** → **Start** → comprueba el **PORT** → tráfico.
- El contenedor es **efímero**: los datos van a una BD externa.
- Tu API es un **Web Service**; un frontend estático sería un **Static Site**.
