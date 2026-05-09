## Bootstrap

```bash
npx nuxi@latest
```

### Nuxt official modules

```bash
npx nuxi module add icon
npx nuxi module add fonts
npx nuxi module add image
npx nuxi module add robots
npx nuxi module add @nuxtjs/sitemap
npx nuxi module add schema-org
npx nuxi module add nuxt-seo-utils
npx nuxi module add nuxt-ai-ready
```

### Nuxt UI + Tailwind

```bash
npm install @nuxt/ui tailwindcss
```

### AI skills

```bash
npx skilld add nuxt-og-image
npx skilld add nuxt-ai-ready
```

## Folder structure

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

## `nuxt.config.ts`

Save as `nuxt.config.ts` in the project root (adjust `site.name` and env usage as needed).

```ts
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

## Environment variables

Copy to `.env` in the project root and fill in values. Do not commit `.env`.

```env
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
