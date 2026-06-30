# Glowfy_deployement


A modern spa booking experience with a client-facing booking page and a staff dashboard for managing appointments, services, promotions, and spa settings.

- **Live URL**: `[Add your deployed URL here]`
- **Repository**: https://github.com/Stella-grid/glowfy-spa-suite.git
- **Version**: `v1.0.0` *(update to match your current release)*
- **License**: MIT *(update if a different license applies)*
- **Contributors**: Stella-grid (repository owner/maintainer) *(add teammates here)*

> **New to this project?** This guide assumes you know nothing about it. Follow the numbered steps in order and you'll have the whole app — frontend **and** backend — running on your own computer using Docker. No Node.js setup, no manual database configuration required.

---

## 1. Overview

Glowfy SPA Suite has two parts that need to run together:

1. **The frontend** — the website itself (booking page + staff dashboard). This runs in a Docker container built from this repo.
2. **The backend** — Supabase (database, login, server logic). You'll run a *local copy* of Supabase on your machine. It also runs entirely inside Docker, managed automatically by a tool called the Supabase CLI — you don't manage its containers by hand.

The whole local setup workflow is:

```text
Install Docker  →  Get the code  →  Start the local backend  →  Configure .env  →  Start the frontend container  →  Open it in your browser
```

> Everything below is written so you can run the full app locally with Docker alone. A non-Docker (plain `npm run dev`) path is mentioned briefly for advanced users, but it is not required.

---

## 2. Application Architecture

Glowfy SPA Suite is a full-stack web application composed of three main layers:

- **Frontend layer**: a React + TypeScript single-page application built with Vite, Tailwind CSS, and shadcn-ui.
- **Application layer**: route-based pages for the client experience and staff dashboard, backed by custom hooks for data access and UI state.
- **Backend layer**: Supabase services for authentication, database storage, API access, and edge functions.

### Main application modules

- **Client experience**: booking flow, service browsing, testimonials, and spa information.
- **Staff dashboard**: daily booking overview, settings management, service configuration, and spa profile management.
- **Admin setup flow**: initial administrator account creation through a dedicated Supabase edge function.

### Architecture diagram

```text
                        ┌─────────────────────────────────────────────┐
                        │              Frontend Layer                  │
                        │  Client Booking UI │ Staff Dashboard │ Admin │
                        │   (React + Vite, Tailwind, shadcn-ui)        │
                        │         → runs in its own Docker container   │
                        └───────────────────────┬───────────────────────┘
                                                 │
                        ┌────────────────────────▼────────────────────┐
                        │              Application Layer               │
                        │  Pages & Routing │ UI Components │ Hooks     │
                        │  (src/pages, src/components, src/hooks)      │
                        └───────────────────────┬───────────────────────┘
                                                 │ HTTPS / HTTP (local)
                        ┌────────────────────────▼────────────────────┐
                        │     Backend Layer — local Supabase stack     │
                        │  Auth │ Postgres DB │ Storage │ Edge Funcs   │
                        │   → runs in Docker containers managed by     │
                        │             the Supabase CLI                 │
                        └───────────────────────────────────────────────┘
```

### Project folder structure

```text
glowfy-spa-suite/
├── public/                      # Static assets
├── src/
│   ├── pages/                   # Public-facing pages and staff dashboard screens
│   ├── components/              # Reusable UI and dashboard components
│   ├── hooks/                   # Data hooks for spa, booking, and auth logic
│   └── integrations/
│       └── supabase/            # Supabase client and generated database types
├── supabase/
│   ├── functions/                # Backend edge functions (e.g. admin bootstrap)
│   └── migrations/               # Database schema changes (auto-applied locally)
├── package.json
└── README.md
```

### Technology stack

