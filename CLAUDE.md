# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static personal portfolio site (`aghilanbaskar.github.io`) served directly by GitHub Pages from the repo root on push to `master`. There is no build step at deploy time: the committed files are what gets served. Root-relative URLs (`/assets/...`) are used because the site is served at the domain root.

## Development

Preview with a static server from the repo root: `python3 -m http.server`. There is no lint/test tooling.

**CSS is precompiled Tailwind v4.** `assets/css/style.css` is generated from `tailwind.input.css`, which scans `index.html` and `resume/index.html`, and the output is committed. After adding or changing Tailwind classes in either page, regenerate it:

```sh
npm i --no-save tailwindcss@4 @tailwindcss/cli@4   # once; node_modules is gitignored
npx tailwindcss -i tailwind.input.css -o assets/css/style.css --minify
```

Then bump the `?v=` query on the stylesheet `<link>` in both pages. Do not switch to the Tailwind CDN play-script: it compiles in the browser and breaks the LCP/CLS targets.

## Structure

- `index.html`: the main portfolio page. It is a single long page with anchor-linked sections: `#about`, `#experience`, `#projects`, `#blogs` (titled "Writing"), `#skills`, `#education` and `#contact`.
  - Icons are an inline SVG `<symbol>` sprite at the top of `<body>`, used via `<use href="#i-…">`.
  - All JS is one small inline script at the bottom: theme toggle, mobile menu, IntersectionObserver reveal/active-nav, and the typewriter.
- `resume/index.html`: a thin wrapper around the Google Doc resume. It has a toolbar (Home, Open in Google Docs, Download PDF) and a full-height `<iframe>` of the Doc's `/preview` URL (search for `RESUME_DOC_ID`). The PDF button uses the Doc's live `/export?format=pdf` URL. Resume content lives only in the Google Doc, so edit it there. The home page's experience/skills are maintained separately.
- `tailwind.input.css`:
  - Theme colours are CSS variables on `:root[data-theme="dark"|"light"]`, mapped into Tailwind via `@theme inline`. So use `bg-bg`, `text-fg`, `text-muted`, `border-line`, `bg-surface`, `text-accent` and so on, rather than `dark:` variants.
  - Shared component classes: `.card`, `.tag`, `.btn`, `.eyebrow`, `.section-title`, `.bullets`, `.link`, `.nav-link`.
- `assets/img/`: images. Prefer WebP with explicit `width`/`height`, plus `loading="lazy" decoding="async"` below the fold.
- `inverter-battery-calculator/index.html`: a standalone, self-contained tool (own HTML/CSS/JS, not linked to `assets/`). It uses Tailwind (via CDN play-script) and Vue 3 (via CDN, global build), with a single `createApp({ setup() {...} })` component inlined in a `<script>` tag. All calculator logic lives in that one `setup()` function. It is excluded from the Tailwind scan.

## Performance and theming rules

These keep LCP, CLS and INP green; Lighthouse mobile currently scores 100.
- No web fonts (system font stack), no third-party scripts, and no hotlinked images.
- The theme is set by the tiny inline script in `<head>` before first paint, so it never flashes. The `js` class it adds on `<html>` is what enables `.reveal` animations. Don't put `.reveal` on the hero, because it would delay LCP.
- Every image has dimensions. The typewriter line has a fixed height and `whitespace-nowrap`, and its phrases must fit at a 360px width.

When adding a new standalone tool, follow the `inverter-battery-calculator/` pattern: its own top-level directory with a self-contained `index.html`.
