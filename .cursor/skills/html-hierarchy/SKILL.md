---
name: html-hierarchy
description: >
  Enforce correct HTML element hierarchy for any web project — covering both heading structure
  (H1–H6 nesting rules) and semantic landmark elements (header, main, nav, section, article,
  aside, footer). Use this skill whenever the user is building, auditing, or fixing HTML page
  structure, heading order, semantic markup, document outline, accessibility tree, or ARIA
  landmark roles. Trigger when the user mentions heading hierarchy, semantic HTML, document
  outline, accessibility structure, screen reader order, H1/H2/H3 nesting, or asks to review
  or correct the structure of any HTML page or component.
---

# HTML Hierarchy — Headings + Semantic Structure

## Overview

HTML hierarchy has two interlocking layers:

1. **Heading hierarchy** — H1 through H6 define the document outline. Used by screen readers, search engines, and AI crawlers to understand content structure.
2. **Semantic landmark structure** — `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>` define page regions. Used by assistive technologies to navigate, and by browsers/search engines to assign content weight.

Both layers must be correct independently, and they must be consistent with each other.

> **All HTML examples in this skill are samples for reference only.** Adapt to your actual page content and component structure.

---

## The Two-Layer Mental Model

```
SEMANTIC LAYER (landmarks)          HEADING LAYER (outline)
─────────────────────────────       ─────────────────────────
<header>                            (no heading required)
<nav>                               (no heading required)
<main>
  <section>                   →     H1 (page title)
    <article>                 →       H2 (article title)
      <section>               →         H3 (subsection)
  <section>                   →     H2 (next major section)
    <aside>                   →       H3 (supporting content)
<footer>                            (no heading required)
```

The heading level should reflect **content depth**, not visual size. Never choose a heading level because of how it looks — control size with CSS.

---

## Part 1: Heading Hierarchy

### Core Rules

| Rule | Detail |
|---|---|
| One `<h1>` per page | The page's single top-level topic. Never use more than one. |
| Never skip levels downward | H1 → H2 → H3 is correct. H1 → H3 (skipping H2) is a violation. |
| Skipping levels upward is allowed | After an H4, going back to H2 is fine — you're closing a subsection. |
| Headings must describe content | Never use headings purely for visual styling. |
| Don't use headings for non-headings | Buttons, captions, labels — not headings. |
| H4–H6 are rarely needed | If you're reaching H4+, consider restructuring the content instead. |

### Correct Structure

```html
<h1>Boutique Hotels in Kerala</h1>           <!-- Page topic -->

  <h2>Backwater Resorts</h2>                 <!-- Major section -->
    <h3>What to Expect</h3>                  <!-- Subsection -->
    <h3>Top Properties</h3>                  <!-- Subsection -->
      <h4>Kumarakom Lake Resort</h4>         <!-- Item (use sparingly) -->

  <h2>Hill Station Retreats</h2>             <!-- Next major section -->
    <h3>Best Time to Visit</h3>
    <h3>Getting There</h3>
```

### Common Violations

```html
<!-- ❌ Multiple H1s -->
<h1>Our Rooms</h1>
<h1>Our Dining</h1>   <!-- Second H1 — wrong -->

<!-- ❌ Skipped level -->
<h1>Hotel Overview</h1>
<h3>Room Types</h3>   <!-- Jumped from H1 to H3 — wrong -->

<!-- ❌ Heading used for styling -->
<h3>Book Now</h3>     <!-- This is a CTA label, not a section heading -->

<!-- ✅ Correct -->
<h1>Hotel Overview</h1>
  <h2>Room Types</h2>
    <h3>Deluxe Rooms</h3>
    <h3>Suite Options</h3>
  <h2>Dining</h2>
```

---

## Part 2: Semantic Landmark Structure

### Element Reference

