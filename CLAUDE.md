# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

An Adobe Edge Delivery Services (AEM EDS) website built on the [aem-boilerplate](https://github.com/adobe/aem-boilerplate/). Uses vanilla JS (ES6+), CSS3, and HTML5 — no build steps, no transpiling, no frameworks. The `*.aem.live` backend delivers CMS-authored content that our code decorates client-side.

**NEVER modify `scripts/aem.js`** — it is the core AEM library managed upstream.

## Commands

- `npm install` — install dependencies
- `npm run lint` — run ESLint + Stylelint (CI runs this on every push)
- `npm run lint:fix` — auto-fix lint issues
- `npx -y @adobe/aem-cli up --no-open --forward-browser-logs` — start local dev server at `http://localhost:3000`
- No test framework is configured; quality is validated via linting and PageSpeed Insights checks against preview URLs.

## Architecture

### Three-Phase Page Loading (`scripts/scripts.js`)

`loadPage()` orchestrates three sequential phases:
1. **Eager** (`loadEager`) — decorates main content, loads first section only (LCP-critical)
2. **Lazy** (`loadLazy`) — loads header/footer blocks, remaining sections, `lazy-styles.css`, fonts
3. **Delayed** (`loadDelayed`) — imports `delayed.js` after 3s for martech/analytics

### Block System

Blocks live in `blocks/{name}/` with matching `{name}.js` and `{name}.css`. Each JS file exports a default `decorate(block)` function that receives the block's DOM element. The framework auto-loads block JS/CSS when the block appears on a page — no manual registration needed.

Current blocks: `cards`, `columns`, `footer`, `fragment`, `header`, `hero`.

### Auto-Blocking (`buildAutoBlocks` in `scripts.js`)

- Links matching `*/fragments/*` auto-load as fragment blocks
- Pages with a leading `<picture>` + `<h1>` get an auto-generated hero block

### Button Convention

Links authored with `**bold**` become `.button.primary`, `*italic*` becomes `.button.secondary`, bold+italic becomes `.button.accent`.

### Content & Markup

- Inspect page content: `curl http://localhost:3000/path/to/page.plain.html`
- Markup reference: https://www.aem.live/developer/markup-sections-blocks
- If no CMS content exists, create static HTML in a `drafts/` folder and start the dev server with `--html-folder drafts`

## Code Style

- **JS**: Airbnb ESLint rules. Always use `.js` extensions in imports. Unix (LF) line endings. `no-param-reassign` allows property mutation.
- **CSS**: Stylelint standard config. Mobile-first with `min-width` breakpoints at 600px/900px/1200px. Scope all selectors to the block (`.blockname .child`, not `.child`). Avoid `{blockname}-container` and `{blockname}-wrapper` class names (reserved by the framework).
- Design tokens (colors, fonts, sizes) are CSS custom properties in `:root` of `styles/styles.css`.

## Environments & Deployment

Derive URLs from `gh repo view --json nameWithOwner` and `git branch`:
- **Preview**: `https://{branch}--{repo}--{owner}.aem.page/`
- **Live**: `https://main--{repo}--{owner}.aem.live/`

PRs **must** include a preview URL in the description demonstrating the change. Use `gh pr checks` to verify CI status. Target PageSpeed score of 100.

## Documentation Search

Search AEM EDS docs: `site:www.aem.live` in web search, or:
```sh
curl -s https://www.aem.live/docpages-index.json | jq -r '.data[] | select(.content | test("KEYWORD"; "i")) | "\(.path): \(.title)"'
```

See `AGENTS.md` for full agent-oriented development guidelines.
