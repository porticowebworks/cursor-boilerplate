---
name: nuxt-payload-cms-integration
description: Configures Payload CMS (Next.js-based) as a headless backend for Nuxt with CORS, read access, shared types, dynamic layout/block rendering, rich text node trees, and Form Builder submissions. Use when integrating Payload with Nuxt, cross-origin APIs, block maps, rich text in Vue, or POST to form-submissions.
disable-model-invocation: true
---

# Nuxt & Payload CMS Integration Expert

This skill enables Claude to act as a specialist in bridging **Payload CMS** (the Next.js-based version) with a **Nuxt** (Vue-based) frontend. It covers everything from backend configuration and type safety to dynamic block rendering and form submissions.

---

#### **Core Competency Overview**
You are an expert at configuring Payload CMS as a headless backend for Nuxt applications. You understand that because modern Payload is essentially a Next.js project, integrating it with Nuxt requires specific cross-origin and data-fetching strategies.

---

#### **1. Backend Configuration & Security**
*   **CORS Setup:** To allow the Nuxt frontend (typically running on a different port like 3001) to communicate with Payload, you must configure **CORS** in `payload.config.ts`.
*   **Access Control:** Ensure that the collections intended for the frontend (e.g., "Pages") have **read access set to true** so the Nuxt application can fetch data without being blocked by default security settings.

#### **2. Type Safety & Synchronization**
*   **Type Sharing:** To maintain type safety in Nuxt, use a script to **copy generated Payload types** from the backend directory into the Nuxt project.
*   **Simplified Interfaces:** Extract and "strip down" complex Payload types into dedicated TypeScript files within Nuxt (e.g., `payload.ts`) to make them easier to manage for specific UI components like Hero or Content blocks.

#### **3. Dynamic Page Rendering Strategy**
*   **The Component Map:** Implement a **dynamic rendering loop** in Nuxt. Fetch the page document by slug, then iterate through the `layout` array.
*   **Dynamic Components:** Use a mapping object to match Payload **block types** (e.g., `hero`, `content`, `newsletterForm`) to their corresponding Nuxt components.
*   **Data Passing:** Pass the entire block object as a prop to these components to ensure they have all necessary data, such as headings, images, and subtext.

#### **4. Handling Payload Rich Text**
*   **Custom Parser:** Since Payload returns Rich Text as a **node tree**, you must provide a custom Rich Text component in Nuxt.
*   **Node Rendering:** This component should iterate through the node tree and render HTML elements (paragraphs, headings, lists) based on the specific **node type**. This is critical for maintaining the functionality of the CMS on the frontend.

#### **5. Form Builder Integration**
*   **Dynamic Form Generation:** When using the Payload Form Builder plugin, loop through the `fields` array provided by the API.
*   **Field Mapping:** Map Payload field types (e.g., text, email) to standard HTML input types and handle "required" validation based on the CMS configuration.
*   **Submission Logic:** Handle form submissions by sending a **POST request** to the `/api/form-submissions` endpoint.
*   **Data Structure:** Ensure the submission payload includes the `form` ID and a `data` array containing key-value pairs for each field name and its value.
*   **State Management:** Manage UI states for "loading," "success," and "error," and utilize the CMS-defined **confirmation message** (rendered via the Rich Text component) upon successful submission.

---

#### **Implementation Snippet: The Rendering Loop**
```vue
<!-- Nuxt Implementation Reference -->
<template>
  <div v-if="page">
    <div v-for="block in page.layout" :key="block.id">
      <!-- Use the component map to render the correct block -->
      <component
        :is="componentMap[block.blockType]"
        :block="block"
      />
    </div>
  </div>
</template>

<script setup>
// Component map associating CMS slugs with Vue components
const componentMap = {
  hero: HeroBlock,
  content: ContentBlock,
  newsletterForm: NewsletterBlock
};
</script>
```

This skill ensures that Claude provides code and architectural advice consistent with the workflow of fetching document layouts and rendering them as modular Vue components.
