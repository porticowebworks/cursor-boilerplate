---
name: seo-on-page
description: >
  Implement and audit on-page SEO for any website type. Covers URL structure, title tags,
  meta descriptions, heading architecture per page type, canonical tags, image SEO, internal
  linking signals, structured data references, page speed signals, and crawlability.
  Use this skill whenever building or reviewing any web page for SEO compliance — including
  new page builds, content audits, technical SEO reviews, or pre-launch checklists.
  Trigger when the user mentions SEO, title tags, meta descriptions, canonical tags, URL slugs,
  alt text, page speed, crawlability, indexing, or asks how to optimise a page for search.
---

# On-Page SEO

## Overview

On-page SEO is the set of signals on a single page that tell search engines — and increasingly
AI systems — what the page is about, how authoritative it is, and how it relates to the rest of
the site. It operates at four layers:

1. **URL and crawl signals** — can the page be found and indexed?
2. **Title and meta signals** — what does the page declare itself to be?
3. **Content structure signals** — does the page prove it matches the query?
4. **Technical signals** — does the page load fast and render correctly?

All four layers must be correct. A perfect title tag on a slow, unindexable page does nothing.

> **Schema markup is not covered here.** Refer to your project-specific schema markup skill for
> structured data implementation. Schema is a fifth SEO layer that builds on top of these four.

> **All examples in this skill are samples for reference only.**

---

## Layer 1: URL and Crawl Signals

### 1.1 URL Structure Rules

| Rule | Correct | Wrong |
|---|---|---|
| Lowercase only | `/rooms/deluxe-sea-view` | `/Rooms/DeluxeSeaView` |
| Hyphens not underscores | `/blog/on-page-seo` | `/blog/on_page_seo` |
| No stop words | `/rooms/deluxe` | `/rooms/the-deluxe-room` |
| Descriptive, not ID-based | `/services/web-design` | `/services/page?id=42` |
| Shallow depth (max 3 levels) | `/blog/seo-guide` | `/en/blog/category/seo/posts/guide` |
| No trailing parameters | `/about` | `/about?ref=home&source=nav` |
| Canonical slug matches H1 | `/hotel-booking-software` | `/hbs` (abbreviation) |

### 1.2 robots.txt

Ensure pages you want indexed are not accidentally blocked.

```txt
User-agent: *
Allow: /

Sitemap: https://www.example.com/sitemap.xml
```

Check for accidental blocks:
- `Disallow: /` — blocks everything
- `Disallow: /blog/` — blocks entire blog
- Missing `Sitemap:` declaration

### 1.3 Meta Robots (Page-Level Indexing Control)

```html
<!-- Default — allow indexing and link following -->
<meta name="robots" content="index, follow" />

<!-- Exclude from index (login pages, thank-you pages, admin) -->
<meta name="robots" content="noindex, follow" />

<!-- Exclude from index AND don't follow links on this page -->
<meta name="robots" content="noindex, nofollow" />
```

Pages that should always be `noindex`:
- Login / account pages
- Checkout and order confirmation pages
- Search results pages (`/search?q=...`)
- Duplicate or near-duplicate content
- Staging / preview URLs

### 1.4 Canonical Tag

Every indexable page must declare its canonical URL. Prevents duplicate content penalties from URL parameters, pagination, and session IDs.

```html
<link rel="canonical" href="https://www.example.com/this-page/" />
```

Rules:
- Canonical must be an absolute URL (not relative)
- Self-referencing canonical on every page — including the home page
- Canonical must match the URL you want indexed exactly (including trailing slash if used)
- Paginated pages (`/blog/page/2`) should NOT canonicalise to page 1 — each gets its own canonical
- Hreflang pages must have canonical + hreflang aligned

### 1.5 sitemap.xml

```xml
<url>
  <loc>https://www.example.com/services/web-design/</loc>
  <lastmod>2026-04-15</lastmod>
  <changefreq>monthly</changefreq>
  <priority>0.8</priority>
</url>
```

Rules:
- Only include indexable pages (no `noindex` pages in sitemap)
- No redirect URLs — only canonical final destinations
- `lastmod` must be accurate — stale dates are ignored by crawlers
- Submit to Google Search Console after every significant change

