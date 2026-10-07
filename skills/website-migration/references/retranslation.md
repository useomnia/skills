# Add-on: rewriting a locale from the source language

Translations captured from a translation proxy or machine translation are often poor. Once the
migration is done, offer to rewrite them. Run it as its own project, with its own agent budget.

## Decisions to ask first

- The source-of-truth locale and the target locales.
- Register and variety (e.g. Spain Spanish with informal "tú"; Brazilian or European Portuguese).
- Whether slugs and collection folders are translated, and old URLs 301.
- How many agents may run in parallel: one translator and one native editor per batch of files works
  well, with as many batches at once as the user allows.
- Product entities and doubtful terms: ask in batches of up to four questions, each with a recommended
  rendering. Never decide a product name alone.

## Infrastructure

1. A **glossary** per locale: term, kind (brand, product, plan, feature, metric, concept, UI, third
   party), definition, rendering, notes (gender, article, capitalisation, first-mention gloss) and an
   avoid-list of wrong renderings that a whole-word search can flag safely.
2. A **status manifest** per file and locale: current, outdated, redo, missing, with the source revision
   each translation was made from.
3. A **check script** that compares a translation with its source: identical structure (keys, list
   lengths, section types), markup, numbers, links, and no glossary avoid-list hits.
4. A **link localizer** that rewrites internal links to the target locale's URLs, and a slug-redirect
   generator that adds 301s when a published slug changes.
5. A **content-creation skill** (or an `AGENTS.md` section) that every future content change follows:
   write the source first, resolve its terms against the glossary, translate into every locale, run the
   checks, stamp the status.

## The pipeline

1. Extract candidate terms from the source corpus; resolve the doubtful ones with the user; write the
   glossary.
2. Mark every target file as redo.
3. Per batch: a translator writes each file from the source only. Never reuse the old translation's
   wording, or the agent will polish the machine translation instead of rewriting it. Then a native editor
   reviews it sentence by sentence: accuracy, register, idiom, glossary, typography, SEO title and
   description lengths. Both run the check script and report new terms.
4. Localize links, generate slug redirects, build, run the link and sitemap audits.
5. Commit and push per batch; stamp the translated files as current.
