# GAUGE — Gaming Assessment & User Gameplay Evaluation

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=flat&logo=typescript&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js_14-000000?style=flat&logo=nextdotjs&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat&logo=nodedotjs&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat&logo=postgresql&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat&logo=prisma&logoColor=white)

Plataforma web *full-stack* orientada a la comunidad de videojuegos para la publicación, consulta y gestión de reseñas fundamentadas. **GAUGE** sienta las bases para un sistema de evaluación transparente, relacionando la experiencia real del usuario con análisis estructurados por género y saga.

---

## 🔗 Despliegues en Producción

* **Frontend (Vercel):** [Todavia en desarrollo](https://gauge-gaming.vercel.app)
* **Backend API (Render):** [Todavia en Desarrollo](https://gauge-api.onrender.com)

---

## 🚀 Características Principales (MVP)

- **Autenticación y Autorización Segura:** Registro e inicio de sesión utilizando cifrado de contraseñas con `bcrypt` y tokens `JWT`.
- **Control de Acceso Basado en Roles (RBAC):** Roles diferenciados (`USER` y `ADMIN`) para la gestión de catálogo y moderación.
- **Catálogo Paginado de Videojuegos:** Exposición de títulos mediante una API REST con respuesta estructurada (`pagination` y `data`).
- **Gestión de Reseñas Específicas:** Creación de publicaciones asociadas a usuarios autenticados mediante extracción segura del `userId` en el token payload.
- **Arquitectura Limpia:** Separación estricta de responsabilidades en 4 capas en backend y desarrollo guiado por características (*feature-driven*) en frontend.

---

## 🛠️ Stack Tecnológico

### Backend
- **Runtime:** Node.js + TypeScript
- **Framework:** Express.js
- **ORM:** Prisma
- **Base de Datos:** PostgreSQL (Supabase / Render Postgres)
- **Seguridad:** `bcrypt` para hashing, `jsonwebtoken` para sesiones, `cors` para políticas de origen.

### Frontend
- **Framework:** Next.js (App Router) + TypeScript
- **Estilos:** Tailwind CSS
- **Arquitectura de UI:** Feature-Driven Development (`auth`, `games`, `reviews`, `common`)
- **Gestión de Peticiones:** Cliente HTTP centralizado con interceptor para la inyección automática del token Bearer y manejo de respuestas `401`.

---

## 📂 Estructura del Proyecto

```text
# Backend
src/
├── domain/          # Entidades de negocio e interfaces de repositorios
├── application/     # Casos de uso (Auth, CRUD de catálogo, interacción)
├── infrastructure/  # Implementación con Prisma, PostgreSQL, Bcrypt y JWT
└── interface/       # Controladores HTTP, routers y middlewares

# Frontend
app/                 # Rutas de Next.js (page.tsx, layout.tsx, not-found.tsx)
src/features/        # Módulos organizados por dominio
├── common/          # Configuración de API, cliente HTTP y componentes globales
├── auth/            # Dominio, servicios, hooks (useAuth) y formularios
├── games/           # Tarjetas, grilla, hooks y paginación del catálogo
└── reviews/         # Gestión y listado de reseñas por usuario

```

---

## 🔑 Credenciales de Prueba (Entorno de Evaluación)

Para probar los roles y rutas protegidas en la aplicación desplegada:

| Rol | Email | Contraseña | Permisos |
| --- | --- | --- | --- |
| **Admin** | `admin@gauge.com` | `AdminPass123!` | Crear, editar y eliminar juegos (`/admin`); ver todas las reseñas. |
| **Usuario** | `user@gauge.com` | `UserPass123!` | Crear reseñas; acceder a `/mis-reseñas`. |

---

## 🧠 Decisiones Técnicas de Arquitectura

### 1. Flujo Completo de Autenticación y Manejo de Sesión

1. El usuario envía sus credenciales mediante `RegisterForm` o `LoginForm`.
2. El backend valida el correo, verifica la contraseña con `bcrypt.compare` y genera un JWT firmado con `JWT_SECRET` y un tiempo de expiración determinado.
3. El payload del token únicamente contiene el `id` y el `role` del usuario para proteger información sensible.
4. En el frontend, el token se almacena en memoria/estado persistente y el cliente HTTP (`http.ts`) intercepta las peticiones salientes para adjuntar el encabezado `Authorization: Bearer <token>`.
5. Al recibir un código de estado `401 Unauthorized` desde la API, el interceptor destruye la sesión activa y redirige automáticamente al usuario hacia `/login`.

### 2. Manejo de Roles y Códigos de Estado (401 vs 403)

* **`401 Unauthorized`:** Se retorna cuando la petición intenta acceder a un recurso protegido sin token, con un token inválido o cuya expiración ha transcurrido. *Ejemplo:* Un usuario sin sesión intenta enviar una reseña (`POST /api/reviews`).
* **`403 Forbidden`:** Se retorna cuando el token es totalmente válido, pero el rol asociado no posee los permisos requeridos para la acción. *Ejemplo:* Un usuario con rol `USER` intenta eliminar un videojuego del catálogo (`DELETE /api/games/:id`).

### 3. Almacenamiento del Token y Consideraciones de Seguridad

En esta versión del proyecto, el token se almacena a nivel de cliente para simplificar la integración con el cliente HTTP. Se reconoce el riesgo de exposición a ataques XSS (*Cross-Site Scripting*); por ello, para la siguiente fase del sistema se contempla migrar la gestión de sesiones a cookies de tipo `httpOnly` y `SameSite` desde el servidor.

---

## 📸 Evidencia de Petición Protegida

*Captura de la pestaña Network del navegador comprobando la inyección automática del token JWT en peticiones privadas.*

---

## ⚙️ Configuración e Instalación Local

### Prerrequisitos

* Node.js (v18 o superior)
* npm o pnpm
* Instancia local o remota de PostgreSQL

### Backend

1. Clonar el repositorio e instalar dependencias:
```bash
git clone [https://github.com/tu-usuario/gauge-backend.git](https://github.com/tu-usuario/gauge-backend.git)
cd gauge-backend
npm install

```


2. Configurar variables de entorno `.env` (basado en `.env.example`):
```env
DATABASE_URL="postgresql://usuario:password@localhost:5432/gauge_db?schema=public"
JWT_SECRET="tu_clave_secreta_super_segura"
JWT_EXPIRES_IN="24h"
PORT=4000

```


3. Ejecutar migraciones de Prisma y el script de semillas (*seed*):
```bash
npx prisma migrate dev
npx prisma db seed

```


4. Iniciar el servidor de desarrollo:
```bash
npm run dev

```



### Frontend

1. Clonar el repositorio e instalar dependencias:
```bash
git clone [https://github.com/tu-usuario/gauge-frontend.git](https://github.com/tu-usuario/gauge-frontend.git)
cd gauge-frontend
npm install

```


2. Configurar variables de entorno `.env.local`:
```env
NEXT_PUBLIC_API_HOST="http://localhost:4000/api"

```


3. Iniciar el servidor de desarrollo:
```bash
npm run dev

```



---

## 🔮 Roadmap Futuro

* [ ] **Integración con Steam Web API:** Verificación automática de horas jugadas mediante OpenID.
* [ ] **Módulo de Análisis con IA:** Evaluación de comentarios mediante LLMs para medir objetividad y prevenir *review bombing*.
* [ ] **Sistema de Gamificación:** Asignación automática de insignias (*Badges*) basadas en el nivel de experiencia demostrado por saga y género.

---

## 📝 Licencia

Este proyecto se distribuye bajo la licencia MIT. Consulta el archivo `LICENSE` para más información.

