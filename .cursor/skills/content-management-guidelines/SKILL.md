---
name: content-management-guidelines
description: >
  Developer guidelines for setting up Payload CMS collections, fields, blocks, and access
  control correctly on Nuxt 4 marketing site projects. Covers collection design principles,
  field type selection, validation rules, block architecture, reusability patterns, role-based
  access control, and field-level access. Use this skill whenever a developer is creating or
  modifying a Payload CMS collection, designing fields, building page builder blocks, setting
  up access control, or reviewing an existing Payload config for correctness.
  Trigger when the user mentions Payload CMS, collections, fields, blocks, access control,
  roles, schema design, or asks how to structure content in Payload.
---

# Payload CMS — Content Management Guidelines

## Core Principle

Payload config is your data contract. Every field you define becomes an API shape, a database
column, and an admin UI element simultaneously. Mistakes at the config level propagate into
the frontend, the database, and the editor experience. Design deliberately — never add fields
speculatively and never leave access control unconfigured.

> **All code examples are samples for reference only.** Adapt slugs, field names, and roles
> to your actual project.

---

## Part 1: Collection Design

### 1.1 Collection Naming Conventions

| Rule | Correct | Wrong |
|---|---|---|
| `slug` is plural, kebab-case | `'pages'`, `'blog-posts'`, `'team-members'` | `'Page'`, `'blogPost'`, `'TeamMember'` |
| `labels` are human-readable | `singular: 'Blog Post'`, `plural: 'Blog Posts'` | Omitted or matching slug |
| File name matches slug | `collections/BlogPosts.ts` | `collections/posts.ts` for `slug: 'blog-posts'` |
| One collection per file | `collections/Pages.ts` | Multiple collections in one file |

```typescript
// collections/BlogPosts.ts
import type { CollectionConfig } from 'payload'

export const BlogPosts: CollectionConfig = {
  slug: 'blog-posts',
  labels: {
    singular: 'Blog Post',
    plural:   'Blog Posts',
  },
  admin: {
    useAsTitle: 'title',          // Which field shows in admin list view
    defaultColumns: ['title', 'status', 'publishedAt', 'updatedAt'],
    group: 'Content',             // Groups collections in sidebar
  },
  fields: [...],
}
```

### 1.2 Standard Fields Every Content Collection Should Have

```typescript
fields: [
  // 1. Title — always first, always required
  {
    name:     'title',
    type:     'text',
    required: true,
  },

  // 2. Slug — URL-safe identifier, auto-generated from title
  {
    name:     'slug',
    type:     'text',
    required: true,
    unique:   true,
    admin: {
      description: 'URL-safe identifier. Auto-generated from title. Do not change after publishing.',
    },
    hooks: {
      beforeValidate: [
        ({ value, data }) =>
          value || data?.title?.toLowerCase().replace(/\s+/g, '-').replace(/[^a-z0-9-]/g, ''),
      ],
    },
  },

  // 3. Status — controls frontend visibility
  {
    name:         'status',
    type:         'select',
    required:     true,
    defaultValue: 'draft',
    options: [
      { label: 'Draft',     value: 'draft' },
      { label: 'Published', value: 'published' },
    ],
  },

  // 4. Published date — for ordering and display
  {
    name: 'publishedAt',
    type: 'date',
    admin: {
      date: { pickerAppearance: 'dayAndTime' },
      description: 'Set to control publish order. Auto-populated on first publish if left blank.',
    },
  },

  // 5. SEO group — always included
  {
    name: 'seo',
    type: 'group',
    fields: [
      {
        name:      'title',
        type:      'text',
        maxLength: 60,
        admin: { description: '50–60 characters. Overrides the page title in search results.' },
      },
      {
        name:      'description',
        type:      'textarea',
        maxLength: 160,
        admin: { description: '140–160 characters. Shown in search result snippets.' },
      },
      {
        name: 'ogImage',
        type: 'upload',
        relationTo: 'media',
        admin: { description: '1200×630px. Used when the page is shared on social media.' },
      },
    ],
  },
]
```

### 1.3 Collection Architecture Decisions

**One collection per content type.** Never combine fundamentally different content types into one collection with a `type` select field to differentiate them.

```typescript
// ❌ One collection for everything
slug: 'content'
fields: [{ name: 'type', type: 'select', options: ['page', 'post', 'room'] }]

// ✅ Separate collections — different schemas, different access rules
slug: 'pages'
slug: 'blog-posts'
slug: 'rooms'
```

**Use Globals for singleton content** — content that has exactly one instance:

