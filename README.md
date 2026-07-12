# Short URL Demo

A URL shortener built as a Laravel API + Vue single-page app, demonstrating token-based auth, Redis-backed lookup caching, click analytics, and a multi-environment Docker setup.

## What it demonstrates

- Laravel API backend with Sanctum authentication and a service-layer design (`UrlShortenerService`) separate from controllers
- Redis-cached short-code lookups with a database fallback, backed by a MySQL store
- Per-click analytics (IP, user agent, referer) recorded on redirect
- Role-based authorization (admin vs. regular user) via a dedicated middleware
- Vue 3 + TypeScript frontend using the Composition API, Pinia, and Radix Vue/Tailwind-based UI primitives
- Auto-generated OpenAPI/Swagger documentation from PHP attribute annotations
- Docker Compose environments for development, testing, and production, each with its own Nginx, MySQL, and Redis services

## What's inside

- **URL shortening**: generates a random 6-character short code, checked for uniqueness against cache and database
- **Redirects**: public `GET /{code}` route resolves the cached (or database) URL, records a click, and increments a counter
- **Click analytics**: `UrlClick` records store IP address, user agent, and referer per redirect
- **Auth**: registration/login/logout via Sanctum, with an `is_admin` flag on the user model
- **Admin endpoints**: list and delete any user's shortened URLs
- **API docs**: Swagger UI generated from controller annotations (`darkaonline/l5-swagger`)
- **Frontend**: forms and list views for creating and managing shortened URLs, built with reusable UI components

## Tech stack

**Backend:** Laravel, Sanctum, MySQL, Redis (Predis client), L5-Swagger, Laravel Pint, Pest

**Frontend:** Vue (Composition API), TypeScript, Vite, Vue Router, Pinia, Axios, Tailwind CSS, Radix Vue, Lucide icons

**Infrastructure:** Docker (multi-stage builds, per-environment Compose files), Nginx

## Quickstart

Requires Docker and Docker Compose.

```bash
cd docker
docker compose -f compose.development.yml up -d --build
```

The entrypoint generates the app key, runs migrations, seeds a test user, and (outside production) publishes the Swagger docs automatically.

- App (via Nginx): http://localhost:8000
- API: http://localhost:8000/api
- API docs: http://localhost:8000/api/documentation
- Vite dev server (hot reload): http://localhost:3000
- MySQL: localhost:3306
- Redis: localhost:6379

Seeded test account: `test@example.com` / `password` (not an admin by default — set `is_admin` on the user record to access the admin endpoints).

Stop with:

```bash
docker compose -f compose.development.yml down
```

Testing and production environments are available via `docker/compose.testing.yml` and `docker/compose.production.yml`; production requires copying `server/.env.production.example` and `web/.env.production.example` to `.env.production` first.

## Structure

```
server/   # Laravel API
  app/Http/Controllers/Api/   # Auth, Url, Admin controllers
  app/Services/               # UrlShortenerService (short-code generation, cache-backed lookup)
  app/Models/                 # ShortenedUrl, UrlClick, User
  database/migrations/
  routes/api.php               # API routes
  routes/web.php                # Public redirect route

web/      # Vue frontend
  src/api/        # API client
  src/components/ # Forms, lists, and UI primitives

docker/   # Compose files per environment, Nginx and PHP configs, entrypoint script
```

## API endpoints

**Auth**
- `POST /api/register`, `POST /api/login`, `POST /api/logout`, `GET /api/user`

**URLs** (authenticated)
- `GET /api/urls` — list the current user's shortened URLs
- `POST /api/urls` — create a shortened URL
- `GET /api/urls/{id}` — URL details with recent clicks
- `DELETE /api/urls/{id}`

**Admin** (authenticated + `is_admin`)
- `GET /api/admin/urls`, `DELETE /api/admin/urls/{id}`

**Public**
- `GET /{code}` — redirect to the original URL and log a click

## License

MIT
