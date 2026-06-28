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
- **Images:** use `<img>` with a same-repo relative path (the viewer inlines it) or an absolute `https://` URL — not CSS `background-image:url()`. Use a system font stack (no web fonts). mp-html caps images at `max-width:100%`.

## Viewer runtime limits (from the miniapp's code)

The viewer resolves a page via the viewer mini-program's `config/index.js`. For a **private** repo it runs in `dataUri` mode — each repo-relative asset is fetched through the GitHub Contents API and inlined as base64. That imposes hard caps; design pages to stay under them:

- **≤ 120 relative images per page.** Only the first `images.maxInline` (currently **120**) repo-relative `<img>` are inlined; any beyond that keep an unresolved relative `src` and render **blank**. Absolute `https://` images load directly and do **not** count toward this cap.
- **Each relative image must be < 1 MB — aim for ≤ 250 KB.** `dataUri` mode uses the Contents API, which only returns base64 `content` for files **≤ 1 MB**; a larger file comes back empty and the image stays blank. 1 MB is the hard ceiling, but **target ≤ 250 KB per photo**: base64 inflates bytes ~33% in memory, and a page of sub-250 KB images inlines far faster and scrolls smoother. See the recompression recipe below.
- **Mind the total inlined payload.** Every relative image is one API call and is held in memory as base64 all at once; a very large total can make the webview sluggish or crash. Keep image-heavy pages reasonable.
- **≤ 8 relative stylesheets per page.** Only the first `css.maxStylesheets` (currently **8**) same-repo `<link rel="stylesheet">` files are fetched and inlined. Inline `<style>` blocks have **no** cap and are always parsed — **prefer inline `<style>`**.
- **`raw` mode (no caps) is public-repo only.** It rewrites images to `raw.githubusercontent.com` with no API calls or caps, but only works for public repos. A private repo must use `dataUri`, so the caps above apply.

> Source of truth: the viewer's `config/index.js` (`images.mode` / `images.maxInline` / `css.inlineExternal` / `css.maxStylesheets`). If those change, update this note.

## Recompressing photos (≤ 250 KB, keep quality)

To hit ≤ 250 KB without making photos look soft, **prefer dropping resolution over crushing JPEG quality**. Squeezing a full-res phone photo (~1280×2275) under 250 KB can force quality ~50 (blocky/smeared); the same 250 KB looks much better at a slightly smaller size with quality ~75. On-screen the viewer caps images at `max-width:100%` and phones are only ~1080–1280 px wide, so downscaling tall photos to ~800–1200 px wide is visually lossless.

Recipe (per file that exceeds the target):

1. **Only touch files over the target.** Leave already-small images alone — re-encoding just degrades them.
2. **Bake in orientation, strip metadata.** Apply the EXIF orientation first (PIL `ImageOps.exif_transpose`), convert to RGB, then save without EXIF (smaller, and avoids sideways photos).
3. **Hold quality in a good band (q ≈ 72–85); shrink resolution to fit.** Find the *largest* scale at which some quality in that band still lands ≤ 250 KB — that maximizes resolution while keeping quality decent. Save JPEG with `optimize=True, progressive=True`.
4. **Keep filenames unchanged** so the page's `<img src>` references stay valid (no HTML edits needed).

Reference (Python / Pillow):

```python
from PIL import Image, ImageOps
import io
TARGET = 250_000
im = ImageOps.exif_transpose(Image.open(path)).convert("RGB")
w, h = im.size
for s in [1.0, .9, .8, .7, .62, .55, .5]:          # largest scale first
    work = im if s == 1 else im.resize((round(w*s), round(h*s)), Image.LANCZOS)
    for q in range(85, 71, -1):                    # highest quality that fits
        buf = io.BytesIO()
        work.save(buf, "JPEG", quality=q, optimize=True, progressive=True)
        if buf.tell() <= TARGET:
            open(path, "wb").write(buf.getvalue()); break
    else:
        continue
    break
```

After recompressing, verify each file is valid and ≤ 250 KB (e.g. `Image.open(p).verify()` + size check), then commit with filenames unchanged.

## Output

Save as one `.html` file and have the user push it to the GitHub repo the viewer reads (relative images must live in that repo).
