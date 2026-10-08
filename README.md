# cricut-svg-splitter

A standalone web app that splits a paper-model SVG (unfolded 3D net) into two files ready for a Cricut machine:

- **Cut file** — outer boundary + gluing tabs, filled solid blue (`#0A91B3`)
- **Emboss file** — fold lines (face-to-face folds + languette/tab fold edges), solid strokes

Both output files share identical page dimensions and coordinate space, so Cricut overlays them with perfect alignment.

## Usage

Open `svg-splitter.html` in any modern browser — no server or build step required. Works from a local `file://` URL or hosted on GitHub Pages.

1. Drop your SVG onto the page (or click to browse), or click **Load sample cube** to try the included cube net
2. Inspect the original preview and the two output previews
3. Download `*_cut.svg` and `*_emboss.svg`

## How it works

The app reads CSS class attributes on SVG path elements to classify them:

| Class | Output file |
|---|---|
| `outer`, `outer_background` | Cut |
| `sticker` (gluing tabs) | Cut |
| `convex`, `concave` | Emboss |
| `freestyle`, `inner_background`, `arrow`, `text` | Dropped |

**Languette fold edges** are auto-computed from the `sticker` subpaths: for each tab, the app finds the edge whose two endpoints both lie on the outer boundary but are not consecutive vertices along it — that edge is the fold line between the tab and its face.

## Input format

SVGs exported from the [Blender Export Paper Model add-on](https://extensions.blender.org/add-ons/export-paper-model/#new) that use the class conventions above. The included `Cube.svg` is a working example.

## Files

| File | Description |
|---|---|
| `svg-splitter.html` | The application — single self-contained HTML file |
| `Cube.svg` | Sample cube net for testing |
