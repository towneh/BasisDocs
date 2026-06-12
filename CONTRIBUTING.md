# Contributing to BasisDocs

BasisDocs is the documentation site at **https://docs.basisvr.org**. It is a
[Next.js](https://nextjs.org) app built with [Fumadocs](https://fumadocs.dev):
pages are written in **MDX** (Markdown plus React components) and the navigation
is driven by small `meta.json` files. This guide explains how to add and edit
documentation in the format the site expects.

## Local development

```bash
npm install        # install dependencies (first time only)
npm run dev        # start the dev server at http://localhost:3000
npm run build      # production build — run this before opening a PR
```

`npm run dev` hot-reloads as you edit. Always run `npm run build` before
submitting: a broken MDX file (a bad import, a stray `<` or `{`, unknown
frontmatter) fails the build, and the same build runs in CI on every push to
`main`.

## Repository layout

| Path | What lives there |
| --- | --- |
| `content/docs/` | All documentation pages (`.mdx`) and navigation (`meta.json`) |
| `public/img/` | Images, grouped into a folder per documentation section |
| `src/` | Site code — layout, i18n config, search. Rarely touched for content work |
| `source.config.ts` | Fumadocs config: frontmatter schema, code themes, languages |
| `.github/workflows/deploy.yml` | Builds and deploys the site to GitHub Pages |

Documentation is organised into sections, each its own folder under
`content/docs/` — for example `content/docs/world/`, `content/docs/avatar/`,
`content/docs/server/`. A section folder contains:

- `index.mdx` — the section landing page.
- One `.mdx` file per page (the file name is the page's URL slug).
- `meta.json` — the section's title, icon, and page order.

## Anatomy of a page

Every page is an `.mdx` file that starts with a YAML **frontmatter** block:

```mdx
---
title: Media Player
description: One sentence describing the page, used for search and link cards.
---

import { Callout } from 'fumadocs-ui/components/callout';

## First heading

Page content here…
```

- **`title`** (required) — the page title shown in the sidebar and at the top.
- **`description`** (optional but recommended) — a one-line summary used by
  search and by `<Card>` link previews.

After the frontmatter, `import` any components you use, then write the body.
Don't add a top-level `# H1`; the `title` provides it. Start the body at `##`.

## Navigation: `meta.json`

Each section folder has a `meta.json` that controls its sidebar entry and the
order of its pages:

```json
{
  "title": "World",
  "icon": "Globe",
  "pages": ["index", "setup", "building", "testing", "scene-props", "media-player"]
}
```

- **`title`** — the section name in the sidebar.
- **`icon`** — a [Lucide](https://lucide.dev/icons) icon name (optional).
- **`pages`** — page slugs in display order. **A new page must be added here**
  or it won't appear in the sidebar.

The top-level `content/docs/meta.json` arranges the sections themselves and
supports two extra forms: `"---Label---"` inserts a sidebar group heading, and
`"...folder"` auto-includes every page in a folder.

## Images

Put images under `public/img/<section>/` and reference them with a root-relative
path (the leading `/img` maps to `public/img`):

```mdx
![Inspector showing the Basis Media Player component](/img/worlds/media-player-inspector.png)
```

Section image folders are named loosely and don't always match the docs folder
name exactly — for example the `world` docs section stores its images under
`public/img/worlds/`. When adding a new section, pick a clear, lowercase folder
name and keep all of that section's images in it. Prefer reasonably sized PNGs
or WebP; very large screenshots slow the page down.

## Localisation

The site ships in **eleven languages**. English is the default and is the only
version you need to write — translations are added separately.

- **Default language: `en`.** English files have **no locale suffix**:
  `setup.mdx`, `meta.json`.
- **Translations** add the locale code before the extension:
  `setup.de.mdx`, `meta.de.json`.
- Supported locales: `en` (default), plus `fr`, `de`, `es`, `ja`, `ru`, `cn`
  (Simplified Chinese), `tw` (Traditional Chinese), `nl`, `it`, `ko`.

When you add or change a page, **edit the English (`.mdx`) version**. If a
translation for a page is missing, the site falls back to English, so a new
English-only page works immediately; the localised versions can follow later.

Translated `meta.<locale>.json` files localise the section **`title`** but keep
the same `icon` and `pages` array as the English `meta.json`. If you reorder or
add pages, mirror the `pages` change into the locale meta files too (the page
slugs themselves are never translated).

### Internal links

Link between pages with a **locale-prefixed** absolute path under `/docs`:

```mdx
See [UI Canvas](/en/docs/world/ui-canvas) for the canvas setup.
```

Use the `/en/...` prefix in English pages; Fumadocs resolves the equivalent
path per locale.

## Components and code blocks

Fumadocs UI components are available by importing them at the top of the page.
The most common:

```mdx
import { Callout } from 'fumadocs-ui/components/callout';
import { Card, Cards } from 'fumadocs-ui/components/card';

<Callout type="info">Helpful aside.</Callout>
<Callout type="warn">Something to watch out for.</Callout>

<Cards>
  <Card title="Next page" description="What it covers" href="/en/docs/world/setup" />
</Cards>
```

Fenced code blocks are syntax-highlighted. The configured languages are
`csharp`, `typescript`, `javascript`, `json`, and `bash` — tag fences
accordingly:

````mdx
```csharp
var player = gameObject.AddComponent<BasisMediaPlayer>();
```
````

## MDX gotchas

MDX parses the body as JSX, so a few characters are special **outside** of code
blocks:

- A raw `<` starts a tag. Don't write `value < 10` in prose — rephrase, or wrap
  it in backticks (`` `value < 10` ``). Inside fenced code blocks it's fine.
- Raw `{` and `}` start a JavaScript expression. Avoid stray braces in prose;
  use backticks for anything containing them.
- Only import components that exist; a wrong import path fails the build.

When in doubt, run `npm run build` — it will point at the offending line.

## Adding a new page — checklist

1. Create `content/docs/<section>/<slug>.mdx` with `title` and `description`
   frontmatter.
2. Add `"<slug>"` to that section's `content/docs/<section>/meta.json` `pages`
   array, in the position you want it.
3. Put any screenshots under `public/img/<section>/` and reference them as
   `/img/<section>/<file>`.
4. Cross-link related pages with `/en/docs/...` links.
5. Run `npm run build` and confirm it passes.
6. Open a pull request. Pushes to `main` deploy automatically.

## Style

- Write for someone doing the task, not reading a spec — lead with what to do.
- Keep sentences short and concrete; prefer step lists for procedures.
- Use `<Callout type="info">` for helpful asides and `type="warn">` for
  pitfalls, rather than burying them in prose.
- Match the voice of the surrounding pages in the section you're editing.

## Submitting

Open a pull request against `main`. The deploy workflow builds the site and
publishes to GitHub Pages on merge; there's nothing to deploy by hand.