```typescript
// globals/SiteSettings.ts
export const SiteSettings: GlobalConfig = {
  slug: 'site-settings',
  fields: [
    { name: 'siteName',     type: 'text', required: true },
    { name: 'contactEmail', type: 'email' },
    { name: 'phone',        type: 'text' },
    { name: 'address',      type: 'textarea' },
  ],
}
```

Globals for: site settings, navigation menus, footer content, global announcement banners.
Collections for: anything with multiple instances — pages, posts, rooms, team members, testimonials.

---

## Part 2: Field Design

### 2.1 Field Type Selection

| Content type | Field type | Notes |
|---|---|---|
| Short text (title, name, label) | `text` | Add `maxLength` |
| Long plain text (description) | `textarea` | Add `maxLength` |
| Rich formatted content | `richText` | Lexical editor — configure toolbars explicitly |
| True/false toggle | `checkbox` | Use for `isFeatured`, `showInNav`, `isActive` |
| Fixed options | `select` | Always define `options` + `defaultValue` |
| Multiple fixed options | `select` with `hasMany: true` | Or `relationship` to a taxonomy collection |
| Number | `number` | Add `min`/`max` where meaningful |
| Date / time | `date` | Set `pickerAppearance` explicitly |
| Email | `email` | Built-in format validation |
| URL | `text` + custom validate | No native URL field — add regex validation |
| Image / file | `upload` | Always `relationTo: 'media'` |
| Link to another document | `relationship` | Specify `relationTo` |
| Grouped related fields | `group` | SEO, address, social links |
| Repeatable rows | `array` | Nav links, gallery items, price tiers |
| Polymorphic layout builder | `blocks` | Page builder sections |
| Colour picker / code | `text` + custom UI | Use `admin.components` for custom input |

### 2.2 Field Rules

**Always set `required: true` on fields the frontend depends on.** Never assume the editor will fill them in.

```typescript
// ❌ Frontend will break if title is empty
{ name: 'title', type: 'text' }

// ✅
{ name: 'title', type: 'text', required: true }
```

**Always set `defaultValue` on select and checkbox fields.**

```typescript
// ❌ Undefined status causes frontend filter logic to fail
{ name: 'status', type: 'select', options: [...] }

// ✅
{ name: 'status', type: 'select', options: [...], defaultValue: 'draft', required: true }
```

**Always set `maxLength` on text and textarea fields shown in UI.**

```typescript
{
  name:      'seoTitle',
  type:      'text',
  maxLength: 60,
  admin: { description: '50–60 characters recommended.' },
}
```

**Always add `admin.description` to fields that need editorial guidance.**

```typescript
{
  name: 'slug',
  type: 'text',
  admin: {
    description: 'URL identifier. Auto-generated from title. Do not change after publishing — this will break existing links.',
  },
}
```

**Never use `type: 'text'` for email fields.**

```typescript
// ❌ No email format validation
{ name: 'contactEmail', type: 'text' }

// ✅ Built-in email validation
{ name: 'contactEmail', type: 'email' }
```

### 2.3 Custom Validation

Add validation when Payload's built-in constraints are insufficient:

```typescript
{
  name: 'websiteUrl',
  type: 'text',
  validate: (value) => {
    if (!value) return true // Optional field — skip if empty
    try {
      new URL(value)
      return true
    } catch {
      return 'Please enter a valid URL including https://'
    }
  },
},

{
  name: 'phone',
  type: 'text',
  validate: (value) => {
    if (!value) return true
    const clean = value.replace(/[\s\-\+\(\)]/g, '')
    if (!/^\d{7,15}$/.test(clean)) return 'Please enter a valid phone number'
    return true
  },
},
```

### 2.4 Relationship Fields

```typescript
// ✅ Single relationship
{
  name:       'featuredImage',
  type:       'upload',
  relationTo: 'media',
  required:   true,
}

// ✅ Relationship to another collection
{
  name:       'author',
  type:       'relationship',
  relationTo: 'team-members',
  required:   true,
}

// ✅ Polymorphic relationship (links to one of multiple collections)
{
  name:       'relatedContent',
  type:       'relationship',
  relationTo: ['blog-posts', 'pages'],
  hasMany:    true,
}
```

**Rules:**
- Never use `type: 'text'` to store a document ID — use `relationship` or `upload`
- Always specify `relationTo` — never leave it ambiguous
- Use `hasMany: true` for lists of related documents
- Set `required: true` if the frontend needs the relationship to render

### 2.5 RichText (Lexical) Configuration

