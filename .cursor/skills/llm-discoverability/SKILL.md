---
name: llm-discoverability
description: >
  Implement page-level technical signals that make any website discoverable, crawlable, and
  citable by AI systems — including ChatGPT, Perplexity, Google AI Overviews, and Claude.
  Use this skill whenever building or auditing a website for AI visibility, configuring robots.txt
  for AI crawlers, implementing llms.txt, choosing between SSR and CSR for AI crawlability,
  optimising HTML signals for AI extraction, or setting up meta tags and Open Graph for AI
  identity. Trigger when the user mentions LLM discoverability, AI crawlers, GEO, AEO, LLMO,
  AI citations, GPTBot, ClaudeBot, PerplexityBot, structured content for AI, or asks why their
  site isn't appearing in AI-generated answers.
---

# LLM Discoverability — Page-Level Technical Implementation

## Overview

AI systems discover and cite websites through two distinct mechanisms:

1. **Training-time crawling** — AI crawlers (GPTBot, ClaudeBot, CCBot) index your site between queries. What they ingest shapes what the model "knows" about you persistently.
2. **Inference-time retrieval** — Real-time search bots (ChatGPT-User, PerplexityBot, Google-Extended) fetch live pages to answer user queries right now.

Both paths require separate configuration. A site can be blocked for one and open for the other. Most LLM visibility failures stem from accidentally blocking the wrong crawler, or serving content that crawlers simply cannot read.

> **Schema markup is not covered here.** It is a critical layer of LLM discoverability. Refer to your project-specific schema markup skill for structured data implementation.

> **All code examples in this skill are samples for reference only.** Adapt to your actual server stack, framework, and domain.

---

## Implementation Stack (in priority order)

| Priority | Layer | What it controls |
|---|---|---|
| 1 | `robots.txt` | Whether AI crawlers can access your site at all |
| 2 | Rendering (SSR vs CSR) | Whether AI crawlers can read your content |
| 3 | HTML content signals | How well AI systems extract and understand your content |
| 4 | Meta tags | How AI systems identify and describe your pages |
| 5 | `llms.txt` | Curated content map for AI systems (forward-looking) |
| 6 | `sitemap.xml` | Crawler discovery of all pages |
| 7 | Page speed + crawl hygiene | Whether crawlers actually complete the crawl |

---

## 1. robots.txt — AI Crawler Access

### Two Categories of AI Crawler (Critical Distinction)

| Type | Purpose | Examples | Blocking impact |
|---|---|---|---|
| **Training crawlers** | Feed content into model weights — no attribution | GPTBot, ClaudeBot, CCBot, Google-Extended | Blocks persistent model knowledge of your site |
| **Search/retrieval crawlers** | Fetch live pages to cite in AI answers | ChatGPT-User, PerplexityBot, Google-Extended | Blocks real-time AI citations |

> `Google-Extended` serves both purposes. Blocking it blocks both AI Overview citations AND Google's AI training. Treat with care.

**Accidentally blocking search crawlers is the most common cause of AI invisibility.** Audit your robots.txt before any other step.

### Recommended robots.txt for AI Visibility

```txt
# ── Traditional search crawlers ──────────────────────
User-agent: Googlebot
Allow: /

User-agent: Bingbot
Allow: /

# ── AI search / retrieval crawlers (cite your content) ──
User-agent: ChatGPT-User
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Google-Extended
Allow: /

# ── AI training crawlers (allow for brand visibility) ──
User-agent: GPTBot
Allow: /
Crawl-delay: 2

User-agent: ClaudeBot
Allow: /
Crawl-delay: 5

User-agent: OAI-SearchBot
Allow: /

# ── Block aggressive / low-value crawlers ────────────
User-agent: Bytespider
Disallow: /

User-agent: CCBot
Disallow: /

# ── Sitemap ───────────────────────────────────────────
Sitemap: https://www.example.com/sitemap.xml
```

> `robots.txt` is advisory only — not all crawlers honour it. Edge-level enforcement (Cloudflare WAF, rate limiting) is needed for hard blocks. For most sites targeting visibility, `Allow: /` across all major AI crawlers is the right default.

### Protecting Specific Paths

