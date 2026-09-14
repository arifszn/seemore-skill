# Configuration

Optional. A folder with no config file works in the browser, in a code editor and as a static build. Create `seemore.config.ts` next to the content only when something below is actually wanted.

`.mts`, `.js`, `.mjs` are also accepted, in that preference order. Use `.ts` unless the project has a reason not to.

## The whole option set

A reference, not a template. A real config sets only what the user asked for or the content needs — usually `title`, often `theme`, rarely more. Every option below already has a working default.

```ts
// seemore.config.ts
export default {
  title: 'My Docs',                    // required once this file exists
  description: 'Everything about the thing.',
  favicon: './favicon.svg',
  base: '/my-repo/',                   // subpath the site is served from
  theme: 'ocean',
  css: './custom.css',                 // appended last, so it wins
  features: { /* only flags being changed */ },
  nav: [{ text: 'GitHub', link: 'https://github.com/you/repo' }],
  footer: { text: '© 2026' },
  editLink: { base: 'https://github.com/you/repo/edit/main/docs' },
  search: 'static',
  pageActions: ['copy-markdown', 'export-html'],
  exclude: ['drafts/**'],
  include: ['.notes'],
  auth: true,                          // password-protect the build; password from SEEMORE_PASSWORD
};
```

**`title` is required as soon as the file exists** — the build fails without it, because that's what names the site in the header. Without a config file at all, the site is titled "Docs".

## Themes

`theme` picks one of twelve presets: `neutral` (default), `black`, `catppuccin`, `dusk`, `ocean`, `purple`, `ruby`, `solar`, `aspen`, `emerald`, `vitepress`, `shadcn`.

Each covers dark and light, follows the system setting, and has a toggle that remembers the user's choice. For a colour request, pick the nearest preset and show them rather than asking which of twelve. Anything a preset can't do goes in `css`, appended after everything else, so it wins without `!important`.

## Feature flags

A map of only the flags being changed, applied over the defaults:

```ts
features: { 'navigation.path': true, 'toc.integrate': true, 'toc.follow': false },
```

| Flag | Default | What it does |
| --- | --- | --- |
| `navigation.instant.prefetch` | on | Preloads a page on hover |
| `navigation.instant.preview` | off | Shows the target page in a popover on hover (needs prefetch) |
| `navigation.footer` | on | Previous/next links at the foot of a page |
| `navigation.top` | on | Top navigation bar |
| `navigation.path` | off | Breadcrumbs |
| `navigation.sections` | off | Groups the sidebar into sections |
| `navigation.prune` | off | Collapses sidebar branches you aren't in |
| `toc.follow` | on | Table of contents highlights as you scroll |
| `toc.integrate` | off | Folds the table of contents into the sidebar (conflicts with `toc.follow`) |
| `content.code.copy` | on | Copy button on code blocks |
| `content.action.edit` | follows `editLink` | "Edit this page" link |
| `content.edit` | on | Double-click inline editing, local preview only, never in a build |
| `content.image.zoom` | on | Click to zoom images |
| `search.suggest` | on | Inline completion in the search box |
| `search.highlight` | on | Carries the query onto the page you land on, so results are shareable |
| `social.cards` | off | Generates social preview images (not with `auth`) |

Two combinations are rejected at load time, with the fix named in the message: `toc.integrate` with `toc.follow` (no separate pane left to scroll), and `navigation.instant.preview` without `navigation.instant.prefetch` (nothing loaded to preview). Pick one side.

## Search

The default `search: 'static'` needs no setup, no server and no account: the index is built from the Markdown and queried in the browser in a Web Worker. Right for almost every site, including large ones.

Hosted indexes drop in when wanted, each needing only its SDK installed:

```ts
search: { provider: 'algolia', appId: '…', apiKey: '…', indexName: '…' },   // algoliasearch
search: { provider: 'orama-cloud', endpoint: '…', apiKey: '…' },            // @orama/core
```

Never put a user's API key into a config file about to be committed without saying so — these are publishable search-only keys by design, but that's the user's call to confirm, not yours to assume.

## Navigation, footer and edit links

```ts
nav: [
  { text: 'Docs', link: '/' },
  { text: 'Community', items: [{ text: 'Discord', link: 'https://…' }] },
],
footer: { text: '© 2026 Acme', links: [{ text: 'Privacy', link: '/privacy' }] },
editLink: { base: 'https://github.com/you/repo/edit/main/docs', text: 'Edit this page' },
```

`editLink.base` is the URL a page's path is appended to — point it at the folder the content lives in (`…/edit/main/docs` for docs in `docs/`). Setting it turns on the "Edit this page" link by itself.

## Page actions

`pageActions` decides what the **Actions** button above each page holds, in order. Both on by default; an empty array removes the button.

```ts
pageActions: ['copy-markdown', 'export-html'],   // the default
```

`copy-markdown` copies the page's source, useful for pasting into an AI chat. `export-html` writes one self-contained HTML file. Dropping `'export-html'` also makes the `seemore export` command refuse to run — intended, not a bug.

## Password protection

```ts
auth: true,                     // visitors stay unlocked a day after their last visit
auth: { remember: '7d' },       // '12h' or '7d'
auth: { id: 'acme-handbook' },  // stable name, so renaming the site keeps visitors unlocked
```

The password never goes in this file — see `references/publishing.md`. `auth` is refused together with `social.cards` or a hosted search provider.

## Excluding and including files

Some folders are skipped by default because they're almost never docs: dot folders (`.github`, `.notes`, …), `node_modules`, `dist`, `build`, `out`, `vendor`, `target`, `venv`, `deps`, `Pods`, `bower_components`.

```ts
exclude: ['drafts/**', '**/internal-*.md'],
include: ['.notes', 'build/reports/**'],
```

`exclude` skips more; `include` brings back something the defaults skip. Never repeat a default in `exclude` — `'**/node_modules/**'` there does nothing. Both take glob patterns relative to the content root; `include` also takes a plain folder name. `exclude` wins over `include`. For a single unfinished page, `draft: true` in its frontmatter is better — stays visible in the preview, drops out of the build.

The defaults only apply *inside* the content root. Serving a folder that itself sits in a dot folder (`npx --yes seemore .github/docs`) needs no config.

## The base path

`base` is the subpath the site is served from, and it is **never inferred**. A site at `username.github.io/my-repo/` needs `base: '/my-repo/'` or every asset link breaks. See `references/publishing.md` — the single most common publishing mistake.
