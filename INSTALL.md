# Installing the seemore skill

This repo holds the **seemore agent skill**: instructions that let an AI agent set seemore up and run it for you, so you never have to type a command yourself. It is not the seemore tool itself — that's [the npm package](https://www.npmjs.com/package/seemore), which the skill runs on demand with `npx`, so nothing is installed beforehand.

The skill is one self-contained folder, `seemore/`, with a `SKILL.md` at its root (its `name` matches the folder, as the spec requires). Every agent loads that same folder; only the parent *skills directory* changes from tool to tool. So installation is always the same three steps: **clone, copy the `seemore/` folder into your agent's skills directory, then clean up.**

---

## Claude Code

### Plugin marketplace (recommended)

1. Register the marketplace:

   ```bash
   /plugin marketplace add arifszn/seemore-skill
   ```

2. Install the plugin from it:

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

Notes:

- **OpenCode** also auto-discovers `~/.claude/skills` and `.claude/skills`, so a Claude install is picked up with no extra step.
- **Codex** scans `.agents/skills` from the current directory up to the repo root, so a project install at `.agents/skills` is visible across the whole repo.
- `.agents/skills` is the closest thing to a cross-agent standard; when in doubt, use it.

After copying, restart or reload your agent so it rescans its skills directory.

---

## Verify

Ask your agent **"what skills do you have?"** and `seemore` should appear in the list. If it does, it's loaded and ready. Then try:

```
Turn this folder of notes into a docs site I can read in my browser
```

You can also invoke it by name instead of waiting for it to trigger. In Claude Code that is `/seemore:seemore`; other agents use the skill's own name, `seemore`.

---

## Requirements

seemore runs on [Node.js](https://nodejs.org) 20 or newer. You won't run into this unless `npx seemore` fails; the skill then tells you plainly that Node is missing. Installing Node is the one step it can't do for you.
