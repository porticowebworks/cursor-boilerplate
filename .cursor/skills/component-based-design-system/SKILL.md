---
name: component-based-design-system
description: >
  Enforce component-based architecture while building websites in Cursor. Ensures UI is broken
  into reusable, prop-driven components from the start — never built as monolithic pages.
  Covers domain understanding, architecture pattern selection, type-safe props, presentational
  vs container separation, composables, shared state via provide/inject, and CSS token systems.
  Use this skill whenever a developer is building a website, UI, landing page, or any frontend
  interface and needs guidance on component decomposition, prop/variable design, CSS variable
  systems, global infrastructure setup, Nuxt 3 / Vue architecture, composables, or reusability
  decisions. Trigger when the user mentions components, design system, reusable UI, frontend
  architecture, scalable website structure, CSS variables, Tailwind config, Vue components,
  Nuxt, composables, Zod validation, or asks how to structure a page or section.
  Only a component-based architecture can build a scalable website.
---

# Component-Based Design System

## The Core Principle

Never build pages. Build components, then assemble pages from them.

A page built as a monolith cannot scale. Every new page starts from scratch. Every design change requires hunting across files. Every inconsistency compounds.

A component-based site is a Lego system. Components are designed once, parameterised to accept variable content, and reused everywhere. Pages become assembly instructions, not constructions.

> **All code examples in this skill are samples for reference only.** Adapt naming conventions, folder structure, and implementation patterns to your actual project.

---

## Step Zero: Understand Domain Before Designing

**Never open a code editor before completing Step Zero.**

Answer these questions first:

1. **What is the problem domain?** What does this page/feature actually do for the user?
2. **What are the main UI concepts and interactions?** Buttons, cards, forms, modals, lists?
3. **What state needs to be managed?** Local component state, shared across siblings, or global?
4. **What are the user flows?** What does the user do, in what order?
5. **How does this fit existing architecture?** Is there an existing component to extend, or is this new?

Only after answering these does pattern identification begin.

### Pattern Identification

Analyse the design (mockup, reference site, or brief) for repeating patterns:

- What elements appear more than once? → Component candidate
- What elements share the same structure but different content? → Single component with props
- What elements appear on multiple pages? → Global component
- What logic repeats across components? → Composable

**Common reusable patterns to identify:**

| Pattern | Component name | Variable inputs |
|---|---|---|
| Image + heading + text + CTA | `FeatureCard` | image, heading, body, ctaLabel, ctaUrl |
| Icon + heading + description | `BentoCard` | icon, heading, description, variant |
| Avatar + quote + name + role | `TestimonialCard` | avatar, quote, name, role |
| Label + heading + subheading | `SectionHeader` | label, heading, subheading, alignment |
| Two columns: text + image | `SplitSection` | heading, body, image, imagePosition (left/right) |
| Navigation bar | `SiteHeader` | logo, navLinks[], ctaLabel, ctaUrl |
| Page footer | `SiteFooter` | logo, columns[], legalText |

### The Reusability Test

Before building any element, ask:

1. **Does this pattern appear elsewhere on this page?** → Component
2. **Could this pattern appear on another page of this site?** → Component
3. **Does this share structure with something that exists but needs different content?** → Extend existing component with a new prop, not a new component
4. **Does logic repeat across multiple components?** → Composable
5. **Is this truly unique to one specific context with no reuse potential?** → Consider a component anyway if complex enough to warrant encapsulation

> **Rule:** When in doubt, make it a component. The cost of over-componentising is low. The cost of refactoring a monolith later is high.

---

## Part 1: Architecture Pattern

### 1.1 Feature-Based vs Layer-Based Structure

**Default: always use feature-based architecture.** Group files by feature, not by technical type.

```
❌ Layer-based (do not use)          ✅ Feature-based (recommended)
─────────────────────────────        ──────────────────────────────
components/                          features/
  LoginForm.vue                        auth/
  RoomCard.vue                           components/
composables/                               LoginForm.vue
  useAuth.ts                             composables/
  useRooms.ts                              useAuth.ts
types/                                   types.ts
  auth.ts                                api.ts
  rooms.ts                               index.ts
```

**Decision flow when starting a feature:**

```
Scan existing codebase structure
        ↓
Feature-based already? → Continue → src/features/[new-feature]/
        ↓
Layer-based? → Propose migration doc → Implement new feature as first feature slice
        ↓
Mixed (mid-migration)? → Check for migration doc → Continue feature-based for new work
```

