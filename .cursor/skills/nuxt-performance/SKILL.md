---
name: nuxt-performance
description: >
  Full performance engineering for Nuxt 4 marketing sites. Covers SSG configuration, NuxtImg
  image optimisation, Nuxt Fonts, bundle splitting, lazy components, lazy hydration, third-party
  script handling via Nuxt Scripts, Core Web Vitals targets, Nitro static output config, prefetch
  strategy, reactive state hygiene, and pre-launch performance checklist.
  Use this skill whenever building, auditing, or optimising a Nuxt 4 website for speed —
  including new page builds, image implementation, component loading decisions, font setup,
  third-party script integration, or Core Web Vitals failures.
  Trigger when the user mentions performance, page speed, LCP, CLS, INP, NuxtImg, lazy loading,
  bundle size, hydration, Nuxt Fonts, Nuxt Scripts, or asks how to make a Nuxt page faster.
  Default rendering strategy: SSG for all pages on marketing sites.
---

# Nuxt 4 Performance Engineering

## Core Principle

Marketing sites have one performance rule: **serve pre-built static HTML from a CDN edge, fully
rendered, with no runtime server required.** Every other optimisation builds on top of this.
SSG is not a choice per project — it is the default. Any exception must be explicitly justified.

> **All code examples are samples for reference only.** Adapt to your actual project structure,
> image paths, and Nuxt 4 module versions.

---

## Performance Targets (Non-Negotiable)

| Metric | Target | What it affects |
|---|---|---|
| **LCP** (Largest Contentful Paint) | < 2.5s | Main content load speed — direct ranking signal |
| **CLS** (Cumulative Layout Shift) | < 0.1 | Visual stability — images, fonts, dynamic inserts |
| **INP** (Interaction to Next Paint) | < 200ms | Responsiveness to clicks and taps |
| **TTFB** (Time to First Byte) | < 200ms | CDN delivery speed |
| **FCP** (First Contentful Paint) | < 1.8s | First visible content |
| **Lighthouse Score** | ≥ 90 (mobile) | Composite performance score |

---

## Part 1: SSG Configuration (Nuxt 4)

### 1.1 Global SSG Setup

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Nuxt 4 app directory structure
  srcDir: 'app/',

  nitro: {
    preset: 'static',           // Output pure static HTML/CSS/JS
    prerender: {
      crawlLinks: true,         // Auto-discover all routes from links
      routes: ['/'],            // Seed route — crawler starts here
      failOnError: false,       // Don't fail build on single page error
    },
  },

  // Nuxt 4 compatibility flag
  future: {
    compatibilityVersion: 4,
  },
})
```

### 1.2 Explicit Route Prerendering

For routes not reachable by link crawling (dynamic routes, sitemaps):

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    prerender: {
      crawlLinks: true,
      routes: [
        '/',
        '/sitemap.xml',   // Ensure sitemap is pre-rendered
      ],
    },
  },

  routeRules: {
    // All routes default to static prerender
    '/**': { prerender: true },

    // Exception: form submission endpoints are server-only
    '/api/**': { prerender: false },
  },
})
```

### 1.3 When NOT to Use SSG (Exceptions Only)

Flag any of these as requiring explicit justification before switching a route off SSG:

| Scenario | Allowed exception | Solution |
|---|---|---|
| User-specific dashboard | ✅ Use CSR for that route only | `routeRules: { '/dashboard/**': { ssr: false } }` |
| Real-time inventory / pricing | ✅ Use client-side fetch after SSG shell loads | SSG page + `useFetch` on mount |
| Form API endpoints | ✅ Server route, not a page | `/server/api/` — never prerendered |
| Everything else on a marketing site | ❌ No exception | Keep as SSG |

> 🚩 **Flag:** If a developer proposes SSR for a marketing page, challenge it. The question is "why can't this be SSG?" not "should we use SSR?"

---

## Part 2: NuxtImg — Image Optimisation

**Rule: Never use a raw `<img>` tag on a Nuxt 4 project.** Always use `<NuxtImg>` or `<NuxtPicture>`.

### 2.1 Module Setup

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/image'],

  image: {
    // Static sites use IPX (built-in) for build-time processing
    provider: 'ipx',

    // Define responsive breakpoints
    screens: {
      xs:  320,
      sm:  640,
      md:  768,
      lg:  1024,
      xl:  1280,
      xxl: 1536,
    },

    // Default image quality
    quality: 80,

    // Allowed formats
    format: ['webp', 'avif', 'jpeg'],
  },
})
```

### 2.2 Hero / Above-Fold Images (LCP Critical)

The hero image is almost always the LCP element. It must load with maximum priority.

```vue
<!-- ✅ Hero image — LCP optimised -->
<NuxtImg
  src="/images/hero.jpg"
  alt="Descriptive alt text matching page topic"
  width="1440"
  height="600"
  format="webp"
  loading="eager"
  fetchpriority="high"
  sizes="100vw"
