---
name: environment-setup
description: >
  Complete environment setup guide for Nuxt 4 + Payload CMS monorepo projects — from first
  git clone to a fully running local development environment. Covers monorepo workspace
  structure, .env configuration, SQLite setup, local dev workflow, and Coolify/VPS deployment
  environment configuration. Use this skill whenever setting up a new project from scratch,
  onboarding a new developer, or configuring environment variables for local, staging, or
  production. Trigger when the user mentions environment setup, .env, local dev, onboarding,
  workspace setup, Coolify deployment config, or asks how to get the project running.
---

# Environment Setup

## Stack

| Layer | Technology |
|---|---|
| Frontend | Nuxt 4 (`apps/web`) |
| CMS / Backend | Payload CMS (`apps/cms`) |
| Database | SQLite (local + staging), PostgreSQL (production optional) |
| Package manager | npm workspaces |
| Deployment | Coolify on VPS |
| Repo structure | Monorepo — isolated apps, API-only integration |

## Architecture Rule

Nuxt and Payload are two independent applications that happen to live in the same repository.
They communicate **only through HTTP API calls** — Nuxt fetches data from Payload's REST API.
Never import code directly from `apps/cms` into `apps/web` or vice versa.

---

## Part 1: Repository Structure

```
project-root/
├── apps/
│   ├── web/                    ← Nuxt 4 frontend
│   │   ├── app/                ← Nuxt 4 src directory
│   │   │   ├── components/
│   │   │   ├── composables/
│   │   │   ├── pages/
│   │   │   └── layouts/
│   │   ├── public/
│   │   ├── .env                ← gitignored
│   │   ├── .env.example        ← committed to repo
│   │   ├── nuxt.config.ts
│   │   └── package.json
│   └── cms/                    ← Payload CMS backend
│       ├── src/
│       │   ├── collections/
│       │   ├── globals/
│       │   ├── access/
│       │   └── payload.config.ts
│       ├── public/
│       ├── .env                ← gitignored
│       ├── .env.example        ← committed to repo
│       └── package.json
├── .gitignore
├── .nvmrc                      ← Node version lock
├── package.json                ← Workspace root
└── README.md
```

### Root `package.json` (Workspace Config)

```json
{
  "name": "project-name",
  "private": true,
  "workspaces": [
    "apps/web",
    "apps/cms"
  ],
  "scripts": {
    "dev":        "npm run dev --workspaces --if-present",
    "dev:web":    "npm run dev --workspace=apps/web",
    "dev:cms":    "npm run dev --workspace=apps/cms",
    "build":      "npm run build --workspaces --if-present",
    "build:web":  "npm run build --workspace=apps/web",
    "build:cms":  "npm run build --workspace=apps/cms"
  },
  "engines": {
    "node": ">=20.0.0",
    "npm":  ">=10.0.0"
  }
}
```

### `.nvmrc`

```
20
```

---

## Part 2: Environment Variables

### Rules

- `.env` is **always gitignored** — never committed
- `.env.example` is **always committed** — contains all variable names with placeholder values and descriptions
- Every variable used in code must have an entry in `.env.example`
- Production secrets (API keys, database URLs) are set in Coolify's environment panel — never in the repo
- No hardcoded values in `nuxt.config.ts` or `payload.config.ts` — always `process.env.VARIABLE`

### `apps/web/.env.example`

```bash
# ─────────────────────────────────────────────
# Nuxt 4 — Web App Environment Variables
# ─────────────────────────────────────────────
# Copy this file to .env and fill in values.
# NEVER commit .env to the repository.
# ─────────────────────────────────────────────

# ── Site ──────────────────────────────────────
# Public URL of this Nuxt app (no trailing slash)
NUXT_PUBLIC_SITE_URL=http://localhost:3000

# ── Payload CMS API ───────────────────────────
# Base URL of the Payload CMS API (no trailing slash)
# Local:      http://localhost:3001
# Staging:    https://cms.staging.yourdomain.com
# Production: https://cms.yourdomain.com
NUXT_PUBLIC_CMS_URL=http://localhost:3001

# API key for server-side Payload requests (set in Payload admin → API Keys)
# Never expose this in client-side code (no NUXT_PUBLIC_ prefix)
CMS_API_KEY=your-api-key-here

# ── Analytics (optional) ──────────────────────
# NUXT_PUBLIC_GA_ID=G-XXXXXXXXXX
# NUXT_PUBLIC_GTM_ID=GTM-XXXXXXX

# ── Feature Flags (optional) ──────────────────
# NUXT_PUBLIC_SHOW_BLOG=true
```