When migration is needed, create `docs/architecture/feature-based-migration.md`:

```markdown
# Feature-Based Architecture Migration Plan
## Current State: layer-based / mixed
## Target: feature-based in src/features/[feature]/
## Strategy: new features go feature-based; migrate existing incrementally
## Progress:
- [x] auth (this PR)
- [ ] rooms
- [ ] booking
```

### 1.2 Folder Structure

```
src/
├── features/                    ← Feature-based grouping (recommended)
│   ├── auth/
│   │   ├── components/          ← Feature-specific components
│   │   ├── composables/         ← Feature-specific composables
│   │   ├── types.ts
│   │   ├── api.ts
│   │   └── index.ts             ← Public exports only
│   └── rooms/
│       ├── components/
│       ├── composables/
│       ├── types.ts
│       └── index.ts
├── components/                  ← Shared/global components only
│   ├── global/                  ← Site-wide: AppHeader, AppFooter, AppNav
│   ├── layout/                  ← Page structure: AppSection, AppContainer
│   ├── ui/                      ← Atoms: AppButton, AppBadge, AppIcon
│   └── forms/                   ← Shared form elements
├── composables/                 ← Shared composables (used by 2+ features)
├── assets/styles/
│   ├── reset.css
│   ├── tokens.css
│   └── utilities.css
└── pages/                       ← Assembly only — no component logic here
```

> In Nuxt 3, `components/` and `composables/` at the root are auto-imported. Feature-specific components under `features/` must be explicitly imported.

---

## Part 2: Global Infrastructure

Set up the global environment before building any component. Components must reference global variables — never hard-code values.

### 2.1 CSS Reset

```css
/* assets/styles/reset.css */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html { -webkit-text-size-adjust: 100%; }

img, video, svg {
  display: block;
  max-width: 100%;
}

input, button, textarea, select { font: inherit; }
```

### 2.2 CSS Custom Properties (Design Tokens)

Define all design decisions as variables. **Components never use raw values** — always reference a token.

```css
/* assets/styles/tokens.css */
:root {

  /* ── Colours ─────────────────────────────────────── */
  --color-primary:        #1a1a2e;
  --color-primary-hover:  #16213e;
  --color-accent:         #e94560;
  --color-accent-hover:   #c73652;
  --color-surface:        #ffffff;
  --color-surface-alt:    #f5f5f7;
  --color-border:         #e2e2e2;
  --color-text-primary:   #1a1a1a;
  --color-text-secondary: #6b6b6b;
  --color-text-inverse:   #ffffff;

  /* ── Typography ──────────────────────────────────── */
  --font-display:   'Your Display Font', serif;
  --font-body:      'Your Body Font', sans-serif;

  --text-xs:   0.75rem;
  --text-sm:   0.875rem;
  --text-base: 1rem;
  --text-lg:   1.125rem;
  --text-xl:   1.25rem;
  --text-2xl:  1.5rem;
  --text-3xl:  1.875rem;
  --text-4xl:  2.25rem;
  --text-5xl:  3rem;

  --font-weight-regular: 400;
  --font-weight-medium:  500;
  --font-weight-bold:    700;

  --line-height-tight:  1.2;
  --line-height-normal: 1.5;
  --line-height-loose:  1.8;

  /* ── Spacing ─────────────────────────────────────── */
  --space-1:  0.25rem;
  --space-2:  0.5rem;
  --space-3:  0.75rem;
  --space-4:  1rem;
  --space-6:  1.5rem;
  --space-8:  2rem;
  --space-10: 2.5rem;
  --space-12: 3rem;
  --space-16: 4rem;
  --space-20: 5rem;
  --space-24: 6rem;

  /* ── Layout ──────────────────────────────────────── */
  --container-sm:  640px;
  --container-md:  768px;
  --container-lg:  1024px;
  --container-xl:  1280px;
  --container-2xl: 1536px;

  /* ── Borders & Radius ────────────────────────────── */
  --radius-sm:    4px;
  --radius-md:    8px;
  --radius-lg:    16px;
  --radius-full:  9999px;
  --border-width: 1px;

  /* ── Shadows ─────────────────────────────────────── */
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.08);
  --shadow-md: 0 4px 12px rgba(0,0,0,0.10);
  --shadow-lg: 0 8px 30px rgba(0,0,0,0.12);

  /* ── Transitions ─────────────────────────────────── */
  --transition-fast:   150ms ease;
  --transition-normal: 250ms ease;
  --transition-slow:   400ms ease;

  /* ── Z-index ─────────────────────────────────────── */
  --z-base:    0;
  --z-raised:  10;
  --z-overlay: 100;
  --z-modal:   200;
  --z-toast:   300;
}
```