---

## Layer 2: Title and Meta Signals

> **Payload CMS projects:** Meta title, description, and OG image fields on content
> collections are managed via `@payloadcms/plugin-seo` — not added manually per collection.
> The fields defined in this skill map directly to the plugin's injected fields.
> `seo.title` → plugin's `meta.title`, `seo.description` → plugin's `meta.description`.

### 2.1 Title Tag

The single most important on-page SEO element. Used as the clickable headline in search results.

**Formula by page type:**

| Page type | Formula | Example |
|---|---|---|
| Homepage | `Brand Name — Primary Value Proposition` | `Portico Webworks — Hotel Websites That Drive Direct Bookings` |
| Service page | `Primary Keyword — Brand Name` | `Hotel Website Design — Portico Webworks` |
| Location page | `Service + Location — Brand Name` | `Web Design Agency in Kochi — Portico Webworks` |
| Blog post | `Post Title (natural, keyword-first) — Brand Name` | `How to Increase Direct Hotel Bookings — Portico Webworks` |
| Product page | `Product Name — Category — Brand Name` | `Deluxe Sea View Room — Rooms — Harbour View Hotel` |
| Category page | `Category Name — Brand Name` | `Hotel Web Design Services — Portico Webworks` |

**Rules:**
- Length: 50–60 characters (Google truncates beyond ~580px display width)
- Primary keyword near the start — not buried at the end
- Every page has a unique title — no duplicates across the site
- Never stuff keywords: `Hotel Web Design | Hotel Website | Hotel SEO | Hotel Marketing` is spam
- Brand name at the end, separated by `—` or `|`
- Write for the human first — the keyword placement is secondary to readability

```html
<title>Hotel Website Design — Portico Webworks</title>
```

### 2.2 Meta Description

Not a direct ranking signal, but controls click-through rate from search results. Poor meta descriptions cost traffic even at high rankings.

**Rules:**
- Length: 140–160 characters
- Contains the primary keyword naturally
- Written as a direct answer or value proposition — not a description of the page
- Ends with a soft CTA or differentiator
- Every page has a unique meta description
- Never auto-generated from body copy — always hand-written

**Formula:** `[What the page delivers] + [Key differentiator] + [Soft CTA]`

```html
<meta name="description" content="Hotel websites built to convert browsers into direct bookings.
Custom design, fast load times, and booking engine integration. Get a free consultation." />
```

**By page type:**

| Page type | Approach |
|---|---|
| Homepage | Brand promise + primary value proposition |
| Service page | What the service delivers + who it's for |
| Location page | Service + location + local differentiator |
| Blog post | What the reader will learn or gain |
| Product/room page | Specific features + emotional benefit |

### 2.3 Open Graph Tags

Controls how the page appears when shared on social media and increasingly used by AI systems for entity identification.

```html
<meta property="og:title"       content="Hotel Website Design — Portico Webworks" />
<meta property="og:description" content="Same as meta description — factual and specific." />
<meta property="og:type"        content="website" />
<meta property="og:url"         content="https://www.example.com/services/hotel-website-design/" />
<meta property="og:image"       content="https://www.example.com/images/og-hotel-web-design.jpg" />
<meta property="og:image:width"  content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:site_name"   content="Portico Webworks" />
```

OG image rules:
- Dimensions: 1200 × 630px minimum
- Contains page title as text — not just a decorative image
- Unique per key page — not a sitewide default for every page
- Under 1MB file size

---

## Layer 3: Content Structure Signals

### 3.1 Heading Architecture Per Page Type

Refer to the `html-hierarchy` skill for full semantic HTML rules. SEO-specific heading rules:

**Every page has exactly one H1.** It must:
- Match (or closely reflect) the primary keyword
- Match (or closely reflect) the title tag topic
- Appear early in the page — ideally the first visible text in `<main>`
- Not be the site name or logo text (that belongs in `<header>`, not as an H1)

**H2s define the page's subtopics** — they are the secondary keywords. Each H2 should answer a question a user searching for this topic would have.

**H3s support H2s** — they drill into specifics. Use sparingly.

