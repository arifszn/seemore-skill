# When something goes wrong

seemore's failures are almost all content errors, and they name the file. Read the message, fix the file, re-run, then tell the user what was wrong in their terms — don't relay a stack trace and wait.

## The dev server

**Nothing appears / "0 pages"**: `pageCount: 0` in the `--json` line means no Markdown was found there. Check `contentRoot` against where the files actually are — usually `npx --yes seemore docs` instead of `npx --yes seemore`.

**"No Markdown files found under …"**: same thing, as a warning. Not fatal in dev — recovers the moment a file appears.

**The URL doesn't load**: check the URL from the JSON line, not an assumed `:4040`. Default port taken → server moved to the next free one.

**A change doesn't show up**: the file is outside the served folder, matched by `exclude`, or lost a duplicate-address collision. The server watches the content root only.

**A page in a dot folder or `build/`-like folder is missing**: those are skipped by default (see `references/configuration.md`). Add the folder to `include` — e.g. `include: ['.notes']` — rather than moving or renaming the user's files.

**"is not part of this site — excluded, or lost a duplicate slug"**: an inline edit was attempted on a file the site doesn't own. Same three causes as above.

## Build failures

Deliberately stricter than the preview — problems you can write past while editing must not ship.

**`Duplicate route /x produced by N files`**: two files slugify to the same address; the message lists all of them. `Deep Dive.md`/`deep-dive.md` is the classic. Rename one, or exclude it with `exclude`. Ask before renaming a user's file.

**`Invalid frontmatter in <file>`**: names the file and the field. `title` must be a string, `order` a number, `draft` a boolean. Unknown keys pass through fine — this is always one of the five typed keys.

**`<file> failed to render`, with `` `<Nope>` is not one of the components seemore provides ``**: an MDX page used a component that doesn't exist. Only six do: `<Callout>`, `<Card>`/`<Cards>`, `<CodeBlockTabs>`, `<Mermaid>`, `<D2>`, `<Pdf>`. Rewrite with one of those or with plain Markdown. `<Tabs>`, `<Accordions>` and `<Files>` look plausible and aren't available.

**A component renders as literal text**: the file is `.md`, not `.mdx`. In plain Markdown a tag isn't JSX — it's dropped, text kept. Rename the file.

**`No Markdown files found`**: fatal in a build, unlike dev. Wrong folder, or everything got excluded.

**`title: required when a config file exists`**: a `seemore.config.ts` was created without a `title`. Add one.

**A dead `[[wikilink]]` or `.md` link**: a warning in dev, a failure in the build — intentional, so you can write a link before the page exists but can't ship it. Create the page or fix the link.

**`` `auth` is on, but SEEMORE_PASSWORD is not set ``**: ask the user for the password and pass it as an env var on the build command, or from a CI secret. Never write it into a file.

**A missing image is only a warning**: the page still builds, because a broken image is visibly wrong on its own. Fix the path anyway.

## Config errors

**A rejected feature-flag combination**: `toc.integrate` with `toc.follow`, and `navigation.instant.preview` without `navigation.instant.prefetch` — both refused at load time, fix named in the message. Pick one side.

**`` `auth` cannot be combined with … ``**: password protection refuses `social.cards` and hosted search, which would publish content outside the encrypted build. Ask the user which one they want.

**An unknown theme name**: must be one of the twelve presets. No custom theme name — custom styling goes in `css`.

**The config isn't being read**: it must be named `seemore.config.ts` (or `.mts`/`.js`/`.mjs`) and sit next to the content being served. Move it, or pass `--config`.

## After publishing

**The live site has no styling, and links 404**: the `base` path. A GitHub Pages project site serves from `/<repo>/`, so it needs `base: '/<repo>/'`. The single most common publishing mistake — looks like a broken build, is a config problem. See `references/publishing.md`.

**GitHub Pages shows a README instead of the site**: **Settings > Pages > Source** is still "Deploy from a branch" — needs **GitHub Actions**. Only the user can click that.

**"This browser can't open password-protected sites"**: not served over HTTPS, or the browser has service workers off (some private windows). Check the URL starts with `https://`.

**The site is stale**: outside GitHub Pages, nothing rebuilds on its own. A change means `npx --yes seemore build` and redeploying — say so plainly.

## Environment

**Node too old or missing**: surfaces as `npx --yes seemore` itself failing — `npx`/`node` not found, or a Node 20+ error. Check `node --version` only then; if old or missing, point the user at https://nodejs.org. Don't try to install Node for them or guess at a version manager.

**`npx` prompts to install the package**: use `npx --yes seemore` so the install confirmation can't block an agent-run command.

**Windows**: paths are handled, but if something looks path-shaped and wrong, say which path and which command — don't paper over it.

## When you're actually stuck

Two failed attempts at the same thing is the limit. Then stop and tell the user: what you tried, what the error says, and the two options you see. Don't loop on a failing command or rearrange their content to dodge an unexplained error.