/>
```

**Rules for hero images:**
- `loading="eager"` — never lazy on above-fold images
- `fetchpriority="high"` — tells browser to prioritise this resource
- `width` and `height` always declared — prevents CLS from layout reflow
- `sizes="100vw"` for full-width heroes — browser picks correct srcset breakpoint
- `format="webp"` — 25–35% smaller than JPEG, supported by all modern browsers

### 2.3 Below-Fold Images (All Other Images)

```vue
<!-- ✅ Standard below-fold image -->
<NuxtImg
  src="/images/room-deluxe.jpg"
  alt="Deluxe sea view room with king bed and floor-to-ceiling windows"
  width="800"
  height="600"
  format="webp"
  loading="lazy"
  sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 800px"
/>
```

### 2.4 Responsive Images with NuxtPicture

Use `<NuxtPicture>` when you need format fallback (AVIF → WebP → JPEG) or art direction (different crops per breakpoint):

```vue
<!-- ✅ Multi-format with fallback -->
<NuxtPicture
  src="/images/team-photo.jpg"
  alt="The Portico Webworks team"
  width="1200"
  height="800"
  :imgAttrs="{ loading: 'lazy', class: 'team-photo' }"
  sizes="(max-width: 768px) 100vw, 1200px"
  format="avif,webp,jpeg"
/>
```

### 2.5 NuxtImg Quick Reference

| Prop | Above-fold | Below-fold | Notes |
|---|---|---|---|
| `loading` | `"eager"` | `"lazy"` | Never lazy on LCP image |
| `fetchpriority` | `"high"` | `"low"` or omit | Above-fold only |
| `format` | `"webp"` | `"webp"` | Always specify |
| `width` + `height` | Required | Required | Prevents CLS — never omit |
| `sizes` | `"100vw"` (full-width) | Responsive sizes string | Enables correct srcset |
| `alt` | Required | Required | Never empty on meaningful images |

### 2.6 Violations to Flag

```vue
<!-- ❌ Raw img tag -->
<img src="/images/hero.jpg" alt="Hero" />

<!-- ❌ NuxtImg without dimensions — causes CLS -->
<NuxtImg src="/images/hero.jpg" alt="Hero" format="webp" />

<!-- ❌ Hero image with lazy loading — kills LCP -->
<NuxtImg src="/images/hero.jpg" loading="lazy" format="webp" width="1440" height="600" />

<!-- ❌ No format specified — browser picks JPEG -->
<NuxtImg src="/images/hero.jpg" width="1440" height="600" loading="eager" />

<!-- ❌ Missing alt on meaningful image -->
<NuxtImg src="/images/team.jpg" width="800" height="600" format="webp" loading="lazy" />
```

---

## Part 3: Nuxt Fonts

Font loading causes two performance problems: render blocking (FCP delay) and layout shift (CLS from font swap). Nuxt Fonts solves both automatically.

### 3.1 Setup

```bash
npx nuxi module add @nuxt/fonts
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/fonts'],

  fonts: {
    // Self-host fonts for privacy and speed (no external DNS lookup)
    defaults: {
      weights: [400, 500, 700],
      styles: ['normal'],
      subsets: ['latin'],
    },
  },
})
```

### 3.2 Usage in CSS

```css
/* assets/styles/tokens.css */
/* Nuxt Fonts intercepts these declarations automatically */
:root {
  --font-display: 'Your Display Font', serif;
  --font-body:    'Your Body Font', sans-serif;
}

body {
  font-family: var(--font-body);
  /* Nuxt Fonts adds font-display: swap + fallback metrics automatically */
}
```

**What Nuxt Fonts does automatically:**
- Self-hosts font files (no Google Fonts DNS lookup on page load)
- Adds `font-display: swap` to prevent render blocking
- Generates fallback font metrics using `fontaine` to minimise CLS during font swap
- Loads only the weights and subsets declared — no full font family downloads

### 3.3 Manual Preload (if not using Nuxt Fonts)

```vue
<!-- In page <head> via useHead() -->
<script setup>
useHead({
  link: [
    {
      rel: 'preload',
      href: '/fonts/your-font.woff2',
      as: 'font',
      type: 'font/woff2',
      crossorigin: '',
    },
  ],
})
</script>
```

---

## Part 4: Lazy Components and Lazy Hydration

### 4.1 Lazy Component Loading

Heavy components not needed on initial render should be lazy-loaded. Nuxt 4 auto-imports `Lazy` prefix components — no import statement needed.

```vue
<!-- ✅ Heavy component loaded only when needed -->
<template>
  <!-- Renders a placeholder until scrolled into view -->
  <LazyTestimonialsCarousel />
  <LazyInteractiveMap />
  <LazyBookingWidget />
</template>
```

For components that need a fallback during load:

```vue
<template>
  <Suspense>
    <template #default>
      <LazyHeavyComponent />
    </template>
    <template #fallback>
      <div class="skeleton" style="height: 400px;" />
    </template>
  </Suspense>
