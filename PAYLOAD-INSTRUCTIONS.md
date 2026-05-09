# Payload CMS — bootstrap prompt

Use this file as the instruction block when scaffolding or extending the `cms/` app for a new client site. Follow steps in order; align env values with the **Environment variables** section in `NUXT-INSTRUCTIONS.md` at the monorepo root (web/Nuxt side).

## Scaffold

```bash
cd cms
npx create-payload-app@latest
```

## Plugins (baseline)

```bash
npm install \
  @payloadcms/plugin-form-builder \
  @payloadcms/plugin-seo \
  @payloadcms/plugin-redirects \
  @payloadcms/plugin-nested-docs
```

Adjust versions to match the generated Payload major version.



## Windows (optional native bindings)

Install if postinstall or builds complain about missing MSVC bindings (run from `cms/` after the project exists):

```bash
npm install @oxc-transform/binding-win32-x64-msvc@^0.126.0 @rollup/rollup-win32-x64-msvc@^4.60.2 @oxc-minify/binding-win32-x64-msvc@^0.126.0 lightningcss-win32-x64-msvc@^1.32.0
```



