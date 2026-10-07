# Astro target

The target is always a static Astro site. Astro changes quickly between major versions: check the
installed version, read its docs and upgrade guide, and prefer what the docs say over memory.

## Sources of truth for Astro

- Docs: https://docs.astro.build (an LLM-friendly index is published at `/llms.txt`).
- The Astro Docs MCP server, if the user agrees to add it (search "Astro Docs MCP server" for its
  current URL), lets you query the docs for the exact installed version.
- `node_modules/astro` types and changelog for anything the docs do not cover.

Scaffold with `npm create astro@latest` (or the user's package manager), TypeScript strict, then pin the
Node version the Astro release requires (`engines` in `package.json`).

## Configuration (`astro.config.mjs`)

| Setting | Recommendation | Why |
| --- | --- | --- |
| `site` | The production URL | Canonicals, sitemaps, feeds and JSON-LD need absolute URLs |
| `output` | `'static'` | No server; no adapter needed |
| `trailingSlash` + `build.format` | Match the live site: `'never'` + `'file'` for `/pricing`, `'always'` + `'directory'` for `/pricing/` | URLs must not change |
| `i18n` | `locales`, `defaultLocale`, `routing.prefixDefaultLocale: false` when the default locale has no prefix | Matches most builder sites (`/` and `/es/...`) |
| `image` | `layout: 'constrained'`, `responsiveStyles: true` | Responsive `srcset` and no layout shift by default |
| `fonts` | The built-in Fonts API with `fontProviders.local()` for self-hosted brand fonts, with fallbacks | Generates optimized fallbacks and preloads; preload only above-the-fold faces |
| `prefetch` | Hover strategy, not prefetch-all | Fast navigation without wasting bandwidth |
| `markdown` | Disable smart punctuation for migrated text | Quotes and dashes must stay exactly as published |

Static output does not turn `redirects` into real HTTP 301s (it writes meta-refresh pages). Keep the
redirect map as data in `src/config/redirects.ts` and compile it into the host's format (`vercel.ts` or
`vercel.json`, Netlify `_redirects` or `netlify.toml`, Cloudflare `_redirects`, ...).

## Integrations and packages

Check each one supports the installed Astro major version before adding it.

| Need | Package | Notes |
| --- | --- | --- |
| Type and content checking | `@astrojs/check` + `typescript` | `astro check` is a phase gate |
| Styling | `tailwindcss` + `@tailwindcss/vite` (Tailwind v4) | Tokens in `@theme`; disable the default palette and type scale so only source tokens exist; `class-variance-authority`, `clsx` and `tailwind-merge` for variants |
| SEO head, JSON-LD graph, build-time checks, llms.txt | `@jdevalk/astro-seo-graph` (+ `schema-dts` types) | One linked JSON-LD graph per page; validates one h1, unique titles, alt text, metadata length, internal links |
| Sitemaps | `@astrojs/sitemap`, or custom endpoints | The official integration cannot pair translated slugs across locales; write `sitemap.xml.ts` endpoints with `xhtml:link` alternates when slugs are translated |
| RSS | `@astrojs/rss` | One feed per locale where live has one |
| Formatting | `prettier` + `prettier-plugin-astro` (+ `prettier-plugin-tailwindcss`) | Keep embedded third-party widgets out of the formatter |
| Hosting config | The host's config package or file (e.g. `@vercel/config` for `vercel.ts`) | Redirects, headers, clean URLs, `noindex` on non-production hosts |

Migration and QA tooling (dev dependencies): `cheerio` (parse cached HTML), `turndown` +
`turndown-plugin-gfm` (rich text to Markdown), `yaml`, `sharp` (WebP conversion, image diffs),
`playwright` (screenshots), `pixelmatch` + `pngjs` (pixel diffs), `@axe-core/playwright` (accessibility).

Prefer no UI framework integration (React, Vue, Svelte...). Builder sites need little JavaScript;
`.astro` components with a bundled `<script>` cover tabs, carousels and filters. Add a framework island
only for a widget that truly needs one.

## Project structure

```
src/
  pages/            thin routes: [...path].astro for marketing pages,
                    [...locale]/[base]/[slug].astro for collection entries
  layouts/          BaseLayout.astro: <html>, SEO head, analytics, header, <main>, footer
  components/
    ui/             primitives without content (Button, Link, Picture, Tag...)
    layout/         Section (vertical rhythm), Container (gutter + max width)
    sections/<Name>/<Name>.astro + schema.ts   one folder per section type, discovered by name
    templates/      one page template per routed collection
    navigation/     header, mega menu, footer, language switcher, breadcrumbs
    seo/            head tags and JSON-LD helpers
  content/<collection>/<locale>/<key>.(md|yaml)   content only, one folder per locale
  content-model/    Zod schemas and collection definitions
  content.config.ts registers every collection
  i18n/             locale registry, typed UI dictionaries per locale and namespace
  config/           site identity, navigation, redirects
  lib/              routing (entry URLs, alternates, localizer), content queries, SEO builders
  styles/global.css tokens and base styles
  assets/           images and fonts processed by Astro
public/             files served verbatim
scripts/            migration/, qa/, audit/ (Node scripts)
```

## Content

- Define collections in `src/content.config.ts` with the `glob()` loader and Zod schemas. Keep the
  locale folder in the entry id (`es/my-post`) so translations pair by key.
- A marketing page is a YAML entry: `slug`, `seo`, header/footer variants, and `sections: [{type, ...}]`.
  A `SectionRenderer` maps `type` to `components/sections/<Name>/` by naming convention, so adding a
  section needs no registry edit.
- Use `reference()` for relations (author, category), `z.coerce.date()` for dates, and the schema
  `image()` helper for images so they are processed by `astro:assets`.
- Long-form bodies are Markdown; keep a little raw HTML only where Markdown cannot match the live page
  (captioned figures, embeds). Store interactive widgets as separate files.
- If schemas live outside `content.config.ts`, the content cache may not invalidate when they change;
  check the installed version's behaviour and clear the cache (or fingerprint the schemas) when needed.

## Routing and i18n

- `[...locale]` as an optional rest param serves `/` and `/es/` from one route file.
- Generate collection pages with `getStaticPaths()`; translated folders (`/es/base-de-conocimiento/...`)
  are a route param, so there is one dynamic route for all collections instead of one file each.
- Build every URL through helpers (`entryPath`, `getAlternates`, a localizer for internal links); never
  hand-build URLs in templates. Hreflang, the language switcher, sitemaps and breadcrumbs all read the
  same helpers.
- UI microcopy goes in typed dictionaries per locale (`t('common.logIn')`), with non-default locales
  type-checked against the default one.

## Images, fonts and performance

- `<Image>` and `<Picture>` from `astro:assets` with width/height-aware sources. The LCP image gets
  `loading="eager"` and `fetchpriority="high"`; everything else is lazy.
- When the largest paint is a CSS background, preload that image explicitly.
- Convert exported PNG screenshots to WebP during migration.
- Iframes hidden at some breakpoints must be `loading="lazy"` (browsers skip hidden lazy iframes).
- Import heavy libraries (animation runtimes, video players) on demand with `import()` when the element
  nears the viewport.

## Version gotchas to check

Record the ones that apply in `AGENTS.md`:

- Newer compilers follow JSX whitespace rules: a newline between inline elements renders no space; keep
  inline runs on one line or add `{' '}`.
- Unclosed tags are errors.
- The Markdown pipeline may differ between versions (remark/rehype plugins vs a newer processor); check
  which plugin API applies before writing Markdown plugins.
- Only one `astro dev` server per project may run; reuse it from every agent, and never run two builds at
  once in the same folder.
