# QA tooling

Build these scripts in phase 0, before any page. They are how you and every subagent prove parity
instead of claiming it. A headless browser library (e.g. Playwright), an image diff library (e.g.
pixelmatch or sharp) and an accessibility engine (e.g. axe-core) cover all of them.

## Visual diff, live vs local

- Input: a list of paths, widths (default 1440 and 390; add 768 in phase 3), and optional selectors to
  compare one section (the live class vs the local id).
- Output: side-by-side tiles (live | local) you can open as images, plus a JSON summary per page with
  the mismatch percentage and the height delta per section.
- Freeze what moves before capturing: animations, carousels, videos, cookie banners, rotating content.
- Keep analytics off while capturing (see the guardrails in `SKILL.md`).

## Before/after pixel diff

- Capture local screenshots with animations frozen into a "before" folder, refactor, capture "after",
  and diff the two folders. A refactor is accepted only when every page is pixel-identical or each
  difference is explained.

## Audits on the build output

| Audit | Checks |
| --- | --- |
| SEO parity | Title, meta description, canonical, robots, hreflang set, Open Graph and Twitter tags, against the live page |
| Links | Every URL in the live manifest is built or redirected; no internal 404s; no links to the old platform's hosts; no cross-locale links |
| Content parity | Visible text of each built page against the live page, ignoring live-only noise |
| JSON-LD | Every graph parses, has no dangling `@id` references, uses absolute URLs, and has each type's required and recommended properties |
| Sitemaps | Sitemaps list exactly the indexable built pages, with reciprocal hreflang alternates; robots.txt and llms.txt are valid |
| Structure | Exactly one `<h1>` per page, every `<img>` has `alt`, landmarks present |
| Accessibility | axe on a sample of pages per template |
| Unused code | Components, exports, dictionary keys and assets that nothing references |

## Hosting checks on a deployment

Run against the preview URL after a push:

- Every redirect answers 301 with the right `Location`.
- Trailing-slash and `.html` variants redirect to the clean URL.
- Unknown URLs in every locale return the 404 page with status 404.
- Hashed assets carry an immutable cache header.
- robots.txt, sitemaps, llms.txt and feeds answer 200 with the right content types.
- Canonical and hreflang point at the production domain.
- Non-production hosts are `noindex`.

## Search-index check

Given a list of URLs a search engine knows (a Search Console export or an SEO tool), request each one on
the new site and report the ones that answer neither 200 nor 301. Run it on the preview before cutover and
on production right after.
