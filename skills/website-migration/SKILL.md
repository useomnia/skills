---
name: website-migration
description: Migrate a live website built on a hosted CMS or site builder (Webflow, WordPress, Framer, Wix, Squarespace, Ghost, a headless CMS...) into a git-based, statically generated Astro site with the same look, URLs and content, and a cleaner component architecture. Use when the user wants to move off a site builder, clone a site 1:1 into Astro, or rebuild a marketing site as a static site in git. Covers scoping, connecting to the source CMS and the deployment platform, git hosting, the Astro setup and plugins, phased delegation to subagents, visual and SEO verification, and cutover.
---

# Website migration

You are the lead engineer and orchestrator of a 1:1 migration of a live website into a statically
generated Astro site kept in git. You own the outcome end to end: scoping, architecture, delegation,
verification and reporting. After one round of questions you work autonomously, stopping only for the
items under "Ask before deciding".

The target is always Astro, built as static output; `references/astro.md` has the configuration,
integrations, project structure and version gotchas. The skill is agnostic of the source platform, the
deployment platform and the git host: detect what you can, ask about the rest, and look up connectors
instead of assuming them.

## Outcome

Hold every decision against these five goals. Restate them verbatim in every subagent brief.

1. **Parity.** Same look and feel, structure, URLs and content as the live site, in every locale it
   serves. The live site is the source of truth. Never invent content: every string, image and link
   comes from the live pages or the CMS export.
2. **Architecture.** Components are semantic and style-based: one component per visual pattern,
   variants via props. Site builders build components by content, so sections that look alike but hold
   different content were built twice; consolidate them. Name components by what they look like and
   do, never by the page or text they hold.
3. **Legibility.** Easy to reason about for agents and humans. Folders organised by what things are
   (primitives, layout, sections, templates, navigation, content), typed content separate from code,
   information-architecture best practices.
4. **SEO.** Canonical, hreflang with x-default, Open Graph, one JSON-LD graph per page generated at
   build time (never injected by client JavaScript), sitemaps, robots.txt, llms.txt, real 301s for every
   legacy URL, fast pages (responsive images, preloaded above-the-fold fonts, lazy third-party embeds).
   Check that every Astro integration you pick supports the installed Astro version.
5. **Nothing lost.** Every URL the live site or search engines know answers 200 or 301 on the new site.

## Step 1: discover before asking

Find out what you can without the user, so the interview only covers real decisions.

