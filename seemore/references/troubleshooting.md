# When something goes wrong

seemore's failures are almost all content errors, and they name the file. Read the message, fix the file, re-run, then tell the user what was wrong in their terms. Don't relay a stack trace and wait for instructions.

## The dev server

**Nothing appears / "0 pages"**: `pageCount: 0` in the `--json` line means no Markdown was found where it looked. Check `contentRoot` in that same line against where the files actually are; the fix is usually `npx --yes seemore docs` rather than `npx --yes seemore`.

**"No Markdown files found under …"**: the same thing, as a warning. In dev it serves an empty site and recovers the moment a file appears, so it's not fatal.

**The URL doesn't load**: check the URL from the JSON line, not an assumed `:4040`. If the default port was taken, the server moved to the next free one and the JSON reflects that.

**A change doesn't show up**: the file is outside the served folder, matched by `exclude` in the config, or lost a duplicate-address collision. The server watches the content root only.

**"is not part of this site — excluded, or lost a duplicate slug"**: an inline edit was attempted on a file the site doesn't own. Same three causes as above.

## Build failures

The build is deliberately stricter than the preview: problems you can write past while editing must not ship.

**`Duplicate route /x produced by N files`**: two files slugify to the same address; the message lists every one of them. `Deep Dive.md` and `deep-dive.md` in one folder is the classic. Rename one, or exclude it with `exclude`. Ask before renaming a user's file.

**`Invalid frontmatter in <file>`**: names the file and the field. `title` must be a string, `order` a number, `draft` a boolean. `order: soon` and `title: 42` both fail. Unknown keys are fine and pass through, so this is always one of the five typed keys.

**`<file> failed to render`, with `` `<Nope>` is not one of the components seemore provides ``**: an MDX page used a component that doesn't exist. Only six do: `<Callout>`, `<Card>`/`<Cards>`, `<CodeBlockTabs>`, `<Mermaid>`, `<D2>`, `<Pdf>`. Rewrite it with one of those or with plain Markdown. `<Tabs>`, `<Accordions>` and `<Files>` look plausible and are not available.

**A component renders as literal text**: the file is `.md`, not `.mdx`. In plain Markdown a tag isn't JSX: it's dropped and its text kept. Rename the file.

**`No Markdown files found`**: fatal in a build, unlike in dev. Wrong folder, or everything got excluded.

**`title: required when a config file exists`**: a `seemore.config.ts` was created without a `title`. Add one; it's what names the site in the header.

**A dead `[[wikilink]]` or `.md` link**: a warning in dev, a failure in the build. That asymmetry is on purpose: you can write a link to a page you haven't created yet, and the build stops you shipping it. Either create the page or fix the link.

**`` `auth` is on, but SEEMORE_PASSWORD is not set ``**: ask the user for the password and pass it as an environment variable on the build command, or from a CI secret. Never write it into a file.

**A missing image is only a warning**: the page still builds, because a page with a broken image is visibly wrong on its own. Fix the path anyway.

## Config errors

**A rejected feature-flag combination**: two are refused at load time, with the fix in the message: `toc.integrate` with `toc.follow`, and `navigation.instant.preview` without `navigation.instant.prefetch`. Pick one side; don't try to route around it.

**`` `auth` cannot be combined with … ``**: password protection refuses `social.cards` and hosted search, which would publish page content outside the encrypted build. Ask the user which one they want.

**An unknown theme name**: `theme` must be one of the twelve presets. There is no custom theme name; custom styling goes in `css`.

**The config isn't being read**: it must be named `seemore.config.ts` (or `.mts`/`.js`/`.mjs`) and sit next to the content being served. If the content is in `docs/` and the config is at the repo root, either move it into `docs/` or pass `--config`.

## After publishing

**The live site has no styling, and links 404**: the `base` path. A GitHub Pages project site is served from `/<repo>/`, so it needs `base: '/<repo>/'`. This is the single most common publishing mistake, and it looks like a broken build rather than a config problem. See `references/publishing.md`.

**GitHub Pages shows a README instead of the site**: the repo's **Settings > Pages > Source** is still "Deploy from a branch" and needs to be **GitHub Actions**. That's a click only the user can make.

**"This browser can't open password-protected sites"**: the site isn't served over HTTPS, or the browser has service workers turned off (some private windows). Check the URL starts with `https://`.

**The site is stale**: outside GitHub Pages, nothing rebuilds on its own. A change means `npx --yes seemore build` and redeploying. Say so plainly rather than letting the user assume it's automatic.

## Environment

**Node too old or missing**: this surfaces as `npx --yes seemore` itself failing — `npx` or `node` not found, or an error about Node 20+. Check `node --version` only then; if it's older or missing, stop and point the user at https://nodejs.org. Don't try to install Node for them or guess at a version manager.

**`npx` prompts to install the package**: use `npx --yes seemore` so the install confirmation cannot block an agent-run command.

**Windows**: paths are handled, but if something looks path-shaped and wrong, say which path and on which command; don't paper over it.

## When you're actually stuck

Two failed attempts at the same thing is the limit. Then stop and tell the user: what you tried, what the error says, and the two options you see. Don't loop on a failing command or start rearranging their content to dodge an error you haven't explained.