**Page-type heading templates:**

```
Homepage:
  H1: [Primary value proposition or brand positioning]
  H2: [Core service / product 1]
  H2: [Core service / product 2]
  H2: [Social proof / testimonials section]
  H2: [CTA section heading]

Service page:
  H1: [Primary service keyword — e.g. "Hotel Website Design"]
  H2: [What's included / what you get]
  H2: [Who it's for]
  H2: [How it works / process]
  H2: [Results / case studies]
  H2: [FAQ]

Blog post:
  H1: [Post title — keyword-first]
  H2: [Major section 1]
    H3: [Subsection if needed]
  H2: [Major section 2]
  H2: [Summary / conclusion]

Location page:
  H1: [Service + Location — e.g. "Hotel Website Design in Kochi"]
  H2: [Why local relevance matters]
  H2: [Our work in this area]
  H2: [Contact / get started]
```

### 3.2 Keyword Placement Checklist

Primary keyword must appear in:
- [ ] Title tag
- [ ] H1
- [ ] First 100 words of body content
- [ ] At least one H2
- [ ] Meta description
- [ ] URL slug
- [ ] At least one image alt attribute

Secondary keywords should appear:
- [ ] In H2 or H3 headings
- [ ] Naturally within body paragraphs
- [ ] In image alt attributes where relevant

**Rules:**
- Never force keyword placement — if it reads awkwardly, rewrite the surrounding sentence
- Keyword density is not a target — natural language is
- Use synonyms and related terms freely — search engines understand semantic context
- One primary keyword per page — never try to rank for two unrelated queries on the same page

### 3.3 Content Quality Signals

| Signal | What it means | How to implement |
|---|---|---|
| **Word count** | Enough depth to satisfy the query | Match competitor depth — not a fixed number |
| **Content freshness** | `dateModified` reflects real updates | Update `lastmod` in sitemap + article schema when content changes |
| **Unique value** | Not a rewording of existing content | Add original data, examples, or perspective |
| **Answer above the fold** | Lead with the answer, then support | Don't bury the key point three paragraphs in |
| **Readability** | Short paragraphs, clear language | Max 3–4 sentences per paragraph; avoid jargon |
| **Outbound links** | Links to authoritative sources | Link to primary sources — not competitors |

### 3.4 Image SEO

Every image must have:

```html
<!-- Descriptive alt text — describes the image AND includes keyword where natural -->
<img
  src="/images/hotel-website-design-kochi.jpg"
  alt="Custom hotel website design for a boutique property in Kochi, Kerala"
  width="1200"
  height="800"
  loading="lazy"
/>
```

**Alt text rules:**
- Describe what is actually in the image — not keyword-stuffed captions
- Include the primary keyword only if it genuinely describes the image
- Decorative images: `alt=""` (empty, not missing)
- Logo: `alt="[Brand Name] logo"`
- Person: `alt="[Name], [Role] at [Company]"` if relevant
- Never: `alt="keyword1 keyword2 keyword3 image photo"`

**Image file rules:**
- File name: descriptive and hyphenated — `hotel-website-design-kochi.jpg` not `IMG_4823.jpg`
- Format: WebP for photos, SVG for logos/icons, PNG only when transparency required
- Size: compress before upload — photos under 200KB, hero images under 400KB
- Dimensions: declare `width` and `height` attributes — prevents layout shift (CLS)
- `loading="lazy"` on all images below the fold; `loading="eager"` on hero/above-fold images

### 3.5 Internal Linking SEO Signals

Refer to the `internal-linking-policy` skill for full linking rules. SEO-specific requirements:

- Anchor text must be descriptive — `Learn about hotel website design` not `click here` or `read more`
- Link to related pages using keyword-rich anchor text naturally within content
- Every page should link out to at least 2–3 relevant internal pages from body content
- Deep pages (3 clicks from home) benefit most from internal links — prioritise them
- Never use the same anchor text to link to two different pages

```html
<!-- ❌ Generic anchor -->
<a href="/services/hotel-websites">Click here</a>

<!-- ✅ Descriptive anchor -->
<a href="/services/hotel-websites">hotel website design services</a>
```

