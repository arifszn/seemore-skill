# Running the live preview

`seemore` with no command starts a dev server that watches the folder and updates the browser as files change. It is the centre of the workflow: get it running early and leave it running.

```bash
npx --yes seemore            # serve the current folder
npx --yes seemore docs       # serve a subfolder
```

The site comes up at `http://localhost:4040` by default.

## Start it in the background, and read the JSON

The server does not exit, so a plain foreground call will hang the session. Start it as a background process and use `--json` so you never have to parse coloured terminal output:

```bash
npx --yes seemore --json
```

One line is printed once it's listening, then the process stays up:

```json
{ "url": "http://localhost:4040/", "port": 4040, "contentRoot": "/Users/me/my-docs", "pageCount": 12 }
```

- `url`: give this to the user verbatim, and open it for them if you can.
- `pageCount`: a cheap sanity check. `0` means it found no Markdown, so it's the wrong folder, or the files are one level down.
- `contentRoot`: the folder actually being served. Check this when the user says pages are missing.

## When the JSON line never shows up

Some shells wrap commands in an output filter that holds all output back until the process exits. A dev server never exits, so the JSON line can fail to arrive even though the site is already being served. Don't sit in a loop waiting for it:

- Check the port is free **before** starting (`curl -s -o /dev/null localhost:4040` should fail), and pass that port explicitly with `--port`. A taken port makes the server quietly move to the next one, even with `--port`, and you'd be checking the wrong one.
- Then poll for either the JSON line or the port answering, whichever comes first. Allow up to a minute on a first run, while `npx` downloads.
- If only the port answered, the URL is `http://localhost:<port>/`. There's no `pageCount`, so sanity-check against the Markdown files you can see in the folder, and open the page to confirm it isn't empty.

## Flags worth knowing

| Flag | Use it for |
| --- | --- |
| `--port 5050` | The default port is taken, or the user wants a stable one per project. |
| `--host` | Serve to another device on the network, such as a phone, a tablet or someone else's laptop. Prints a LAN address as well. |
| `--open` | Open a browser on start. Off by default; prefer opening the URL yourself so you stay in control of which browser. |
| `--config <path>` | The config file isn't next to the content. |
| `--base /my-repo/` | Preview exactly as it will be served from a subpath. Rarely needed in dev. |

## One server per project

Don't start a second server for the same folder. The first one is already live, and a second just confuses which URL is current. If you've lost track of whether one is running, check the port rather than starting another.

If the port is in use by something else, seemore's underlying dev server moves to the next free port and the `url` in the JSON line reflects that. Always use the URL from the JSON, never an assumed `:4040`.

## What "live" covers

Worth telling the user, because it's more than hot-reloading text:

- Adding, deleting, renaming or retitling a file updates the sidebar, the navigation and the search index immediately.
- Editing a page's body updates it in place without losing scroll position.
- A content error that would fail a build (a dead link, a duplicate address) is a **warning** in dev, not a crash, so a half-finished edit doesn't take the site down. Catch them here, in the warnings — don't run `seemore build` to check for problems: a build writes `dist/` into the user's folder, which they didn't ask for.
- `draft: true` pages are visible in the preview and excluded from the build.

## Editing from the page

The preview is also an editor, and for a non-technical user it's often the best way to fix their own text: double-click any paragraph, heading, list item, quote or table cell and that block's **Markdown source** opens in place, so `**bold**` stays `**bold**` and tables stay tables. **Save** writes it back to the file on disk, leaving everything around it untouched.

Mention this once, early. It's local-preview only, never in a build, and it can be switched off with `'content.edit': false` if the user doesn't want it.

## Nothing is written to their folder

The dev server writes nothing into the content folder: seemore's own app is the Vite root and caches go to the OS temp directory. Say so if the user is nervous about pointing a tool at their notes. Files change only when they save an inline edit, or when you run `build` or `export`.

## Stopping it

Stop the background process when the work is done, and say you've done it. Leaving a stray server on a port the user doesn't know about is the kind of mess they can't debug themselves.
