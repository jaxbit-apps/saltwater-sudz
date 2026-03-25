# Gallery Masonry Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the placeholder before/after gallery with a masonry grid of real vehicle photos and a click-to-enlarge lightbox.

**Architecture:** Single-file static HTML site. All CSS is inline in `<style>`, all JS inline in `<script>`. No build tools, no frameworks. Images served from `docs/images/`. Gallery uses CSS `column-count` masonry with a vanilla JS lightbox overlay.

**Tech Stack:** HTML, CSS (columns masonry), vanilla JS, `sips` (macOS) for image optimization.

**Spec:** `docs/superpowers/specs/2026-03-25-gallery-masonry-redesign.md`

**Ordering:** Tasks must be executed in order. Task 1 must complete before Task 3 because the `<img>` width/height attributes depend on the actual output dimensions from image optimization.

---

### Task 1: Optimize and Copy Images

**Files:**
- Source: `Images/*.png`, `Images/*.PNG`, `Images/*.jpeg` (13 files)
- Create: `docs/images/gallery-01.jpg` through `docs/images/gallery-13.jpg`

All portrait source images are 1206x2622 PNG (~8-10 MB each). All landscape source images are 3000-6000px wide JPEG (~3-7 MB each). We need to resize and compress for web.

- [ ] **Step 1: Resize and convert portrait images to JPEG**

Portrait images (1206x2622) are fine at current width for retina. Resize to max 800px wide (plenty for a masonry column ~400px on desktop at 2x). Convert to JPEG quality 80.

```bash
cd /Users/jackcauthen/developer/webdev/studio/saltwater-sudz

# Gallery ordering: mix portraits and landscapes for visual balance
# gallery-01: BMW X5 white (portrait) - IMG_6466
sips -s format jpeg -s formatOptions 80 --resampleWidth 800 "Images/IMG_6466 - Ryan Haag.PNG" --out docs/images/gallery-01.jpg

# gallery-02: Exterior detail action shot (landscape) - Exterior Full package.jpeg
sips -s format jpeg -s formatOptions 80 --resampleWidth 1200 "Images/Exterior Full package.jpeg" --out docs/images/gallery-02.jpg

# gallery-03: Black truck (portrait) - IMG_7518
sips -s format jpeg -s formatOptions 80 --resampleWidth 800 "Images/IMG_7518 - Ryan Haag.png" --out docs/images/gallery-03.jpg

# gallery-04: Porsche 911 Turbo (portrait) - IMG_6467
sips -s format jpeg -s formatOptions 80 --resampleWidth 800 "Images/IMG_6467 - Ryan Haag.PNG" --out docs/images/gallery-04.jpg

# gallery-05: Interior detail action shot (landscape) - Interior.jpeg
sips -s format jpeg -s formatOptions 80 --resampleWidth 1200 "Images/Interior.jpeg" --out docs/images/gallery-05.jpg

# gallery-06: Dark BMW X3 (portrait) - IMG_7520
sips -s format jpeg -s formatOptions 80 --resampleWidth 800 "Images/IMG_7520 - Ryan Haag.png" --out docs/images/gallery-06.jpg

# gallery-07: Classic VW Jetta (portrait) - IMG_7519
sips -s format jpeg -s formatOptions 80 --resampleWidth 800 "Images/IMG_7519 - Ryan Haag.png" --out docs/images/gallery-07.jpg

# gallery-08: Exterior detail shot (landscape) - exterior.jpeg
sips -s format jpeg -s formatOptions 80 --resampleWidth 1200 "Images/exterior.jpeg" --out docs/images/gallery-08.jpg

# gallery-09: Ford interior clean (portrait) - IMG_7522
sips -s format jpeg -s formatOptions 80 --resampleWidth 800 "Images/IMG_7522 - Ryan Haag.png" --out docs/images/gallery-09.jpg

# gallery-10: IMG_6468 vehicle shot (portrait)
sips -s format jpeg -s formatOptions 80 --resampleWidth 800 "Images/IMG_6468 - Ryan Haag.PNG" --out docs/images/gallery-10.jpg

# gallery-11: IMG_7521 vehicle shot (portrait)
sips -s format jpeg -s formatOptions 80 --resampleWidth 800 "Images/IMG_7521 - Ryan Haag.png" --out docs/images/gallery-11.jpg

# gallery-12: IMG_7523 vehicle shot (portrait)
sips -s format jpeg -s formatOptions 80 --resampleWidth 800 "Images/IMG_7523 - Ryan Haag.png" --out docs/images/gallery-12.jpg

# gallery-13: IMG_7524 vehicle shot (portrait)
sips -s format jpeg -s formatOptions 80 --resampleWidth 800 "Images/IMG_7524 - Ryan Haag.png" --out docs/images/gallery-13.jpg
```