```txt
# Allow crawling of public content but protect user data areas
User-agent: GPTBot
Allow: /blog/
Allow: /about/
Allow: /services/
Disallow: /account/
Disallow: /checkout/
Disallow: /admin/
```

### Audit Your Current robots.txt

Check live at: `https://yourdomain.com/robots.txt`

Red flags to fix:
- `User-agent: * Disallow: /` — blocks everything including AI crawlers
- Explicit `Disallow: /` for `GPTBot`, `ClaudeBot`, or `PerplexityBot`
- Missing `Sitemap:` declaration

---

## 2. Rendering — SSR vs CSR

**This is the single biggest hidden cause of AI invisibility.**

Most major AI training crawlers (GPTBot, ClaudeBot) do not execute JavaScript. A React, Vue, Angular, or Svelte app using client-side rendering delivers a near-empty HTML shell to these crawlers. Your content simply does not exist for them.

| Rendering Method | AI Crawlability | Notes |
|---|---|---|
| **Server-Side Rendering (SSR)** | ✅ Full | HTML delivered pre-rendered. All crawlers see full content. |
| **Static Site Generation (SSG)** | ✅ Full | Pre-built HTML. Best for content-heavy pages. |
| **Client-Side Rendering (CSR)** | ⚠️ Partial | PerplexityBot and Google-Extended handle JS. GPTBot/ClaudeBot do not. |
| **CSR with no prerender** | ❌ Invisible | GPTBot and ClaudeBot see an empty div. |

### Rule

**Any page you want AI systems to discover and cite must serve fully rendered HTML on first request.** JavaScript hydration can happen after — but critical content cannot depend on it.

### Framework Guidance

| Framework | AI-Safe Approach |
|---|---|
| Next.js | Use `getServerSideProps` or `generateStaticParams` for key pages |
| Nuxt 3 | Use `useFetch` with SSR mode on (default); avoid `client-only` components for critical content |
| SvelteKit | Use `+page.server.ts` load functions |
| Plain HTML / WordPress | Already SSR by default ✅ |
| React SPA (CRA) | Add a prerender service (Prerender.io) or migrate key pages to Next.js |

### Content Visibility Rules

Even on SSR pages, content hidden behind JavaScript interactions is often missed:

```html
<!-- ❌ AI crawlers may not see content inside collapsed accordions triggered by JS -->
<div class="accordion" data-expanded="false">
  <div class="content">Key policy information here</div>
</div>

<!-- ✅ Critical content should be visible in initial HTML, not JS-dependent -->
<section>
  <h2>Cancellation Policy</h2>
  <p>Key policy information here</p>
</section>
```

---

## 3. HTML Content Signals

AI systems extract meaning from your HTML structure. Clean, semantically correct HTML produces cleaner AI extraction than div-soup.

### Content Hierarchy

Place your most important content as high in the `<body>` as possible. AI crawlers use content position as a relevance signal.

```html
<!-- ✅ Key facts early, before supplementary content -->
<main>
  <h1>Page Topic</h1>
  <p>Clear, specific description of what this page is about.</p>

  <section>
    <h2>Key Facts</h2>
    <!-- Critical information visible in initial HTML -->
  </section>
</main>
```

Refer to the `html-hierarchy` skill for full semantic HTML and heading structure rules.

### Definition Patterns (High AI Extraction Value)

AI systems are optimised to extract direct answers. Use definition-style content structures where possible.

```html
<!-- Definition list pattern — high extraction confidence -->
<dl>
  <dt>Check-in time</dt>
  <dd>2:00 PM — 11:00 PM</dd>

  <dt>Check-out time</dt>
  <dd>By 12:00 PM noon</dd>
</dl>

<!-- Table pattern — strong for comparative data -->
<table>
  <thead>
    <tr><th>Room Type</th><th>Size</th><th>Max Occupancy</th></tr>
  </thead>
  <tbody>
    <tr><td>Deluxe</td><td>32 sqm</td><td>2 guests</td></tr>
  </tbody>
</table>
```

### Summary / TLDR Block

Place a concise summary at the top of long content pages. AI systems pull from summary blocks for citations.

```html
<section aria-label="Summary">
  <p><strong>In brief:</strong> [One or two sentences summarising the page's core answer or topic.]</p>
</section>
```

