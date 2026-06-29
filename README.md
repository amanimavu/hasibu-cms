# Hasibu CMS

A self-hosted [Directus](https://directus.io) CMS, run via Docker Compose. Based on the Directus "Blank" starter template, configured to use **MySQL** for the database and **Redis** for caching.

## Stack

| Service    | Image                      | Purpose                       |
| ---------- | -------------------------- | ----------------------------- |
| `directus` | `directus/directus:11.17.4`| CMS application + Admin UI/API |
| `database` | `mysql:8`                  | Primary data store            |
| `cache`    | `redis:6`                  | Cache + WebSocket adapter      |

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose v2 (`docker compose`)

## Setup

All Docker configuration lives in the `directus/` directory.

1. Create your environment file from the example:

   ```bash
   cd directus
   cp .env.example .env
   ```

2. Edit `.env` and set, at minimum:
   - `DIRECTUS_SECRET` — a random secret (e.g. `openssl rand -hex 32`)
   - `DB_PASSWORD` / `DB_ROOT_PASSWORD` — strong passwords
   - Review ports (see [Ports](#ports) below)

3. Start the stack:

   ```bash
   docker compose up -d
   ```

4. Open the Admin app at the `PUBLIC_URL` (default <http://localhost:8055>). The blank
   template uses onboarding to create the first admin user on first launch.

To stop:

```bash
docker compose down
```

## Configuration

Configuration is driven entirely by `directus/.env`. Key variables:

### Database

| Variable           | Default          | Notes                                                              |
| ------------------ | ---------------- | ------------------------------------------------------------------ |
| `DB_PORT`          | `3306`           | Port Directus uses to reach the DB **inside the Docker network**. Keep `3306`. |
| `DB_HOST_PORT`     | `3308`           | Port the DB is exposed on **the host** (`localhost:3308`). Change if taken. |
| `DB_DATABASE`      | `directus`       | Database name. Auto-created on first boot.                         |
| `DB_USER`          | `directus`       | App user. Auto-created by the MySQL image with `ALL PRIVILEGES` on `DB_DATABASE`. |
| `DB_PASSWORD`      | —                | Password for `DB_USER`.                                            |
| `DB_ROOT_PASSWORD` | —                | MySQL `root` password.                                             |

> **Note on ports:** `DB_PORT` (3306) is the *internal* container port Directus
> connects to over the Docker network — leave it alone. `DB_HOST_PORT` is the
> *host-side* mapping for connecting from your machine (GUI/CLI). Pick a free
> host port; the default is `3308` to avoid clashing with a local MySQL on 3306
> or another container on 3307.

### Directus

| Variable           | Default                 | Notes                          |
| ------------------ | ----------------------- | ------------------------------ |
| `DIRECTUS_PORT`    | `8055`                  | Host port for the Admin UI/API |
| `DIRECTUS_SECRET`  | —                       | **Change this.** Signing secret |
| `PUBLIC_URL`       | `http://localhost:8055` | Public-facing URL              |
| `CORS_ENABLED` / `CORS_ORIGIN` | `true` / `*` | Tighten `CORS_ORIGIN` in production |

Other settings (cache, WebSockets, cookies, CSP, email/SMTP) are documented inline
in `.env.example`.

## Data persistence

Bind-mounted under `directus/`:

- `data/database/` — MySQL data files
- `uploads/`       — uploaded assets
- `extensions/`    — custom Directus extensions (auto-reloaded when `EXTENSIONS_AUTO_RELOAD=true`)

> **Warning:** `data/database/` is bound to MySQL's data directory. If switching
> database engines or starting fresh, delete it first (`sudo rm -rf data/database`)
> — a non-empty data dir from a different engine will prevent MySQL from starting.

## Production notes

- Set a strong, unique `DIRECTUS_SECRET` and DB passwords.
- Restrict `CORS_ORIGIN` to your real origins (no `*`).
- Set `*_COOKIE_SECURE=true` and the correct `*_COOKIE_DOMAIN` when serving over HTTPS.
- Configure `EMAIL_*` for transactional email (password resets, invites).