</template>
```

### 4.2 Lazy Hydration (Nuxt 4)

SSG pre-renders HTML, but Vue still hydrates every component on the client. Lazy hydration defers this for off-screen components — reducing INP and Time to Interactive.

```vue
<template>
  <!-- Hydrate only when scrolled into viewport -->
  <LazyTestimonialsSection hydrate-on-visible />

  <!-- Hydrate only on user interaction -->
  <LazyChatWidget hydrate-on-interaction="mouseenter" />

  <!-- Hydrate after browser is idle -->
  <LazyFooterNewsletter hydrate-on-idle />

  <!-- Never hydrate — pure static HTML, no Vue reactivity needed -->
  <LazyStaticInfoBlock hydrate-never />
</template>
```

**Hydration strategy by component type:**

| Component type | Strategy |
|---|---|
| Hero, above-fold content | Eager (default — no directive needed) |
| Below-fold sections | `hydrate-on-visible` |
| Interactive widgets (chat, forms) | `hydrate-on-interaction` |
| Footer, legal content | `hydrate-on-idle` |
| Pure static content (no JS needed) | `hydrate-never` |

---

## Part 5: Third-Party Scripts (Nuxt Scripts)

Third-party scripts (analytics, chat, maps, pixels) are the most common cause of INP failures and LCP delays. Never load them with a raw `<script>` tag.

### 5.1 Setup

```bash
npx nuxi module add @nuxt/scripts
```

### 5.2 Deferred Script Loading

```vue
<script setup>
// ✅ Load analytics only after page is interactive
const { onLoaded } = useScript('https://www.googletagmanager.com/gtm.js?id=GTM-XXXXX', {
  trigger: 'onNuxtReady',   // After hydration completes
})

// ✅ Load chat widget only on user interaction
const { load } = useScript('https://cdn.crisp.chat/js/sdk.js', {
  trigger: 'manual',        // Load only when explicitly called
})

// Trigger on first user click anywhere on page
onMounted(() => {
  document.addEventListener('click', () => load(), { once: true })
})
</script>
```

### 5.3 Script Loading Triggers

| Trigger | When it loads | Use for |
|---|---|---|
| `'onNuxtReady'` | After hydration | Analytics, tag managers |
| `'manual'` | When `load()` is called | Chat widgets, heavy embeds |
| `'immediate'` | As early as possible | Critical scripts only (rare) |
| Event-based | On first interaction | Anything that can wait |

> 🚩 **Flag:** Any `<script src="...">` tag in a component or page — replace with `useScript()`.

---

## Part 6: Bundle Optimisation

### 6.1 What Nuxt 4 Handles Automatically

- Per-page code splitting — each page downloads only its own JS chunk
- Async component chunks — lazy components get their own chunk
- Tree shaking via Vite — unused code removed at build time
- Module deduplication — shared dependencies bundled once

### 6.2 Analyse Bundle Size

```bash
# Install bundle analyser
npx nuxi module add nuxt-bundle-analyzer

# Run build with analysis
npx nuxt build --analyze
```

Red flags in bundle analysis:
- Any single chunk > 250KB (uncompressed)
- Large libraries loaded globally that are only used on one page (moment.js, lodash full build)
- Duplicate packages at different versions

### 6.3 Dynamic Imports for Page-Specific Heavy Libraries

```vue
<script setup>
// ✅ Heavy library loaded only on this page, not globally
const { default: Chart } = await import('chart.js/auto')
</script>
```

### 6.4 Reactive State Hygiene (Reduces Hydration Cost)

```typescript
// ❌ Everything reactive — forces Vue to track changes on static data
const data = ref({
  config: { apiUrl: '/api', version: '1.0' },  // Never changes
  items: [],                                     // Changes
})

// ✅ Only reactive what actually changes
const CONFIG = { apiUrl: '/api', version: '1.0' }  // Plain constant
const items = ref([])                               // Reactive only
```

### 6.5 Event Listener Cleanup

```typescript
// ❌ Memory leak — listener never removed
onMounted(() => {
  window.addEventListener('resize', handleResize)
})

// ✅ Always pair with cleanup
onMounted(() => {
  window.addEventListener('resize', handleResize)
})
onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
```

---

## Part 7: NuxtLink and Prefetch Strategy

`<NuxtLink>` replaces all `<a>` tags for internal navigation. It adds smart prefetching automatically.

```vue
<!-- ✅ Always use NuxtLink for internal links -->
<NuxtLink to="/rooms/deluxe">View Deluxe Room</NuxtLink>

<!-- Disable prefetch for heavy pages not likely to be visited -->
<NuxtLink to="/full-gallery" :prefetch="false">View All Photos</NuxtLink>

<!-- Prefetch only on hover (not on viewport entry — reduces bandwidth) -->
<NuxtLink to="/booking" prefetch-on="interaction">Book Now</NuxtLink>
```

> 🚩 **Flag:** Any `<a href="/internal-page">` — replace with `<NuxtLink to="/internal-page">`.

---

## Additional Resources

- For full baseline config, Core Web Vitals failure matrix, launch checklist, and validation tools, see [reference.md](reference.md).
