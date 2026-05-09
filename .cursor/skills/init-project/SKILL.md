# Initialise New Project

You are scaffolding a new Nuxt 4 + Payload CMS monorepo from an empty repository.
Before writing a single file, ask the developer the questions below. Do not proceed
until all required answers are collected.

---

## Step 1 — Ask These Questions First

Ask all questions in one message. Do not scaffold anything yet.

```
Before I set up the project, I need a few details:

REQUIRED
1. Project name — lowercase, hyphenated (used for folder name, package names, and staging URLs)
   e.g. kowju, spunic, welnez

2. Production domain (client's live domain, no trailing slash)
   e.g. kowju.com

3. Node version preference
   Default: 20 — confirm or specify otherwise

OPTIONAL (skip to use defaults)
4. Database — SQLite (default) or PostgreSQL?
5. Payload plugins needed beyond defaults?
   Defaults: @payloadcms/plugin-form-builder, @payloadcms/plugin-seo,
             @payloadcms/plugin-redirects, @payloadcms/plugin-nested-docs
6. Any specific collections needed from day one?
   Defaults created: pages, blog-posts, rooms, media, users

The following will be auto-derived from your answers:
  Staging web:  [project-name]-web-staging.zescloud.net
  Staging CMS:  [project-name]-cms-staging.zescloud.net
  Production CMS: cms.[production-domain]
  Development: always http://localhost:3000 (web) and http://localhost:3001 (cms)
```

Wait for answers before proceeding to Step 2.

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
See `web/.env.example` and `cms/.env.example` for required variables.

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

Create `web/.env.example`:
```bash
# ─────────────────────────────────────────────
# Nuxt 4 — Web App Environment Variables
# Copy to .env and fill in values. Never commit .env.
# ─────────────────────────────────────────────

# Public URL of this Nuxt app (no trailing slash)
# Development: http://localhost:3000
# Staging:     https://[project-name]-web-staging.zescloud.net
# Production:  https://[production-domain]
NUXT_PUBLIC_SITE_URL=http://localhost:3000

# Site name — used by sitemap, schema-org, og-image modules
NUXT_PUBLIC_SITE_NAME=[Project Name]

# Payload CMS API base URL (no trailing slash)
# Development: http://localhost:3001
# Staging:     https://[project-name]-cms-staging.zescloud.net
# Production:  https://cms.[production-domain]
NUXT_PUBLIC_CMS_URL=http://localhost:3001

# Server-side API key for Payload requests (never expose to browser)
CMS_API_KEY=your-api-key-here
```

Update `web/nuxt.config.ts`:
```typescript
export default defineNuxtConfig({
  srcDir: 'app/',

  future: {
    compatibilityVersion: 4,
  },

  modules: [
    '@nuxt/icon',
    '@nuxt/fonts',
    '@nuxt/image',
    '@nuxt/ui',
    '@nuxtjs/robots',
    '@nuxtjs/sitemap',
    'nuxt-schema-org',
    'nuxt-seo-utils',
    'nuxt-og-image',
    'nuxt-ai-ready',
  ],

  nitro: {
    preset: 'static',
    prerender: {
      crawlLinks: true,
      routes: ['/', '/sitemap.xml'],
      failOnError: false,
    },
    compressPublicAssets: true,
  },

  routeRules: {
    '/**': { prerender: true },
    '/api/**': { prerender: false },
  },

  // Site identity — required by sitemap, schema-org, og-image
  site: {
    url:  process.env.NUXT_PUBLIC_SITE_URL,
    name: '[Project Name]',  // Replace with actual project name from Step 1
  },

  image: {
    provider: 'ipx',
    quality: 80,
    format: ['webp', 'avif', 'jpeg'],
  },

  fonts: {
    defaults: {
      weights: [400, 500, 700],
      styles: ['normal'],
      subsets: ['latin'],
    },
  },

  runtimeConfig: {
    cmsApiKey: process.env.CMS_API_KEY,
    public: {
      cmsUrl:  process.env.NUXT_PUBLIC_CMS_URL,
      siteUrl: process.env.NUXT_PUBLIC_SITE_URL,
    },
  },
})
```