---

## Layer 4: Technical Signals

### 4.1 Core Web Vitals Targets

| Metric | Target | What it measures |
|---|---|---|
| **LCP** (Largest Contentful Paint) | < 2.5s | How fast the main content loads |
| **CLS** (Cumulative Layout Shift) | < 0.1 | How much the page jumps while loading |
| **INP** (Interaction to Next Paint) | < 200ms | How fast the page responds to clicks |
| **TTFB** (Time to First Byte) | < 200ms | Server response time |
| **FCP** (First Contentful Paint) | < 1.8s | When the first content appears |

### 4.2 Performance Checklist

- [ ] Images in WebP format and compressed
- [ ] `width` and `height` on all `<img>` tags (prevents CLS)
- [ ] Hero image uses `loading="eager"` and `fetchpriority="high"`
- [ ] All below-fold images use `loading="lazy"`
- [ ] CSS and JS minified
- [ ] Unused CSS removed (no full framework loaded for a few utilities)
- [ ] Fonts loaded with `font-display: swap`
- [ ] Third-party scripts (analytics, chat, embeds) deferred or async
- [ ] Server-side rendering or static generation for key pages (not client-side only)
- [ ] HTTPS enforced sitewide

### 4.3 Mobile SEO

```html
<!-- Required in <head> -->
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

Rules:
- All tap targets (buttons, links) minimum 48 × 48px
- No horizontal scrolling on any viewport
- Font size minimum 16px for body text (prevents zoom on mobile)
- Test with Google Search Console Mobile Usability report

### 4.4 Hreflang (Multilingual Sites Only)

If the site serves multiple languages or regions:

```html
<link rel="alternate" hreflang="en"    href="https://www.example.com/page/" />
<link rel="alternate" hreflang="en-IN" href="https://www.example.com/in/page/" />
<link rel="alternate" hreflang="x-default" href="https://www.example.com/page/" />
```

Rules:
- Every language version must reference all other language versions
- `x-default` points to the fallback version
- Hreflang and canonical must not conflict

---

## Page-Type SEO Checklist

### Every page
- [ ] Unique, keyword-first title tag (50–60 chars)
- [ ] Unique meta description (140–160 chars)
- [ ] Self-referencing canonical tag
- [ ] Correct `robots` meta tag (`index, follow` or `noindex`)
- [ ] Open Graph tags complete
- [ ] Exactly one H1 — matches title tag topic
- [ ] H1 appears early in `<main>` — not in `<header>`
- [ ] Primary keyword in H1, first 100 words, at least one H2
- [ ] All images have descriptive alt text
- [ ] All images have `width` and `height` declared
- [ ] Descriptive anchor text on all internal links
- [ ] Page included in sitemap (if indexable)
- [ ] No broken links

### Homepage additionally
- [ ] Brand name in title (usually at end)
- [ ] H1 reflects primary value proposition
- [ ] Links to all major sections/services from body content
- [ ] OG image includes brand name as text

### Blog post additionally
- [ ] `datePublished` and `dateModified` in page metadata
- [ ] Author attributed
- [ ] At least 2 internal links to related content
- [ ] At least 1 outbound link to authoritative source

### Service / product page additionally
- [ ] Primary keyword in URL slug
- [ ] FAQ section with H2 or H3 headings (supports AI citation)
- [ ] At least one CTA above the fold

---

## Validation Tools

| Tool | What it checks |
|---|---|
| [Google Search Console](https://search.google.com/search-console) | Indexing, coverage, Core Web Vitals, mobile usability |
| [Google Rich Results Test](https://search.google.com/test/rich-results) | Schema markup validity |
| [PageSpeed Insights](https://pagespeed.web.dev) | Core Web Vitals + performance |
| [Screaming Frog](https://www.screamingfrog.co.uk/seo-spider/) | Sitewide crawl — titles, metas, canonicals, broken links |
| [Ahrefs Webmaster Tools](https://ahrefs.com/webmaster-tools) | Backlinks, keyword rankings, site audit (free tier) |
| Chrome DevTools → Lighthouse | Full SEO + performance + accessibility audit |
