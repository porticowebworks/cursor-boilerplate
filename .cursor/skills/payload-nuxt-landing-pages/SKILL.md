---

name: payload-nuxt-landing-pages

description: >

  Build modular, block-based landing pages using Payload CMS (backend) and Nuxt.js (frontend).

  Use this skill whenever the user is architecting or developing a CMS-driven landing page system

  with Payload blocks, Nuxt dynamic routing, form handling, or rich text rendering. Trigger when

  the user mentions Payload CMS blocks, Nuxt page builder, dynamic block rendering, form submissions

  via Payload, or any block-based page architecture combining these two technologies.

---



# Payload CMS + Nuxt.js: Modular Landing Pages



## Overview



Architecture: **Payload CMS** manages content via a blocks-based layout field. **Nuxt.js** renders that data, but pages/components call only Nuxt API routes (`/api/cms/*`) instead of calling Payload directly.

Integration rule for this skill:

- Pages/components call `/api/*` in Nuxt.
- Nuxt server routes call Payload REST.
- Use one generic Nuxt API layer for all collections (no one-route-per-collection pattern).



> **Note:** All blocks (Hero, Content, NewsletterForm) and the Pages collection defined in this skill are **samples for reference only**. They illustrate the pattern — not a prescribed schema. Adapt field names, types, and structure to your actual project requirements.



---



## 1. Project Initialization



```bash

npx create-payload-app        # Choose blank template + preferred DB (SQLite recommended for dev)

npm install @payloadcms/plugin-form-builder

```



In `payload.config.ts`:

```ts

import { formBuilder } from '@payloadcms/plugin-form-builder'



export default buildConfig({

  plugins: [formBuilder()],  // Auto-generates Forms + FormSubmissions collections

})

```



---



## 2. Block Definitions



Create each block as a separate file. Import into the Pages collection.



### `blocks/Hero.ts`

| Field | Type | Notes |

|---|---|---|

| heading | text | Required |

| subheading | richText | Styled secondary text |

| image | relationship | Points to `media` collection |

| cta | group | Contains `label` (text) + `url` (text) |



### `blocks/Content.ts`

| Field | Type | Notes |

|---|---|---|

| heading | text | Section heading |

| body | richText | Main content, supports lists/links |



### `blocks/NewsletterForm.ts`

| Field | Type | Notes |

|---|---|---|

| heading | text | Optional |

| form | relationship | References `forms` collection (from plugin) |



---



## 3. Pages Collection



```ts

// collections/Pages.ts

import { Hero } from '../blocks/Hero'

import { Content } from '../blocks/Content'

import { NewsletterForm } from '../blocks/NewsletterForm'



export const Pages: CollectionConfig = {

  slug: 'pages',

  fields: [

    { name: 'title', type: 'text', required: true },

    { name: 'slug', type: 'text', required: true, unique: true },

    {

      name: 'layout',

      type: 'blocks',

      blocks: [Hero, Content, NewsletterForm],

    },

  ],

}

```



---



## 4. Nuxt.js Frontend



### Generic Nuxt API Layer (recommended)

Create one scalable pass-through endpoint:

```ts
// server/api/cms/[...path].ts
export default defineEventHandler(async (event) => {
  const config = useRuntimeConfig()
  const path = getRouterParam(event, 'path') || ''
  const method = event.method
  const query = getQuery(event)
  const body = method === 'GET' || method === 'HEAD' ? undefined : await readBody(event)

  const payloadURL = `${config.payloadBaseURL}/api/${path}`

  return await $fetch(payloadURL, {
    method,
    query,
    body,
    headers: config.payloadApiKey
      ? { Authorization: config.payloadApiKey }
      : undefined,
  })
})
```

This keeps auth, headers, caching, and response shaping centralized in one place and scales to any Payload collection.

### Data Fetching — `pages/[slug].vue`






```ts

const route = useRoute()

const { data } = await useFetch('/api/cms/pages', {

  query: { where: { slug: { equals: route.params.slug } } }

})

const page = data.value?.docs?.[0]  // Payload returns array; take first

```



### Dynamic Block Rendering



**Component map** — create files in `components/blocks/`:

- `HeroBlock.vue`

- `ContentBlock.vue`

- `NewsletterBlock.vue`



**Resolver function:**

```ts

const blockMap: Record<string, any> = {

  hero: HeroBlock,

  content: ContentBlock,

  newsletterForm: NewsletterBlock,

}



function resolveBlockComponent(blockType: string) {

  return blockMap[blockType] ?? null

}

```



**Template loop:**

```vue

<template>

  <div v-for="block in page.layout" :key="block.id">

    <component

      :is="resolveBlockComponent(block.blockType)"

      :block="block"

    />

  </div>

</template>

```



> [Uncertain — verify before implementing] Vue dynamic component syntax above follows standard Vue 3 patterns but should be verified against your Nuxt version.



### Rich Text Rendering



Payload returns rich text as a Lexical/Slate JSON object — not raw HTML. Use a dedicated renderer:



```vue

<!-- components/RichText.vue -->

<script setup>

defineProps<{ content: any }>()

</script>

<!-- Use @payloadcms/richtext-lexical or a custom renderer to parse content -->

```



---



## 5. Form Submissions



**Inside `NewsletterBlock.vue`:**



```ts

// Loop through form.fields from CMS to render inputs dynamically

// POST to Nuxt API layer; server route forwards to Payload



const handleSubmit = async () => {

  await $fetch('/api/cms/form-submissions', {

    method: 'POST',

    body: {

      form: props.block.form.id,

      submissionData: formFields.value.map(field => ({

        field: field.name,

        value: formValues.value[field.name],

      })),

    },

  })

}

```



**Required payload structure:**

```json

{

  "form": "<form_document_id>",

  "submissionData": [

    { "field": "email", "value": "user@example.com" },

    { "field": "firstName", "value": "John" }

  ]

}

```



**State management checklist:**

- [ ] `loading` — disable submit button during POST

- [ ] `success` — display `form.confirmationMessage` from CMS

- [ ] `error` — show fallback error message



---



## 6. Key Architecture Decisions & Tradeoffs



| Decision | KISS | Balanced | Best Practice |

|---|---|---|---|

| Block rendering | Manual `v-if` per block type | Component map object | Auto-import from `/blocks` dir with Nuxt auto-imports |

| Rich text | `v-html` with serialized string | Custom Lexical renderer | `@payloadcms/richtext-lexical` React-to-Vue port |

| Nuxt-Payload integration | Pages call Payload directly | Mixed direct + server routes | Pages call only Nuxt `/api/cms/*`; generic catch-all route forwards to Payload |

| Form fields | Hardcoded inputs | Dynamic render from `form.fields` | Full form builder with validation schema from CMS |



---



## 7. Common Pitfalls



- **`blockType` casing** — Payload uses camelCase (`newsletterForm`), ensure component map keys match exactly

- **Rich text as JSON** — never try to render Payload rich text directly as HTML without a serializer

- **Form ID** — the `form` relationship field returns either an ID string or a populated object depending on depth; always check `form.id ?? form`

- **Slug uniqueness** — enforce at collection level or you'll get ambiguous API responses

- **Bypassing Nuxt API layer** — avoid direct browser calls to Payload; it spreads auth/caching/transform logic across the app



---



## References



- Payload Form Builder plugin: `@payloadcms/plugin-form-builder`

- Nuxt submission endpoint: `POST /api/cms/form-submissions` (forwarded to Payload `POST /api/form-submissions`)

- Payload REST API depth param: `?depth=2` to populate relationships in layout blocks

