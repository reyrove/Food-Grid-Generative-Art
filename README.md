# Food-Grid-Generative-Art

A seed-based generative art catalogue for **Food Grid**, a computational illustration system. Built as a single-file HTML page with p5.js, styled as a gallery-ready portfolio and licensing surface.

**Live:** `https://reyrove.github.io/Food-Grid-Generative-Art/`

---

## Overview

**Food Grid** renders a 16 × 16 grid of tiles from numeric seeds. Every tile contains a small illustrated food drawn from a list of fourteen options — apple, orange, pear, cherry, banana, strawberry, grape, avocado, donut, croissant, chocolate, macaron, sushi, cheese. Each tile has its own rotation, its own size variation, and its own colour from a harmonic palette of eight hues.

The page functions as a **catalogue and licensing surface** for fashion houses, textile studios, and surface designers. Every composition is reproducible by seed, meaning the same number always produces the identical grid.

---

## Features

### Generative Engine (p5.js)

- **16 × 16 tile grid** — 256 illustrated foods per composition
- **14 illustrated foods** — apple, orange, pear, cherry, banana, strawberry, grape, avocado, donut, croissant, chocolate, macaron, sushi, cheese
- **Per-seed palette** — 8 colours at 45° hue intervals, from a random base hue; saturation 50–90, brightness 60–95
- **Per-tile variation** — small rotation (−30° to +30°), size variation (0.5–0.9 of tile), and a random colour from the palette
- **Soft background wash** — a low-saturation pastel wash in a random hue
- **Shape vocabulary** — each food is built from its own combination of `ellipse`, `triangle`, `rect`, `curveVertex`, `beginShape` / `endShape`, so no two foods share a silhouette

### Same Seed → Many Surfaces

A single seed drives every instance on the page:

| Instance | Purpose | Tiling |
|----------|---------|--------|
| **Cover** | Hero composition | 1 × 1 |
| **Plate** | Main work (master) | 1 × 1 |
| **Frame** | Framed archival print | 1 × 1 |
| **Print** | Archival print surface | 1 × 1 |
| **Scarf** | Silk scarf (3:1) | 3 × 1 |
| **Textile** | Fabric yardage (4:3) | 2 × 2 |
| **Wall** | Wallpaper (2:3) | 1 × 2 |
| **Archive** | 8 unique seeds | 1 × 1 each |

### Controls

- **New Seed** button / `R` key — regenerate with a fresh seed
- **Download** button / `S` key — export the plate canvas as PNG
- **Click any archive plate** — load that seed into the main view

### Performance

- Debounced resize handling (250ms)
- `IntersectionObserver` pauses off-screen sketches
- `visibilitychange` pauses rendering when the tab is hidden
- `pixelDensity(1)` for a lighter canvas footprint
- Two-frame render (`maxFrames: 2`) so tiles draw once and stop

---

## File Structure

```
Food-Grid-Generative-Art/
├── index.html          # Single-file page (structure + styles + generative engine)
├── README.md
└── images/
    ├── fav.svg         # Favicon
    ├── tote.png        # Mockup: tote bag
    ├── tee.png         # Mockup: t-shirt
    └── cushion.png     # Mockup: cushion
```

No build step. No dependencies to install. p5.js is loaded from a CDN.

---

## Getting Started

1. Clone the repo:
   ```bash
   git clone https://github.com/reyrove/Food-Grid-Generative-Art.git
   cd Food-Grid-Generative-Art
   ```
2. Add the three mockup images to `images/` (optional — they appear in the Commission section).
3. Open `index.html` in a browser.

That's it. The page self-initialises on `DOMContentLoaded`.

---

## How the Generative System Works

### Seeded Random

A linear congruential generator replaces p5's native randomness inside each sketch:

```js
class SeededRandom {
  constructor(s) { this.state = (Math.abs(s) % 2147483647) || 1; }
  rand() { this.state = (this.state * 9301 + 49297) % 233280; return this.state / 233280; }
  range(min, max) { return this.rand() * (max - min) + min; }
  int(min, max) { return Math.floor(this.range(min, max + 1)); }
  pick(arr) { return arr[Math.floor(this.rand() * arr.length)]; }
}
```

Every aesthetic decision — background hue, base palette hue, tile food, tile colour, tile rotation, tile size — is drawn from this generator in a fixed order. Same seed, same sequence, same composition.

### Parameter Space (per seed)