### Avoid These HTML Patterns for Critical Content

```html
<!-- ❌ Content inside lazy-loaded components may be missed -->
<div data-lazy="true">...</div>

<!-- ❌ Content inside iframes is not crawled -->
<iframe src="/content.html"></iframe>

<!-- ❌ Content only accessible after click/toggle -->
<button onclick="showContent()">Show Details</button>
<div id="content" style="display:none">Critical info here</div>

<!-- ❌ Important text as image — not readable by crawlers -->
<img src="pricing-table.png" alt="pricing" />
```

---

## 4. Meta Tags for AI Identity

Meta tags help AI systems correctly identify, describe, and attribute your pages.

### Essential Meta Tags

```html
<head>
  <!-- Page identity -->
  <title>Specific Page Topic | Site Name</title>
  <meta name="description" content="A specific, factual 150–160 character description of this page's content. Write as if it will be read aloud as an AI answer." />
  <link rel="canonical" href="https://www.example.com/this-page/" />

  <!-- Robots directives — control indexing per page -->
  <meta name="robots" content="index, follow" />

  <!-- To block a specific AI crawler from indexing (not crawling) this page: -->
  <!-- <meta name="robots" content="noindex" /> blocks all -->
  <!-- <meta name="googlebot-extended" content="noindex" /> blocks Google AI only -->

  <!-- Open Graph — used by AI systems for entity identification -->
  <meta property="og:title" content="Specific Page Topic" />
  <meta property="og:description" content="Same as meta description — factual and specific." />
  <meta property="og:type" content="website" />
  <meta property="og:url" content="https://www.example.com/this-page/" />
  <meta property="og:image" content="https://www.example.com/images/page-og.jpg" />
  <meta property="og:site_name" content="Site Name" />

  <!-- Authorship / publication signals -->
  <meta name="author" content="Author Name or Organisation" />
  <meta property="article:published_time" content="2026-01-15T08:00:00+00:00" />
  <meta property="article:modified_time" content="2026-04-20T10:00:00+00:00" />
</head>
```

### Meta Description as an AI Answer

Write `meta description` as if an AI system will read it aloud as the answer to a question. Specific, factual, complete in one sentence.

```html
<!-- ❌ Vague — provides no extractable facts -->
<meta name="description" content="We offer great rooms and amazing service at our hotel." />

<!-- ✅ Specific — extractable as a direct AI answer -->
<meta name="description" content="The Harbour View Hotel is a 42-room boutique property in Kochi, Kerala, with seafront views, rooftop pool, and direct beach access. Rates from ₹5,000 per night." />
```

### Controlling AI Indexing Per Page (robots meta tag)

```html
<!-- Allow all crawlers to index and follow links (default) -->
<meta name="robots" content="index, follow" />

<!-- Prevent all AI + search crawlers from indexing this page -->
<meta name="robots" content="noindex, follow" />

<!-- Block only Google's AI systems (AI Overviews, Gemini) from this page -->
<meta name="googlebot-extended" content="noindex" />

<!-- Block only GPTBot from indexing this page -->
<meta name="GPTBot" content="noindex" />
```

> `robots.txt` controls whether a crawler accesses a page at all. `<meta name="robots">` controls whether an accessible page gets indexed. Both layers are required for full control.

---

## 5. llms.txt — AI Content Map

`llms.txt` is a Markdown-formatted file at your domain root that provides AI systems with a curated map of your site's most important content.

> **Current status (2026):** [Unverified — verify before prioritising] No major AI company (OpenAI, Anthropic, Google) has formally confirmed they read `llms.txt` during inference or training. Server log data from multiple sources shows minimal AI crawler requests to the file. Implement it as a low-effort future-proofing measure — not as a primary discoverability strategy. Prioritise layers 1–4 first.

### File Location

```
https://www.example.com/llms.txt        ← root-level content map (required)
https://www.example.com/llms-full.txt   ← full content dump (optional — use with caution)
```

> Do not use `llms-full.txt` unless you are comfortable exposing your entire content library. Most practitioners recommend the curated map format only.

### llms.txt Format