| Layer | Technology |
|---|---|
| Build tool | Vite |
| Language | TypeScript |
| UI framework | React |
| Component library | shadcn-ui |
| Styling | Tailwind CSS |
| Data fetching / caching | TanStack Query |
| Animation | Framer Motion |
| Backend | Supabase (Auth, Postgres, Edge Functions) — run locally via Docker |
| Frontend container | Docker (multi-stage build, served by nginx) |

---

## 3. Network Architecture

When running locally, everything talks over `localhost` — nothing leaves your computer:

```text
Your Browser
     |
     | http://localhost:8080
     v
Frontend Container (nginx serving the built React app)
     |
     | http://localhost:54321  (local Supabase API, started by the Supabase CLI)
     v
Local Supabase Stack (Docker containers)
  ├── Auth        → handles login/sessions
  ├── Postgres    → stores spa, service, booking, staff data
  ├── Storage     → file storage (if used)
  └── Edge Funcs  → e.g. the admin bootstrap function
     |
     | http://localhost:54323
     v
Supabase Studio (local dashboard to view/edit your database in the browser)
```

### Default local ports

| Service | URL | Purpose |
|---|---|---|
| Frontend app | `http://localhost:8080` | The actual website you interact with |
| Supabase API | `http://localhost:54321` | What the frontend talks to (Auth, database, functions) |
| Supabase Studio | `http://localhost:54323` | Visual dashboard to inspect your local database |
| Postgres (direct) | `localhost:54322` | Direct database connection, if you need a SQL client |

> Exact port numbers are assigned by the Supabase CLI when it starts and are printed in your terminal — use those values if they differ.

---

## 4. Script Overview

Once Docker is installed, these are the only commands you need:

| Command | Purpose |
|---|---|
| `supabase start` | Starts the local backend (Postgres, Auth, Storage, Edge Functions) in Docker, and applies database migrations automatically |
| `supabase stop` | Stops the local backend containers |
| `docker compose up --build` | Builds and starts the frontend container |
| `docker compose down` | Stops the frontend container |

**Typical first-run workflow:**

```text
supabase start
    ↓ (copy the printed API URL + anon key into .env)
docker compose up --build
    ↓
open http://localhost:8080
```

*(Optional, for advanced/non-Docker frontend development: `npm install` then `npm run dev` — not required for the Docker workflow.)*

---

## 5. Prerequisites

You only need two things installed on your computer:

