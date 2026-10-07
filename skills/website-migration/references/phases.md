# Phases

Each phase ends with its gate, a commit and a push. A phase's reports feed the next phase's brief.
Agent counts assume a budget of about five subagents in total; scale them to the user's budget.

## Phase 0: capture and foundation (orchestrator only)

1. **Capture.** Crawl every live URL (sitemaps plus link discovery) in every locale into a gitignored
   cache: rendered HTML, the CMS export, every asset, all CSS in cascade order (deduplicated into one
   file you can grep for exact values), and a manifest of live URLs with their status codes. Make the
   crawl re-runnable so content can be re-synced right before cutover.
2. **QA tools.** Build them before any page; see `qa-tooling.md`.
3. **Contract.** Write `AGENTS.md` (symlink it as `CLAUDE.md` so every agent finds it): architecture map,
   commands, content model, i18n rules, styling rules, SEO rules, code style, and the gotchas of the
   framework version. Keep it current; it is the contract for every agent and every future session.
4. **Foundation.** Scaffold the Astro project and its integrations as described in `astro.md`. Then routing (locale prefixes, translated slugs), i18n helpers, the page model (a page is
   SEO metadata plus an ordered list of typed sections), the base layout with the SEO head, header,
   footer, language switcher, UI primitives, design tokens taken from the source CSS, and ONE reference
   section built end to end: schema, component, and an extractor that turns the cached HTML into
   content. Every later section copies this pattern.
5. **Gate:** type check with 0 errors, build passes, the reference section matches live in the visual
   diff. Commit and push.

## Phase 1: design system and content (2 agents in parallel, split by layer)

**Design-system agent** (owns components, styles, layouts, page content files, page extractors):

1. Inventory every section class or block type found in the cache. Study its markup, CSS and live
   rendering.
2. Group them into a catalog of visual patterns and write it to a README in the sections folder: name,
   purpose, variants and props, source classes replaced, pages that use it. Several near-identical
   template sections usually collapse into one component with variants.
3. Implement each pattern as component + schema + extractor, following the reference section.
4. Generate every static page in every locale from the cache. Store sections repeated across pages
   once, as reusable blocks, and deduplicate them inside the generator so it stays reproducible.
5. Port only the interactions the live site has (tabs, carousels, video facades, animations), with
   minimal progressive JavaScript and `prefers-reduced-motion` respected.
6. Iterate with the visual diff on every static page at 1440 and 390 px (and a sample in every locale)
   until the remaining differences are unavoidable.

**Content agent** (owns content collections, their schemas, query helpers, collection export scripts):

1. Decide which CMS collections are actually rendered on the site; model only those, and report the rest.
2. Model them semantically: arrays instead of numbered fields (`faqs: [{question, answer}]`, not
   `faq_q1..faq_q12`), references by key, readable enum values, dates as dates, images as local assets
   with alt text, SEO fields matching what the live `<head>` serves.
3. Export every locale. If translations exist only in rendered pages (translation proxies), extract them
   from the cached HTML and align each item with its default-locale counterpart.
4. Write typed query helpers for templates (sorted, featured, by author or category, related items).
5. Write a verify script: item counts per collection and locale match live, references resolve, required
   SEO fields are present.

**Gate:** static pages within tolerance in the visual diff; the verify script passes; type check and
build green.

## Phase 2: templates and listings (2 agents in parallel, split by domain)

For example, an editorial agent (blog, knowledge base, changelog, authors, categories) and a catalog
agent (comparisons, alternatives, use cases, case studies). They own different template folders and
share one header component for article-like pages.

- Detail pages for every routed collection, through one dynamic route per folder (no duplicated
  template code per locale).
- Listing pages with the live filters, search and pagination; feeds (RSS) where live has them.
- Page-specific JSON-LD (Article, BlogPosting, FAQPage, BreadcrumbList, VideoObject, SoftwareApplication,
  ItemList...) built from typed helpers.

**Gate:** every live URL in the manifest is built or redirected; visual diff on every listing and at
least three entries per template, in every locale; build green.

## Phase 3: QA and hardening (1 agent)

1. Make the audits trustworthy first: remove noise from live-only artefacts (hidden elements, form
   success messages, obfuscated emails, placeholder table-of-contents headings).
2. Consolidate: components with the same look under different names, styles copied between components,
   client-side list logic implemented several times, misplaced or misnamed folders, unused components,
   exports, dictionary keys and assets. Every refactor needs pixel-identical before/after screenshots.
3. Run every audit on a fresh build over all live URLs; fix what is ours.
4. Visual diff at 1440, 768 and 390 px in every locale: every static page, every listing, three entries
   per template. Tablet width is where layout bugs hide.
5. Accessibility with axe (WCAG 2.1 AA) on two pages per template, every listing, home and pricing, in
   every locale. Report brand-colour contrast issues instead of changing the brand.
6. Performance with PageSpeed Insights or Lighthouse on the preview, before and after: LCP image priority,
   image sizes, layout shift, render-blocking resources, script weight, font preloading.
7. Hosting checks on the preview deployment: see `qa-tooling.md`.

**Gate:** every audit clean or each finding explained; the final deviations list and the flagged list
handed to the user.

## After phase 3

Apply the user's decisions on the flagged list, then follow `cutover.md`. If translations copied from a
proxy are poor, offer the add-on in `retranslation.md`.
