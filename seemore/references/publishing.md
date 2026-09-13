# Building and publishing

```bash
npx --yes seemore build            # prerender the whole site into dist/
npx --yes seemore build docs       # build a subfolder
npx --yes seemore build --out site # somewhere other than dist/
```

`dist/` is plain web files, with no server and no Node on the host. Every page is prerendered to its own `index.html`, beside a `404.html` every static host honours, and the host-specific files are written for you: `_redirects` (Netlify, Cloudflare Pages), `200.html` (Surge), `.nojekyll` (GitHub Pages). So there is nothing to configure on the host beyond pointing it at the folder.

Build before offering to publish. A build failure is a content error that names its file (see `references/troubleshooting.md`), and it's much cheaper to fix locally than after a deploy.

## Get the base path right first

This is the one thing that silently produces a broken site: a page with no styling and no working links.

- Served from a domain root (`docs.example.com`, a Netlify or Surge subdomain, Cloudflare Pages): nothing to do.
- Served from a **subpath**, which GitHub Pages does by default (`username.github.io/my-repo/`): the site needs `base: '/my-repo/'` in `seemore.config.ts`, or `--base /my-repo/` on the build.

Set it deliberately, because it is never inferred. When the build runs inside GitHub Actions with no `base` set, it
prints the exact line to add (with the repo name filled in from the environment); a local build stays silent,
so don't wait for a warning that only appears in CI.

## Password protection

For a private site ("team only", "behind a password"), add `auth: true` to `seemore.config.ts`. The build encrypts the whole site and visitors unlock it in the browser, on any host below (HTTPS required).

```bash
SEEMORE_PASSWORD='a long passphrase' npx --yes seemore build
```

On Windows PowerShell:

```powershell
$env:SEEMORE_PASSWORD='a long passphrase'; npx --yes seemore build
```

- **The password comes from `SEEMORE_PASSWORD` only.** Never write it into the config, a workflow or any committed file. In CI, pass a secret on the build step (`SEEMORE_PASSWORD: ${{ secrets.SEEMORE_PASSWORD }}`); the user creates it in the repo's settings.
- **Ask the user for the password.** Don't invent one or repeat it back. Suggest a long passphrase: a short one can be guessed offline.
- Only the build is protected, never the preview.
- To remove someone's access, change the password and rebuild. Remind the user to share the password separately from the URL.

## The login reality

Some hosts need a one-time account login, and that step is **interactive**: Surge prompts for email and password, while Netlify, Cloudflare and Vercel open a browser. You cannot drive it from a tool call, and it will not work through the agent prompt's `!` shell either, because `!` runs a command but can't feed a multi-prompt interactive stdin, so `npx surge login` just hangs at `email:` with nowhere to type. GitHub Pages does not need a CLI login, but the user must select GitHub Actions as the repository's Pages source in Settings.

So the login happens in the **user's own terminal window**, meaning the real Terminal or iTerm app, not this session. Tell them the one command to run there, wait, and take the work back afterwards: the credential is saved to disk (`~/.netrc` for Surge, the CLI's own config for the others), and every later deploy reads it non-interactively.

This is the single exception to "you run the commands". Frame it that way to the user: one login, once, and then you handle the rest.

If they'd rather not, a token skips it entirely: Surge takes `SURGE_LOGIN` + `SURGE_TOKEN`, Netlify takes `--auth`, Cloudflare takes a `CLOUDFLARE_API_TOKEN` env var, Vercel takes `--token`. If they paste one, use it and skip the terminal step.

## Option A: GitHub Pages

The right default when the docs live in a GitHub repo: free, no extra account, and it redeploys itself on every push. No interactive login at all, because GitHub Actions is already authenticated.

1. Set `base: '/<repo-name>/'` in `seemore.config.ts`.
2. Add `.github/workflows/deploy-docs.yml`:

   ```yaml
   name: Deploy docs
   on:
     push:
       branches: [main]
   permissions:
     contents: read
     pages: write
     id-token: write
   concurrency:
     group: deploy-docs
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v5
         - uses: actions/setup-node@v5
           with:
             node-version: 22
         - run: npx --yes seemore build
         - uses: actions/configure-pages@v5
         - uses: actions/upload-pages-artifact@v3
           with:
             path: dist
     deploy:
       needs: build
       runs-on: ubuntu-latest
       environment:
         name: github-pages
       steps:
         - uses: actions/deploy-pages@v4
   ```

   Point `npx --yes seemore build` at the docs folder if they aren't at the repo root, and set `path:` to match `--out`.

3. In the repo's **Settings > Pages**, set **Source** to **GitHub Actions**. That is a click the user makes; it can't be done from the CLI.
4. Push, then give them `https://<user>.github.io/<repo>/`. First deploy takes a couple of minutes.

## Option B: Netlify

```bash
npx netlify deploy --dir=dist --prod
```

Check auth with `npx netlify status`; if it's not logged in, hand over `npx netlify login`. The first deploy asks to create or link a site; accept the create option. Prints a `https://….netlify.app` URL, and redeploying updates the same one.

## Option C: Cloudflare Pages

```bash
npx wrangler pages deploy dist --project-name=<name>
```

Check auth with `npx wrangler whoami`; if it's not logged in, hand over `npx wrangler login`, which opens a browser for the one-time login. Prints a `https://….pages.dev` URL. Good when the user already has a Cloudflare account.

## Option D: Surge

```bash
npx surge dist <name>.surge.sh
```

The fastest path to a URL, and the only one whose login is email and password rather than a browser. The user runs `npx surge login` once in their own terminal. Pass an explicit `<something>.surge.sh` domain or it prompts for one; re-deploying the same domain overwrites it, so the URL stays put. Leaves nothing behind in the folder.

## Option E: Vercel

```bash
npx vercel deploy dist --yes --prod
```

Check auth with `npx vercel whoami`; if it's not logged in, hand over `npx vercel login`, which opens a browser. Prints a production `https://….vercel.app` URL, and redeploying the same project updates the same one. Take it down later from the Vercel dashboard.

## After deploying

1. Open the live URL yourself and check it actually rendered: the stylesheet loaded, the sidebar is there, a link works. A wrong `base` looks exactly like an unstyled page.
2. Give the user the URL and one line about what to do next: it works on any device, share it anywhere.
3. Leave the link folder alone. Netlify and Vercel drop a `.netlify` or `.vercel` folder that ties the directory to the created site. It isn't junk: keeping it lets the next deploy reuse the same site and URL without relinking. Remove it only when this was a throwaway and the user won't redeploy.
4. Tell them how it updates. On GitHub Pages, pushing to `main` is the whole story. Elsewhere, say plainly that a change means rebuilding and redeploying, and offer to do it whenever they ask.

## Sharing one page instead of a site

When the user wants to send someone a single document, not host a site:

```bash
npx --yes seemore export docs/spec.md            # writes spec.html next to the Markdown
npx --yes seemore export docs/spec.md --out ~/Desktop
```

One self-contained HTML file, with styles inlined, images embedded and diagrams intact, that opens offline from a double-click, with no host and no link to manage. The same file the **Actions > Export as HTML** button produces. Exporting is refused if `pageActions` leaves out `'export-html'`.