```markdown
# Site Name

> Brief description of what this site/organisation does and its core expertise.
> One to two sentences. Specific and factual.

## Core Pages
- [Home](https://www.example.com/): Overview of the site
- [About](https://www.example.com/about/): Organisation background and mission
- [Services](https://www.example.com/services/): What we offer

## Key Content
- [Guide Title](https://www.example.com/blog/guide/): Brief description of what this page answers
- [FAQ](https://www.example.com/faq/): Common questions about our products and policies

## Policies
- [Privacy Policy](https://www.example.com/privacy/): Data handling and user rights
- [Terms of Service](https://www.example.com/terms/): Usage conditions
```

### What to Include / Exclude

| Include | Exclude |
|---|---|
| Core product/service pages | Login, account, checkout pages |
| Evergreen content and guides | Thin or duplicate content |
| FAQ and policy pages | Outdated posts or archived content |
| About and contact pages | Internal admin URLs |
| High-value blog content | Session-specific or personalised pages |

---

## 6. sitemap.xml — Crawler Discovery

Ensure your sitemap is current, submitted, and accessible to AI crawlers.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://www.example.com/</loc>
    <lastmod>2026-04-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://www.example.com/about/</loc>
    <lastmod>2026-01-15</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
</urlset>
```

**Checklist:**
- [ ] `Sitemap:` URL declared in `robots.txt`
- [ ] Submitted to Google Search Console
- [ ] All canonical URLs included — no redirects, no `noindex` pages
- [ ] `lastmod` kept current — stale dates reduce crawl priority
- [ ] Image sitemap added if visual content is important

---

## 7. Page Speed + Crawl Hygiene

AI crawlers operate with short timeouts and limited crawl budgets. Slow or broken pages are skipped.

### Speed Targets for AI Crawlability

| Metric | Target |
|---|---|
| Time to First Byte (TTFB) | < 200ms |
| Largest Contentful Paint (LCP) | < 2.5s |
| Page size | < 1MB uncompressed HTML |

### Crawl Hygiene Rules

- No broken internal links (`404`) — crawlers waste budget and may drop the site
- No redirect chains longer than 1 hop — `A → B → C` should be `A → C`
- No `noindex` on pages you want cited
- No login walls, CAPTCHA, or cookie consent blocking body content from crawlers
- Compress images; defer non-critical scripts
- Enable HTTP/2 or HTTP/3 on your server

---

## 8. Implementation Checklist

### Foundation (do first)
- [ ] Audit `robots.txt` — confirm major AI crawlers are not blocked
- [ ] Confirm key pages use SSR or SSG — not client-side only rendering
- [ ] All critical content visible in initial HTML — not JS-dependent

### Identity Signals
- [ ] `<title>` tags specific and descriptive on every page
- [ ] `meta description` written as a direct, factual AI answer
- [ ] `canonical` tags correct on every page
- [ ] Open Graph tags complete on every page
- [ ] `article:published_time` and `article:modified_time` on content pages

### Crawler Infrastructure
- [ ] `sitemap.xml` current and declared in `robots.txt`
- [ ] No broken links or redirect chains
- [ ] TTFB under 200ms on key pages
- [ ] No cookie walls or CAPTCHAs blocking crawlers

### Schema Markup
- [ ] Refer to your project-specific schema markup skill

### Forward-Looking (lower priority)
- [ ] `llms.txt` created at domain root with curated page map
- [ ] Server logs reviewed for AI crawler activity (look for GPTBot, ClaudeBot, PerplexityBot)

---

## 9. Known AI Crawler User-Agent Strings

For server log monitoring and `robots.txt` targeting:

| Crawler | Organisation | Purpose |
|---|---|---|
| `GPTBot` | OpenAI | Training |
| `ChatGPT-User` | OpenAI | Real-time retrieval |
| `OAI-SearchBot` | OpenAI | Search index |
| `ClaudeBot` | Anthropic | Training |
| `Google-Extended` | Google | Training + AI Overviews |
| `PerplexityBot` | Perplexity | Real-time retrieval |
| `Bytespider` | ByteDance/TikTok | Training (aggressive) |
| `CCBot` | Common Crawl | Open training datasets |
| `Meta-ExternalAgent` | Meta | Training |
| `Applebot-Extended` | Apple | Training |