### 2.3 Utility Classes

```css
/* assets/styles/utilities.css */

.container {
  width: 100%;
  max-width: var(--container-xl);
  margin-inline: auto;
  padding-inline: var(--space-6);
}

.flex         { display: flex; }
.flex-col     { display: flex; flex-direction: column; }
.flex-center  { display: flex; align-items: center; justify-content: center; }
.flex-between { display: flex; align-items: center; justify-content: space-between; }
.flex-wrap    { flex-wrap: wrap; }

.grid-2 { display: grid; grid-template-columns: repeat(2, 1fr); gap: var(--space-6); }
.grid-3 { display: grid; grid-template-columns: repeat(3, 1fr); gap: var(--space-6); }
.grid-4 { display: grid; grid-template-columns: repeat(4, 1fr); gap: var(--space-6); }

.text-left   { text-align: left; }
.text-center { text-align: center; }
.text-right  { text-align: right; }

.mt-auto { margin-top: auto; }
.gap-4   { gap: var(--space-4); }
.gap-6   { gap: var(--space-6); }
.gap-8   { gap: var(--space-8); }

.sr-only {
  position: absolute; width: 1px; height: 1px;
  padding: 0; margin: -1px; overflow: hidden;
  clip: rect(0,0,0,0); white-space: nowrap; border: 0;
}
```

---

## Part 3: Type-Safe Props (Preventing Primitive Obsession)

Raw primitives (`string`, `number`, `boolean`) passed as props carry no validation. A prop typed as `string` accepts anything — an empty string, an invalid email, a malformed URL. Use validated types to catch errors at the boundary.

### 3.1 Zod Schemas (for runtime validation — forms, API responses)

```typescript
// features/auth/types.ts
import { z } from 'zod'

export const EmailSchema = z.string().email().min(1)
export const UserIdSchema = z.string().uuid()
export const SlugSchema   = z.string().min(1).regex(/^[a-z0-9-]+$/)

export type Email  = z.infer<typeof EmailSchema>
export type UserId = z.infer<typeof UserIdSchema>
export type Slug   = z.infer<typeof SlugSchema>

// Use in composables or server routes
export function parseEmail(value: unknown): Email {
  return EmailSchema.parse(value) // throws ZodError on invalid
}
```

### 3.2 Branded Types (for compile-time safety without runtime overhead)

```typescript
// types/branded.ts
declare const __brand: unique symbol
type Brand<T, B> = T & { [__brand]: B }

export type RoomId    = Brand<string, 'RoomId'>
export type Price     = Brand<number, 'Price'>
export type ImagePath = Brand<string, 'ImagePath'>

export function createRoomId(id: string): RoomId {
  if (!id) throw new Error('RoomId cannot be empty')
  return id as RoomId
}

export function createPrice(value: number): Price {
  if (value < 0) throw new Error('Price cannot be negative')
  return value as Price
}
```

### 3.3 When to Use Which

| Situation | Use |
|---|---|
| Form field validation | Zod schema |
| API response parsing | Zod schema |
| Internal type safety between components | Branded type |
| Props that are IDs, slugs, or domain values | Branded type |
| Props that are plain UI configuration | Standard TypeScript interface |

### 3.4 Typed Props in Vue 3 / Nuxt 3

```vue
<script setup lang="ts">
import type { ImagePath } from '~/types/branded'

interface Props {
  heading:      string
  body:         string
  icon?:        string
  image?:       ImagePath                              // branded — not just any string
  variant?:     'default' | 'highlight' | 'dark'      // string enum
  ctaLabel?:    string
  ctaUrl?:      string
  isFullWidth?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  variant:     'default',
  isFullWidth: false,
})
</script>
```

---

## Part 4: Component Types

Every component belongs to one of two categories. **Never mix them.**

### 4.1 Presentational Components (UI Only)

- No state management
- No API calls or side effects
- Props-driven — all data comes from outside
- Fully reusable across features
- Easiest to test

