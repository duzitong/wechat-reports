---
name: mp-html-page
description: Generate or convert a self-contained HTML page that renders correctly in the WeChat mp-html viewer (the GH HTML viewer mini program, which bundles only the mp-html `style` plugin). Use when the user wants to author a new HTML page for the GH HTML viewer miniapp, convert an existing webpage/HTML into mp-html-safe markup, or fix a page that renders wrong in the miniapp (white background, missing styles, broken layout).
---

# mp-html Page Generator

Author a single self-contained `.html` file that renders in the miniapp's `mp-html` component (only the `style` plugin is bundled). Design the page however fits the content — just stay inside the constraints below, then re-read the result and confirm it breaks none of them.

## Constraints

- **One self-contained file.** All CSS in a `<style>` in `<head>`, placed BEFORE the elements it styles (the plugin matches top-down).
- **Literal values only** — no `var()` / custom properties (not resolved). No `@media`, `@keyframes`, `@font-face`, `@import` (all dropped).
- **Supported selectors:** tag, `.class`, `#id`, intersection (`.a.b`), descendant (`.a .b`), child (`.a > b`), grouping (`a, b`), `:before` / `:after`. Dropped: `*`, `[attr]`, `:hover` / `:nth-child` / other pseudo-classes, `~`, `+`.
- **Set `background` + `color` on a wrapper** that holds all content — otherwise it falls back to the device default (white bg / black text).
- **No interactivity** — `<script>`, `<canvas>`, `<iframe>`, forms, animations and transitions don't work.
- **Images:** use `<img>` (not CSS `background-image:url()`) with a system font stack (no web fonts); mp-html caps images at `max-width:100%`. **Pick `src` by how central the image is (see runtime limits): photos that ARE the content (a photo atlas, a travel journal) → a relative same-repo path, so the viewer pre-fetches them all and they appear together, fully loaded; a lone incidental/decorative image → an absolute `raw.githubusercontent.com` URL, so it lazy-loads and never blocks first paint. A private repo can only use relative (raw URLs won't authenticate), so its images always pre-load.**

## Viewer runtime limits (from the miniapp's code)

The viewer resolves a page via the viewer mini-program's `config/index.js`, which runs in `dataUri` mode for **every** repo (public or private). In that mode each **relative** `<img>`/asset is fetched through the GitHub Contents API, then **saved to an on-device cache file** whose local path is handed to the `<img>` (a base64 data URI is only a write-failure fallback). Absolute `https://` images are left untouched (see the last two bullets). Some hard caps apply to the relative path; design pages to stay under them:

- **≤ 120 relative images per page.** Only the first `images.maxInline` (currently **120**) repo-relative `<img>` are fetched; any beyond that keep an unresolved relative `src` and render **blank**. Absolute `https://` images load directly and do **not** count toward this cap.
- **Each relative image must be < 1 MB — aim for ≤ 250 KB.** The fetch still uses the Contents API, which only returns `content` for files **≤ 1 MB**; a larger file comes back empty and the image stays blank — 1 MB is the hard ceiling. **Target ≤ 250 KB per photo anyway**: first-load downloads run only a few at a time (concurrency-limited), so smaller files mean a faster first paint and a smaller on-device cache. Absolute `https://` images skip the API entirely. See the recompression recipe below.
- **Images are cached on-device, not held in memory.** Each downloaded image is written to a local cache file and the `<img>` points at that path; the cache **persists across restarts** and only refreshes via the viewer's **“清缓存”** button. With lazy-loading on, image-heavy pages are far lighter than the old base64-inline approach — but every image must still clear the per-image cap above on its first download.
- **≤ 8 relative stylesheets per page.** Only the first `css.maxStylesheets` (currently **8**) same-repo `<link rel="stylesheet">` files are fetched and inlined. Inline `<style>` blocks have **no** cap and are always parsed — **prefer inline `<style>`**.
- **A config `raw` mode exists but is currently OFF.** `config.images.mode` ships hardcoded to `dataUri`, so the viewer **never auto-rewrites** relative images to `raw.githubusercontent.com` — every relative `<img>` is pre-downloaded via the Contents API regardless of repo visibility. The only way to get lazy behavior today is to **author the absolute raw URL yourself** (next bullets).
- **Relative `src` = pre-downloaded together; absolute `https` = native lazy-load.** The viewer collects every **relative** `<img>`, fetches each through the Contents API (≤ 120/page, < 1 MB each, 5 at a time) behind a "加载图片 n/N…" progress line, then swaps them all to on-device paths at once — so the images land **together, fully loaded** (a cohesive reveal, no half-drawn page). An **absolute `https://` `src` is skipped** by that collector and passed straight to mp-html's native `lazy-load`: it streams in on scroll and never blocks first paint, but before it arrives it can briefly show as a **collapsed thin line** (no reserved height).
- **So choose by the image's role, not by repo type.** Photo-heavy pages where the images ARE the content (flower atlas, travel guide) read better with **relative** paths — everything appears at once, ready to browse. A lone banner or decoration is better as an **absolute raw URL** — lazy, no first-paint stall. **A private repo has no choice: raw URLs won't authenticate, so always use relative** (which is also why the ≤ 120 / < 1 MB caps matter there). Absolute URLs must point at an already-pushed file to resolve.

## Recompressing photos (≤ 250 KB, keep quality)

To hit ≤ 250 KB without making photos look soft, **prefer dropping resolution over crushing JPEG quality**. Squeezing a full-res phone photo (~1280×2275) under 250 KB can force quality ~50 (blocky/smeared); the same 250 KB looks much better at a slightly smaller size with quality ~75. On-screen the viewer caps images at `max-width:100%` and phones are only ~1080–1280 px wide, so downscaling tall photos to ~800–1200 px wide is visually lossless.

Recipe (per file that exceeds the target):

1. **Only touch files over the target.** Leave already-small images alone — re-encoding just degrades them.
2. **Bake in orientation, strip metadata.** Apply the EXIF orientation first (PIL `ImageOps.exif_transpose`), convert to RGB, then save without EXIF (smaller, and avoids sideways photos).
3. **Hold quality in a good band (q ≈ 72–85); shrink resolution to fit.** Find the *largest* scale at which some quality in that band still lands ≤ 250 KB — that maximizes resolution while keeping quality decent. Save JPEG with `optimize=True, progressive=True`.
4. **Keep filenames unchanged** so the page's `<img src>` references stay valid (no HTML edits needed).

After recompressing, verify each file is valid and ≤ 250 KB, then commit with filenames unchanged.

## Output

Save as one `.html` file and have the user push it to the GitHub repo the viewer reads (relative images must live in that repo).
