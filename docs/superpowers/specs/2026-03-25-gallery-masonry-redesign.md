# Gallery Masonry Redesign

## Summary

Replace the current before/after placeholder gallery with a CSS masonry grid of real portrait-orientation vehicle photos, plus a click-to-enlarge lightbox. No external dependencies.

## Motivation

- Current gallery has 6 before/after placeholder items with gradient blocks and no real images
- Ryan's actual photos are portrait-oriented single shots (not before/after pairs)
- A masonry grid is the standard portfolio pattern for mixed/portrait images and lets visitors see more work at once

## Design

### Hero Section

Keep the existing hero structure. Update copy only:

- **Tagline**: "See the Results" (unchanged)
- **Title**: "Our Work" (unchanged)
- **Subtitle**: "Real results from Saltwater Sudz customers across the Lowcountry. Every vehicle gets meticulous attention to detail."
- **Meta tags**: Update `<title>`, `<meta description>`, OG tags, and Twitter tags to remove "Before & After" language
- **Schema.org JSON-LD**: Update description to remove "Before and after" phrasing

### Masonry Grid

Replaces the entire `.gallery-grid` section (lines 219-331 of current `docs/gallery.html`).

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

### Section Header

Replace the "Before & After / Transformations" header text:

- **Eyebrow**: "Portfolio"
- **Heading**: "Recent Work"
- **Subheading**: Remove the placeholder instruction paragraph entirely

### Images (13 total)

Copied from `Images/` to `docs/images/` for serving:

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

Images will be renamed to web-friendly filenames when copied (e.g., `gallery-01.png`, `gallery-02.png`, etc.) to avoid spaces and special characters in URLs.

Each `<img>` tag gets a descriptive `alt` attribute and `loading="lazy"`.

### Lightbox

Vanilla JS, no dependencies. Behavior:

- **Open**: Click any gallery image -> overlay appears with full-size image
- **Overlay**: `position: fixed`, `inset: 0`, `background: rgba(0,0,0,0.9)`, `z-index: 2000`
- **Image**: Centered via flexbox, `max-width: 90vw`, `max-height: 90vh`, `object-fit: contain`
- **Close**: Click anywhere on overlay, or press Escape key
- **No prev/next navigation** - keeps it simple
- **Body scroll lock**: `overflow: hidden` on `<body>` while lightbox is open

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
```

JS (~20 lines, inline in existing `<script>` block):

- Add click listeners to all `.gallery-masonry__item img` elements
- On click: set lightbox image `src`, add `.active` class, lock body scroll
- On overlay click or Escape: remove `.active`, unlock body scroll

## Files Changed

1. **`docs/gallery.html`** - Rewrite gallery section CSS, HTML, and JS
2. **`design/approved/gallery.html`** - Mirror the same changes
3. **`docs/images/`** - Copy and rename 13 images from `Images/`

## Out of Scope

- Image optimization/compression (can be done later)
- Filtering/categories
- Prev/next navigation in lightbox
- Ryan's headshot (separate task)