Create folder structure inside `web/`:
```
web/
└── app/
    ├── assets/
    │   └── styles/
    │       ├── reset.css
    │       ├── tokens.css
    │       └── utilities.css
    ├── components/
    │   ├── global/
    │   ├── layout/
    │   ├── ui/
    │   ├── blocks/
    │   └── forms/
    ├── composables/
    ├── features/
    ├── layouts/
    │   └── default.vue
    └── pages/
        └── index.vue
```

Populate `web/app/assets/styles/reset.css`, `tokens.css`, and `utilities.css` using the
component-based-design-system skill.

---

## Step 5 — Scaffold `cms/` (Payload CMS)

Run:
```bash
cd cms
npx create-payload-app@latest . --no-git --template blank
```

Install plugins:
```bash
npm install @payloadcms/plugin-form-builder \
            @payloadcms/plugin-seo \
            @payloadcms/plugin-redirects \
            @payloadcms/plugin-nested-docs
```

Update `cms/src/payload.config.ts`:
```typescript
import { buildConfig }       from 'payload'
import { formBuilderPlugin } from '@payloadcms/plugin-form-builder'
import { seoPlugin }         from '@payloadcms/plugin-seo'
import { redirectsPlugin }   from '@payloadcms/plugin-redirects'
import { nestedDocsPlugin }  from '@payloadcms/plugin-nested-docs'
import { Pages }             from './collections/Pages'
import { Rooms }             from './collections/Rooms'
import { Media }             from './collections/Media'
import { Users }             from './collections/Users'
import { SiteSettings }      from './globals/SiteSettings'
import { BlogPosts }         from './collections/blog/BlogPosts'
import { BlogCategories }    from './collections/blog/BlogCategories'
import { BlogTags }          from './collections/blog/BlogTags'

export default buildConfig({
  // Order controls sidebar position within each admin group.
  // Blog group renders: Posts → Categories → Tags (matches array order below)
  collections: [Pages, Rooms, BlogPosts, BlogCategories, BlogTags, Media, Users],
  globals:     [SiteSettings],
  plugins: [
    formBuilderPlugin({ fields: { payment: false } }),
    seoPlugin({
      uploadsCollection: 'media',
      collections: ['pages', 'blog-posts', 'rooms'],
      generateTitle:       ({ doc }) => `${doc.title} — ${process.env.PAYLOAD_PUBLIC_SITE_NAME || ''}`,
      generateDescription: ({ doc }) => doc.excerpt || '',
      generateURL: ({ doc, collectionSlug }) => {
        const base = process.env.CORS_URL || ''
        if (collectionSlug === 'blog-posts') return `${base}/blog/${doc.slug}`
        return `${base}/${doc.slug}`
      },
    }),
    redirectsPlugin({ collections: ['pages', 'blog-posts'] }),
    nestedDocsPlugin({ collections: ['pages'] }),
  ],
  admin: {
    user: Users.slug,
    meta: { titleSuffix: '— CMS' },
  },
  typescript: { outputFile: 'types/payload-types.ts' },
  cors: [process.env.CORS_URL || ''],
  csrf: [process.env.CORS_URL || ''],
})
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
    │   ├── blog/
    │   │   ├── BlogPosts.ts
    │   │   ├── BlogCategories.ts
    │   │   └── BlogTags.ts
    │   ├── blocks/
    │   ├── Pages.ts
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

### Blog Collections — Scaffold All Three

Create `cms/src/collections/blog/BlogPosts.ts`:

```typescript
import type { CollectionConfig } from 'payload'
import {
  lexicalEditor, BoldFeature, ItalicFeature, UnderlineFeature,
  StrikethroughFeature, SubscriptFeature, SuperscriptFeature,
  InlineCodeFeature, LinkFeature, UnorderedListFeature, OrderedListFeature,
  ChecklistFeature, BlockquoteFeature, HeadingFeature,
  HorizontalRuleFeature, UploadFeature, BlocksFeature,
} from '@payloadcms/richtext-lexical'
import { isAdmin, isAdminOrEditor, isPublishedOrLoggedIn } from '../../access'

