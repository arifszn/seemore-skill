---
name: seemore
description: Turn a folder of Markdown into a real documentation site with seemore, and keep it running afterwards. Covers installing it, scaffolding pages, writing and editing content, starting a live preview, building the static site, and publishing it to a live URL. Use this skill whenever someone wants Markdown to be readable as a site instead of raw files. That includes when they ask for docs, documentation, a docs site, a handbook, a wiki, a knowledge base, release notes or a changelog; when they point at a folder of notes, specs, drafts or .md files and want it browsable, searchable, nicer-looking or shareable; when they want to preview, render, reorganise, reorder, retheme, restyle, build, deploy or publish Markdown; when they want to add, rewrite or reorder a page in a docs site that already exists; or when they mention seemore or npx seemore by name. Trigger it even when they never say 'docs site' and only describe the outcome in plain language.
---

# seemore

Turn a folder of Markdown into a real documentation site, with navigation, search, themes and diagrams, and publish it without the user ever opening a terminal.

## The rule that defines this skill

**You run the commands. The user describes what they want.** Someone reaching for this skill has told you, implicitly, that they don't want to install packages, learn flags, or debug a port conflict. So never hand them a command to paste, and never make progress conditional on them running one. Run it, read the output, fix what broke, and report in plain language what they can now see.

Three things follow from it:

- **Say what happened, not what you typed.** "Your site is running at http://localhost:4040, with 12 pages" beats pasting the command and its log. Keep flags, file paths and stack traces out of your messages unless the user asks, or you need their decision.
- **Never move or rewrite their files to suit the tool.** seemore reads a folder where it already is; that is the whole point of it. If the layout is awkward, say so and offer. Don't reorganise someone's notes unasked.
- **There is exactly one thing the user must sometimes do themselves, an interactive login when publishing.** That is the single documented exception, and `references/publishing.md` covers how to hand it over cleanly.

## What seemore actually is (so you don't over-build)

seemore points at a folder of `.md`/`.mdx` files and serves it as a site. There is **no scaffold command, no project to create, and no config file required**. `npx seemore` in a folder of Markdown is a complete, working setup. Three commands is the entire surface:

```
seemore [dir]           start the live dev server (default http://localhost:4040)
seemore build [dir]     build a static site into dist/
seemore export <file>   export one page as a standalone HTML file
```

Requires Node.js 20 or newer. Nothing is written into the user's folder by the dev server, and nothing leaves their machine.

Because of this, **the most common correct answer is very little work**. If the user already has Markdown, skip straight to the preview (Step 3). Don't create a `docs/` folder, a config file or a package.json that nobody asked for.

## Step 1. Work out what they've got

Look before you ask. Check the working directory for `.md`/`.mdx` files and a `seemore.config.ts`. Then place the request in one of three cases:

| What you find | What to do |
| --- | --- |
| **Markdown already there** (notes, a `docs/` folder, AI-written specs, a README) | Nothing to scaffold. Go to Step 2, then preview it. This is the common case. |
| **An empty or near-empty folder**, and the user wants a docs site | Scaffold a starting structure. See `references/scaffolding.md`. |
| **A seemore site already set up** (`seemore.config.ts` present, or they say "my docs site") | They want a change, not a setup: add a page, restyle, rebuild, republish. Jump to the step that matches. |

Ask a question only when you genuinely can't tell what they want documented, and ask one, not a list. If Markdown exists in more than one plausible place, name the folders you found and let them pick.

## Step 2. Get seemore runnable

`npx seemore` downloads and runs it on demand — there is nothing to install beforehand. Don't check Node up front; just run the command. If it fails (`npx` or `node` not found, or an error about needing Node 20+), check `node --version` then. If it's older or missing, stop and tell the user plainly that Node.js 20+ is needed, and point them at https://nodejs.org. That is an install you cannot do for them, and guessing at version managers wastes their time.

## Step 3. Start the preview

This is the moment the work becomes visible, so get here early, often before writing a single page.

The dev server is **long-running** and does not exit. Start it as a background process, never as a blocking call that hangs the session. Use the machine-readable flag so you don't have to screen-scrape coloured output:

```bash
npx seemore --json
```

It prints one JSON line once it's listening, then keeps running:

```json
{ "url": "http://localhost:4040/", "port": 4040, "contentRoot": "/Users/me/my-docs", "pageCount": 12 }
```

Read `url` and `pageCount` from it. Then:

1. Give the user the URL and the page count. Open it in a browser for them if you can.
2. Tell them the one thing that makes the preview worth keeping open: **it's live**. Adding, renaming, retitling or deleting a file updates the site immediately, navigation and search included, so they can leave it open while you both work.
3. Tell them they can **edit from the page itself**: double-click any paragraph, heading, list item, quote or table cell and that block's Markdown opens in place; **Save** writes it back to the file. It's the fastest way for a non-technical user to fix their own typo, and it only works in the local preview.

Point it at a subfolder when the Markdown lives deeper: `npx seemore docs`. `references/preview.md` covers the port already being in use, serving to another device, and keeping one server per project.