- [ ] **Step 2: Verify output sizes and get dimensions**

```bash
ls -lh docs/images/gallery-*.jpg
# Expected: each file under 500 KB

for f in docs/images/gallery-*.jpg; do
  echo "$(basename $f): $(sips -g pixelWidth -g pixelHeight "$f" 2>/dev/null | grep pixel | tr '\n' ' ')"
done
# Expected portrait: 800x1738
# Expected landscape: 1200x~800 (varies by aspect ratio)
```

Record the exact width and height for each image — these are needed for the `<img>` tags in Task 3.

- [ ] **Step 3: Commit images**

```bash
git add docs/images/gallery-*.jpg
git commit -m "Add optimized gallery images (13 photos, JPEG q80)"
```

---

### Task 2: Update Gallery CSS

**Files:**
- Modify: `docs/gallery.html` (lines 92-108 old CSS, lines 151-160 responsive overrides)

- [ ] **Step 1: Remove old gallery CSS**

In `docs/gallery.html`, delete these CSS rules (lines 92-108):

```css
/* GALLERY GRID */
.gallery-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:var(--space-lg)}
.gallery-item{border-radius:var(--radius-lg);overflow:hidden;box-shadow:var(--card-shadow);transition:transform 0.35s cubic-bezier(0.25,0.46,0.45,0.94),box-shadow 0.35s ease}
.gallery-item:hover{transform:translateY(-4px);box-shadow:var(--card-shadow-hover)}
.gallery-pair{display:grid;grid-template-columns:1fr 1fr;gap:0}
.gallery-before,.gallery-after{height:220px;display:flex;align-items:center;justify-content:center;position:relative}
.gallery-before{background:linear-gradient(135deg,#4A6274,#2D3436)}
.gallery-after{background:linear-gradient(135deg,var(--cta-bg),#00A5CF)}
.gallery-label{
  font-family:var(--font-heading);font-size:0.72rem;font-weight:600;
  letter-spacing:0.15em;text-transform:uppercase;color:#FFFFFF;
  background:rgba(0,0,0,0.3);backdrop-filter:blur(8px);
  padding:6px 14px;border-radius:var(--radius-sm);
}
.gallery-caption{padding:var(--space-md);background:var(--bg-secondary)}
.gallery-caption h3{font-family:var(--font-heading);font-size:0.95rem;font-weight:600;color:var(--text-primary);margin-bottom:4px}
.gallery-caption p{font-size:0.8rem;color:var(--text-muted);line-height:1.4}
```

- [ ] **Step 2: Add masonry CSS in the same location**

Replace the removed CSS with:

```css
/* MASONRY GALLERY */
.gallery-masonry{columns:3;column-gap:var(--space-md)}
.gallery-masonry__item{break-inside:avoid;margin-bottom:var(--space-md);border-radius:var(--radius-lg);overflow:hidden;box-shadow:var(--card-shadow);cursor:pointer;transition:transform 0.35s cubic-bezier(0.25,0.46,0.45,0.94),box-shadow 0.35s ease}
.gallery-masonry__item:hover{transform:translateY(-4px);box-shadow:var(--card-shadow-hover)}
.gallery-masonry__item img{width:100%;height:auto;display:block}
```

- [ ] **Step 3: Add lightbox CSS after the masonry CSS**

```css
/* LIGHTBOX */
.lightbox{display:none;position:fixed;inset:0;background:rgba(0,0,0,0.9);z-index:2000;align-items:center;justify-content:center;cursor:pointer}
.lightbox.active{display:flex}
.lightbox img{max-width:90vw;max-height:90vh;object-fit:contain;border-radius:var(--radius-md);cursor:default}
.lightbox__close{position:absolute;top:var(--space-md);right:var(--space-md);color:#fff;font-size:2rem;cursor:pointer;background:none;border:none;line-height:1;padding:8px}
.lightbox__close:hover{opacity:0.7}
```

- [ ] **Step 4: Update responsive breakpoints**

In the `@media(max-width:1024px)` rule (line 151), replace **only** `.gallery-grid{grid-template-columns:repeat(2,1fr)}` with `.gallery-masonry{columns:2}`. **Preserve** the `.footer-grid{grid-template-columns:1fr 1fr}` rule on the same line — do not delete it. The result should be:

