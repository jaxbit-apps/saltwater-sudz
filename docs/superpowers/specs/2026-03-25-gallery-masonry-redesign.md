# Gallery Masonry Redesign

## Summary

Replace the current before/after placeholder gallery with a CSS masonry grid of real portrait-orientation vehicle photos, plus a click-to-enlarge lightbox. No external dependencies.

## Motivation

- Current gallery has 6 before/after placeholder items with gradient blocks and no real images
- Ryan's actual photos are portrait-oriented single shots (not before/after pairs)
- A masonry grid is the standard portfolio pattern for mixed/portrait images and lets visitors see more work at once

## Design

### Hero Section

Keep the existing hero structure. Update copy:

- **Tagline**: "See the Results" (unchanged)
- **Title**: "Our Work" (unchanged)
- **Subtitle**: Changed from "Before and after transformations from real Saltwater Sudz customers..." to "Real results from Saltwater Sudz customers across the Lowcountry. Every vehicle gets meticulous attention to detail."
- **Meta tags**: Update `<title>`, `<meta description>`, OG tags, and Twitter tags to remove "Before & After" language
- **Schema.org JSON-LD**: Update description to remove "Before and after" phrasing

### Masonry Grid

Replaces the entire `.gallery-grid` section (lines 219-331 of current `docs/gallery.html`).

**Remove old CSS**: Delete all `.gallery-grid`, `.gallery-item`, `.gallery-pair`, `.gallery-before`, `.gallery-after`, `.gallery-label`, `.gallery-caption` rules (lines 92-108) and their responsive overrides.

**CSS approach**: Uses `column-count` / `columns` property.

```css
.gallery-masonry {
  columns: 3;
  column-gap: var(--space-md);
}
.gallery-masonry__item {
  break-inside: avoid;
  margin-bottom: var(--space-md);
  border-radius: var(--radius-lg);
  overflow: hidden;
  box-shadow: var(--card-shadow);
  cursor: pointer;
  transition: transform 0.35s cubic-bezier(0.25,0.46,0.45,0.94), box-shadow 0.35s ease;
}
.gallery-masonry__item:hover {
  transform: translateY(-4px);
  box-shadow: var(--card-shadow-hover);
}
.gallery-masonry__item img {
  width: 100%;
  height: auto;
  display: block;
}
```

**Responsive breakpoints** (matching existing site patterns):

- Desktop (>1024px): 3 columns
- Tablet (768-1024px): 2 columns
- Mobile (<768px): 1 column

**No captions** on grid items. The photos speak for themselves.

**Scroll reveal**: Each item gets the existing `.reveal` class with staggered delays (`.reveal-d1`, `.reveal-d2`, `.reveal-d3` cycling).

**Column ordering note**: CSS `column-count` fills top-to-bottom per column, not left-to-right by row. This is standard masonry behavior and acceptable for a portfolio gallery where images have no sequential relationship.

### Section Header

Replace the "Before & After / Transformations" header text:

- **Eyebrow**: "Portfolio"
- **Heading**: "Recent Work"
- **Subheading**: Remove the placeholder instruction paragraph entirely

### Images (13 total)

Copied from `Images/` to `docs/images/` for serving.

**Image optimization (required)**: The source PNGs are 8-10 MB each (~114 MB total), which is unusable on mobile. During the copy step, convert all images to optimized JPEG at quality 80 using `sips` (macOS built-in). Target: each image under 500 KB. This brings the total page weight to a reasonable ~5-7 MB.

**Naming convention**: Rename to web-friendly filenames preserving the correct extension: `gallery-01.jpg`, `gallery-02.jpg`, etc. All output files will be `.jpg` since we're converting to JPEG.

**Image ordering**: Implementer's discretion to balance portrait and landscape images visually across columns.

