# Initialise New Project

You are scaffolding a new Nuxt 4 + Payload CMS monorepo from an empty repository.
Before writing a single file, ask the developer the questions below. Do not proceed
until all required answers are collected.

---

## Step 1 — Ask These Questions First

Ask only these two questions. Do not ask anything else. Do not scaffold anything yet.

```
Before I set up the project, I need two details:

1. Project name — lowercase, hyphenated
   e.g. kowju, spunic, welnez

2. Production domain (no trailing slash)
   e.g. kowju.com
```

All other settings are fixed defaults — do not ask about them:

| Setting | Fixed value |
|---|---|
| Node version | 20 (compatible with Nuxt 4 and latest Payload) |
| Database | SQLite |
| Payload plugins | `plugin-form-builder`, `plugin-seo`, `plugin-redirects`, `plugin-nested-docs` |
| Collections | `pages`, `blog-posts` (Posts/Categories/Tags), `rooms`, `media`, `users` |

The following are auto-derived — never ask the developer for these:

| Variable | Value |
|---|---|
| Staging web | `[project-name]-web-staging.zescloud.net` |
| Staging CMS | `[project-name]-cms-staging.zescloud.net` |
| Production CMS | `cms.[production-domain]` |
| Local web | `http://localhost:3000` |
| Local CMS | `http://localhost:3001` |

Wait for the two answers before proceeding to Step 2.

---

## Step 2 — Rename Folder and Invalidate Boilerplate Git

Before creating any files, handle the boilerplate repo hygiene. The developer cloned this
from the boilerplate repo — the folder name and git history must be replaced.

### 2a — Instruct folder rename

Output this message to the developer and wait for confirmation before proceeding:

```
⚠️  ACTION REQUIRED BEFORE I CONTINUE

The current folder is the boilerplate. You must rename it to your project name
before I scaffold anything — otherwise the IDE workspace path will be wrong.

Steps:
1. Close this IDE / Cursor window completely
2. In your file explorer or terminal, rename the folder:
   FROM: portico-foundation  (or whatever the boilerplate is named)
   TO:   [project-name]      (the name you gave in Step 1)
3. Reopen Cursor from the renamed folder
4. Reopen this chat and type "renamed" to continue

Do not skip this — the folder name is used in package.json, README, and git config.
```

Do not proceed until the developer types "renamed" or confirms the folder has been renamed.

### 2b — Invalidate boilerplate git

Once the developer confirms the rename, strip the boilerplate git history and reinitialise:

```bash
# Remove boilerplate git history
rm -rf .git

# Initialise a clean new repo
git init

# Set default branch to main
git branch -M main
```

Confirm with the developer: "Git reinitialised. The boilerplate remote has been disconnected.
Your new repo has no remote yet — we'll set that up after scaffolding."

---

## Step 3 — Scaffold Root Structure

Create the following at the repo root:

### `package.json` (workspace root)

```json
{
  "name": "[project-name]",
  "private": true,
  "workspaces": ["web", "cms"],
  "scripts": {
    "dev":       "npm run dev --workspaces --if-present",
    "dev:web":   "npm run dev --workspace=web",
    "dev:cms":   "npm run dev --workspace=cms",
    "build":     "npm run build --workspaces --if-present",
    "build:web": "npm run build --workspace=web",
    "build:cms": "npm run build --workspace=cms"
  },
  "engines": {
    "node": ">=[node-version].0.0",
    "npm": ">=10.0.0"
  }
}
```

### `.nvmrc`
```
[node-version]
```

### `.gitignore`
```gitignore
# Dependencies
node_modules/

# Environment variables
.env
.env.local
.env.*.local

# Nuxt build output
web/.nuxt/
web/.output/
web/dist/

# Payload build output
cms/.next/
cms/dist/
cms/build/

# SQLite
cms/*.db
cms/*.db-journal
cms/*.db-shm
cms/*.db-wal

# Media uploads
cms/public/media/
!cms/public/media/.gitkeep

# Logs
*.log
npm-debug.log*

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/settings.json
.idea/
```

