# Installing the seemore skill

This repo holds the **seemore agent skill**: instructions that let an AI agent set seemore up and run it for you, so you never type a command yourself. It is not the seemore tool itself — that's [the npm package](https://www.npmjs.com/package/seemore), which the skill runs on demand with `npx`, so nothing is installed beforehand.

The skill is one folder, `seemore/`, with a `SKILL.md` at its root (its `name` matches the folder, as the spec requires). Every agent loads that same folder; only the parent skills directory changes. Installation is always the same three steps: **clone, copy `seemore/` into your agent's skills directory, clean up.**

---

## Claude Code

### Plugin marketplace (recommended)

```bash
/plugin marketplace add arifszn/seemore-skill
```

```bash
/plugin install seemore
```

### Standalone skill

Not using the plugin? Use the template below.

---

## Every other agent: one template

Pick your agent's skills directory from the table, set it as `DEST`, and run:

```bash
DEST=~/.claude/skills   # ← change this line per the table below
git clone --depth 1 https://github.com/arifszn/seemore-skill /tmp/seemore-skill-repo
mkdir -p "$DEST"
cp -r /tmp/seemore-skill-repo/seemore "$DEST/seemore"
rm -rf /tmp/seemore-skill-repo
```

| Agent | Global / user `DEST` | Project / workspace `DEST` |
| --- | --- | --- |
| **Claude Code** | `~/.claude/skills` | `.claude/skills` |
| **OpenCode** | `~/.config/opencode/skills` | `.opencode/skills` |
| **Codex** | `~/.agents/skills` | `.agents/skills` |
| **Antigravity** | `~/.gemini/config/skills` | `.agents/skills` |
| **Gemini CLI / Cursor / Windsurf / Kimi / others** | see your agent's skills docs | `.agents/skills` |

- **OpenCode** also auto-discovers `~/.claude/skills` and `.claude/skills` — a Claude install is picked up free.
- **Codex** scans `.agents/skills` from the current directory up to the repo root, so a project install is visible repo-wide.
- `.agents/skills` is the closest thing to a cross-agent standard — use it when in doubt.

Restart or reload your agent after copying so it rescans its skills directory.

---

## Verify

Ask your agent **"what skills do you have?"** — `seemore` should appear in the list. Then try:

```
Turn this folder of notes into a docs site I can read in my browser
```

You can also invoke it by name: `/seemore:seemore` in Claude Code, or just `seemore` elsewhere.

---

## Requirements

seemore runs on [Node.js](https://nodejs.org) 20 or newer. If `npx --yes seemore` fails, the skill tells you plainly that Node is missing — installing it is the one step it can't do for you.