- **Source platform:** fetch the home page and read response headers and markup (`x-wf-*` headers or
  `data-wf-*` attributes mean Webflow; `wp-content` or `/wp-json` means WordPress; `generator` meta
  tags, `framerusercontent.com`, `static.wixstatic.com`, `squarespace-cdn.com`, Ghost's `/ghost/api`...).
  Note translation proxies (e.g. Weglot, Localize) and CDNs in front of the site.
- **Site size:** read `/sitemap.xml` (and per-locale sitemaps) to count pages per locale and template.
- **Local tooling:** check which CLIs are installed and authenticated: git hosts (`gh auth status`,
  `glab auth status`), deployment CLIs (`vercel whoami`, `netlify status`, `wrangler whoami`...), and
  the runtime (`node`, `pnpm`/`npm`, `python`).
- **Connectors already available to you:** list the MCP servers and tools in your session (CMS, hosting,
  docs/tracking tools such as Notion, Linear or GitHub Issues, analytics, SEO data).
- **Your own capabilities:** can you spawn subagents or run multi-agent workflows? If not, plan each
  phase as a fresh session with the same brief (see `references/subagent-brief.md`).

## Step 2: interview (one round)

Ask everything still open in a single round of multiple-choice questions, each with a recommended
option and a one-line reason. Use your structured-question tool if you have one. Skip anything the user
already stated. Typical questions:

| Decision | Recommended default |
| --- | --- |
| Deployment platform | Ask; offer the platforms whose CLI or connector you found first, plus "undecided" (host-agnostic static output, redirects kept as data) |
| Astro version | The latest stable release; check the integrations you need support it (`references/astro.md`) |
| Where CMS content lives | Astro content collections (Markdown/YAML with Zod schemas) in the repo; a one-time export, after which the source CMS is retired |
| Locales | Astro's native i18n routing: templates and components shared, content duplicated per locale as files, only UI microcopy in typed dictionaries |
| Styling | Tailwind CSS v4 through `@tailwindcss/vite`, with only the design tokens taken from the source CSS (default palette disabled); or tokens + scoped `<style>` in `.astro` components |
| Git hosting | See step 4 |
| Agents in parallel | Ask how many agents may run at the same time (recommend 2: one per layer or domain, so file ownership never overlaps), and whether there is a cap on the total |
| Tracking | Where to keep the per-phase task list and the "Flagged for owner" list (an existing doc tool, an issue tracker, or a `MIGRATION.md` in the repo) |
| Deliberate deviations | Fix obvious live bugs and list them; keep everything else exactly as live |
| Product naming | Whether there are naming rules or renamed products the copy must follow |

Record every answer where it survives the session (`AGENTS.md`, the tracking page, or your memory) so
it is never asked twice.

## Step 3: connect to the source CMS and the deployment platform

Connectors change often, so search the web for the current options instead of relying on memory.
`references/connectors.md` has the search queries, the evaluation rules and starting hints per platform.

1. Search for an official MCP server, an official CLI, and the export or content API of the source
   platform and of the chosen deployment platform.
2. Rank the options: official MCP or CLI first, then the official API with a token, then community tools
   (only if maintained and widely used), and finally crawling the rendered site, which always works and
   is required anyway for parity checks.
3. Present the options with the exact setup steps: what to install, how to authenticate, which
   permissions or scopes are needed (read-only for the source is enough).
4. **Never ask the user to paste secrets into the chat.** Have them run the login command themselves (in
   Claude Code: type `! <command>` in the prompt) or put tokens in a gitignored `.env` file.
5. Verify each connection with a read-only call (list sites, collections, or projects) before relying
   on it.

Ask the user to export what APIs rarely expose: the platform's 301 redirect list, form and integration
settings, and any analytics or tag-manager configuration. If they have Search Console (or another index
export), get the list of URLs Google knows; old URLs still need 301s.

## Step 4: git hosting

- If you are aware of the user having a GitHub account (for example `gh auth status` succeeds), create a
  GitHub repository, configure it as the `origin` remote and push the changes in meaningful commits.
  Confirm the owner (personal account or organisation), the name and the visibility in the interview;
  default to private.
- With another host (GitLab, Bitbucket, Gitea...), do the same through its CLI or API if it is
  authenticated; otherwise ask the user to create an empty repository and give you the remote URL.
- Connect the repository to the deployment platform so every push builds a preview, if the platform
  supports it. Never deploy to production or change DNS yourself.
- **Meaningful commits:** one commit per coherent change (foundation, a section family, a collection,
  an audit tool, a fix), with a message that says what changed and why. Commit and push after each
  milestone whose gates pass. Never amend, rebase or force-push pushed commits, and report whether each
  pushed commit builds on the deployment platform.

## Step 5: execute in phases

The phases, their deliverables and their gates are in `references/phases.md`. Summary:

| Phase | Who | Delivers |
| --- | --- | --- |
| 0. Capture and foundation | You, no subagents | Crawl cache of the live site, QA tools, the Astro project and its integrations, `AGENTS.md`, routing, i18n, layout, one reference section |
| 1. Design system and content | Agents in parallel, split by layer | Section catalog, every static page, typed content collections in every locale |
| 2. Templates and listings | Agents in parallel, split by domain | Every collection detail page and listing, feeds, page-specific JSON-LD |
| 3. QA and hardening | 1 agent | Component consolidation, full audits, visual, accessibility, performance, hosting checks |
| Cutover | The user, with you | Owner decisions applied, DNS switched, sitemaps submitted (`references/cutover.md`) |

Build the QA tools in phase 0, before any page; their specs are in `references/qa-tooling.md`. Set up
the Astro project as described in `references/astro.md` before the foundation.

## Delegation rules

- Brief every subagent using `references/subagent-brief.md`: the Outcome section verbatim,
  `AGENTS.md` as mandatory reading, its role, an explicit file-ownership list (and what is not
  theirs), ordered tasks, gates, and the report schema.
- Subagents never spawn subagents and never commit; you review and commit. Parallel agents share one
  dev server, and only one production build runs at a time.
- Never run more agents at once than the user allowed. Split parallel work so ownership never overlaps:
  by layer (UI vs data) or by domain (editorial vs catalog), never by page. With one agent at a time,
  run the roles of a phase one after another; with more than two, split by domain within each layer
  (e.g. one agent per collection family).
- Check every report against the disk (`git status`, `git diff`) before accepting it. Feed its open
  issues and notes into the next phase's brief, and add owner decisions to the flagged list.

## Verification standard

- "Done" means verified, not written. Say what you checked, how, and the result, and say what you did
  not check.
- Type check: `astro check` with 0 errors after every change set.
- Visual parity: a live-vs-local screenshot diff per page and section at 1440, 768 and 390 px, in every
  locale. Report each page outside tolerance with the reason.
- Refactors: before/after screenshots must be pixel-identical, or every difference explained.
- Every phase ends green: type check, a build with no missing-h1, missing-alt or broken-link warnings,
  and every audit clean or each finding explained.

## Ask before deciding

Keep the live behaviour and add these to the "Flagged for owner" list, unless the interview settled them:

- Bugs or typos on the live site (fix the obvious ones and list them).
- Translation quality problems in non-default locales (see `references/retranslation.md` for the add-on).
- Redirects the platform does not expose.
- Drafts that are still live, and pages not worth migrating (test, template or utility pages).
- Missing or placeholder alt text, cookie consent, and third-party widgets that only work on the
  production domain.
- Product naming and terminology.

## Guardrails

- Analytics, tag managers and session recording load only on the production host, plus an opt-in query
  flag for testing previews. Screenshot runs must never send traffic to production analytics.
- Never deploy to production, change DNS, or delete the source site, its CMS data or the capture cache.
- Keep secrets out of the repository and out of the chat.
- Code style: no unnecessary comments; document each module's purpose and complex function signatures;
  extract well-named functions instead of writing inline comments; match the surrounding code.

## Communication

- After each phase, report: what landed (with numbers), what is verified and how, deviations from live,
  the flagged list, and the pushed commit with its deployment status.
- When the user asks for status, answer with a short table per agent: done, in progress, blocked.
- Keep the tracking page current: tick tasks as they land and add new flagged items as they appear.
- Common mistakes to avoid are listed in `references/pitfalls.md`; read it before phase 0.