```vue
<!-- components/ui/FeatureCard.vue — PRESENTATIONAL -->
<script setup lang="ts">
interface Props {
  icon?:     string
  heading:   string
  body:      string
  variant?:  'default' | 'highlight' | 'dark'
  ctaLabel?: string
  ctaUrl?:   string
}
withDefaults(defineProps<Props>(), { variant: 'default' })
</script>

<template>
  <article :class="['feature-card', `feature-card--${variant}`]">
    <div v-if="icon" class="feature-card__icon">{{ icon }}</div>
    <h3 class="feature-card__heading">{{ heading }}</h3>
    <p class="feature-card__body">{{ body }}</p>
    <NuxtLink v-if="ctaLabel && ctaUrl" :to="ctaUrl" class="feature-card__cta">
      {{ ctaLabel }}
    </NuxtLink>
  </article>
</template>

<style scoped>
.feature-card {
  padding: var(--space-8);
  border-radius: var(--radius-lg);
  border: var(--border-width) solid var(--color-border);
  background: var(--color-surface);
  transition: box-shadow var(--transition-normal);
}
.feature-card:hover      { box-shadow: var(--shadow-md); }
.feature-card--highlight { background: var(--color-primary); color: var(--color-text-inverse); border-color: transparent; }
.feature-card--dark      { background: var(--color-surface-alt); }
.feature-card__heading   { font-size: var(--text-xl); font-weight: var(--font-weight-bold); margin-block: var(--space-3); }
.feature-card__body      { font-size: var(--text-base); color: var(--color-text-secondary); line-height: var(--line-height-normal); }
</style>
```

### 4.2 Container Components (Logic + State)

- Manages state
- Calls composables
- Handles data fetching and side effects
- Passes data to presentational components as props
- Contains no visual styling of its own

```vue
<!-- features/rooms/components/RoomListContainer.vue — CONTAINER -->
<script setup lang="ts">
import { useRooms } from '../composables/useRooms'
import RoomCard from '~/components/ui/RoomCard.vue'

const { rooms, isLoading, error } = useRooms()
</script>

<template>
  <AppLoadingState v-if="isLoading" />
  <AppErrorState v-else-if="error" :message="error.message" />
  <div v-else class="grid-3">
    <RoomCard
      v-for="room in rooms"
      :key="room.id"
      :heading="room.name"
      :body="room.description"
      :image="room.image"
      cta-label="View Room"
      :cta-url="`/rooms/${room.slug}`"
    />
  </div>
</template>
```

---

## Part 5: Composables (Reusable Logic)

Extract any logic that could be reused across two or more components into a composable. **Single responsibility: one composable does one thing.**

### 5.1 Data Fetching Composable

```typescript
// features/rooms/composables/useRooms.ts
export function useRooms() {
  const rooms      = ref<Room[]>([])
  const isLoading  = ref(false)
  const error      = ref<Error | null>(null)

  const fetchRooms = async () => {
    isLoading.value = true
    error.value = null
    try {
      rooms.value = await $fetch('/api/rooms')
    } catch (err) {
      error.value = err as Error
    } finally {
      isLoading.value = false
    }
  }

  onMounted(fetchRooms)
  return { rooms, isLoading, error, fetchRooms }
}
```

### 5.2 Form State Composable

```typescript
// composables/useFormState.ts — shared across features
export function useFormState<T extends Record<string, unknown>>(initialValues: T) {
  const values      = reactive({ ...initialValues })
  const errors      = reactive<Partial<Record<keyof T, string>>>({})
  const isSubmitting = ref(false)

  function setValue<K extends keyof T>(key: K, value: T[K]) {
    values[key] = value
    delete errors[key]
  }

  function setError(key: keyof T, message: string) {
    errors[key] = message
  }

  function reset() {
    Object.assign(values, initialValues)
    Object.keys(errors).forEach(k => delete errors[k as keyof T])
  }

  return { values, errors, isSubmitting, setValue, setError, reset }
}
```

### 5.3 Composable Rules

- File name: `use` prefix — `useRooms.ts`, `useFormState.ts`
- Returns a plain object — never a class
- Single responsibility — one concern per composable
- Feature-specific → `features/[feature]/composables/`
- Shared (used by 2+ features) → root `composables/`
- Nuxt 3 auto-imports from root `composables/` — no import statement needed

---

## Part 6: Shared State (provide / inject)

Use `provide` / `inject` when state needs to be shared across 3 or more component levels.

**Rule:** Do not use for state that only passes between parent and immediate child — use props for that.