### `apps/cms/.env.example`

```bash
# ─────────────────────────────────────────────
# Payload CMS — Environment Variables
# ─────────────────────────────────────────────
# Copy this file to .env and fill in values.
# NEVER commit .env to the repository.
# ─────────────────────────────────────────────

# ── Server ────────────────────────────────────
# Port Payload runs on locally
PORT=3001

# ── Database ──────────────────────────────────
# SQLite database file path (relative to apps/cms)
# Local/staging: file-based SQLite
# Production: switch to DATABASE_URL with PostgreSQL if required
DATABASE_URI=file:./database.db

# ── Payload ───────────────────────────────────
# Secret used to sign JWTs — minimum 32 characters — generate with:
# node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
PAYLOAD_SECRET=replace-with-32-char-minimum-random-string

# ── CORS ──────────────────────────────────────
# URL of the Nuxt frontend (no trailing slash)
# Local:      http://localhost:3000
# Staging:    https://staging.yourdomain.com
# Production: https://yourdomain.com
CORS_URL=http://localhost:3000

# ── Admin ─────────────────────────────────────
# First admin user — used only on initial seed, ignored after
ADMIN_EMAIL=admin@yourdomain.com
ADMIN_PASSWORD=change-this-immediately-after-first-login

# ── Media ─────────────────────────────────────
# Absolute path where uploaded media files are stored
# Local: leave blank (uses default public/media)
# Production: set to persistent VPS path outside the app directory
# MEDIA_DIR=/var/www/project/media
```

### Variable Naming Conventions

| Prefix | Scope | Example |
|---|---|---|
| `NUXT_PUBLIC_` | Nuxt — exposed to browser | `NUXT_PUBLIC_CMS_URL` |
| No prefix (Nuxt) | Nuxt — server-side only | `CMS_API_KEY` |
| No prefix (Payload) | Payload — server-side only | `PAYLOAD_SECRET` |

**Rule:** Never use `NUXT_PUBLIC_` prefix on secrets. API keys, secrets, and database URIs must never be prefixed with `NUXT_PUBLIC_` — they would be exposed in the client bundle.

---

## Part 3: First-Time Local Setup

Run through these steps in order. Do not skip steps.

### Prerequisites

```bash
# Verify Node version (must be >=20)
node --version

# Install correct Node version if needed (using nvm)
nvm install 20
nvm use 20

# Verify npm version (must be >=10)
npm --version
```

### Step 1 — Clone and install

```bash
# Clone the repository
git clone git@github.com:your-org/project-name.git
cd project-name

# Install all workspace dependencies (installs apps/web and apps/cms together)
npm install
```

### Step 2 — Configure environment variables

```bash
# Web app
cp apps/web/.env.example apps/web/.env

# CMS
cp apps/cms/.env.example apps/cms/.env
```

Open both `.env` files and fill in:
- `apps/web/.env` — `NUXT_PUBLIC_CMS_URL` set to `http://localhost:3001`
- `apps/cms/.env` — generate a `PAYLOAD_SECRET` (command is in the `.env.example`)
- `apps/cms/.env` — set `ADMIN_EMAIL` and `ADMIN_PASSWORD` for initial login

### Step 3 — Generate `PAYLOAD_SECRET`

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
# Copy output into PAYLOAD_SECRET in apps/cms/.env
```

### Step 4 — Start development servers

```bash
# Start both apps concurrently from workspace root
npm run dev

# Or start individually in separate terminals:
npm run dev:cms   # Payload on http://localhost:3001
npm run dev:web   # Nuxt on http://localhost:3000
```

> Start CMS first on initial setup — Payload creates the SQLite database file and runs
> migrations on first start. Nuxt will error if the CMS API is unreachable on boot.

### Step 5 — Create first admin user

1. Open `http://localhost:3001/admin`
2. Payload prompts for first admin setup on fresh database
3. Use the email and password from `apps/cms/.env`
4. Change the password immediately after first login

### Step 6 — Verify connection

```bash
# Confirm Payload API is responding
curl http://localhost:3001/api/pages

# Open Nuxt frontend
open http://localhost:3000
```

If Nuxt returns a fetch error, check:
- `NUXT_PUBLIC_CMS_URL` in `apps/web/.env` matches Payload's port
- Payload started without errors (check terminal output)
- `CORS_URL` in `apps/cms/.env` includes `http://localhost:3000`

---

## Part 4: `.gitignore`

Root `.gitignore` — covers both apps:

