## Part 8: nuxt.config.ts — Full Performance Config

```typescript
// nuxt.config.ts — marketing site SSG baseline
export default defineNuxtConfig({
  srcDir: 'app/',

  future: {
    compatibilityVersion: 4,
  },

  modules: [
    '@nuxt/image',
    '@nuxt/fonts',
    '@nuxt/scripts',
  ],

  nitro: {
    preset: 'static',
    prerender: {
      crawlLinks: true,
      routes: ['/', '/sitemap.xml'],
      failOnError: false,
    },
    // Compress output
    compressPublicAssets: true,
  },

  routeRules: {
    '/**': { prerender: true },
    '/api/**': { prerender: false },
  },

  image: {
    provider: 'ipx',
    quality: 80,
    format: ['webp', 'avif', 'jpeg'],
    screens: {
      xs: 320, sm: 640, md: 768,
      lg: 1024, xl: 1280, xxl: 1536,
    },
  },

  fonts: {
    defaults: {
      weights: [400, 500, 700],
      styles: ['normal'],
      subsets: ['latin'],
    },
  },

  // Vite build optimisations
  vite: {
    build: {
      cssCodeSplit: true,       // Split CSS per page
      rollupOptions: {
        output: {
          manualChunks: {
            // Vendor chunk for shared dependencies
            vendor: ['vue', 'vue-router'],
          },
        },
      },
    },
  },

  // Experimental performance features
  experimental: {
    payloadExtraction: false,   // Smaller HTML for SSG — no JSON payload embedded
    renderJsonPayloads: false,
  },
})
```

---

## Part 9: Core Web Vitals — What Breaks Each Metric

### LCP (Largest Contentful Paint) — Common Causes

| Cause | Fix |
|---|---|
| Hero image using `loading="lazy"` | Switch to `loading="eager"` + `fetchpriority="high"` |
| Hero image not using `<NuxtImg>` | Replace raw `<img>` with `<NuxtImg>` |
| No `width`/`height` on hero | Add dimensions — browser can't reserve space without them |
| Large uncompressed image | Use `format="webp"` + `quality="80"` |
| Render-blocking font | Use Nuxt Fonts or add manual `preload` |
| Slow TTFB (server not on CDN) | Deploy to CDN edge (Netlify, Cloudflare Pages, Vercel) |

### CLS (Cumulative Layout Shift) — Common Causes

| Cause | Fix |
|---|---|
| `<NuxtImg>` without `width`/`height` | Always declare dimensions |
| Font swap causing text reflow | Use Nuxt Fonts (generates matching fallback metrics) |
| Dynamic content inserted above existing content | Reserve space with `min-height` on container |
| Ads or embeds without size reservation | Wrap in fixed-size container |

### INP (Interaction to Next Paint) — Common Causes

| Cause | Fix |
|---|---|
| Heavy JS executing on main thread | Break into chunks with `scheduler.yield()` |
| Third-party scripts blocking interaction | Load with Nuxt Scripts + deferred trigger |
| Unhydrated components responding slowly | Use `hydrate-on-interaction` — defer hydration |
| Large reactive objects re-rendering | Make only changing data reactive |
| Event listeners not cleaned up | Always pair `onMounted` with `onUnmounted` cleanup |

---

## Part 10: Pre-Launch Performance Checklist

### SSG
- [ ] `nitro.preset` set to `'static'`
- [ ] `crawlLinks: true` enabled
- [ ] All pages confirmed prerendered in build output (`dist/` or `.output/public/`)
- [ ] No marketing pages using SSR or CSR

### Images
- [ ] All images use `<NuxtImg>` — no raw `<img>` tags
- [ ] Hero/above-fold image: `loading="eager"` + `fetchpriority="high"`
- [ ] All other images: `loading="lazy"`
- [ ] `width` and `height` declared on every `<NuxtImg>`
- [ ] `format="webp"` on every `<NuxtImg>`
- [ ] `sizes` attribute set on full-width and responsive images

### Fonts
- [ ] Nuxt Fonts module installed and configured
- [ ] No Google Fonts `<link>` tags in HTML head
- [ ] Only required weights and subsets declared

### Scripts
- [ ] No raw `<script src="...">` tags in components
- [ ] Analytics loaded with `trigger: 'onNuxtReady'`
- [ ] Chat / support widgets loaded with `trigger: 'manual'`

### Components
- [ ] Heavy below-fold sections use `Lazy` prefix
- [ ] `hydrate-on-visible` applied to below-fold lazy components
- [ ] `hydrate-on-idle` or `hydrate-never` on footer-level components
- [ ] Event listeners cleaned up in `onUnmounted`
- [ ] Only changing data is `ref()` / `reactive()` — static data is plain constants

### Links
- [ ] All internal links use `<NuxtLink>` — no `<a href>` on internal routes

### Validation
- [ ] PageSpeed Insights ≥ 90 on mobile
- [ ] Lighthouse ≥ 90 on mobile
- [ ] No CLS > 0.1 in PageSpeed field data
- [ ] LCP < 2.5s on mobile (throttled connection)
- [ ] Bundle analyser run — no chunk > 250KB

---

## Validation Tools

| Tool | What to check |
|---|---|
| [PageSpeed Insights](https://pagespeed.web.dev) | CWV field data + lab diagnostics |
| Chrome DevTools → Lighthouse | Full performance audit |
| Chrome DevTools → Performance tab | Main thread blocking, long tasks |
| `npx nuxt build --analyze` | Bundle size and chunk breakdown |
| [WebPageTest](https://www.webpagetest.org) | Waterfall, TTFB, CDN delivery |
| Nuxt DevTools | Hydration timeline, module analysis |
