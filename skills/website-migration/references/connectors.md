# Finding connectors

Connectors (MCP servers, CLIs, APIs) appear and change every month. Search for the current ones; treat
the hints at the end as starting points to verify, not as facts.

## Search queries

Run these for the source platform and for the deployment platform (replace `<platform>`):

- `<platform> official MCP server`
- `<platform> MCP server site:github.com`
- `<platform> CLI` and `<platform> CLI export content`
- `<platform> CMS API export collections` (source) or `<platform> deploy static site CLI` (target)
- `<platform> 301 redirects export` (source)
- `<platform> API token scopes read only` (source)

Open the official docs page for every option you keep; a search snippet is not enough to know how it
authenticates or whether it is still maintained.

## Ranking

1. **Official MCP server** published by the platform: structured access, usually the least setup.
2. **Official CLI**: scriptable, re-runnable exports and deploys.
3. **Official REST/GraphQL API with a token**: write a small re-runnable export script.
4. **Community MCP server or CLI**: only if recently maintained and widely used; read its code paths
   that touch credentials before recommending it.
5. **Crawling the rendered site**: always possible and always needed for parity checks. For the source
   it is the fallback when no API exposes the content.

Prefer read-only scopes for the source platform. The migration never writes to it.

## What to present to the user

For each option you recommend: what it gives you, what to install, how to authenticate (the user runs
the login command or stores a token in a gitignored `.env`), the scopes needed, and how you will verify
the connection (a read-only list call). Recommend one option and say why.

## What APIs usually do not expose

Ask the user to export these from the platform's dashboard:

- The 301 redirect rules.
- Form destinations and integrations (CRM, newsletter, scheduling widgets).
- Tag manager, analytics and consent configuration.
- Translation-proxy settings (translated slugs, excluded pages).
- Drafts versus published state, when the API mixes them.

## Starting hints (verify before use)

Source platforms:

- **Webflow:** an official MCP server and a Data API (sites, pages, CMS collections and items) that
  works with a site token; redirects are a site setting.
- **WordPress:** the REST API at `/wp-json/wp/v2/`, WP-CLI on the server, and the built-in WXR export.
- **Ghost:** the Content and Admin APIs, and a JSON export from the admin panel.
- **Wix, Squarespace, Framer:** check what the platform's API or export currently allows; crawling may be
  the main source for page content.
- **Headless CMSs (Contentful, Sanity, Strapi, Storyblok...):** official APIs and CLIs with export
  commands; content may already be structured, so map it to typed collections directly.
- **Translation proxies (Weglot, Localize...):** translations usually exist only in the rendered pages;
  crawl each locale.

Deployment platforms:

- **Vercel, Netlify, Cloudflare:** each has an official CLI and, at the time of writing, MCP servers;
  check how each one expresses redirects, headers and preview deployments.
- **GitHub Pages, GitLab Pages:** deploy through CI; redirects need HTML or JavaScript fallbacks, so
  prefer a host with real 301s when the site has many legacy URLs.
- **Undecided:** produce host-agnostic static output and keep redirects as a data file that can be
  compiled into any host's format later.
