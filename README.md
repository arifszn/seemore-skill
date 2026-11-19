# seemore agent skill

An agent skill that turns a folder of Markdown into a documentation site with [seemore](https://github.com/arifszn/seemore). You describe what you want, and your AI agent runs everything.

**[seemore](https://github.com/arifszn/seemore)** · **[Docs](https://arifszn.github.io/seemore)** · **[Install](#installation)**

This repository holds the skill only. seemore itself is an [npm package](https://www.npmjs.com/package/seemore), which the skill runs with `npx` when it needs it, so there is nothing else to install first.

## How it works

1. **Point it at your Markdown.** Notes, specs, drafts, a README, or nothing at all: say what you want documented.
2. **It sets seemore up.** It starts a live preview in your browser, and creates starter pages if the folder is empty.
3. **You describe changes.** Add a page, fix the wording, reorder the sidebar, make it dark blue. The preview updates as it goes.
4. **You get a site.** Built to static files and published to a live URL, if you want one.

You run no commands yourself. Publishing can need a one-time login to the host or a change in your repository settings. The skill tells you exactly what to do, then takes the work back.

## What it does

- **Starts where your files are**: no project to create, no `docs/` layout to adopt, nothing moved or rewritten.
- **Runs a live preview**: pages, navigation and search update as files change, and you can double-click any paragraph to fix its text.
- **Writes real content**: pages built from what you gave it, not placeholders.
- **Knows the syntax**: `[[wikilinks]]`, Mermaid and D2 diagrams, alerts, code-block titles and diff markers, and which MDX components exist.
- **Publishes it**: to GitHub Pages, Netlify, Cloudflare Pages, Surge or Vercel, with the base path set correctly, and optionally behind a password.
- **Fixes its own errors**: build errors name a file, and the skill reads them instead of handing you a stack trace.

## Installation

### Claude Code

```bash
/plugin marketplace add arifszn/seemore-skill
```

```bash
/plugin install seemore
```

### Other agents

The skill works in any agent that supports the `SKILL.md` format, such as Codex, OpenCode, Antigravity, Gemini CLI and Cursor. Paste this to your agent:

```
Fetch and follow the install instructions from
https://raw.githubusercontent.com/arifszn/seemore-skill/refs/heads/main/INSTALL.md
```

Or install it yourself: [INSTALL.md](INSTALL.md) has the skills folder for each agent.

## Usage

Describe what you want in plain language:

```
Turn this folder of notes into a docs site I can read in my browser
```

```
Add a page about how deploys work, and put it after the getting-started page
```

```
Publish this to GitHub Pages behind a password
```

Or call the skill by name: `/seemore:seemore` in Claude Code, `seemore` in other agents.

## Requirements

[Node.js](https://nodejs.org) 22 or newer. If it is missing, the skill tells you, because installing Node is the one step it does not do for you.

## License

[MIT](LICENSE)
