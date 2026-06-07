# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static marketing website for **Dr. Sergey Morozov** (medlogic.co), a Clinical AI Strategist. There is no build system, package manager, framework, or backend. Every page is a hand-authored, self-contained `.html` file served directly. Tailwind CSS is loaded at runtime from the CDN — there is no compile step, no `node_modules`, and no `package.json`.

## Running / previewing

Open any `.html` file directly in a browser, or serve the directory:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

There is no build, lint, or test tooling. Validate changes by loading the page in a browser and checking the layout, the mobile nav toggle, and form behavior.

## Pages

The site is a flat set of pages at the repo root, cross-linked by relative `*.html` hrefs (no router):

- `index.html` — home / landing
- `about.html`, `services.html`, `case-studies.html`, `insights.html`, `contact.html`
- `assessment.html` — "AI Readiness Assessment" lead-capture page
- `presentation-ai-monitoring.html` — **standalone slide deck**, see note below

When adding or renaming a page, update the nav links and the footer links in **every** other page — they are duplicated inline per file, not shared.

## Conventions to preserve

These patterns are copy-pasted across all the main pages. When editing one, keep the others consistent.

- **Styling**: Tailwind via `<script src="https://cdn.tailwindcss.com?plugins=forms,container-queries">`. The custom Material-Design color palette and `borderRadius`/`fontFamily` overrides live in an inline `tailwind.config` (`<script id="tailwind-config">`) in each page's `<head>`. Fonts are Inter + Material Symbols Outlined from Google Fonts. Reuse the existing palette tokens (e.g. `secondary` `#0f57d0`, `tertiary` `#006d48`, `surface` `#f9f9f9`) rather than introducing new hex values.
- **Shared CSS helpers** defined inline per page: `.clinical-gradient`, `.editorial-shadow`, and the `.material-symbols-outlined` font-variation rule.
- **Brand wordmark**: `DR. MOROZOV` in the nav, linking to `index.html`.
- **Layout container**: `max-w-[1280px] mx-auto px-8` with a sticky, blurred header.
- **Mobile nav**: a `#mobile-nav` overlay toggled by a small inline IIFE (`menu-toggle` / `menu-close`). The same script is duplicated in each page near the closing tags.
- **Email obfuscation**: contact email is assembled in JS at runtime — elements with class `js-email` get their `href`/`text` set by an inline IIFE that concatenates `sergey` + `medlogic.co`. Do not hardcode the full address in markup; use the `js-email` pattern.
- **Each page sets its own** `<title>`, SEO `meta description`, `canonical`, Open Graph, and Twitter Card tags pointing at `https://medlogic.co/<page>.html`. Update these when content changes.
- **Footer**: dark `#172B4D`, copyright reads `© 2026 Dr. Sergey Morozov. Clinical Curator & AI Strategist.`

## Forms

`contact.html`, `assessment.html`, and `insights.html` submit to Formspree:

```
action="https://formspree.io/f/mrerzzpz" method="POST"
```

There is no client-side submit handler or scoring logic — forms POST directly to Formspree. The assessment page presents the "six dimensions" as static content, not an interactive calculator.

## The presentation page is different

`presentation-ai-monitoring.html` is a **self-contained slide deck** and does **not** follow the conventions above. It does not use Tailwind, the shared nav/footer, or the site color tokens. Instead it defines its own design system with CSS custom properties (`:root { --navy, --blue, --cyan, ... }`) and uses `.slide` / `.slide-title` sections. Treat it as an independent document; do not try to unify it with the marketing pages.

## Assets

- `logo.png` (~1.1 MB) and `favicon.svg` at the root, referenced by relative path.
- `.DS_Store` is committed at the root (macOS artifact); avoid adding more.