| Parameter | Range |
|-----------|-------|
| Grid size | 16 × 16 |
| Background hue | 0 – 360 |
| Background saturation | 10 – 20 |
| Background brightness | 95 |
| Base palette hue | 0 – 360 |
| Palette colours | 8 (45° apart) |
| Palette saturation | 50 – 90 |
| Palette brightness | 60 – 95 |
| Tile rotation | −30° to +30° |
| Tile size | 0.5 – 0.9 × tile |

### The Fourteen Foods

Each food is defined by a small `switch` in `drawFood`. The shape vocabulary:

| Food | Shape |
|------|-------|
| Apple | Ellipse + stem rect |
| Orange | Ellipse |
| Pear | Triangle + stem rect |
| Cherry | Two small ellipses + two stroke lines |
| Banana | curveVertex path |
| Strawberry | Ellipse + triangle calyx |
| Grape | Six stacked ellipses |
| Avocado | Ellipse + pit ellipse |
| Donut | Ellipse + white hole ellipse |
| Croissant | curveVertex path |
| Chocolate | Rounded rect |
| Macaron | Flattened ellipse |
| Sushi | Rounded rect + rice ellipse |
| Cheese | Quadrilateral wedge |

Adding a new food means adding a case to the `switch` and adding the name to `FOODS`.

### Tiling Logic

Rectangular surfaces (scarf, textile, wall) tile the square composition N×M. Each tile is drawn into its own translated coordinate space, centred within its cell:

```js
const tileW = canvasW / tileCols;
const tileH = canvasH / tileRows;
const tileSize = Math.min(tileW, tileH);
// draw square of side tileSize at (ox, oy) for each cell
```

This preserves the square composition's proportions regardless of surface aspect ratio.

---

## Customisation

### Change the default seed

Edit the init function:

```js
function init() {
  seed = 123456789;  // fixed seed instead of random
  // ...
}
```

### Adjust the grid resolution

Edit the constant near the top of the script:

```js
const GRID_SIZE = 16;
```

Higher values (24, 32) produce finer, denser illustrations. Lower values (8, 10) produce larger, more legible individual foods. The grid is always square.

### Add a new food

1. Add the name to `FOODS`:
   ```js
   const FOODS = ['apple', 'orange', ..., 'your-new-food'];
   ```
2. Add a `case` to the `switch` inside `drawFood`:
   ```js
   case 'your-new-food':
     // drawing code, using `size` for scale
     break;
   ```

The `size` parameter scales with the tile, so all shapes automatically fit.

### Change the palette

Edit inside `drawComposition`:

```js
const palette = [];
for (let i = 0; i < 8; i++) {
  palette.push([
    (baseHue + i * 45) % 360,   // hue step
    sr.range(50, 90),            // saturation
    sr.range(60, 95)             // brightness
  ]);
}
```

- Change `45` to a smaller step (30, 22.5) for a tighter, more harmonious palette.
- Change the saturation and brightness ranges for a softer or more vivid look.

### Change the background

```js
const bgHue = sr.rand() * 360;
const bgSat = sr.range(10, 20);
p.background(bgHue, bgSat, 95);
```

Replace with a fixed hue for a consistent ground across all seeds, or widen the saturation range for more variance.

### Change surface tiling

Edit `surfaceDefs`:

```js
const surfaceDefs = [
  { id: 'surfacePrint',   wrapId: 'wrap-surfacePrint',   tile: { cols: 1, rows: 1 } },
  { id: 'surfaceScarf',   wrapId: 'wrap-surfaceScarf',   tile: { cols: 3, rows: 1 } },
  // ...
];
```

The `tile` object accepts any `{ cols, rows }` pair.

### Replace mockup images

Swap the files in `images/` or update the `src` attributes in the Commission section:

```html
<img src="images/your-mockup.png" alt="..." loading="lazy" />
```

---

## Design System

| Token | Value | Use |
|-------|-------|-----|
| `--bg` | `#0a0a0a` | Page background |
| `--paper` | `#f4f1ea` | Light surface (Surfaces section) |
| `--gold` | `#b8935a` | Primary accent |
| `--gold-soft` | `#d4b483` | Italic emphasis |
| `--serif` | Cormorant Garamond | Headings, titles |
| `--mono` | DM Mono | Labels, metadata, UI |

Type scale, spacing, and section rhythm match the Skyline, Portrait, Girih, Gothic Grid, Order to Chaos, Wallpaper Groups, and Wandering Tiles templates. Responsive breakpoints at 900px, 720px, 560px, and 400px.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Regenerate with a new seed |
| `S` | Download plate as PNG |
| `Enter` / `Space` | Activate focused archive item |
| `Esc` | Close legal modal |

---