| Source filename | Description |
|---|---|
| IMG_7518 - Ryan Haag.png | Black truck with business number (portrait) |
| IMG_7519 - Ryan Haag.png | Classic VW Jetta (portrait) |
| IMG_7520 - Ryan Haag.png | Dark BMW X3 in driveway (portrait) |
| IMG_7521 - Ryan Haag.png | Vehicle detail shot (portrait) |
| IMG_7522 - Ryan Haag.png | Ford interior - clean (portrait) |
| IMG_7523 - Ryan Haag.png | Vehicle detail shot (portrait) |
| IMG_7524 - Ryan Haag.png | Vehicle detail shot (portrait) |
| IMG_6466 - Ryan Haag.PNG | White BMW X5 in driveway (portrait) |
| IMG_6467 - Ryan Haag.PNG | Black Porsche 911 Turbo (portrait) |
| IMG_6468 - Ryan Haag.PNG | Vehicle detail shot (portrait) |
| Exterior Full package.jpeg | Exterior detailing action shot - BMW grille (landscape) |
| exterior.jpeg | Exterior detail shot (landscape) |
| Interior.jpeg | Interior detailing action shot (landscape) |

Each `<img>` tag gets:
- Descriptive `alt` attribute
- `loading="lazy"`
- Explicit `width` and `height` attributes (measured from the optimized files) to prevent layout shift (CLS)

### Lightbox

Vanilla JS, no dependencies. Behavior:

- **Open**: Click any gallery image -> overlay appears with full-size image
- **Overlay**: `position: fixed`, `inset: 0`, `background: rgba(0,0,0,0.9)`, `z-index: 2000`
- **Image**: Centered via flexbox, `max-width: 90vw`, `max-height: 90vh`, `object-fit: contain`. No `loading="lazy"` on the lightbox image (it's shown on demand).
- **Close**: Click anywhere on overlay, press Escape key, or click visible X button in top-right corner
- **No prev/next navigation** - keeps it simple
- **Body scroll lock**: `overflow: hidden` on `<body>` while lightbox is open
- **Accessibility**: Lightbox div gets `role="dialog"`, `aria-modal="true"`, `aria-label="Image viewer"`. Focus returns to the triggering image on close.
- **DOM placement**: Lightbox `<div>` placed just before `</body>`, outside all sections, to avoid stacking context issues.

```css
.lightbox {
  display: none;
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.9);
  z-index: 2000;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}
.lightbox.active {
  display: flex;
}
.lightbox img {
  max-width: 90vw;
  max-height: 90vh;
  object-fit: contain;
  border-radius: var(--radius-md);
}
.lightbox__close {
  position: absolute;
  top: var(--space-md);
  right: var(--space-md);
  color: #fff;
  font-size: 2rem;
  cursor: pointer;
  background: none;
  border: none;
  line-height: 1;
}
```

**Lightbox CSS** goes in the existing `<style>` block. **Lightbox JS** (~25 lines) goes in the existing `<script>` block.

JS behavior:
- Add click listeners to all `.gallery-masonry__item img` elements
- On click: store reference to triggering element, set lightbox image `src`, add `.active` class, lock body scroll
- On overlay click, X button click, or Escape: remove `.active`, unlock body scroll, return focus to triggering element

## Unchanged Sections

- CTA banner section ("Want These Results for Your Vehicle?") and surrounding waves - no changes
- Footer - no changes
- Header/nav - no changes

## Files Changed

1. **`docs/gallery.html`** - Rewrite gallery section: remove old before/after CSS/HTML, add masonry CSS/HTML, add lightbox CSS/HTML/JS, update hero copy and meta tags
2. **`design/approved/gallery.html`** - Make identical to the updated `docs/gallery.html`
3. **`docs/images/`** - Copy, optimize (JPEG quality 80 via `sips`), and rename 13 images from `Images/`

## Out of Scope

- Filtering/categories
- Prev/next navigation in lightbox
- Ryan's headshot (separate task)
- Full focus trapping in lightbox (basic accessibility included; full keyboard trap deferred)