```gitignore
# Dependencies
node_modules/

# Environment variables — NEVER commit
.env
.env.local
.env.*.local

# Nuxt build output
apps/web/.nuxt/
apps/web/.output/
apps/web/dist/

# Payload build output
apps/cms/.next/
apps/cms/dist/
apps/cms/build/

# SQLite database — local data, not committed
apps/cms/*.db
apps/cms/*.db-journal
apps/cms/*.db-shm
apps/cms/*.db-wal

# Media uploads — not committed (too large, managed separately in production)
apps/cms/public/media/
!apps/cms/public/media/.gitkeep

# Logs
*.log
npm-debug.log*

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/settings.json
.idea/
*.swp
```

> Media uploads are gitignored. In production, media is stored at a persistent path on the VPS
> outside the app directory. In staging, uploads are ephemeral — acceptable for testing.

---

## Part 5: Coolify / VPS Deployment Configuration

### Environment Separation

| Environment | Branch | Domain | Auto-deploy |
|---|---|---|---|
| Local | — | `localhost:3000` / `localhost:3001` | Manual (`npm run dev`) |
| Staging | `develop` | `staging.yourdomain.com` | On push |
| Production | `main` | `yourdomain.com` | On push (or manual confirm) |

### Coolify App Setup — Nuxt (Web)

```
Repository:        github.com/your-org/project-name
Branch:            main (production) / develop (staging)
Root Directory:    apps/web
Build Command:     npm run build
Start Command:     node .output/server/index.mjs
Publish Directory: .output/public   ← for static; omit for SSR
Port:              3000
```

> For SSG (marketing sites): Build command is `npm run generate`.
> Output directory is `.output/public`.
> No start command needed — serve static files from CDN/Nginx.

### Coolify App Setup — Payload (CMS)

```
Repository:        github.com/your-org/project-name
Branch:            main (production) / develop (staging)
Root Directory:    apps/cms
Build Command:     npm run build
Start Command:     npm run start
Port:              3001
```

### Environment Variables in Coolify

Set these in Coolify's **Environment Variables** panel for each app — not in the repo.

**Nuxt app (Coolify):**

```
NUXT_PUBLIC_SITE_URL=https://yourdomain.com
NUXT_PUBLIC_CMS_URL=https://cms.yourdomain.com
CMS_API_KEY=<production-api-key>
```

**Payload app (Coolify):**

```
PORT=3001
DATABASE_URI=file:/var/www/project/database.db
PAYLOAD_SECRET=<production-secret-minimum-32-chars>
CORS_URL=https://yourdomain.com
ADMIN_EMAIL=admin@yourdomain.com
MEDIA_DIR=/var/www/project/media
```

### Persistent Storage on VPS

SQLite and media uploads must be stored outside the app directory — Coolify redeploys wipe
the container filesystem.

In Coolify → Storage:
```
Host path:      /var/www/project-name/data
Container path: /var/www/project

# This maps to:
DATABASE_URI=file:/var/www/project/database.db
MEDIA_DIR=/var/www/project/media
```

> Create the host paths before first deploy:
> ```bash
> mkdir -p /var/www/project-name/data/media
> ```

---

## Part 6: Adding a New Developer

Checklist for onboarding a new team member:

```
□ Grant GitHub repo access
□ Share .env values securely (password manager or encrypted message — never Slack/email plaintext)
□ Confirm Node >=20 installed on their machine
□ Confirm they've run: git clone → npm install → cp .env.example .env → fill values → npm run dev
□ Confirm Payload admin accessible at localhost:3001/admin
□ Confirm Nuxt frontend accessible at localhost:3000
□ Create their admin user in Payload (or have them register on first-run)
□ Walk through branch strategy (see git-workflow skill)
```

---

## Quick Reference

```
FIRST TIME SETUP:
  git clone → npm install
  cp apps/web/.env.example  apps/web/.env   (fill in values)
  cp apps/cms/.env.example  apps/cms/.env   (fill in values + generate PAYLOAD_SECRET)
  npm run dev:cms            (start CMS first)
  npm run dev:web            (then Nuxt)
  Open localhost:3001/admin  (create first admin)
  Open localhost:3000        (verify frontend)

DAILY DEV:
  npm run dev                (both apps)
  npm run dev:web            (Nuxt only)
  npm run dev:cms            (Payload only)

ENV RULES:
  .env          → gitignored, never committed
  .env.example  → committed, always up to date
  NUXT_PUBLIC_  → browser-safe variables only
  No prefix     → server-side only (secrets, API keys, DB URIs)

PRODUCTION:
  Secrets set in Coolify env panel — never in repo
  SQLite + media on persistent VPS volume outside app directory
  Staging → develop branch
  Production → main branch
```