## Browser Support

Modern evergreen browsers with:

- Canvas 2D (universal)
- `IntersectionObserver` (universal in modern browsers)
- `backdrop-filter` (Safari 9+, Chrome 76+, Firefox 103+)
- CSS custom properties (universal)

Fallback: if `IntersectionObserver` is unavailable, sketches render continuously.

---

## Accessibility

- Archive items are keyboard-focusable with `role="button"` and `aria-label`
- Legal modal traps focus, closes on `Esc` and overlay click
- `prefers-reduced-motion` disables transitions and smooth scroll
- All interactive elements have visible focus states (via browser default)

---

## Section Map

| Section | ID | Purpose |
|---------|-----|---------|
| Cover | — | Hero title + live food grid |
| Statement | `#statement` | Artist statement |
| Plate | `#work` | Main work + metadata + controls |
| Surfaces | `#application` | Framed print + 4 tiled surfaces |
| Process | `#process` | Three process cards |
| Archive | `#archive` | 8 selectable seeds |
| Commission | `#commission` | Services + mockups + CTA |
| Colophon | — | Footer, links, legal |
| Legal modal | `#legalModal` | Licensing / Terms / Credits |

---

## Deployment (GitHub Pages)

1. Push the repo to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, select `Deploy from a branch`.
4. Choose branch `main` and folder `/ (root)`.
5. Save — the site goes live at `https://reyrove.github.io/Food-Grid-Generative-Art/`.

---

## Notes on the Source Sketch

The generative engine is adapted almost verbatim from the original p5 sketch. The adaptations that turn it into a catalogue page:

1. **Seeded random replaces p5's `random()`** — a local `SeededRandom` class drives every choice. The original sketch calls `setSeeds()` for fxhash compatibility; that's replaced with the seeded generator directly.
2. **Sketch factory** — `makeSketch({ slotId, seedValue, ... })` returns a p5 sketch that can be instantiated into any DOM slot, enabling cover / plate / frame / surfaces / archive from one seed.
3. **DOM-driven canvas sizing** — `cs = min(windowHeight, windowWidth)` becomes slot-rect dimensions.
4. **Tiling support** — the `tile: { cols, rows }` parameter repeats the square composition across rectangular surfaces.
5. **Drawing logic takes `p` explicitly** — the original used p5's global mode; here `p` is passed so the factory closure works.
6. **`color(...)` replaced with `[h, s, b]` arrays** — the original sketch uses p5's `color()` to build the palette. Inside the factory, this is replaced with plain arrays passed to `p.fill()`. Same visual result, faster execution.
7. **`windowResized` replaced by a debounced `handleResize`** — all sketches are destroyed and recreated on resize (250ms debounce), since their canvases are locked to slot dimensions.
8. **`triggerPreview` removed** — the original calls it for fxhash thumbnail generation; not needed in the plain-browser catalogue.

---

## Related Repositories

| Repo | Theme |
|------|-------|
| [`Skyline-Series`](https://github.com/reyrove/Skyline-Series) | Themed cityscapes |
| [`Portrait-Series-Generative-Art`](https://github.com/reyrove/Portrait-Series-Generative-Art) | Generative faces |
| [`Girih-2-Generative-Art`](https://github.com/reyrove/Girih-2-Generative-Art) | Islamic geometric ornament |
| [`Gothic-Grid-Generative-Art`](https://github.com/reyrove/Gothic-Grid-Generative-Art) | Mosaic grid composition |
| [`Order-to-Chaos-Generative-Art`](https://github.com/reyrove/Order-to-Chaos-Generative-Art) | Gradient grid composition |
| [`Wallpaper-Groups-Generative-Art`](https://github.com/reyrove/Wallpaper-Groups-Generative-Art) | Crystallographic symmetry |
| [`Wandering-Tiles-Generative-Art`](https://github.com/reyrove/Wandering-Tiles-Generative-Art) | Line-drawing composition |
| **`Food-Grid-Generative-Art`** | Illustrated food grid *(this repo)* |

All eight share the same catalogue template, design tokens, and seed-based reproducibility model.

---

## Credits

- **Design & Generative System** — Reyhaneh Daneshdoost
- **Typefaces** — Cormorant Garamond, DM Mono
- **Engine** — p5.js 1.9.0
- **Edition** — Food Grid, Autumn 2026

---

## License

All artwork, code, and generative systems are the intellectual property of the artist. Compositions may not be reproduced, redistributed, resold, or adapted without a written license.

For licensing, commissions, or private systems: **reyhanehdaneshdoost@gmail.com**