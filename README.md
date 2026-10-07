# Omnia skills

Agent skills from the team at [Omnia](https://www.useomnia.com). Each skill is a folder under
[`skills/`](skills) with a `SKILL.md` and its reference files. They work in Claude Code, Codex, and any
agent that reads the `SKILL.md` format.

| Skill | What it does |
| --- | --- |
| [website-migration](skills/website-migration) | Migrates a site built on Webflow, WordPress, Framer or another hosted CMS into a static Astro site in git: same look, URLs and content, cleaner components, verified with visual and SEO audits |

## Install

**Claude Code**: add this repository as a plugin marketplace, then install the skills you want:

```
/plugin marketplace add useomnia/skills
/plugin install website-migration@useomnia
```

**Any agent, with the skills CLI**:

```
npx skills add useomnia/skills
```

**Manually**: copy a skill folder into your agent's skills directory, for example
`~/.claude/skills/website-migration` for Claude Code or `~/.codex/skills/website-migration` for Codex.

## website-migration

Ask your agent something like *"Migrate www.example.com from Webflow to Astro"*. The skill then:

1. **Discovers** the source platform, the site's size, your installed CLIs and connected MCP servers.
2. **Interviews you once** about what it cannot detect: deployment platform, git hosting, locales,
   styling, how many agents may run in parallel, and where to track the work.
3. **Finds connectors** for your CMS and your host (MCP servers, CLIs or APIs) by searching the web,
   and walks you through connecting them without pasting secrets into the chat.
4. **Sets up git**: if you have a GitHub account it creates the repository, sets it as `origin`, and
   pushes the work in meaningful commits.
5. **Captures the live site** and builds QA tools first: live-vs-local screenshot diffs, pixel diffs
   for refactors, and SEO, link, content, JSON-LD, sitemap and accessibility audits.
6. **Builds in phases**: foundation; then design system and content in parallel; then templates and
   listings; then QA and hardening. Each phase must pass its gates (a type check, a clean build and
   audits) before it is committed.
7. **Keeps you in charge** of the decisions that are yours. Live bugs, translation problems, missing
   redirects and naming questions go on a list for you instead of being decided silently.
8. **Prepares the cutover** with a checklist; DNS and production stay in your hands.

It works best with the most capable model at high reasoning effort, and with an agent that can run
subagents in parallel. Without subagents, it runs each phase as a fresh session. Expect one to two
days of mostly unattended work for a site of a few hundred pages.

The skill comes from our own migration of [useomnia.com](https://www.useomnia.com) from Webflow and
Weglot to Astro: about 800 pages in two languages and 18 content collections.

## Contributing

Issues and pull requests are welcome. A new skill goes in `skills/<name>/` with a `SKILL.md` whose
frontmatter has `name` and `description`, plus an entry in
[`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).

## License

[MIT](LICENSE)