```typescript
// features/auth/composables/useAuth.ts
import type { InjectionKey, Ref, ComputedRef } from 'vue'
import type { Email } from '../types'

interface AuthState {
  user:            Ref<User | null>
  isAuthenticated: ComputedRef<boolean>
  login:           (email: Email, password: string) => Promise<void>
  logout:          () => Promise<void>
}

const AUTH_KEY = Symbol('auth') as InjectionKey<AuthState>

export function provideAuth() {
  const user            = ref<User | null>(null)
  const isAuthenticated = computed(() => !!user.value)

  async function login(email: Email, password: string) {
    user.value = await $fetch('/api/auth/login', { method: 'POST', body: { email, password } })
  }

  async function logout() {
    await $fetch('/api/auth/logout', { method: 'POST' })
    user.value = null
  }

  provide(AUTH_KEY, { user, isAuthenticated, login, logout })
}

export function useAuth() {
  const auth = inject(AUTH_KEY)
  if (!auth) throw new Error('useAuth() must be called within a component that calls provideAuth()')
  return auth
}
```

```vue
<!-- layouts/default.vue — provide at layout level -->
<script setup lang="ts">
import { provideAuth } from '~/features/auth/composables/useAuth'
provideAuth()
</script>
```

**When to use provide/inject:**
- Auth/session state across the entire app
- Theme or user preferences
- Cart or booking state
- Any state consumed 3+ levels deep

**When NOT to use:**
- State local to one component → `ref` / `reactive`
- State shared between siblings → lift to parent, pass as props
- State only needed by immediate child → props

---

## Part 7: Tailwind CSS Integration