Configure the Lexical toolbar explicitly. Never use the default — it exposes formatting options editors will misuse.

```typescript
import { lexicalEditor, BoldFeature, ItalicFeature,
         LinkFeature, UnorderedListFeature, HeadingFeature } from '@payloadcms/richtext-lexical'

{
  name: 'content',
  type: 'richText',
  editor: lexicalEditor({
    features: ({ defaultFeatures }) => [
      HeadingFeature({ enabledHeadingSizes: ['h2', 'h3'] }), // H1 is the page title — never in body
      BoldFeature(),
      ItalicFeature(),
      UnorderedListFeature(),
      LinkFeature(),
      // Do NOT include: AlignFeature (use CSS), ColorFeature (breaks design tokens),
      // IndentFeature (formatting crutch)
    ],
  }),
}
```

---

## Part 3: Block Architecture

Blocks are the page builder units. They are the most reused and most misdesigned part of most Payload setups.

### 3.1 Block Design Rules

**One block = one clearly named, self-contained section.** Blocks should map to visible page sections, not to abstract layout concepts.

```
✅ Good block names          ❌ Bad block names
────────────────────         ──────────────────
HeroSection                  Column
FeatureGrid                  Content
TestimonialsCarousel         Section
CtaBanner                    Block1
SplitSection                 Layout
RoomListing                  Component
```

**Blocks should be reusable across page types.** A `CtaBanner` block used on the homepage should be the same block used on a rooms page — not a copy with slight differences.

**Blocks should not be too granular.** A "Heading" block and a "Paragraph" block are too granular — combine into a `ContentSection` block. Editors should not be assembling typography, they should be assembling sections.

**Blocks should not be too monolithic.** A single "Page" block that contains everything is no different from not using blocks at all.

### 3.2 Block File Structure

```
collections/
  blocks/
    HeroSection.ts
    FeatureGrid.ts
    SplitSection.ts
    TestimonialsCarousel.ts
    CtaBanner.ts
    RoomListing.ts
    FaqSection.ts
```

Each block in its own file. Import into collections:

```typescript
// collections/Pages.ts
import { HeroSection }          from './blocks/HeroSection'
import { FeatureGrid }          from './blocks/FeatureGrid'
import { SplitSection }         from './blocks/SplitSection'
import { TestimonialsCarousel } from './blocks/TestimonialsCarousel'
import { CtaBanner }            from './blocks/CtaBanner'

export const Pages: CollectionConfig = {
  slug: 'pages',
  fields: [
    { name: 'title', type: 'text', required: true },
    {
      name: 'layout',
      type: 'blocks',
      blocks: [
        HeroSection,
        FeatureGrid,
        SplitSection,
        TestimonialsCarousel,
        CtaBanner,
      ],
    },
  ],
}
```

### 3.3 Block Config Template

```typescript
// collections/blocks/CtaBanner.ts
import type { Block } from 'payload'

export const CtaBanner: Block = {
  slug:        'cta-banner',
  labels: {
    singular:  'CTA Banner',
    plural:    'CTA Banners',
  },
  fields: [
    {
      name:     'heading',
      type:     'text',
      required: true,
      maxLength: 80,
      admin: { description: 'Main call to action headline.' },
    },
    {
      name:     'subheading',
      type:     'text',
      maxLength: 120,
    },
    {
      name:         'variant',
      type:         'select',
      defaultValue: 'default',
      options: [
        { label: 'Default',  value: 'default' },
        { label: 'Dark',     value: 'dark' },
        { label: 'Accent',   value: 'accent' },
      ],
      admin: { description: 'Controls the visual style of the banner.' },
    },
    {
      name: 'primaryCta',
      type: 'group',
      fields: [
        { name: 'label', type: 'text',     required: true },
        { name: 'url',   type: 'text',     required: true },
        { name: 'openInNewTab', type: 'checkbox', defaultValue: false },
      ],
    },
    {
      name: 'secondaryCta',
      type: 'group',
      admin: { description: 'Optional secondary action.' },
      fields: [
        { name: 'label', type: 'text' },
        { name: 'url',   type: 'text' },
      ],
    },
  ],
}
```

### 3.4 Block Field Violations to Flag