### `README.md`
```markdown
# [Project Name]

Nuxt 4 + Payload CMS monorepo.

## Stack
- Frontend: Nuxt 4 (`web/`)
- CMS: Payload CMS (`cms/`)
- Database: SQLite (local) / [database choice] (production)
- Deployment: Coolify / VPS

## Environments

| Environment | Web | CMS |
|---|---|---|
| Local | http://localhost:3000 | http://localhost:3001 |
| Staging | https://[project-name]-web-staging.zescloud.net | https://[project-name]-cms-staging.zescloud.net |
| Production | https://[production-domain] | https://cms.[production-domain] |

## DNS Setup (Cloudflare)
See DNS Configuration section below before deploying to staging or production.

## Quick Start
See **Environment variables** in `NUXT-INSTRUCTIONS.md` (repo root) and `cms/.env.example` for required variables.

\`\`\`bash
npm install
npm run dev:cms   # Start CMS first — http://localhost:3001
npm run dev:web   # Then Nuxt — http://localhost:3000
\`\`\`
```

---

## Step 4 — Scaffold `web/` (Nuxt 4)

Run:
```bash
cd web
npx nuxi@latest init . --no-git
```

Then install modules using official commands — run each in order from inside `web/`:

```bash
# Nuxt official modules
npx nuxi module add icon
npx nuxi module add fonts
npx nuxi module add image
npx nuxi module add robots
npx nuxi module add @nuxtjs/sitemap
npx nuxi module add schema-org
npx nuxi module add nuxt-seo-utils
npx nuxi module add nuxt-ai-ready

# Nuxt UI + Tailwind
npm install @nuxt/ui tailwindcss

# AI skills
npx skilld add nuxt-og-image
npx skilld add nuxt-ai-ready
```

Install dev dependencies:
```bash
npm install -D typescript vue-tsc
```

Create `web/.env` using the **Environment variables** section in `NUXT-INSTRUCTIONS.md`.

Add `web/nuxt.config.ts` using the **`nuxt.config.ts`** section in `NUXT-INSTRUCTIONS.md`.

Create the folder structure shown under **Folder structure** in `NUXT-INSTRUCTIONS.md`.

Populate `web/app/assets/styles/reset.css`, `tokens.css`, and `utilities.css` using the
component-based-design-system skill.

---

## Step 5 — Scaffold `cms/` (Payload CMS)

Use `PAYLOAD-INSTRUCTIONS.md` (repo root) for ports, baseline plugins, `package.json` script shape, and env alignment.

Run:
```bash
cd cms
npx create-payload-app@latest . --no-git --template blank
```

Install plugins:
```bash
npm install @payloadcms/plugin-form-builder
```

If PostgreSQL was selected in Step 1:
```bash
npm install @payloadcms/db-postgres
```

Create `cms/.env.example`:
```bash
# ─────────────────────────────────────────────
# Payload CMS — Environment Variables
# Copy to .env and fill in values. Never commit .env.
# ─────────────────────────────────────────────

# Port Payload runs on
PORT=3001

# Database URI
# SQLite:     file:./database.db
# PostgreSQL: postgresql://user:password@localhost:5432/dbname
DATABASE_URI=file:./database.db

# JWT secret — minimum 32 characters
# Generate: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
PAYLOAD_SECRET=replace-with-32-char-minimum-random-string

# Nuxt frontend URL for CORS (no trailing slash)
# Development: http://localhost:3000
# Staging:     https://[project-name]-web-staging.zescloud.net
# Production:  https://[production-domain]
CORS_URL=http://localhost:3000

# First admin credentials (used only on initial database seed)
ADMIN_EMAIL=admin@[production-domain]
ADMIN_PASSWORD=change-this-immediately-after-first-login

# Persistent media storage path (production/staging only)
# MEDIA_DIR=/var/www/[project-name]/media
```

Create folder structure inside `cms/`:
```
cms/
└── src/
    ├── collections/
    │   ├── blocks/
    │   ├── Pages.ts
    │   ├── BlogPosts.ts
    │   ├── Rooms.ts
    │   ├── Media.ts
    │   └── Users.ts
    ├── globals/
    │   └── SiteSettings.ts
    ├── access/
    │   └── index.ts
    └── payload.config.ts
```

Scaffold each collection, global, and access file using the
`content-management-guidelines` skill — standard fields, access control, and the Media
collection image sizes.

Create `cms/public/media/.gitkeep` (empty file to preserve the directory in git).

---

## Step 6 — Install All Dependencies

From repo root:
```bash
npm install
```

This installs dependencies for both workspaces in one command.

---

## Step 7 — Copy `.env.example` to `.env`

```bash
# Create web/.env from **Environment variables** in NUXT-INSTRUCTIONS.md
cp cms/.env.example cms/.env
```