```css
@media(max-width:1024px){.gallery-masonry{columns:2}.footer-grid{grid-template-columns:1fr 1fr}}
```

In the `@media(max-width:768px)` rule (lines 152-160), replace:
```css
.gallery-grid{grid-template-columns:1fr}
.gallery-before,.gallery-after{height:180px}
```
with:
```css
.gallery-masonry{columns:1}
```

- [ ] **Step 5: Verify no old gallery class references remain in CSS**

Search the `<style>` block for any remaining `.gallery-grid`, `.gallery-item`, `.gallery-pair`, `.gallery-before`, `.gallery-after`, `.gallery-label`, `.gallery-caption` references. There should be none.

---

### Task 3: Update Gallery HTML

**Files:**
- Modify: `docs/gallery.html` (lines 6-17 meta tags, line 205 subtitle, lines 162-176 JSON-LD, lines 219-331 gallery section)

- [ ] **Step 1: Update meta tags**

Line 6 — change `<title>`:
```html
<title>Gallery | Saltwater Sudz LLC - Mobile Detailing Portfolio Bluffton SC</title>
```

Line 7 — change `<meta description>`:
```html
<meta name="description" content="See real results from Saltwater Sudz mobile car detailing in Bluffton, SC. Exterior, interior, and full detail work across the Lowcountry.">
```

Line 10 — change OG title:
```html
<meta property="og:title" content="Gallery | Saltwater Sudz LLC - Mobile Detailing Portfolio Bluffton SC">
```

Line 11 — change OG description:
```html
<meta property="og:description" content="See real results from Saltwater Sudz mobile car detailing in Bluffton, SC. Exterior, interior, and full detail work.">
```

Line 15 — change Twitter title:
```html
<meta name="twitter:title" content="Gallery | Saltwater Sudz LLC - Detailing Portfolio">
```

Line 16 — change Twitter description:
```html
<meta name="twitter:description" content="See real results from Saltwater Sudz mobile car detailing in Bluffton, SC.">
```

- [ ] **Step 2: Update JSON-LD schema**

Lines 162-176 — change the `description` field:
```json
"description": "Portfolio of mobile car detailing work by Saltwater Sudz in Bluffton, SC."
```

- [ ] **Step 3: Update hero subtitle**

Line 205 — change to:
```html
<p class="hero-subtitle">Real results from Saltwater Sudz customers across the Lowcountry. Every vehicle gets meticulous attention to detail.</p>
```

- [ ] **Step 4: Replace gallery section HTML**

Replace lines 219-331 (the entire `<section class="section">` containing the gallery grid) with the masonry gallery. Use the actual `width` and `height` values measured in Task 1 Step 2.

```html
<section class="section">
  <div class="wide-width">
    <p style="font-family:var(--font-heading);font-size:0.72rem;font-weight:600;letter-spacing:0.2em;text-transform:uppercase;color:var(--cta-bg);text-align:center;margin-bottom:var(--space-sm);" class="reveal">Portfolio</p>
    <h2 style="font-family:var(--font-heading);font-size:clamp(1.6rem,3vw,2.441rem);font-weight:700;line-height:1.15;color:var(--text-primary);text-align:center;margin-bottom:var(--space-xl);text-wrap:balance;" class="reveal">Recent Work</h2>

    <div class="gallery-masonry">
      <div class="gallery-masonry__item reveal"><img src="images/gallery-01.jpg" alt="White BMW X5 freshly detailed in Bluffton SC driveway" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal reveal-d1"><img src="images/gallery-02.jpg" alt="Exterior detail in progress - brush cleaning BMW grille" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal reveal-d2"><img src="images/gallery-03.jpg" alt="Black truck detailed by Saltwater Sudz" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal reveal-d3"><img src="images/gallery-04.jpg" alt="Black Porsche 911 Turbo freshly detailed" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal"><img src="images/gallery-05.jpg" alt="Interior detailing - dashboard cleaning and conditioning" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal reveal-d1"><img src="images/gallery-06.jpg" alt="Dark BMW X3 detailed in driveway" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal reveal-d2"><img src="images/gallery-07.jpg" alt="Classic VW Jetta after full exterior detail" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal reveal-d3"><img src="images/gallery-08.jpg" alt="Exterior vehicle detail - full wash and finish" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal"><img src="images/gallery-09.jpg" alt="Ford truck interior fully cleaned and conditioned" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal reveal-d1"><img src="images/gallery-10.jpg" alt="Vehicle freshly detailed by Saltwater Sudz" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal reveal-d2"><img src="images/gallery-11.jpg" alt="Detailed vehicle - Saltwater Sudz mobile detailing" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal reveal-d3"><img src="images/gallery-12.jpg" alt="Vehicle detail completed - Lowcountry SC" width="WIDTH" height="HEIGHT" loading="lazy"></div>
      <div class="gallery-masonry__item reveal"><img src="images/gallery-13.jpg" alt="Mobile detailing results - Saltwater Sudz Bluffton" width="WIDTH" height="HEIGHT" loading="lazy"></div>
    </div>
  </div>
</section>
```