When using Tailwind, map your design tokens into the config so Tailwind classes reference the same values.

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary:       '#1a1a2e',
        accent:        '#e94560',
        surface:       '#ffffff',
        'surface-alt': '#f5f5f7',
      },
      fontFamily: {
        display: ['Your Display Font', 'serif'],
        body:    ['Your Body Font', 'sans-serif'],
      },
    },
  },
}
```

> Even with Tailwind, every component must live in its own file. Copy-pasting Tailwind markup across pages defeats the purpose entirely. The component is still the unit — Tailwind is just how it gets styled.

---

## Part 8: Enforcement Rules for Cursor

When reviewing or writing code, Claude checks for these violations and flags them inline.

### 🚩 Violations to Flag

**1. Duplicate markup**
```html
<!-- ❌ Same structure copy-pasted with different content -->
<div class="card"><img src="pool.jpg" /><h3>Rooftop Pool</h3></div>
<div class="card"><img src="gym.jpg" /><h3>Fitness Centre</h3></div>
```
> 🚩 **Component opportunity:** Extract to a `Card` component with `image` and `heading` props.

**2. Hard-coded values in styles**
```css
/* ❌ */ .card { padding: 32px; color: #1a1a1a; border-radius: 8px; }
```
> 🚩 **Token violation:** Use `var(--space-8)`, `var(--color-text-primary)`, `var(--radius-md)`.

**3. Component logic in page files**
```vue
<!-- ❌ pages/rooms.vue with scoped component styles and markup -->
<style scoped>.room-card { padding: 24px; }</style>
<div class="room-card">...</div>
```
> 🚩 **Architecture violation:** Move to `components/ui/RoomCard.vue`. Pages contain assembly only.

**4. Variant controlled from outside**
```html
<!-- ❌ --> <FeatureCard style="background: navy; color: white;" />
```
> 🚩 **Encapsulation violation:** Add `variant="dark"` prop. Handle styling internally.

**5. Hard-coded content inside component**
```vue
<!-- ❌ --> <template><h1>Welcome to Our Hotel</h1></template>
```
> 🚩 **Prop gap:** `heading` must be a prop. Hard-coded content makes this single-use.

**6. Primitive obsession on domain values**
```vue
<!-- ❌ --> defineProps({ email: String, roomId: String })
```
> 🚩 **Type safety gap:** `email` → `EmailSchema` (Zod). `roomId` → branded `RoomId` type.

**7. Logic mixed into presentational component**
```vue
<!-- ❌ Presentational component making its own API call -->
<script setup>
const { data: rooms } = await useFetch('/api/rooms')
</script>
```
> 🚩 **Separation violation:** Move `useFetch` to a container component or composable. Presentational components receive data only via props.

**8. Layer-based structure for new features**
```
❌ components/RoomCard.vue + composables/useRooms.ts + types/room.ts
```
> 🚩 **Architecture pattern:** Create `features/rooms/` with components, composables, and types co-located.

---

## Part 9: Design Plan Output Format

Before implementing any feature with state, multiple components, or shared logic — produce a design plan. Never jump straight to code.

```
🎨 DESIGN PLAN — [Feature Name]

Domain:
  What this feature does and who uses it.

Core Types:
  ✅ RoomId (branded type) — prevents invalid ID strings
  ✅ RoomSlug (Zod schema) — validated slug format for URLs

Components:
  ✅ RoomCard (Presentational)
     Props: heading, body, image, ctaLabel, ctaUrl, variant
     Responsibility: UI only

  ✅ RoomListContainer (Container)
     Responsibility: fetch rooms, handle loading/error, render RoomCard list
     Uses: useRooms composable

Composables:
  ✅ useRooms
     Returns: { rooms, isLoading, error, fetchRooms }
     Responsibility: API call + state management

Shared State:
  ✅ None required — state is local to this feature

Feature Structure:
  📁 features/rooms/
    ├── components/
    │   ├── RoomCard.vue
    │   └── RoomListContainer.vue
    ├── composables/
    │   └── useRooms.ts
    ├── types.ts
    ├── api.ts
    └── index.ts

Design Decisions:
  - RoomCard is presentational for reusability and testability
  - Container pattern isolates fetch logic from UI
  - useRooms composable is reusable across room detail and listing pages
```

---

## Part 10: Component Documentation Standard

```typescript
/**
 * FeatureCard
 *
 * Presentational card: icon, heading, body, optional CTA.
 * Used in: Features section, Amenities page, Services listing.
 * Type: Presentational — no state, no side effects.
 *
 * Props:
 *   icon         {string}   — Emoji or SVG. Optional.
 *   heading      {string}   — Card title. Required.
 *   body         {string}   — Supporting description. Required.
 *   variant      {string}   — 'default' | 'highlight' | 'dark'. Default: 'default'.
 *   ctaLabel     {string}   — CTA text. Optional.
 *   ctaUrl       {string}   — CTA href. Optional. Required if ctaLabel is set.
 *   isFullWidth  {boolean}  — Fill container width. Default: false.
 *
 * Example:
 *   <FeatureCard heading="Rooftop Pool" body="Open sunrise to sunset." variant="highlight" />
 */
```

---

## Part 11: Page Assembly Pattern

Pages contain only assembly — no component markup, no styles, no logic.

```vue
<!-- pages/index.vue — ASSEMBLY ONLY -->
<script setup lang="ts">
const features = [
  { icon: '🏊', heading: 'Rooftop Pool',   body: 'Open sunrise to sunset.' },
  { icon: '🏋️', heading: 'Fitness Centre', body: 'Open 24 hours.' },
  { icon: '🍽️', heading: 'Fine Dining',    body: 'Locally sourced, seasonally inspired.' },
]
</script>

<template>
  <AppHeader />
  <HeroSection heading="A Hotel Above the Rest" subheading="Direct beachfront, Kochi" />
  <FeatureGrid :features="features" />
  <RoomListContainer />
  <Testimonials />
  <CtaBanner
    heading="Book Direct for Best Rates"
    cta-label="Check Availability"
    cta-url="/booking"
  />
  <AppFooter />
</template>
```

---

## Quick Reference Checklist

### Before starting any feature
- [ ] Domain questions answered (state, flows, existing architecture)
- [ ] Design plan produced for anything non-trivial
- [ ] Repeating patterns identified
- [ ] Architecture pattern confirmed (feature-based)
- [ ] Global tokens defined

### For every component
- [ ] Classified as Presentational or Container — not mixed
- [ ] Lives in its own file in the correct location
- [ ] All content passed via typed props — nothing hard-coded
- [ ] Visual variants handled via `variant` prop internally
- [ ] Style values reference CSS variables — no raw values
- [ ] Styles scoped to the component (`<style scoped>`)
- [ ] Documentation comment block present

### For every composable
- [ ] Single responsibility — one concern only
- [ ] `use` prefix on filename and function
- [ ] Feature-specific → `features/[feature]/composables/`
- [ ] Shared (2+ features) → root `composables/`

### For every page
- [ ] Assembly only — no component markup
- [ ] No styles at page level
- [ ] No logic that belongs in a component or composable

### Type safety
- [ ] Domain values use Zod schemas or branded types
- [ ] No plain `String` props for IDs, slugs, emails, or prices