```typescript
// ❌ Block with no variant control — single visual style, not reusable
export const HeroSection: Block = {
  slug: 'hero-section',
  fields: [
    { name: 'heading', type: 'text' },
    // No variant, no layout options — every hero looks identical
  ],
}

// ❌ Block storing a raw colour value — bypasses design tokens
{ name: 'backgroundColor', type: 'text', admin: { description: 'e.g. #ff0000' } }
// ✅ Use a select with predefined token-mapped options instead
{ name: 'backgroundColor', type: 'select', options: [
    { label: 'White',   value: 'surface' },
    { label: 'Light',   value: 'surface-alt' },
    { label: 'Dark',    value: 'primary' },
  ], defaultValue: 'surface',
}

// ❌ Block with hard-coded image path as text
{ name: 'image', type: 'text', admin: { description: 'Paste image URL here' } }
// ✅ Always use upload relationship
{ name: 'image', type: 'upload', relationTo: 'media' }

// ❌ Duplicate block for minor variation
// HeroSectionDark.ts — identical to HeroSection with background colour changed
// ✅ Add variant prop to the original block instead
```

---

## Part 4: Access Control

### 4.1 Role Model

Define roles on the Users collection. Three roles cover all marketing site scenarios:

```typescript
// collections/Users.ts
export const Users: CollectionConfig = {
  slug:   'users',
  auth:   true,
  fields: [
    {
      name:         'role',
      type:         'select',
      required:     true,
      defaultValue: 'editor',
      options: [
        { label: 'Admin',  value: 'admin' },   // Full access — developer/agency
        { label: 'Editor', value: 'editor' },  // Content only — client team
      ],
      access: {
        // Only admins can change roles — editors cannot promote themselves
        update: ({ req }) => req.user?.role === 'admin',
      },
    },
  ],
}
```

### 4.2 Access Control Functions

Define reusable functions in a shared file — never inline the same logic in multiple collections:

```typescript
// access/index.ts
import type { Access } from 'payload'

// Any authenticated user
export const isLoggedIn: Access = ({ req }) => !!req.user

// Admins only
export const isAdmin: Access = ({ req }) =>
  req.user?.role === 'admin'

// Admins or editors
export const isAdminOrEditor: Access = ({ req }) =>
  req.user?.role === 'admin' || req.user?.role === 'editor'

// Public read — no auth required
export const isPublic: Access = () => true

// Published content only for public; all content for authenticated users
export const isPublishedOrLoggedIn: Access = ({ req }) => {
  if (req.user) return true
  return {
    status: { equals: 'published' },
  }
}
```

### 4.3 Collection-Level Access

Apply access functions to every collection's `access` config. **Never leave access unconfigured** — Payload's default is open to all authenticated users, which is usually too permissive.

```typescript
// Public-facing content collection (pages, blog posts, rooms)
access: {
  read:   isPublishedOrLoggedIn,  // Public reads published only
  create: isAdminOrEditor,        // Editors can create
  update: isAdminOrEditor,        // Editors can update
  delete: isAdmin,                // Only admins can delete
},

// Internal/settings collection (site settings, navigation)
access: {
  read:   isLoggedIn,   // Any logged-in user can read
  create: isAdmin,      // Only admins can create
  update: isAdmin,      // Only admins can update
  delete: isAdmin,      // Only admins can delete
},

// Media collection
access: {
  read:   isPublic,           // Media URLs must be publicly accessible
  create: isAdminOrEditor,
  update: isAdminOrEditor,
  delete: isAdmin,
},
```

### 4.4 Field-Level Access

Use field-level access for fields editors should not modify directly:

```typescript
{
  name: 'slug',
  type: 'text',
  access: {
    // Editors can set slug on create but only admins can change it after publish
    // (changing slug breaks existing URLs)
    update: isAdmin,
  },
},

{
  name: 'publishedAt',
  type: 'date',
  access: {
    update: isAdmin,  // Only admins control publish dates
  },
},

{
  name: 'internalNotes',
  type: 'textarea',
  access: {
    read:   isAdmin,   // Editors cannot see internal notes
    create: isAdmin,
    update: isAdmin,
  },
},
```

### 4.5 Access Control Violations to Flag

```typescript
// ❌ No access config — defaults to all authenticated users
export const Pages: CollectionConfig = {
  slug: 'pages',
  fields: [...],
  // Missing access property entirely
}

// ❌ Public read without status filter — drafts exposed to frontend
access: {
  read: isPublic,  // Returns ALL documents including drafts
}
// ✅
access: {
  read: isPublishedOrLoggedIn,  // Public gets published only
}

// ❌ Editors can delete content
access: {
  delete: isAdminOrEditor,  // Editors should not delete — only admins
}
// ✅
access: {
  delete: isAdmin,
}

// ❌ Role field editable by editors — privilege escalation risk
{
  name: 'role',
  type: 'select',
  // No field-level access — any editor can promote themselves to admin
}
// ✅
{
  name: 'role',
  type: 'select',
  access: { update: isAdmin },
}

// ❌ Inline access logic — duplicated across collections
access: {
  read: ({ req }) => req.user?.role === 'admin' || req.user?.role === 'editor',
  // Same logic copy-pasted in 5 collections
}
// ✅ Use shared access functions from access/index.ts
access: {
  read: isAdminOrEditor,
}
```