**Important:** Replace every `WIDTH` and `HEIGHT` placeholder with the actual pixel dimensions from Task 1 Step 2. Portrait images will be `800` x `1738` (approximately). Landscape images will be `1200` x their proportional height.

---

### Task 4: Add Lightbox HTML and JS

**Files:**
- Modify: `docs/gallery.html` (add lightbox div before `</body>`, update `<script>` block)

- [ ] **Step 1: Add lightbox HTML**

Insert this just before the `<script>` tag (which is currently at line 385):

```html
<!-- LIGHTBOX -->
<div class="lightbox" id="lightbox" role="dialog" aria-modal="true" aria-label="Image viewer">
  <button class="lightbox__close" aria-label="Close">&times;</button>
  <img src="" alt="">
</div>
```

- [ ] **Step 2: Add lightbox JS to the script block**

Add this code to the existing `<script>` block, after the IntersectionObserver code:

```javascript
// Lightbox
var lightbox=document.getElementById('lightbox');
var lightboxImg=lightbox.querySelector('img');
var lightboxClose=lightbox.querySelector('.lightbox__close');
var triggerEl=null;
document.querySelectorAll('.gallery-masonry__item img').forEach(function(img){
  img.addEventListener('click',function(){
    triggerEl=this;
    lightboxImg.src=this.src;
    lightboxImg.alt=this.alt;
    lightbox.classList.add('active');
    document.body.style.overflow='hidden';
  });
});
function closeLightbox(){
  lightbox.classList.remove('active');
  document.body.style.overflow='';
  if(triggerEl){triggerEl.focus();triggerEl=null;}
}
lightbox.addEventListener('click',function(e){if(e.target===lightbox)closeLightbox()});
lightboxClose.addEventListener('click',closeLightbox);
document.addEventListener('keydown',function(e){if(e.key==='Escape'&&lightbox.classList.contains('active'))closeLightbox()});
```

- [ ] **Step 3: Verify the complete script block**

The full `<script>` block should contain (in order):
1. Nav toggle click handler
2. Scroll handler for header
3. IntersectionObserver for `.reveal` elements
4. Lightbox code (just added)

---

### Task 5: Sync Design File and Commit

**Files:**
- Create: `design/approved/gallery.html` (overwrite with contents of `docs/gallery.html`)

- [ ] **Step 1: Copy docs/gallery.html to design/approved/gallery.html**

```bash
cp docs/gallery.html design/approved/gallery.html
```

- [ ] **Step 2: Verify both files are identical**

```bash
diff docs/gallery.html design/approved/gallery.html
# Expected: no output (files are identical)
```

- [ ] **Step 3: Commit all HTML changes**

```bash
git add docs/gallery.html design/approved/gallery.html
git commit -m "Replace before/after gallery with masonry grid and lightbox

- CSS columns masonry layout (3/2/1 columns responsive)
- 13 real vehicle photos with lazy loading
- Click-to-enlarge lightbox with Escape/click-to-close
- Updated meta tags and hero copy to remove before/after language
- Accessible lightbox with ARIA attrs and focus management"
```

---

### Task 6: Visual Verification

- [ ] **Step 1: Open the gallery page in a browser**

```bash
open docs/gallery.html
```

Verify:
- All 13 images load and display in a 3-column masonry grid
- Portrait images are tall, landscape images span full column width
- Hover effect (slight lift + shadow) works on each image
- Scroll reveal animations fire as images come into view
- Clicking an image opens the lightbox overlay with the full image
- Clicking the X, clicking the dark overlay, or pressing Escape closes the lightbox
- Resize browser to tablet width (~900px): grid becomes 2 columns
- Resize to mobile width (~500px): grid becomes 1 column
- Hero text reads "Our Work" / "Real results from Saltwater Sudz..."
- CTA banner and footer are unchanged

- [ ] **Step 2: Check page weight**

```bash
du -sh docs/images/gallery-*.jpg | tail -1
# Expected: total under 7 MB

wc -c docs/gallery.html
# Expected: HTML file size reasonable (under 20 KB)
```
