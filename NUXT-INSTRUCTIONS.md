## Bootstrap

```bash
npx nuxi@latest init
```

### Nuxt official modules

```bash
npx nuxi module add icon fonts image robots @nuxtjs/sitemap schema-org nuxt-seo-utils nuxt-ai-ready

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

### Windows (optional native bindings)

Install if postinstall or builds complain about missing MSVC bindings (run from the Nuxt project root after `npm install`):

```bash
npm install @oxc-transform/binding-win32-x64-msvc@^0.126.0 @rollup/rollup-win32-x64-msvc@^4.60.2 @oxc-minify/binding-win32-x64-msvc@^0.126.0 lightningcss-win32-x64-msvc@^1.32.0
```

### AI skills

```bash
npx skilld add nuxt-og-image nuxt-ai-ready

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



