# seemore skill

AI agent skill to turn a folder of Markdown into a documentation site with [seemore](https://github.com/arifszn/seemore). You describe what you want; your agent runs everything.

**[seemore →](https://github.com/arifszn/seemore)** · **[Docs →](https://arifszn.github.io/seemore)**

## How it works

1. **Point it at your Markdown.** Notes, specs, drafts, a README — or nothing at all, and say what you want documented.
2. **It sets seemore up.** Installs what's needed, scaffolds pages if the folder is empty, starts a live preview in your browser.
3. **You describe changes.** Add a page, fix the wording, reorder the sidebar, make it dark blue. The preview updates as it goes.
4. **You get a site.** Built to static files and published to a live URL, if that's what you want.

No terminal commands to run yourself. Publishing may need a one-time login or a repository-settings change, which the skill hands over cleanly and then takes back.

## Features

- **Starts where your files are**: no project to create, no `docs/` layout to adopt, nothing moved or rewritten.
- **Live preview**: pages, navigation and search update as files change; double-click any paragraph to fix its text.
- **Writes real content**: pages seeded from what you actually gave it, not placeholders.
- **Knows the syntax**: `[[wikilinks]]`, Mermaid and D2 diagrams, admonitions, code-block titles and diff markers, the six components that exist and the ones that only look like they do.
- **Publishes it**: GitHub Pages, Netlify, Cloudflare Pages, Surge or Vercel, base-path trap handled, optionally behind a password.
- **Fixes its own errors**: build failures name a file, and the skill reads them instead of handing you a stack trace.

## Installation

### Claude Code (Plugin Marketplace)

```bash
/plugin marketplace add arifszn/seemore-skill
```

```bash
/plugin install seemore
```

### Other agents (Codex, OpenCode, Antigravity, Gemini, Cursor, …)

Standalone skill, works in any agent supporting the `SKILL.md` format. See **[INSTALL.md](INSTALL.md)** for per-agent skills paths.

Or tell your agent to install it by pasting this:

```
Fetch and follow the install instructions from
https://raw.githubusercontent.com/arifszn/seemore-skill/refs/heads/main/INSTALL.md
```

## Usage

```
Turn this folder of notes into a docs site I can read in my browser
Add a page about how deploys work, and put it after the getting-started page
```

Or invoke by name: `/seemore`

## Requirements

[Node.js](https://nodejs.org) 20 or newer. The skill checks and tells you if it's missing — installing Node is the one step it can't do for you.

## License

[MIT](LICENSE)