## Step 4. Write and edit content

Now the ordinary work: adding pages, fixing wording, restructuring. Read `references/content-authoring.md` before writing anything non-trivial. It has the syntax seemore adds on top of plain Markdown, and the small set of components that exist.

The parts that bite most often, so they're here rather than one file away:

- **A page's address comes from its filename.** `getting-started.md` becomes `/getting-started`, `guide/index.md` becomes `/guide`. Renaming a file changes its URL.
- **Something should claim the home page.** A root `index.md` or `README.md` becomes `/`. With neither, seemore generates a card grid of every page. That's fine as a starting point, but a real `index.md` is better once the site has a shape.
- **Ordering is explicit or alphabetical.** `meta.json` in a directory (`{ "pages": ["getting-started", "..."] }`) wins; then frontmatter `order`, lower first; then alphabetical by title. If the user says the sidebar is "in the wrong order", that's `meta.json`.
- **Frontmatter keys seemore acts on** are `title`, `description`, `icon`, `order` and `draft`. Other keys pass through untouched, so Markdown written for another tool still builds.
- **Components need the `.mdx` extension**, and only six exist: `<Callout>`, `<Card>`/`<Cards>`, `<CodeBlockTabs>`, `<Mermaid>`, `<D2>`, `<Pdf>`. Any other tag fails the build by name. In a plain `.md` file a tag isn't JSX at all; it's dropped and its text kept.
- **Write `[[wikilinks]]`, not relative paths**, when linking between pages. `[[Page|label]]` and `[[Page#Heading]]` both work, and neither breaks when a file moves.

With the preview running, every save is visible immediately, so make a change and then tell the user what to look at, rather than describing it.

## Step 5. Configure it, only if there's a reason

A folder with no config file builds correctly. Create `seemore.config.ts` next to the content when the user wants something it controls: a site title, a theme, a nav link, a footer, a logo, "edit this page" links, excluded drafts.

```ts
// seemore.config.ts
export default {
  title: 'My Docs',
  description: 'Everything about the thing.',
  theme: 'ocean',
};
```

`title` is **required as soon as a config file exists**. The build fails without it, because that's what names the site in the header. Twelve built-in themes are available; the full option list, feature flags and hosted-search setup are in `references/configuration.md`.

Translate, don't quiz. "Can it be dark blue?" is a `theme` choice you should just make and show them, not a question about colour tokens.

## Step 6. Build the static site

When they want something to keep, host or hand over:

```bash
npx seemore build
```

That prerenders every page into `dist/` as plain web files, with no server needed. Host-specific files (`_redirects`, `200.html`, `.nojekyll`) and a `404.html` are written for you.

Build errors are content errors, and they name the file: two pages claiming one address, an unknown component, invalid frontmatter, a dead link. Fix them and rebuild. Don't report a failed build to the user without having tried. `references/troubleshooting.md` has the specific messages.

To share a **single page** rather than a site, `npx seemore export docs/spec.md` writes one self-contained HTML file (styles inlined, images embedded, diagrams intact) that opens from a double-click. It's the right answer for "can you send this to someone who doesn't have this folder".

## Step 7. Publish it

Offer this once a build succeeds; it's usually what "I want a docs site" ultimately meant. Read `references/publishing.md` and **let the user pick the host**. GitHub Pages, Netlify, Cloudflare Pages and Surge are all covered there.

One trap worth carrying here, because it silently produces a site with no styling: on GitHub Pages the site lives at `username.github.io/my-repo/`, not at the root, so it needs `base: '/my-repo/'` in the config (or `--base /my-repo/` on the build). A local build won't warn you about it, since the reminder only prints when the build runs inside GitHub Actions, so set it when you set up the deploy, not after.

Publishing is also the one place the user may have to act: the hosts need a one-time interactive login that cannot be driven from a tool call. `references/publishing.md` explains how to hand that over and take the work back afterwards.

## Talking to the user

Their vocabulary is pages, sidebar, theme, link, publish. Yours should match:

- Say "your site", "a page", "the sidebar order", "publish it". Not "the content root", "MDX compilation", "the prerender step", "frontmatter".
- **Frontmatter** is the one internal term that leaks, because they'll see it if they open a file. Call it "the settings block at the top of the page" the first time, then use whatever they use.
- Report outcomes, not commands: what they can see, at what URL, and what to try next.
- When something fails, say what broke and what you're doing about it, then do it. Don't paste a stack trace and wait.
- Offer the obvious next move at each stop: preview it, add a page, change the look, publish it.

## Reference files

Read these as you reach them, not all at once:

- `references/scaffolding.md` for starting a docs folder from nothing (Step 1)
- `references/preview.md` for running the dev server and ports (Step 3)
- `references/content-authoring.md` for Markdown and MDX syntax, components, ordering, page addresses (Step 4)
- `references/configuration.md` for every `seemore.config.ts` option, themes, feature flags, search (Step 5)
- `references/publishing.md` for building and deploying to a live URL (Steps 6 and 7)
- `references/troubleshooting.md` for the errors that actually come up, and their fixes (any step)