| Tool | Why you need it | Download |
|---|---|---|
| **Docker Desktop** | Runs both the frontend container and the local Supabase backend | [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/) |
| **Git** | To download (clone) this repository | [git-scm.com](https://git-scm.com/downloads) |

### Installing Docker Desktop

- **Windows**: download the installer from the link above, run it, and enable WSL2 if prompted (the installer guides you through this). Restart your computer if asked.
- **macOS**: download the `.dmg`, drag Docker into Applications, then open it (look for the whale icon in the menu bar — it should say "Docker Desktop is running").
- **Linux**: follow the distribution-specific instructions on the Docker site, or install Docker Engine + Docker Compose plugin via your package manager.

### Verify Docker is installed and running

```sh
docker --version
docker compose version
```

Both commands should print a version number. If you get a "command not found" error, reinstall Docker Desktop and make sure it's actually open/running (the whale icon should be active, not greyed out).

### Supported operating systems

- Windows 10/11 (with WSL2)
- macOS (Intel or Apple Silicon)
- Ubuntu 22.04 / 24.04, Debian 12, or most modern Linux distributions

### References

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Git](https://git-scm.com/)
- [Supabase Local Development](https://supabase.com/docs/guides/local-development)

---

## 6. Environment Installation

### Purpose

Gets the project code onto your computer and prepares the configuration file the app needs to find its backend.

### Step 1 — Clone the repository

```sh
git clone https://github.com/Stella-grid/glowfy-spa-suite.git
cd glowfy-spa-suite
```

### Step 2 — Install the Supabase CLI

The Supabase CLI is what tells Docker to start the backend. Pick whichever matches your system:

```sh
# macOS (Homebrew)
brew install supabase/tap/supabase

# Windows (Scoop)
scoop bucket add supabase https://github.com/supabase/scoop-bucket.git
scoop install supabase

# Linux / no package manager — run it without installing, via npx
npx supabase --version
```

(If you use `npx`, prefix every `supabase` command below with `npx` as well, e.g. `npx supabase start`.)

### Step 3 — Create your `.env` file

Create a file named `.env` in the project root. You'll fill in the real values in Step 8 — for now, create it with placeholders:

```env
# Supabase project configuration (you'll fill these in after running `supabase start`)
VITE_SUPABASE_URL=http://localhost:54321
VITE_SUPABASE_ANON_KEY=replace-with-the-anon-key-printed-by-supabase-start

# Used by the Docker container
APP_PORT=8080
```

| Variable | Description | Required | Default |
|---|---|---|---|
| `VITE_SUPABASE_URL` | URL of your local Supabase API | Yes | `http://localhost:54321` |
| `VITE_SUPABASE_ANON_KEY` | Public anon key, printed when you run `supabase start` | Yes | N/A |
| `APP_PORT` | Port the frontend container listens on | No | `8080` |

### Expected result

```text
Environment installation completed.
Repository cloned, Supabase CLI available, .env file created.
```

---

## 7. Environment Validation

Before moving on, confirm everything from the previous step is in place:

```sh
docker --version          # Docker is installed
docker compose version    # Docker Compose is available
supabase --version        # Supabase CLI is installed (or npx supabase --version)
git --version              # Git is installed
ls .env                   # The .env file exists
```

### Validation output

```text
[PASS] Docker
[PASS] Docker Compose
[PASS] Supabase CLI
[PASS] Git
[PASS] .env file present
```

If any of these fail, go back to Section 5 and re-check that installation step before continuing.

---

## 8. Application Installation

This is where the app actually gets built and the backend gets created — all in Docker.

### Step 4 — Start the local backend

From the project root:

```sh
supabase start
```

The first run downloads several Docker images, so it may take a few minutes. When it finishes, it prints something like:

```text
         API URL: http://localhost:54321
     GraphQL URL: http://localhost:54321/graphql/v1
          DB URL: postgresql://postgres:postgres@localhost:54322/postgres
      Studio URL: http://localhost:54323
        anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
service_role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

This command also **automatically applies** the database migrations in `supabase/migrations`, so your local database is ready to use immediately — no manual migration step needed.

### Step 5 — Update your `.env` with the real values

Copy the `API URL` and `anon key` from the output above into your `.env` file:

```env
VITE_SUPABASE_URL=http://localhost:54321
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...   # use your real printed value
APP_PORT=8080
```

> Save the file before continuing — the frontend reads these values when its Docker image is built.

### Step 6 — Build and start the frontend container

```sh
docker compose up --build
```

This builds the React app and serves it through nginx inside a container, using the Dockerfile included in this repo (see [Section 10](#10-docker-deployment)).

### Expected result

```text
Backend running locally via Docker (Supabase CLI).
Frontend container built and running.
Application installation completed.
```

---

## 9. Application Validation

Confirm everything is actually working:

| Check | How to verify |
|---|---|
| Backend containers running | `docker ps` shows several containers with names starting `supabase_` |
| Frontend container running | `docker ps` shows a `glowfy-spa-suite` container |
| Frontend loads | Open `http://localhost:8080` — the booking page should appear |
| Database reachable | Service/booking data loads on the client booking page |
| Studio accessible | Open `http://localhost:54323` — you should see your tables under Table Editor |
| Auth working | Staff login succeeds and redirects to the dashboard |

### Backend services overview

- **Authentication** — Supabase Auth manages staff/admin login, session persistence, and access control for the dashboard.
- **Bookings** — stored in Postgres; the client app reads/writes appointment records through Supabase's auto-generated REST API (PostgREST), scoped per spa (multi-tenant).
- **Services & Promotions** — service catalog and promotional data, managed by staff through the dashboard and stored in Postgres.
- **Admin Bootstrap (Edge Function)** — creates the first administrator account on initial setup, so the dashboard isn't accessible without a valid admin user.

---

## 10. Docker Deployment

This section contains everything needed to run the frontend in Docker — both for local use and as a base for production deployment.

### Dockerfile

```dockerfile
# ---- Build stage ----
FROM node:20-alpine AS build
WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# ---- Production stage ----
FROM nginx:alpine

COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
```

### nginx.conf

```nginx
server {
    listen 8080;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    # SPA fallback: any unknown route serves index.html
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location /assets/ {
        expires 7d;
        add_header Cache-Control "public, immutable";
    }
}
```

### docker-compose.yml

```yaml
version: "3.9"

services:
  glowfy-spa-suite:
    build: .
    container_name: glowfy-spa-suite
    ports:
      - "8080:8080"
    env_file:
      - .env
    restart: unless-stopped
```

> Save the three files above as `Dockerfile`, `nginx.conf`, and `docker-compose.yml` in the project root if they aren't already present.

### Everyday commands

```sh
# Start everything (backend + frontend), after the first-time setup above
supabase start
docker compose up -d

# Stop everything
docker compose down
supabase stop

# Rebuild the frontend after pulling new code or changing .env
docker compose up --build

# View frontend container logs
docker compose logs -f
```

> Reminder: since this is a static frontend, `VITE_SUPABASE_*` values are baked in **at build time**. If you change `.env`, you must rebuild with `docker compose up --build` for the change to take effect.

### Deploying beyond your local machine (optional)

The same `Dockerfile` can be used to deploy to any container host (e.g., a VPS, Render, Railway, Fly.io). For a Supabase **cloud** project instead of the local one, just point `VITE_SUPABASE_URL` / `VITE_SUPABASE_ANON_KEY` in `.env` to your hosted Supabase project's values before building the image. Static hosts like Vercel or Netlify are also an option if you'd rather not run a container at all — build command `npm run build`, output directory `dist`.

---

## 11. Troubleshooting

| Issue | Possible Cause | Recommended Solution |
|---|---|---|
| `docker: command not found` | Docker Desktop not installed or not running | Install Docker Desktop and make sure it's open (whale icon active) |
| `supabase start` hangs or fails | Docker isn't running, or ports are already in use | Make sure Docker Desktop is open; run `supabase stop` then `supabase start` again |
| Port `54321`/`54322`/`54323`/`8080` already in use | Another app or a previous run is using the port | Run `supabase stop` and `docker compose down`, close the conflicting app, then start again |
| Blank page / "Failed to fetch" errors | `.env` has the wrong Supabase URL/key, or container was built before `.env` was filled in | Update `.env`, then re-run `docker compose up --build` |
| Frontend container builds but shows old data after `.env` change | Vite bakes env vars in at build time | Always run `docker compose up --build` (not just `up`) after editing `.env` |
| Auth/login not working | Wrong anon key copied from `supabase start` output | Re-copy the `anon key` value exactly, rebuild the frontend |
| Booking/service data not loading | Migrations didn't apply | Run `supabase db reset` to reapply all migrations from scratch on your local database |
| `docker ps` shows no `supabase_*` containers | Backend was stopped or never started | Run `supabase start` |
| Everything was working, now nothing connects | Computer was restarted and Docker Desktop isn't running | Open Docker Desktop, then run `supabase start` and `docker compose up -d` again |

---

## 12. References

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Supabase Local Development Guide](https://supabase.com/docs/guides/local-development)
- [Supabase CLI Reference](https://supabase.com/docs/reference/cli/introduction)
- [Vite Documentation](https://vitejs.dev/)
- [React Documentation](https://react.dev/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [shadcn-ui Documentation](https://ui.shadcn.com/)