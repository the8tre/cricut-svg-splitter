# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the app

No build step. Serve `index.html` with any static file server from the project root:

```bash
python3 -m http.server 8744
# then open http://localhost:8744
```

To verify JS syntax without running a browser:

```bash
python3 -c "
import re
content = open('index.html').read()
m = re.search(r'<script>(.*?)</script>', content, re.DOTALL)
open('/tmp/chk.js','w').write(m.group(1))
"
node --check /tmp/chk.js
```

## Architecture

Single self-contained file: `index.html` (HTML + CSS + JS, ~800 lines). No dependencies, no build tooling.

### Input

SVGs exported from the [Blender Export Paper Model add-on](https://extensions.blender.org/add-ons/export-paper-model/#new). Path elements carry CSS `class` attributes that classify their role:

| Class | Role |
|---|---|
| `outer` | Cut boundary (single closed polygon — already encodes languettes/tabs) |
| `sticker` | Gluing tabs (multi-subpath `M...Z` — redundant with `outer`, used only for fold-edge detection) |
| `convex`, `concave` | Deboss/fold lines |
| `freestyle`, `inner_background`, `arrow`, `text`/`tspan` | Extra (passed through to combined output) |

### Processing pipeline (`split()`)

1. **Walk** the SVG tree → classify each element into `cutNodes`, `debossNodes`, `extraNodes`
2. **Cut**: `mergeCutNodes(cutNodes, outerDAttr)` — uses `outer` path directly (sticker paths are geometrically redundant); outputs single `<path>` with inline `style="fill:…"` (no `<style>` block — Cricut ignores CSS)
3. **Deboss**: `mergeDebossNodes(debossNodes, languetteFoldPathStr)` — collects all `M…L` segments, deduplicates by canonical edge key, merges into one path
4. **Combined**: `buildCricutSVG(…)` — union of cut + deboss paths + extra nodes in a single SVG

### Cricut SVG sizing (critical)

Cricut Design Space ignores `width`/`height` canvas size and fits the shape's bounding box to the canvas height, scaling width proportionally. **Do not use full-page viewBox.**

Working approach (confirmed):
- `viewBox` = tight path bounding box (`minX minY w h`)
- `width`/`height` in **inches** matching that bbox: `(w/25.4).toFixed(4) + 'in'`
- Applied to cut, deboss, and combined SVGs

Full-page approaches that were tried and failed: `width="210mm"`, `width="793.70px"`, `viewBox="0 0 210 297"` with inch dimensions.

### Previews

`setupPreview()` uses `<img src=blobUrl>` (not `innerHTML`) to isolate the SVG from the page's CSS cascade and prevent XSS. Blob URLs are tracked in `_blobUrls` map and revoked before replacement.

### Key functions

| Function | Purpose |
|---|---|
| `mergeCutNodes(nodes, outerD)` | Extract outer path, add inline fill/stroke style |
| `mergeDebossNodes(nodes, extraPathStr)` | Deduplicate M…L segments, merge into one path |
| `buildCricutSVG(…)` | Combine cut + deboss + extra into one SVG with tight union bbox |
| `pathBBox(dAttr)` | Compute bounding box from path d-attribute |
| `extractLanguetteFoldEdges(…)` | Find fold lines: edges on outer boundary but not consecutive in it |
| `parseStickerSubpaths(dAttr)` | Parse multi-subpath sticker `d` into array of point arrays |