const CalloutBlock = {
  slug: 'callout',
  labels: { singular: 'Callout', plural: 'Callouts' },
  fields: [
    {
      name: 'type', type: 'select', defaultValue: 'info',
      options: [
        { label: 'Info',    value: 'info'    },
        { label: 'Warning', value: 'warning' },
        { label: 'Tip',     value: 'tip'     },
      ],
    },
    { name: 'content', type: 'textarea', required: true },
  ],
}

export const BlogPosts: CollectionConfig = {
  slug: 'blog-posts',
  labels: { singular: 'Post', plural: 'Posts' },
  admin: {
    useAsTitle: 'title',
    group: 'Blog',
    defaultColumns: ['title', 'status', 'publishedAt', 'author', 'updatedAt'],
    description: 'Blog posts, articles, and news.',
    livePreview: {
      url: ({ data }) => `${process.env.CORS_URL}/blog/${data?.slug}/preview`,
    },
  },
  versions: {
    maxPerDoc: 20,
    drafts: {
      autosave: { interval: 2000 },
    },
  },
  access: {
    read:         isPublishedOrLoggedIn,
    create:       isAdminOrEditor,
    update:       isAdminOrEditor,
    delete:       isAdmin,
    readVersions: isAdminOrEditor,
  },
  hooks: {
    beforeChange: [
      ({ data, operation }) => {
        if (data.status === 'published' && !data.publishedAt && operation === 'update') {
          return { ...data, publishedAt: new Date().toISOString() }
        }
        return data
      },
    ],
  },
  fields: [
    { name: 'title',    type: 'text',   required: true, maxLength: 120 },
    {
      name: 'slug', type: 'text', required: true, unique: true, index: true,
      admin: { position: 'sidebar', description: 'Auto-generated from title. Do not change after publishing.' },
      hooks: {
        beforeValidate: [
          ({ value, data }) =>
            value || data?.title?.toLowerCase().trim().replace(/\s+/g, '-').replace(/[^a-z0-9-]/g, ''),
        ],
      },
    },
    {
      name: 'excerpt', type: 'textarea', required: true, maxLength: 300,
      admin: { description: 'Short summary for blog listing, social shares, and search snippets. 150–300 characters.' },
    },
    {
      name: 'featuredImage', type: 'upload', relationTo: 'media', required: true,
      admin: { description: 'Main image. Used in listings and as OG image. Minimum 1200×630px.' },
    },
    {
      name: 'content', type: 'richText', required: true,
      editor: lexicalEditor({
        features: () => [
          HeadingFeature({ enabledHeadingSizes: ['h2', 'h3', 'h4'] }),
          BoldFeature(), ItalicFeature(), UnderlineFeature(), StrikethroughFeature(),
          SubscriptFeature(), SuperscriptFeature(), InlineCodeFeature(),
          LinkFeature(), UnorderedListFeature(), OrderedListFeature(),
          ChecklistFeature(), BlockquoteFeature(), HorizontalRuleFeature(),
          UploadFeature(),
          BlocksFeature({ blocks: [CalloutBlock] }),
        ],
      }),
    },
    {
      name: 'status', type: 'select', required: true, defaultValue: 'draft',
      options: [
        { label: 'Draft',     value: 'draft'     },
        { label: 'Published', value: 'published' },
        { label: 'Archived',  value: 'archived'  },
      ],
      admin: { position: 'sidebar', description: 'Draft = hidden. Archived = hidden but kept.' },
    },
    {
      name: 'publishedAt', type: 'date',
      admin: { position: 'sidebar', date: { pickerAppearance: 'dayAndTime' }, description: 'Auto-set on first publish.' },
    },
    {
      name: 'author', type: 'relationship', relationTo: 'users', required: true,
      admin: { position: 'sidebar' },
    },
    {
      name: 'categories', type: 'relationship', relationTo: 'blog-categories', hasMany: true,
      admin: { position: 'sidebar' },
    },
    {
      name: 'tags', type: 'relationship', relationTo: 'blog-tags', hasMany: true,
      admin: { position: 'sidebar' },
    },
    {
      name: 'readingTimeMinutes', type: 'number',
      admin: { position: 'sidebar', readOnly: true, description: 'Auto-calculated from content.' },
      hooks: {
        beforeChange: [
          ({ data }) => {
            const words = JSON.stringify(data?.content || '').split(/\s+/).length
            return Math.max(1, Math.ceil(words / 200))
          },
        ],
      },
    },
    {
      name: 'isFeatured', type: 'checkbox', defaultValue: false,
      admin: { position: 'sidebar', description: 'Pin to featured position on blog listing.' },
    },
    {
      name: 'disableIndex', type: 'checkbox', defaultValue: false,
      admin: { position: 'sidebar', description: 'Add noindex to this post.' },
    },
    {
      name: 'relatedPosts', type: 'relationship', relationTo: 'blog-posts', hasMany: true, maxRows: 3,
      admin: { description: 'Manually curated related posts. Max 3.' },
    },
  ],
}
```

Create `cms/src/collections/blog/BlogCategories.ts`:

```typescript
import type { CollectionConfig } from 'payload'
import { isAdmin, isAdminOrEditor, isPublic } from '../../access'

