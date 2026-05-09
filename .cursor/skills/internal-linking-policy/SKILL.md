---
name: internal-linking-policy
description: >
  Enforce internal linking policy while a developer builds pages in Cursor. Flags orphaned pages,
  2-click rule violations, and missing inbound links in real-time as new pages are written.
  Use this skill whenever a developer is building or reviewing a web page and needs to ensure
  it complies with internal linking rules — no orphaned pages, every page reachable within 2
  clicks from home. Trigger when the user is building a new page, adding links to a page, asking
  Claude to review page structure, or mentions internal linking, site navigation, or page
  reachability. Header and footer links are explicitly excluded from internal linking policy.
---

# Internal Linking Policy

## The Two Rules

| Rule | Definition |
|---|---|
| **No orphaned pages** | Every page must have at least one inbound link from another page (excluding header and footer) |
| **2-click rule** | Every page must be reachable within 2 clicks from the home page via body content links only |

> **Scope:** Internal linking policy covers **body content links only** — `<main>`, `<section>`, `<article>`, `<aside>`. Links inside `<header>` and `<footer>` are navigation infrastructure and are explicitly excluded from this policy. Do not count them when auditing or flagging.

---

## What Claude Has Access To

Claude operates with **the current file only**. It cannot see other pages in the site. This constrains what it can verify versus what it must flag for the developer to resolve.

| Check | What Claude can do |
|---|---|
| Outbound links from current page | ✅ Read and verify directly |
| Whether current page links to others correctly | ✅ Verify against declared site map |
| Inbound links to current page | ⚠️ Cannot verify — must flag and ask developer to confirm |
| 2-click depth of current page | ⚠️ Cannot verify — must flag and instruct developer to confirm |
| Header/footer link exclusion | ✅ Apply automatically |

---

## Session Setup (Required)

At the start of any session where internal linking is being reviewed, Claude must request the site map before proceeding. Without it, Claude cannot check the 2-click rule.

**Claude should say:**

> "To enforce the 2-click rule, I need a minimal site map. Please provide:
> 1. A list of all existing pages (page name + URL slug)
> 2. For each page, which other pages link to it from body content (not header/footer)
>
> Format example:
> ```
> home (/)
>   → about (/about)
>   → rooms (/rooms)
>     → deluxe-room (/rooms/deluxe)
>     → suite (/rooms/suite)
>   → blog (/blog)
>     → post-1 (/blog/post-1)
> ```
> If you don't have this yet, I'll flag linking gaps as I review each page and leave resolution to you."

If the developer does not provide a site map, proceed with file-level review and flag all inbound link gaps explicitly.

---

## How to Review a Page

When a developer shares a page file, run through these checks in order.

### Step 1 — Identify the page being built

Determine the page's URL slug from the file name, route definition, or developer declaration. If unclear, ask: "What is the URL slug for this page?"

### Step 2 — Exclude header and footer links

Strip all links found inside `<header>`, `<footer>`, `<nav>` (when nav is inside header/footer), and any element with a class/role indicating site-wide navigation. Do not include these in any link count or reachability check.

```html
<!-- These links are EXCLUDED from policy -->
<header>
  <nav>
    <a href="/rooms">Rooms</a>       <!-- excluded -->
    <a href="/contact">Contact</a>   <!-- excluded -->
  </nav>
</header>

<footer>
  <a href="/privacy">Privacy</a>     <!-- excluded -->
  <a href="/sitemap">Sitemap</a>     <!-- excluded -->
</footer>

<!-- These links ARE subject to policy -->
<main>
  <section>
    <a href="/rooms/deluxe">View Deluxe Room</a>   <!-- counted -->
  </section>
</main>
```

### Step 3 — Audit outbound links from this page

List every internal link found in body content. Check:
- Does each linked page exist in the declared site map?
- Is the link in body content (not header/footer)?
- Is the `href` a relative or absolute internal URL (not external)?

Flag any link pointing to a page not in the site map:
> ⚠️ **Undeclared page linked:** `/rooms/new-room` is linked from this page but does not appear in the site map. Add it to the site map or correct the link.

### Step 4 — Check inbound links to this page

Claude cannot see other files. Apply this rule:

**If no site map was provided:**
> 🚩 **Orphan risk:** No inbound links to `[current page slug]` are visible in this file. Ensure at least one other page links to this page from its body content. Header and footer links do not count.

**If a site map was provided and this page has no inbound body links declared:**
> 🚩 **Orphaned page:** `[slug]` has no inbound body content links in the site map. This page cannot be reached without header/footer navigation. Add a link to it from at least one page that is itself reachable from home.

