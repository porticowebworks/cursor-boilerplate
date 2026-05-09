# Payload CMS — bootstrap prompt

Use this file as the instruction block when scaffolding or extending the `cms/` app for a new client site. Follow steps in order; align env values with the **Environment variables** section in `cursor-boilerplate/web/README.md`.

## Scaffold

```bash
cd cms
npx create-payload-app@latest
```

## Plugins (baseline)

```bash
npm install \
  @payloadcms/plugin-form-builder \
  @payloadcms/plugin-seo \
  @payloadcms/plugin-redirects \
  @payloadcms/plugin-nested-docs
```

Adjust versions to match the generated Payload major version.

## Windows (optional native bindings)

Install if postinstall or builds complain about missing MSVC bindings (run from `cms/` after the project exists):

```bash
npm install @oxc-transform/binding-win32-x64-msvc@^0.126.0 @rollup/rollup-win32-x64-msvc@^4.60.2 @oxc-minify/binding-win32-x64-msvc@^0.126.0 lightningcss-win32-x64-msvc@^1.32.0
```

## Environment

Copy the block below to `cms/.env` (do not commit `.env`). Replace placeholders.

```env
# ─────────────────────────────────────────────
# Payload CMS — Environment Variables
# ─────────────────────────────────────────────

PORT=3001

# SQLite:     file:./database.db
# PostgreSQL: postgresql://user:password@localhost:5432/dbname
DATABASE_URI=file:./database.db

# JWT secret — minimum 32 characters
# node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
PAYLOAD_SECRET=replace-with-32-char-minimum-random-string

# Must match Nuxt NUXT_PUBLIC_SITE_URL (no trailing slash)
CORS_URL=http://localhost:3000

ADMIN_EMAIL=admin@[production-domain]
ADMIN_PASSWORD=change-this-immediately-after-first-login

# Persistent media (production/staging)
# MEDIA_DIR=/var/www/[project-name]/media
```

## Cross-check with Nuxt

- `CORS_URL` === `NUXT_PUBLIC_SITE_URL`
- Nuxt `NUXT_PUBLIC_CMS_URL` points at this app base URL (e.g. `http://localhost:3001` in dev)

## Agent constraints

- Keep `show_in_rest` and REST-exposed meta aligned with how the Nuxt app consumes collections (see workspace CPT/builder exposure rules if applicable).
- Do not commit secrets or `.env`; add `cms/.env.example` with dummy values if the repo needs a template.