Then generate a `PAYLOAD_SECRET`:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Insert the output into `cms/.env` as `PAYLOAD_SECRET`.

Prompt the developer:
```
.env files created. Before starting the servers:

1. Open cms/.env and:
   - Replace PAYLOAD_SECRET with the generated value above
   - Set ADMIN_EMAIL and ADMIN_PASSWORD for first login

2. Open web/.env and confirm:
   - NUXT_PUBLIC_CMS_URL=http://localhost:3001

Ready to start? Type "start" to launch both servers.
```

---

## Step 8 — Initial Git Commit and Remote Setup

```bash
# Stage all scaffolded files
git add .

# Initial commit
git commit -m "Initial project scaffold — [project-name]"
```

Then prompt the developer:

```
✅ Initial commit created.

Now connect to your new remote repository:

1. Create a new empty repo on GitHub/GitLab (do NOT initialise with README)
2. Copy the remote URL and run:

   git remote add origin [your-repo-url]
   git push -u origin main

Type "pushed" once done, or "skip" to do this later.
```

---

## Step 9 — DNS Configuration (Cloudflare)

After the developer confirms the git push (or skips), output this DNS checklist:

```
🌐 DNS SETUP REQUIRED — Configure before deploying to staging or production

All DNS should be managed through Cloudflare. Set up the following records:

STAGING (point to your VPS IP)
┌─────────────────────────────────────────────────────────────────┐
│ Type  │ Name                                    │ Value         │
│ A     │ [project-name]-web-staging.zescloud.net │ [VPS IP]      │
│ A     │ [project-name]-cms-staging.zescloud.net │ [VPS IP]      │
└─────────────────────────────────────────────────────────────────┘

PRODUCTION (point to your VPS IP)
┌──────────────────────────────────────────────┐
│ Type  │ Name                  │ Value        │
│ A     │ [production-domain]   │ [VPS IP]     │
│ A     │ cms.[production-domain] │ [VPS IP]   │
└──────────────────────────────────────────────┘

Cloudflare settings per record:
  - Proxy status: Proxied (orange cloud) ✅
  - TTL: Auto
  - SSL/TLS mode: Full (strict) in Cloudflare SSL/TLS settings

After DNS propagates (usually 1–5 minutes with Cloudflare):
  - Configure Coolify with matching domains
  - Enable SSL in Coolify — Cloudflare handles the certificate
  - Verify staging is live before touching production DNS

Your VPS IP: check your Coolify server settings or VPS provider dashboard.
```

---

## Step 10 — Start Development Servers

When the developer confirms:

```bash
# Terminal 1 — start CMS first
npm run dev:cms

# Terminal 2 — then Nuxt
npm run dev:web
```

Confirm both are running:
- Payload admin: `http://localhost:3001/admin`
- Nuxt frontend: `http://localhost:3000`

Output final summary:
```
✅ Project scaffolded successfully.

Project:   [project-name]
Structure: web/ → Nuxt 4 | cms/ → Payload CMS

Environments:
  Local       web: http://localhost:3000
  Local       cms: http://localhost:3001/admin
  Staging     web: https://[project-name]-web-staging.zescloud.net
  Staging     cms: https://[project-name]-cms-staging.zescloud.net
  Production  web: https://[production-domain]
  Production  cms: https://cms.[production-domain]

Next steps:
1. Open http://localhost:3001/admin — create your first admin user
2. Review web/.env and cms/.env — update any remaining placeholder values
3. Configure DNS records in Cloudflare (see Step 9 above)
4. Push to your remote repo if not already done
5. Configure Coolify environments once DNS is pointing
6. Read environment-setup skill for full Coolify deployment configuration
7. Read git-workflow skill for branching and commit conventions

Project is ready to build.
```

---

## Rules for This Task

- Ask all questions in Step 1 before creating any file
- Wait for explicit "renamed" confirmation before proceeding past Step 2a
- Auto-derive all staging domains from project name — never ask the developer for staging URLs
- Development is always localhost — never add localhost as an env-specific question
- Substitute all domain placeholders with real values derived from Step 1 answers before writing any file
- Never create a `shared/` or `packages/` folder — this is a no-shared-code monorepo
- Start CMS before Nuxt — Payload must create its database before Nuxt boots
- Reference the relevant skill for each scaffolding step — do not invent field names, config values, or access patterns from memory
- Output the DNS checklist in Step 9 regardless of whether the developer pushed to git or skipped