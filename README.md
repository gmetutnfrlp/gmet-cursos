# GMET — Cursos y Oportunidades

Plataforma web de cursos y oportunidades de formación en tecnología para mujeres de GIDAS, UTN FRLP.  
Reemplaza el tablero de Miro con un sitio propio con panel admin, estados automáticos por fecha y un agente de IA para carga de cursos.

---

## Stack

| Capa | Tecnología |
|---|---|
| Framework | Next.js 14 (App Router) + TypeScript |
| Estilos | Tailwind CSS |
| Base de datos + Auth | Supabase (PostgreSQL + Supabase Auth) |
| Agente IA | Google Gemini 3.6 Flash |
| Scraping de URLs | Jina AI (sin API key) |
| Íconos | Lucide React |

---

## Requisitos previos

- Node.js 18 o superior
- npm
- Acceso a la cuenta de Supabase de GMET — pedíselo a Zoe o Agus si no lo tenés
- Acceso al repositorio `gmetutnfrlp/cursos-gmet` en GitHub

---

## Levantar el proyecto en local

### 1. Clonar el repositorio

```bash
git clone https://github.com/gmetutnfrlp/cursos-gmet.git
cd cursos-gmet
```

### 2. Instalar dependencias

```bash
npm install
```

### 3. Configurar las variables de entorno

Creá un archivo `.env.local` en la raíz del proyecto.  
Los valores de Supabase los encontrás en el dashboard del proyecto GMET en **Project Settings → API**.

```env
# Supabase — Project Settings → API del proyecto GMET
NEXT_PUBLIC_SUPABASE_URL=https://tu-proyecto.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=tu-anon-key

# Gemini API — conseguila en aistudio.google.com con la cuenta de GMET
GEMINI_API_KEY=tu-gemini-key
```

> **Importante:** `.env.local` está en el `.gitignore` — nunca lo subas al repositorio.

### 4. Correr en modo desarrollo

```bash
npm run dev
```

Abrí [http://localhost:3000](http://localhost:3000) para ver el sitio público.

---

## Base de datos

**No necesitás correr ningún SQL.** La base de datos ya está configurada en la cuenta de Supabase de GMET con todas las tablas, datos y políticas.

Las tablas principales son:

- `courses` — cursos con título, descripción, proveedor, categoría, fecha límite, URL, rango de edad y estado (`draft` / `published`)
- `providers` — proveedores precargados (Santander, Accenture, Google, Globant, etc.) con opción de agregar nuevos desde el panel

Si por alguna razón necesitás recrear la base de datos desde cero, pedile el SQL de migración a Agus.

---

## Acceder al panel admin

El panel está en `/admin` y usa autenticación real con **Supabase Auth** (email + contraseña).

1. Entrá a [http://localhost:3000/admin/login](http://localhost:3000/admin/login)
2. Usá el email y contraseña de admin — pedíselos a Zoe o Agus
3. Para crear un usuario nuevo: dashboard de Supabase → **Authentication → Users → Invite user**

Desde el panel podés:
- Cargar cursos manualmente con el formulario
- Cargar cursos con el agente de IA (pegando texto libre o una URL)
- Editar y eliminar cursos existentes
- Publicar o dejar en borrador cada curso
- Buscar y filtrar cursos por proveedor, categoría y estado

---

## Estados de los cursos (sitio público)

Los cursos se muestran con colores según la fecha de inscripción:

| Color | Estado | Cuándo |
|---|---|---|
| 🟢 Verde | Activo | Inscripción abierta con más de 7 días |
| 🟡 Amarillo | Por vencer | Vence en los próximos 7 días |
| ⚫ Gris | Vencido | Fecha de inscripción pasada |
| 🩷 Rosa | Sin fecha | Sin fecha límite definida |

Los cursos vencidos se muestran al final con opacidad reducida. Solo son visibles en el sitio público los cursos con `status = published` e `is_active = true`.

---

## El agente de IA

En el panel admin hay una sección **"Cargar curso con IA"** con dos modos:

**Modo texto** — pegás una descripción en texto libre:
> "Accenture abrió inscripciones para Pentesting Foundations, curso gratuito para mujeres de 18 a 35 años, inscripción hasta el 30 de octubre en accenture.com/ar/pentesting"

**Modo link** — pegás la URL del curso directamente. El agente lee la página web real usando Jina AI y extrae la información desde el contenido original.

En ambos casos el formulario se pre-rellena para que lo revises antes de guardar.  
Todo curso cargado por el agente queda en estado **Borrador** hasta que lo publicás manualmente.

> **Limitación conocida:** algunos sitios bloquean scrapers automáticos. En esos casos el agente devuelve un error y podés usar el modo texto como alternativa.

---

## Estructura del proyecto

```
src/
  app/
    page.tsx                  → sitio público de cursos
    admin/
      page.tsx                → panel admin (requiere sesión activa)
      login/page.tsx          → login con Supabase Auth
    api/
      extract-course/
        route.ts              → API route del agente IA (texto o URL → campos)
  components/
    CourseCard.tsx            → tarjeta de curso con estado visual
    CourseGrid.tsx            → grid responsivo con filtros por categoría
    StatusBadge.tsx           → badge de estado (activo / por vencer / vencido)
    AdminCourseForm.tsx       → formulario de carga/edición de cursos
    AdminCourseTable.tsx      → tabla con filtros, buscador y ordenamiento
    AICoursLoader.tsx         → sección del agente IA (modo texto y modo link)
  lib/
    supabase-browser.ts       → cliente Supabase para Client Components
    supabase-server.ts        → cliente Supabase para Server Components
    courseUtils.ts            → lógica de estados por fecha (active / expiring_soon / expired)
  types/
    course.ts                 → tipos Course y Provider
  middleware.ts               → protección de rutas /admin via Supabase Auth
```

---

## Verificar que todo funciona

Después de levantar el proyecto en local, chequeá:

- [ ] [http://localhost:3000](http://localhost:3000) muestra las tarjetas de cursos cargados
- [ ] Los filtros por categoría funcionan
- [ ] [http://localhost:3000/admin/login](http://localhost:3000/admin/login) muestra el formulario de login
- [ ] Podés iniciar sesión con las credenciales de admin
- [ ] El panel muestra la tabla de cursos existentes
- [ ] El agente en modo texto extrae campos correctamente
- [ ] El agente en modo link lee una URL y pre-rellena el formulario

---

## Deploy (cuando corresponda)

El deploy lo maneja Zoe o Agus. El proyecto está configurado para Vercel:

1. Importar el repo desde [vercel.com/new](https://vercel.com/new)
2. Cargar las variables de entorno del paso 3
3. Deploy — Vercel detecta Next.js automáticamente

Si cambiás alguna variable de entorno, hay que redeploy: **Vercel → Deployments → Redeploy**.

---

## Contacto

Dudas sobre el proyecto: **Agus** (desarrollo) o **Zoe** (coordinación GMET).  
Repo: [github.com/gmetutnfrlp/cursos-gmet](https://github.com/gmetutnfrlp/cursos-gmet)
