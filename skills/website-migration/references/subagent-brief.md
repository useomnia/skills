# Subagent brief

Every subagent gets the same preamble, then its own role. If your environment cannot spawn subagents,
run each role as a fresh session with this brief and the previous phase's reports pasted in.

## Template

```markdown
You are one of a small team of coding agents migrating {{SITE}} from {{SOURCE_PLATFORM}} to a static
{{FRAMEWORK}} site in {{REPO}}.
Read AGENTS.md first (its conventions are mandatory) and skim the foundation code it maps.

What the owner asked for (every decision must serve it):
{{the Outcome section of the skill, verbatim}}

Team rules:
- Do NOT spawn subagents. Do NOT commit.
- Edit only your ownership list: {{globs}}. Not yours: {{globs}}. If you must change another file,
  make the smallest additive change and report it.
- Another agent works in this folder at the same time. Reuse the shared dev server at {{URL}}; never
  start a second one. Check that no other build is running before you build.
- The live site and the capture cache ({{CACHE_PATH}}) are the source of truth. Never invent content.
- Verify with the visual diff tool ({{COMMAND}}). Use it a lot.
- Code style: no unnecessary comments; document module purpose and complex signatures; extract
  well-named functions instead of inline comments; match the surrounding code.

Your role: {{role}}

Tasks, in order:
{{numbered tasks}}

Gates: {{e.g. type check 0 errors; build passes; no missing-h1 or missing-alt warnings on your pages}}

When you finish, return the structured report. Be concrete and honest: what is verified and how, what
is not, and every deviation from the live site.
```

## Report schema

| Field | Content |
| --- | --- |
| `summary` | What was built, in a few sentences with numbers |
| `created` | Each file or folder created, with its purpose |
| `verification` | Commands run, pages compared, results |
| `deviationsFromLive` | Every known difference from the live site, with the reason |
| `openIssues` | What is unfinished or broken |
| `notesForNextPhase` | What the next agents must know |

If your environment supports structured output, enforce this schema.

## Writing good role briefs

- State the ownership list as globs, and list explicitly what belongs to the other agent.
- Order tasks so the agent produces the catalog or model first and the bulk work second.
- Name the out-of-scope items (things the next phase builds) so the agent leaves them alone.
- Include the decisions the user already made that touch this role.
- In later phases, list concrete candidates found in your own review (e.g. two components with the same
  name, two components painting the same texture) rather than asking for a generic cleanup.
