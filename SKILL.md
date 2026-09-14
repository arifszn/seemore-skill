---
name: seemore
description: Turn a folder of Markdown into a real documentation site with seemore, and keep it running afterwards. Covers installing it, scaffolding pages, writing and editing content, starting a live preview, building the static site, and publishing it to a live URL. Use this skill whenever someone wants Markdown to be readable as a site instead of raw files. That includes when they ask for docs, documentation, a docs site, a handbook, a wiki, a knowledge base, release notes or a changelog; when they point at a folder of notes, specs, drafts or .md files and want it browsable, searchable, nicer-looking or shareable; when they want to preview, render, reorganise, reorder, retheme, restyle, build, deploy or publish Markdown; when they want to add, rewrite or reorder a page in a docs site that already exists; or when they mention seemore or npx seemore by name. Trigger it even when they never say 'docs site' and only describe the outcome in plain language.
---

# seemore

Turn a folder of Markdown into a documentation site with navigation, search, themes and diagrams, and publish it without the user touching a terminal.

## The rule that defines this skill

**You run the commands. The user describes what they want.** Never hand them a command to paste; never make progress conditional on them running one. Run it, fix what breaks, report in plain language what they can now see.

- **Say what happened, not what you typed.** "Your site is running at http://localhost:4040, with 12 pages" — not the command and its log.
- **Never move or rewrite their files to suit the tool.** seemore reads a folder where it already is. Don't reorganise notes or add unasked files (a config, a `dist/`) — suggest, let them say yes.
- **One exception: an interactive login when publishing.** See `seemore/references/publishing.md`.

## What seemore actually is

Points at a folder of `.md`/`.mdx` files and serves it as a site. No scaffold command, no project, no required config. `npx --yes seemore` in a folder of Markdown is a complete setup.

```
seemore [dir]           start the live dev server (default http://localhost:4040)
seemore build [dir]     build a static site into dist/
seemore export <file>   export one page as a standalone HTML file
```

Nothing is written into the user's folder by the dev server; nothing leaves their machine.

The correct answer is usually very little work. Markdown already exists → skip to the preview (Step 3). Don't create a `docs/` folder, config file, or package.json unasked.

## Step 1. Work out what they've got

If the request already says the Markdown is here ("this folder of notes", "my docs"), don't look: go straight to Step 3. The `pageCount` it reports is your check. Otherwise, take one quick look at the working directory for `.md`/`.mdx` files and `seemore.config.ts`.

| What you find | What to do |
| --- | --- |
| Markdown already there (notes, `docs/`, AI-written specs, a README) | Step 2, then preview. Common case. |
| Empty/near-empty folder, wants a docs site | Scaffold — `seemore/references/scaffolding.md` |
| Site already set up (`seemore.config.ts`, or "my docs site") | Jump to the step matching their ask |

Ask only when you genuinely can't tell what they want documented — one question, not a list. If Markdown exists in more than one plausible place, name the folders and let them pick.

## Step 2. Get seemore runnable

**No pre-flight checks.** Starting the preview (Step 3) is how you find out whether seemore runs. Don't check `node --version`, `npx --version` or `which`, and don't look at package-manager files first. The user may just be trying it out, so every command before the site is live is wasted time.

If the user has already asked for a specific runner, use it. If `npx --yes seemore` fails, retry with the project's apparent runner: `pnpm dlx seemore`, `yarn dlx seemore`, `bunx seemore`. Use whichever succeeds for the rest of the task. Only once the command has failed because `npx`/`node` is missing or too old (seemore needs Node 20+) should you run `node --version`. If Node is old or missing, stop and point the user at https://nodejs.org. That's the one install you can't do for them.

## Step 3. Start the preview

Get here before anything else — no questions asked first, nothing created first. The site should be live within seconds of the request.

The dev server is **long-running** and never exits — start it in the background with `--json --port 4040`. It prints one JSON line once listening (`url`, `port`, `contentRoot`, `pageCount`), then keeps running.

**Don't wait on that line alone** — some shells buffer output until the process exits, which a server never does. Use the **single command** in `seemore/references/preview.md`. It picks a free port, starts the server in the background, and waits for either the JSON line or the port answering, all in one shell call (it allows up to a minute for a first-run `npx` download). Don't split it into separate checks, and don't write your own wait. If it prints the server's error output instead of a URL, that's the Step 2 failure path.

Then:
1. Give the user the URL and page count. Open it in a browser if you can.
2. Once it's live (never before), check what's being served: list the `.md`/`.mdx` files under `contentRoot`, skipping `node_modules`. If some clearly aren't the user's notes, name them and offer to hide them with `exclude`. Typical examples are agent or tooling files (`.claude/`, `.github/`, `CLAUDE.md`, `AGENTS.md`) and vendored folders. Don't add the config until they say yes.
3. Tell them it's **live**: adding, renaming or deleting a file updates the site immediately, nav and search included.
4. Tell them they can **edit from the page**: double-click any paragraph, heading, list item, quote or table cell and that block's Markdown opens in place; **Save** writes it back to the file. Local preview only.