---

## Part 5: Media Collection

Every project needs a properly configured Media collection. Never rely on Payload's default.

```typescript
// collections/Media.ts
export const Media: CollectionConfig = {
  slug:   'media',
  upload: {
    staticDir:    'public/media',
    staticURL:    '/media',
    imageSizes: [
      { name: 'thumbnail', width: 400,  height: 300,  crop: 'centre' },
      { name: 'card',      width: 800,  height: 600,  crop: 'centre' },
      { name: 'hero',      width: 1440, height: 600,  crop: 'centre' },
      { name: 'og',        width: 1200, height: 630,  crop: 'centre' },
    ],
    adminThumbnail: 'thumbnail',
    mimeTypes: ['image/jpeg', 'image/png', 'image/webp', 'image/svg+xml'],
  },
  access: {
    read:   isPublic,
    create: isAdminOrEditor,
    update: isAdminOrEditor,
    delete: isAdmin,
  },
  fields: [
    {
      name: 'alt',
      type: 'text',
      required: true,
      admin: { description: 'Describe the image for screen readers and SEO. Required.' },
    },
    {
      name: 'caption',
      type: 'text',
      admin: { description: 'Optional caption displayed below the image.' },
    },
  ],
}
```

**Rules:**
- `alt` is required on Media — enforced at the CMS level, not hoped for at the frontend
- Define `imageSizes` at setup — retrofitting image sizes requires re-uploading all media
- `mimeTypes` whitelist prevents non-image uploads to the image library
- `read: isPublic` is correct for media — image URLs must be publicly accessible

---

## Part 6: Payload Config Assembly

```typescript
// payload.config.ts
import { buildConfig } from 'payload'
import { Pages }       from './collections/Pages'
import { BlogPosts }   from './collections/BlogPosts'
import { Rooms }       from './collections/Rooms'
import { Media }       from './collections/Media'
import { Users }       from './collections/Users'
import { SiteSettings } from './globals/SiteSettings'

export default buildConfig({
  collections: [Pages, BlogPosts, Rooms, Media, Users],
  globals:     [SiteSettings],

  admin: {
    user: Users.slug,
    meta: {
      titleSuffix: '— CMS',
    },
  },

  // TypeScript strict mode
  typescript: {
    outputFile: 'types/payload-types.ts',
  },

  // Always enable in production
  cors:        [process.env.NUXT_PUBLIC_SITE_URL || ''],
  csrf:        [process.env.NUXT_PUBLIC_SITE_URL || ''],
})
```

---

## Quick Reference Checklist

### Every collection
- [ ] `slug` is plural and kebab-case
- [ ] `labels` singular and plural defined
- [ ] `admin.useAsTitle` set to the title field
- [ ] `admin.group` set for sidebar organisation
- [ ] `access` fully configured — no missing operations
- [ ] `status` field with `draft`/`published` + `defaultValue: 'draft'`
- [ ] `slug` field with auto-generation hook
- [ ] SEO group with title, description, ogImage

### Every field
- [ ] `required: true` on fields the frontend depends on
- [ ] `defaultValue` on all `select` and `checkbox` fields
- [ ] `maxLength` on all `text` and `textarea` fields
- [ ] `admin.description` on any field needing editorial guidance
- [ ] `email` type (not `text`) for email fields
- [ ] `upload` + `relationTo: 'media'` for image fields (not text URL)

### Every block
- [ ] Lives in its own file under `collections/blocks/`
- [ ] Has a `variant` select field if visual variations exist
- [ ] Image fields use `upload` + `relationTo: 'media'` — not text URLs
- [ ] Colour/style fields use predefined `select` options — not free-text colour values
- [ ] Not duplicated for minor variations — extend with props instead

### Access control
- [ ] Shared access functions defined in `access/index.ts`
- [ ] No inline access logic duplicated across collections
- [ ] `read` uses `isPublishedOrLoggedIn` for public content — not `isPublic`
- [ ] `delete` restricted to `isAdmin` — editors cannot delete
- [ ] `role` field on Users has `update: isAdmin` field-level access
- [ ] Media `read` is `isPublic` — image URLs must be accessible
