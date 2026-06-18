---
name: mp-html-page
description: Generate or convert a self-contained HTML page that renders correctly in the WeChat mp-html viewer (the miniapp-github-html-viewer mini program, which bundles only the mp-html `style` plugin). Use when the user wants to author a new HTML page for the GH HTML viewer miniapp, convert an existing webpage/HTML into mp-html-safe markup, or fix a page that renders wrong in the miniapp (white background, missing styles, broken layout).
---

# mp-html Page Generator

Author a single self-contained `.html` file that renders in the miniapp's `mp-html` component (only the `style` plugin is bundled). Design the page however fits the content — just stay inside the constraints below, then re-read the result and confirm it breaks none of them.

## Constraints

- **One self-contained file.** All CSS in a `<style>` in `<head>`, placed BEFORE the elements it styles (the plugin matches top-down).
- **Literal values only** — no `var()` / custom properties (not resolved). No `@media`, `@keyframes`, `@font-face`, `@import` (all dropped).
- **Supported selectors:** tag, `.class`, `#id`, intersection (`.a.b`), descendant (`.a .b`), child (`.a > b`), grouping (`a, b`), `:before` / `:after`. Dropped: `*`, `[attr]`, `:hover` / `:nth-child` / other pseudo-classes, `~`, `+`.
- **Set `background` + `color` on a wrapper** that holds all content — otherwise it falls back to the device default (white bg / black text).
- **No interactivity** — `<script>`, `<canvas>`, `<iframe>`, forms, animations and transitions don't work.
- **Images:** use `<img>` with a same-repo relative path (the viewer inlines it) or an absolute `https://` URL — not CSS `background-image:url()`. Use a system font stack (no web fonts). mp-html caps images at `max-width:100%`.

## Output

Save as one `.html` file and have the user push it to the GitHub repo the viewer reads (relative images must live in that repo).
