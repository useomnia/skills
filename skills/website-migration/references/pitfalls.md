# Pitfalls

Read before phase 0. Each one cost real time in a migration this skill is based on.

- **Analytics leak.** Screenshot runs against localhost loaded the production analytics tags and recorded
  pageviews. Gate every tag by the production hostname from the first commit.
- **Ambiguous agent budget.** "At most 5 agents" can mean total or concurrent. Ask, and state your
  reading when you start.
- **Progress claims without changes.** An agent reported work that was not on disk. Check `git status`
  and `git diff` before accepting any report.
- **Framework cache quirks.** Content schemas kept outside the framework's main config were not
  invalidated after edits, and the dev server crashed on stale data. Record every such gotcha in
  `AGENTS.md` and, where possible, guard against it in code.
- **Platform limits that look like content.** A builder's list showed at most 100 items, so the live
  index looked shorter than the real collection. Decide with the user whether to keep or lift such limits,
  and list the decision as a deviation.
- **Heavy images.** Exported originals were about 400 MB, mostly PNG screenshots. Converting them to WebP
  cut that to under 140 MB with no visible loss.
- **Placeholder alt text.** CMS images carried placeholder alt values. Migrate them empty and flag them
  for real descriptions; never ship the placeholder.
- **Third-party embeds.** Some iframes only size themselves on the production domain, so the visual diff
  reports noise there. Note it once; don't chase it.
- **Markdown and embeds.** Raw `<script>` and `<style>` blocks inside Markdown may be rewritten by the
  framework's Markdown pipeline. Store interactive widgets as separate files referenced by key, and keep
  them out of the formatter.
- **Whitespace in templates.** Some compilers follow JSX whitespace rules, where a newline between inline
  elements renders no space. Check the framework's rules before porting inline markup.
- **Live bugs copied verbatim.** Copying a broken layout or a typo is not parity the user wants. Fix the
  obvious ones and list them; flag the rest.
- **Rules arriving mid-run.** Each new rule forces a re-plan. Ask everything in the first interview.