| Element | Role | Rules |
|---|---|---|
| `<header>` | Site or section header | One per page at root level. Can also appear inside `<article>` / `<section>` as a local header. Never nest a root `<header>` inside `<main>`. |
| `<nav>` | Navigation block | Use for primary, secondary, and footer nav. Add `aria-label` when multiple `<nav>` elements exist on a page. |
| `<main>` | Primary page content | **Exactly one per page.** Never repeated. Must not be nested inside `<article>`, `<aside>`, `<footer>`, `<header>`, or `<nav>`. |
| `<section>` | Thematic grouping | Must have a heading (H2–H6) to be meaningful. Without a heading, use `<div>` instead. |
| `<article>` | Self-contained content | Content that makes sense independently: blog post, review, news item, product card. Can be nested (e.g. comments inside a post). |
| `<aside>` | Supplementary content | Tangentially related to surrounding content. Sidebars, pull quotes, related links. Not for content that's critical to the page. |
| `<footer>` | Site or section footer | One at root level for site footer. Can also appear inside `<article>` / `<section>`. |
| `<div>` | No semantic meaning | Layout and styling only. Use when no semantic element fits. |

### Page-Level Template

```html
<body>
  <header>                              <!-- Site header -->
    <nav aria-label="Primary">          <!-- Main navigation -->
      ...
    </nav>
  </header>

  <main>                                <!-- One per page -->

    <section aria-labelledby="intro">   <!-- Thematic section -->
      <h1 id="intro">Page Title</h1>
      <p>Intro content...</p>
    </section>

    <section aria-labelledby="rooms">
      <h2 id="rooms">Our Rooms</h2>

      <article>                         <!-- Self-contained unit -->
        <h3>Deluxe Room</h3>
        <p>...</p>
      </article>

      <article>
        <h3>Suite</h3>
        <p>...</p>
      </article>
    </section>

    <aside aria-label="Related offers">  <!-- Supplementary -->
      <h2>You May Also Like</h2>
      ...
    </aside>

  </main>

  <footer>                              <!-- Site footer -->
    <nav aria-label="Footer">
      ...
    </nav>
  </footer>

</body>
```

### Common Violations

```html
<!-- ❌ Multiple <main> elements -->
<main>...</main>
<main>...</main>   <!-- Only one allowed -->

<!-- ❌ <section> without a heading -->
<section>
  <p>Some content with no heading</p>   <!-- Use <div> instead -->
</section>

<!-- ❌ <header> nested inside <main> at root level -->
<main>
  <header>Site Header</header>   <!-- Root <header> must be outside <main> -->
</main>

<!-- ❌ Multiple <nav> without aria-label -->
<nav>...</nav>      <!-- Which nav is this? -->
<nav>...</nav>      <!-- Ambiguous to screen readers -->

<!-- ✅ Correct -->
<nav aria-label="Primary navigation">...</nav>
<nav aria-label="Footer navigation">...</nav>

<!-- ❌ <aside> used for critical content -->
<aside>
  <h2>Book Your Stay</h2>   <!-- Primary CTA — not supplementary -->
</aside>
<!-- Use <section> instead -->

<!-- ❌ <div> used where semantic element fits -->
<div class="article-post">   <!-- Use <article> -->
  <div class="article-header">   <!-- Use <header> -->
    <div class="title">...</div>   <!-- Use <h2> or appropriate heading -->
  </div>
</div>
```

---

## Part 3: Headings + Landmarks Together

The heading level must match the semantic depth of its container.

### Rule: H1 lives in `<main>`, not in `<header>`

```html
<!-- ❌ Wrong -->
<header>
  <h1>Site Name</h1>   <!-- This is a logo/brand, not the page topic -->
</header>

<!-- ✅ Correct -->
<header>
  <a href="/" aria-label="Home">
    <img src="logo.png" alt="Site Name" />
  </a>
</header>
<main>
  <h1>Page Topic</h1>   <!-- H1 belongs here -->
</main>
```

### Rule: Each `<section>` introduces a new heading level

```html
<main>
  <h1>Destinations</h1>

  <section>
    <h2>Kerala</h2>         <!-- H2 for section inside main -->

    <section>
      <h3>Backwaters</h3>   <!-- H3 for nested section -->

      <article>
        <h4>Alleppey</h4>   <!-- H4 for article inside nested section -->
      </article>
    </section>
  </section>

  <section>
    <h2>Goa</h2>            <!-- Back to H2 — new sibling section -->
  </section>
</main>
```

