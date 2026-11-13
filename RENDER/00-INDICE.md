# 🚀 CARPETA RENDER — ÍNDICE DE ESTUDIO

Teoría de **despliegue de APIs en Render** explicada desde la base, con ejemplos
reales **copiados de tus proyectos** del Bootcamp.

> ⚠️ Los proyectos **NO** se han modificado. Solo se han copiado fragmentos de su código.
> 🔒 Las contraseñas de los `compose.yaml` / `.env.example` se han sustituido por
> `TU_PASSWORD` y **no** se ha copiado ningún archivo `.env` real.

---

## 📖 Documentos de esta carpeta

| # | Archivo | Qué vas a aprender |
|---|---------|--------------------|
| 1 | `01-que-es-render-y-como-funciona-un-despliegue.md` | Qué es "desplegar", IaaS/PaaS/SaaS, qué hace Render por dentro (build → run), tipos de servicio |
| 2 | `02-preparar-tu-proyecto-para-produccion.md` | `build` vs `start`, `process.env.PORT`, variables de entorno sin `.env`, CORS, versión de Node, `devDependencies` |
| 3 | `03-desplegar-paso-a-paso.md` | Crear el Web Service campo a campo, Root Directory, auto-deploy, `render.yaml` (Blueprint) |
| 4 | `04-variables-logs-health-checks-y-plan-gratuito.md` | Environment, Secret Files, logs, Health Check, *cold start*, límites del plan Free, regiones |
| 5 | `05-errores-comunes-y-diagnostico.md` | Cómo leer un fallo de build / de arranque / de BD y arreglarlo |
| 6 | `06-ejemplos-completos-de-proyectos.md` | Los archivos relevantes de tus dos proyectos desplegables, enteros |

📌 Las **conclusiones conjuntas** de Render + Supabase están en
`../CONCLUSIONES-Render-y-Supabase.md`.

---

## 🗂️ Proyectos de los que se han extraído los ejemplos

| Proyecto | Archivo | Temas que cubre |
|----------|---------|-----------------|
| `postgresql-render-atlas/nestjs-postgresql-supabase-render/` | `README.md` | Tabla de campos de Render (Build/Start/Root/Variables) |
| ″ | `src/main.ts` | `process.loadEnvFile()` solo si existe `.env`, `process.env.PORT`, CORS |
| ″ | `src/database/database.module.ts` | `Pool` de `pg` leyendo `DATABASE_URL` (la de Supabase) |
| ″ | `package.json` | Scripts `build` (`nest build`) y `start:prod` (`node dist/main`) |
| `mongo-atlas-render/express-mongo-atlas-render/` | `README.md` | Despliegue de Express + MongoDB Atlas en Render |
| ″ | `src/index.ts` | `const PORT = process.env.PORT ?? 3000` |
| ″ | `src/db.ts` | Fallar rápido si falta `MONGODB_URI` |
| ″ | `package.json` | `dev` con `tsx --env-file`, `build` con `tsc`, `start` con `node dist/index.js` |

---

## 🚀 Orden de estudio recomendado

1. `01` → entender **qué problema resuelve** Render (sin esto los campos del formulario son magia).
2. `02` → lo más importante: **qué tiene que cumplir tu código** para poder vivir en un servidor ajeno.
3. `03` → hacer el despliegue de verdad.
4. `04` → operar el servicio: variables, logs, salud y límites del plan gratis.
5. `05` → cuando algo falla (y fallará), saber **dónde mirar**.
6. `06` → repaso con el código real completo.

---

## 💡 La idea en una imagen

```
  TU ORDENADOR                       GITHUB                         RENDER
 ┌──────────────┐   git push    ┌──────────────┐   webhook    ┌────────────────────────┐
 │ código + .env│ ────────────▶ │ código       │ ───────────▶ │ 1. git clone           │
 │ (localhost)  │               │ (SIN .env)   │              │ 2. Build Command       │
 └──────────────┘               └──────────────┘              │ 3. Start Command       │
                                                              │ 4. PORT + variables    │
                                                              │ 5. https://xxx.onrender.com
                                                              └───────────┬────────────┘
                                                                          │ DATABASE_URL
                                                                          ▼
                                                             Supabase (Postgres) / Atlas (Mongo)
```

> 📌 Render **no ve tu ordenador**: solo ve lo que hay en **GitHub** y lo que tú
> escribas en su panel (variables de entorno). Todo lo que tu app necesite tiene
> que salir de uno de esos dos sitios.