Subfolder: `npx --yes seemore docs`. Ports, LAN serving, one-server-per-project: `seemore/references/preview.md`.

## Step 4. Write and edit content

Read `seemore/references/content-authoring.md` before writing anything non-trivial. The parts that bite most often:

- **A page's address comes from its filename.** `getting-started.md` → `/getting-started`, `guide/index.md` → `/guide`. Renaming a file changes its URL.
- **Something should claim the home page.** Root `index.md` or `README.md` → `/`. With neither, seemore generates a card grid of every page.
- **Ordering: `meta.json` > frontmatter `order` (lower first) > alphabetical.** "Sidebar's in the wrong order" → `meta.json`.
- **Frontmatter keys seemore acts on**: `title`, `description`, `icon`, `order`, `draft`. Others pass through untouched.
- **Components need `.mdx`.** Only six exist: `<Callout>`, `<Card>`/`<Cards>`, `<CodeBlockTabs>`, `<Mermaid>`, `<D2>`, `<Pdf>`. Any other tag fails the build by name. In plain `.md` a tag isn't JSX; it's dropped and its text kept.
- **Write `[[wikilinks]]`, not relative paths**, between pages. `[[Page|label]]` and `[[Page#Heading]]` both work and survive a file moving.

With the preview running, every save is visible immediately — tell the user what to look at rather than describing it.

## Step 5. Configure it, only when asked

A folder with no config file builds correctly — never delay a first run for configuration. Never create `seemore.config.ts` on your own initiative, even for a good reason (agent-instruction files cluttering the page list, no site title). Fold the suggestion into the message reporting the live URL, and move on.

Create or edit the config only when the user asks for something it controls: a title, theme, nav link, footer, logo, "edit this page" links, excluding files, or pages in a folder skipped by default (dot folders, `build/`, …) that need `include`.

```ts
// seemore.config.ts
export default {
  title: 'My Docs',
  description: 'Everything about the thing.',
  theme: 'ocean',
};
```

`title` is **required as soon as a config file exists** — the build fails without it. Twelve built-in themes; full option list, feature flags, hosted search: `seemore/references/configuration.md`.

Translate, don't quiz. "Can it be dark blue?" → pick a `theme` and show them, don't ask about colour tokens.

Write only the keys that request needs. Every other option already has a working default, so the reference's full option list is not a template: no feature flags the user didn't ask for (`navigation.sections`, for one, changes the whole sidebar), no `exclude` entries for folders skipped anyway (`node_modules`, dot folders, `dist`, `build`, …), no explanatory comments.

## Step 6. Build the static site

Only when they want something to keep, host or hand over — never as a sanity check, never on a first run. Writes a `dist/` folder into theirs.

```bash
npx --yes seemore build
```

Prerenders every page into `dist/` as plain web files, plus host-specific files (`_redirects`, `200.html`, `.nojekyll`) and a `404.html`.

Build errors are content errors and name the file: duplicate address, unknown component, invalid frontmatter, dead link. Fix and rebuild — details in `seemore/references/troubleshooting.md`.

For a **single page**, `npx --yes seemore export docs/spec.md` writes one self-contained HTML file (styles inlined, images embedded, diagrams intact) — the right answer for "send this to someone without this folder".

## Step 7. Publish it

Offer once a build succeeds. Read `seemore/references/publishing.md` and **let the user pick the host** — GitHub Pages, Netlify, Cloudflare Pages, Surge, Vercel.

One silent trap: GitHub Pages serves from `username.github.io/my-repo/`, not root, so it needs `base: '/my-repo/'` in the config (or `--base` on the build). A local build won't warn you — the reminder only prints inside GitHub Actions — so set it when you set up the deploy, not after.

Publishing is the one place the user may need to act: some hosts need a one-time interactive login, and GitHub Pages needs GitHub Actions selected in repo settings. `seemore/references/publishing.md` covers handing those off cleanly and taking the work back.

Private site: `auth: true` password-protects the build via `SEEMORE_PASSWORD` — never in a file. Same reference file.

## Talking to the user

Their vocabulary: pages, sidebar, theme, link, publish. Not: content root, MDX compilation, prerender step, frontmatter.

- **Frontmatter** leaks because they'll see it in a file — call it "the settings block at the top of the page" once, then use their word.
- Report outcomes, not commands: what they can see, at what URL, what to try next.
- When something fails, say what broke and what you're doing about it, then do it.
- Offer the next move at each stop: preview it, add a page, change the look, publish it.

## Reference files

Read as you reach them, not all at once:

- `seemore/references/scaffolding.md` — starting from nothing (Step 1)
- `seemore/references/preview.md` — dev server, ports (Step 3)
- `seemore/references/content-authoring.md` — Markdown/MDX syntax, components, ordering, page addresses (Step 4)
- `seemore/references/configuration.md` — every config option, themes, feature flags, search (Step 5)
- `seemore/references/publishing.md` — building and deploying to a live URL (Steps 6–7)
- `seemore/references/troubleshooting.md` — the errors that actually come up, and their fixes (any step)