### Rule: `<article>` resets the heading context

An `<article>` represents independent content. Its first heading should reflect its own hierarchy, not the parent page's.

```html
<section>
  <h2>Latest Reviews</h2>

  <article>
    <h3>Taj Hotel Review</h3>       <!-- H3 — first heading inside article -->
      <h4>Rooms</h4>
      <h4>Dining</h4>
  </article>

  <article>
    <h3>ITC Grand Review</h3>
      <h4>Location</h4>
  </article>
</section>
```

---

## Part 4: ARIA Labels for Landmark Disambiguation

When the same landmark type appears more than once, `aria-label` or `aria-labelledby` is required so screen readers can distinguish them.

```html
<!-- Multiple <nav> elements -->
<nav aria-label="Primary navigation">...</nav>
<nav aria-label="Breadcrumb">...</nav>
<nav aria-label="Footer navigation">...</nav>

<!-- Multiple <section> elements — use aria-labelledby pointing to heading -->
<section aria-labelledby="rooms-heading">
  <h2 id="rooms-heading">Rooms</h2>
</section>

<section aria-labelledby="dining-heading">
  <h2 id="dining-heading">Dining</h2>
</section>

<!-- <aside> with label -->
<aside aria-label="Related articles">...</aside>
```

---

## Part 5: Audit Checklist

Run through this for any page or component.

### Heading Audit
- [ ] Exactly one `<h1>` on the page
- [ ] H1 describes the page topic (not the site name)
- [ ] No heading levels skipped downward (H1→H3 without H2)
- [ ] Headings used only for actual section titles, not styling or CTAs
- [ ] H4+ used sparingly — if needed at all
- [ ] Heading text is descriptive enough to work as a standalone outline

### Semantic Structure Audit
- [ ] Exactly one `<main>` on the page
- [ ] `<main>` is not nested inside `<header>`, `<footer>`, `<nav>`, or `<aside>`
- [ ] Every `<section>` has a heading
- [ ] `<div>` not used where a semantic element fits
- [ ] Multiple `<nav>` elements each have a distinct `aria-label`
- [ ] Multiple `<section>` / `<aside>` elements each have `aria-labelledby` or `aria-label`
- [ ] `<aside>` only contains supplementary content — nothing critical to the page
- [ ] `<article>` used only for self-contained, independently meaningful content
- [ ] `<header>` and `<footer>` at root level are outside `<main>`

### Combined Check
- [ ] H1 is inside `<main>`, not inside `<header>`
- [ ] Heading levels reflect semantic container depth, not visual size
- [ ] No heading levels jump across semantic boundaries unexpectedly

---

## Part 6: Quick Reference — Element Decision Tree

```
Is this the primary content area of the page?
  → YES: <main>
  → NO: continue

Is this a site-wide or section-level header/footer?
  → header/footer → <header> / <footer>

Is this a navigation block?
  → YES: <nav> (+ aria-label if multiple on page)

Is this content self-contained and independently meaningful?
  → YES: <article>

Is this a thematic group of content with a heading?
  → YES: <section>

Is this supplementary, tangentially related content?
  → YES: <aside>

None of the above — purely for layout/styling?
  → <div>
```

---

## Part 7: Tools for Validation

| Tool | What it checks |
|---|---|
| [W3C Markup Validator](https://validator.w3.org) | HTML validity |
| [WAVE Accessibility Tool](https://wave.webaim.org) | Heading order + landmark structure |
| Chrome DevTools → Accessibility Tree | Live landmark and heading outline |
| [Headingsmap (browser extension)](https://chromewebstore.google.com/detail/headingsmap/flbjommegcjonpdmenkdiocclhjacmbi) | Visual heading hierarchy per page |
| Lighthouse (DevTools → Audits) | Accessibility score including heading issues |
| Screen reader test (NVDA / VoiceOver) | Real-world landmark navigation |
