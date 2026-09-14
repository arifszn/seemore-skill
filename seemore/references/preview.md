# Running the live preview

`seemore` with no command starts a dev server that watches the folder and updates the browser as files change. It's the centre of the workflow — get it running early and leave it running.

```bash
npx --yes seemore            # serve the current folder
npx --yes seemore docs       # serve a subfolder
```

Comes up at `http://localhost:4040` by default.

## Start it in the background, and read the JSON

The server does not exit, so a plain foreground call hangs the session. Start it in the background and use `--json` so you never parse coloured terminal output:

```bash
npx --yes seemore --json
```

One line is printed once it's listening, then the process stays up:

```json
{ "url": "http://localhost:4040/", "port": 4040, "contentRoot": "/Users/me/my-docs", "pageCount": 12 }
```

- `url`: give this to the user verbatim, open it for them if you can.
- `pageCount`: a cheap sanity check. `0` means no Markdown found — wrong folder, or files one level down.
- `contentRoot`: the folder actually being served. Check this when the user says pages are missing.

## Start it with one command

Run this as one shell call. Don't check Node, the port or the folder first, because this command covers all of them:

```bash
PORT=4040; while curl -s -o /dev/null "localhost:$PORT"; do PORT=$((PORT+1)); done
LOG="${TMPDIR:-/tmp}/seemore-$PORT.log"
nohup npx --yes seemore --json --port "$PORT" >"$LOG" 2>&1 &
PID=$!
for i in $(seq 60); do
  grep -m1 '"url"' "$LOG" && break
  curl -s -o /dev/null "localhost:$PORT" && { echo "{\"url\":\"http://localhost:$PORT/\"}"; break; }
  kill -0 "$PID" 2>/dev/null || { echo "seemore exited:"; cat "$LOG"; break; }
  sleep 1
done
```

(Add `docs` after `seemore` for a subfolder. On Windows without a POSIX shell, follow the same steps in PowerShell.)

Why it's shaped this way:

- **Free port first, passed explicitly.** A taken port makes the server quietly move to the next one even with `--port` set, so you'd poll the wrong port.
- **JSON line or port, whichever comes first.** Some shells buffer output until the process exits, which a dev server never does, so the JSON line may never appear.
- **It stops if the process dies.** Then the log is printed, and that's your failure. Only now is it worth looking at the cause, such as `node`/`npx` missing or Node older than 20 (see `troubleshooting.md`).
- **Only the port answered?** The URL is `http://localhost:<port>/`, but there's no `pageCount`. Open the page to confirm it isn't empty.

## Flags worth knowing

| Flag | Use it for |
| --- | --- |
| `--port 5050` | The default port is taken, or the user wants a stable one per project. |
| `--host` | Serve to another device on the network — a phone, a tablet, someone else's laptop. Prints a LAN address too. |
| `--open` | Open a browser on start. Off by default; prefer opening the URL yourself. |
| `--config <path>` | The config file isn't next to the content. |
| `--base /my-repo/` | Preview exactly as it will be served from a subpath. Rarely needed in dev. |

## One server per project

Don't start a second server for the same folder — the first is already live, and a second just confuses which URL is current. Check the port rather than starting another if you've lost track.

If the port's in use by something else, seemore moves to the next free port and the `url` in the JSON reflects that. Always use the URL from the JSON, never an assumed `:4040`.

## What "live" covers

Worth telling the user — it's more than hot-reloading text:

- Adding, deleting, renaming or retitling a file updates the sidebar, navigation and search index immediately.
- Editing a page's body updates it in place without losing scroll position.
- A content error that would fail a build (a dead link, a duplicate address) is a **warning** in dev, not a crash, so a half-finished edit doesn't take the site down. Catch them here — don't run `seemore build` just to check: a build writes `dist/` into the user's folder, unasked.
- `draft: true` pages are visible in the preview and excluded from the build.

## Editing from the page

The preview is also an editor: double-click any paragraph, heading, list item, quote or table cell and that block's **Markdown source** opens in place, so `**bold**` stays `**bold**` and tables stay tables. **Save** writes it back to the file, leaving everything around it untouched.

Mention this once, early — it's often the best way for a non-technical user to fix their own text. Local-preview only, never in a build; switch off with `'content.edit': false` if unwanted.

## Nothing is written to their folder

The dev server writes nothing into the content folder: seemore's own app is the Vite root, and caches go to the OS temp directory. Say so if the user's nervous about pointing a tool at their notes. Files change only from an inline-edit save, or from `build`/`export`.

## Stopping it

Stop the background process when the work is done, and say you've done it. A stray server on a port the user doesn't know about is a mess they can't debug themselves.