export const BlogCategories: CollectionConfig = {
  slug: 'blog-categories',
  labels: { singular: 'Category', plural: 'Categories' },
  admin: {
    useAsTitle: 'name',
    group: 'Blog',
    defaultColumns: ['name', 'slug', 'description'],
    description: 'Top-level content groupings for blog posts.',
  },
  access: {
    read:   isPublic,
    create: isAdminOrEditor,
    update: isAdminOrEditor,
    delete: isAdmin,
  },
  fields: [
    { name: 'name', type: 'text', required: true, maxLength: 80 },
    {
      name: 'slug', type: 'text', required: true, unique: true, index: true,
      admin: { description: 'Auto-generated from name. Do not change after publishing.' },
      hooks: {
        beforeValidate: [
          ({ value, data }) =>
            value || data?.name?.toLowerCase().trim().replace(/\s+/g, '-').replace(/[^a-z0-9-]/g, ''),
        ],
      },
    },
    {
      name: 'description', type: 'textarea', maxLength: 200,
      admin: { description: 'Shown on the category archive page.' },
    },
    {
      name: 'coverImage', type: 'upload', relationTo: 'media',
      admin: { description: 'Optional hero image for the category archive page.' },
    },
    {
      name: 'parent', type: 'relationship', relationTo: 'blog-categories',
      admin: { description: 'Optional parent for hierarchical categories.' },
    },
    { name: 'order', type: 'number', defaultValue: 0, admin: { description: 'Sort order. Lower = first.' } },
  ],
}
```

Create `cms/src/collections/blog/BlogTags.ts`:

```typescript
import type { CollectionConfig } from 'payload'
import { isAdmin, isAdminOrEditor, isPublic } from '../../access'

export const BlogTags: CollectionConfig = {
  slug: 'blog-tags',
  labels: { singular: 'Tag', plural: 'Tags' },
  admin: {
    useAsTitle: 'name',
    group: 'Blog',
    defaultColumns: ['name', 'slug'],
    description: 'Granular labels for filtering and discovery.',
  },
  access: {
    read:   isPublic,
    create: isAdminOrEditor,
    update: isAdminOrEditor,
    delete: isAdmin,
  },
  fields: [
    { name: 'name', type: 'text', required: true, maxLength: 50 },
    {
      name: 'slug', type: 'text', required: true, unique: true, index: true,
      admin: { description: 'Auto-generated from name.' },
      hooks: {
        beforeValidate: [
          ({ value, data }) =>
            value || data?.name?.toLowerCase().trim().replace(/\s+/g, '-').replace(/[^a-z0-9-]/g, ''),
        ],
      },
    },
    {
      name: 'description', type: 'text', maxLength: 160,
      admin: { description: 'Optional description for the tag archive page.' },
    },
  ],
}
```

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
cp web/.env.example web/.env
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