**If a site map was provided and this page has inbound links:**
> ✅ Inbound links confirmed: `[slug]` is linked from `[parent page]`.

### Step 5 — Check 2-click rule

**If no site map was provided:**
> 🚩 **2-click rule unverifiable:** No site map provided. Confirm manually that `[slug]` is reachable within 2 body-content clicks from the home page.

**If a site map was provided:**

Trace the shortest path from home to this page using body content links only.

- **0 clicks** = this is the home page ✅
- **1 click** = home page body content links directly to this page ✅
- **2 clicks** = a page linked from home body content links to this page ✅
- **3+ clicks** = violation 🚩

> 🚩 **2-click rule violation:** `[slug]` is only reachable in [N] clicks from home via body content links. Reachability path: home → `[page-a]` → `[page-b]` → `[slug]`. Add a link to `[slug]` from either home or a page directly linked from home.

---

## Flag Format

Use consistent, scannable flags so developers can action them quickly.

| Flag | Meaning |
|---|---|
| 🚩 **Orphaned page** | Page has no inbound body content links |
| 🚩 **2-click rule violation** | Page is more than 2 body-content clicks from home |
| 🚩 **2-click rule unverifiable** | No site map — developer must confirm manually |
| ⚠️ **Undeclared page linked** | Outbound link points to a page not in the site map |
| ✅ **Compliant** | Page passes both rules |

### Inline flag format (for Cursor inline comments)

```
// 🚩 INTERNAL LINKING: [page-slug] has no inbound body content links.
//    Ensure at least one page reachable from home links to this page
//    from body content. Header/footer links do not count.

// 🚩 INTERNAL LINKING: 2-click rule unverifiable — no site map provided.
//    Confirm /rooms/new-room is reachable within 2 body clicks from home.

// ⚠️ INTERNAL LINKING: /rooms/new-room is linked here but not in site map.
//    Add to site map or correct the href.
```

---

## Inline Suggestion Format

When Claude flags a violation, it must always suggest a fix — never just flag.

### Orphaned page — suggestion

> 🚩 **Orphaned page:** `/blog/post-1` has no inbound body content links in the site map.
>
> **Suggested fix:** Add a link to this post from the `/blog` listing page body content, e.g.:
> ```html
> <a href="/blog/post-1">Read: [Post Title]</a>
> ```
> The `/blog` page is already 1 click from home, so this satisfies the 2-click rule as well.

### 2-click violation — suggestion

> 🚩 **2-click rule violation:** `/rooms/deluxe/gallery` is 3 clicks from home (home → /rooms → /rooms/deluxe → /rooms/deluxe/gallery).
>
> **Suggested fix (choose one):**
> 1. Link to `/rooms/deluxe/gallery` directly from `/rooms` body content (reduces to 2 clicks)
> 2. Link to it from the home page body content (reduces to 1 click — only if contextually appropriate)

---

## What Is NOT an Internal Link (Exclusion List)

Never count the following as internal body content links:

- Any `<a>` inside `<header>`
- Any `<a>` inside `<footer>`
- Any `<a>` inside `<nav>` that is a child of `<header>` or `<footer>`
- Any `<a>` with `rel="nofollow"`
- Any `<a>` pointing to an external domain
- Any `<a>` pointing to an anchor on the same page (`href="#section"`)
- Any `<a>` inside a cookie banner, modal overlay, or notification bar
- Canonical tags, hreflang tags, or any link in `<head>`
- Sitemap XML links

---

## Edge Cases

| Situation | Handling |
|---|---|
| Home page itself | Exempt from inbound link rule. Always 0 clicks from home. |
| 404 / error pages | Exempt from both rules. |
| `noindex` pages (privacy policy, terms) | Flag as low-priority — note they are exempt if intentionally excluded from crawl |
| Paginated pages (`/blog/page/2`) | The root (`/blog`) must satisfy the rules. Paginated variants are exempt if the root is compliant. |
| Redirect pages | Treat the destination URL as the canonical page for link counting. |
| Pages only linked from search results | These count as orphaned — search is not a body content link. |
| Same page linked multiple times | Count as one inbound link regardless of how many times it appears. |

---

## Quick Reference Card

```
ON EVERY PAGE REVIEW:

1. Strip header/footer links — don't count them
2. List all body content outbound links
3. Check each outbound href exists in site map
4. Check inbound links to THIS page (from site map or flag)
5. Verify 2-click depth (from site map or flag)
6. Flag violations with inline suggestion
7. Mark compliant pages explicitly

NEVER:
- Count header/footer links toward policy compliance
- Flag without suggesting a fix
- Leave 2-click status silent if site map is absent
```
