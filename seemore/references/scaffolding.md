# Starting a docs folder from nothing

There is no `seemore init`, and nothing needs one: seemore renders whatever Markdown it finds. "Scaffolding" here means writing a few sensible files, not generating a project. No `package.json`, no build config, no `docs/` convention to obey.

Reach for this only when the folder is empty or has nothing worth rendering. If Markdown already exists, preview it instead.

## The smallest thing that works

One file:

```
my-docs/
└── index.md
```

That is a working site. Everything below is what you add as the content grows, when there's a reason, not up front.

## A shape that scales

```
my-docs/
├── index.md              -> /            the home page
├── getting-started.md    -> /getting-started
├── guide/
│   ├── index.md          -> /guide       the section's landing page
│   └── writing.md        -> /guide/writing
├── meta.json             sidebar order for the root
└── seemore.config.ts     only if you need a title, theme, nav…
```

Keep the nesting shallow. Two levels is plenty for almost every site, and a deep tree makes the sidebar worse, not more organised.

## Write the home page first

Whatever else happens, something should claim `/`. A root `index.md` or `README.md` does it. With neither, seemore generates a card-grid index of every page. A fine placeholder, but it says nothing about what the site is.

```md
---
title: Home
description: What this is, in one line.
order: 1
---

# Project name

One paragraph on what this is and who it's for.

## Where to go next

- [[getting-started|Getting started]], to install it and see something work
- [[guide/writing|Writing pages]], to learn how to add to these docs
```

## Frontmatter

The settings block at the top of a page. Five keys are acted on; anything else passes through untouched, so content written for another tool still builds.

```md
---
title: Getting started
description: Install it and see something work.
icon: Rocket
order: 2
draft: false
---
```

| Key | Effect |
| --- | --- |
| `title` | Sidebar label, tab title, search result title. Falls back to the first heading. |
| `description` | Shown under the title and in search results. |
| `icon` | A [Lucide](https://lucide.dev) icon name for the sidebar. |
| `order` | Sidebar position, lower first. Second in precedence to `meta.json`. |
| `draft` | Kept in the local preview, dropped from the build. How you park an unfinished page. |

## Ordering the sidebar

Three mechanisms, in precedence order.

1. **`meta.json` in a directory**, an explicit list, where `"..."` stands in for everything you didn't name:

   ```json
   { "pages": ["index", "getting-started", "guide", "..."] }
   ```

   Use this for the root and for any section the user cares about the order of. It's the only mechanism that survives someone renaming a title.

2. **Frontmatter `order`**, lower numbers first. Good for a handful of pages; gets tedious past a dozen because inserting a page means renumbering.

3. **Alphabetical by title**, which is what happens to anything the first two don't cover.

Don't mix `meta.json` and `order` in the same directory. Pick one per directory so the next person can tell what's in charge.

## Naming files

The filename becomes the URL, so name for the address, not the title:

- Lowercase, hyphenated: `getting-started.md`, not `Getting Started.md`. Spaces work, and slugify to `/getting-started`, but the file is then awkward everywhere else.
- A section is a folder with an `index.md`, so `/guide` has a real page and isn't a dead sidebar heading.
- Two files that slugify to the same address is a build error naming both. `Deep Dive.md` and `deep-dive.md` in one folder collide.

## Seeding content the user actually wanted

The point of a scaffold is to hand back something real, not lorem ipsum. Write the starting pages from what the user told you they're documenting: their project's name, their actual sections. Three real pages beat ten placeholders, and a placeholder page nobody fills in is worse than no page.

If they came with material, whether pasted notes, a README or a pile of AI-written specs, that *is* the content. Put it into pages and give it an order; don't ask them to restate it.

## Then preview it

Stop scaffolding as soon as there's a home page and one real page, and start the dev server (`references/preview.md`). Everything after that is easier to judge on screen than in a file tree.
