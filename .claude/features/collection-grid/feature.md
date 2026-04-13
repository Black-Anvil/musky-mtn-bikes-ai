# Collection Grid

## Brief

New product grid section modeled basically after the existing one (main-collection-product-grid.liquid), but with a customized layout.  The images in the reference > feature-screenshots folder shows

1. the staggered grid with a unique product image outline shape
2. the outline color change on hover
3. the second product image revealed on hover
4. a progressive elliptical reveal of the second image on hover
5. a two-column layout on mobile

There should be a section setting to allow for a certain number of product cards to be displayed, along with paginated navigation buttons.


---

## Scoping Questions

Generated: 2026-04-05
Chosen approach: A — Fully custom section + custom card snippet

### Q1: Collection Source — How should the section get its products?

This section could pull products from a collection picker (most flexible for the store owner) or be tied to the current collection page template.

- [ ] a) **Collection picker setting** — store owner selects a collection in the customizer. Section can be added to any page (homepage, landing pages, etc.)
- [ ] b) **Current collection** — section uses `collection.products` from the current page. Only works on collection template pages.
- [x] c) **Both** — collection picker with a fallback to the current page's collection if none is selected

**Notes:**


### Q2: Pagination Style — How should navigation between pages work?

The brief mentions "paginated navigation buttons."

- [x] a) **Standard pagination** — numbered page links (1, 2, 3...) with prev/next arrows, full page reload. Matches the existing theme pattern.
- [ ] b) **Prev/Next only** — simple previous and next buttons, no page numbers. Cleaner look, full page reload.
- [ ] c) **Load More button** — a single button that appends more products via AJAX (no page reload). More modern feel but requires JS.
- [ ] d) **Infinite scroll** — products load automatically as the user scrolls down. Requires JS.

I'd recommend (a) or (b) for simplicity — we can reuse the existing `pagination` snippet for (a). If you want (c) or (d), that adds JS complexity.

**Notes:**


### Q3: Card Border Shape — The concave corner cutout

Looking at the screenshots closely, the bottom-right corner has a concave curved cutout. I see two implementation approaches:

- [x] a) **CSS clip-path** — define the shape with a polygon/path. Clean, scalable, no extra assets. Slight limitation: the visible border line would need to be done via a pseudo-element or SVG overlay since clip-path cuts the border too.
- [ ] b) **SVG border** — use an SVG as the card outline. Most precise control over the border shape and color changes on hover. Slightly more complex markup.

I'm leaning toward (a) with a pseudo-element for the border, but (b) gives pixel-perfect control if the shape needs to match the reference exactly. What matters more — exact shape fidelity or simpler code?

**Notes:**


### Q4: Accent Color — Should it be configurable?

The reference shows a neon yellow/green accent color used for the hover border and the price text.

- [x] a) **Single section setting** — one color picker for the accent color, applied to both hover border and price
- [ ] b) **Use the theme's color scheme** — pull from the existing color scheme variables (less control but stays consistent with theme)
- [ ] c) **Two separate settings** — one for hover border color, one for price color (more flexibility)

I'd recommend (a) — one accent color setting keeps it simple and matches the reference where both use the same color.

**Notes:**


### Q5: Card Border — Default (non-hover) state

In the reference, the non-hovered cards have a thin light gray/white outline. On hover, it changes to the accent color.

- [ ] a) **Configurable border color** — a color picker for the default border, separate from the accent
- [x] b) **Derived from color scheme** — use a low-opacity version of the foreground color (e.g., `rgba(var(--color-foreground), 0.2)`) to match the theme automatically
- [ ] c) **Hardcoded subtle gray** — simple, matches the dark background in the reference

I'd recommend (b) — it adapts automatically if the section is placed on different background colors.

**Notes:**


### Q6: Products Per Page — How many products to show?

- [x] a) **Range slider** — store owner picks a number (e.g., 4–24, step of 2 or 4)
- [ ] b) **Fixed default** — we pick a sensible default (e.g., 8) and that's it, no setting

I'd recommend (a) with a default of 8 based on the reference showing roughly 8 products per view.

**Notes:**


### Q7: Desktop Columns — How many columns on desktop?

The reference shows a 4-column layout on desktop.

- [x] a) **Fixed 4 columns** — matches the reference exactly, no setting needed
- [ ] b) **Configurable** — let the store owner choose (3 or 4 columns)

I'd recommend (a) — the staggered layout is designed around 4 columns and changing the count would affect the visual rhythm.

**Notes:**


### Q8: Mobile Layout

The reference shows a clean 2-column layout on mobile with no stagger offset.

- [x] a) **Fixed 2 columns on mobile** — matches the reference, no setting
- [ ] b) **Configurable** — let store owner choose 1 or 2 columns on mobile

I'd recommend (a) since the reference is clear about this.

**Notes:**


### Q9: Product Info Display

The reference shows product title and price below the card. Should we include anything else?

- [x] a) **Title + price only** — matches the reference exactly
- [ ] b) **Title + price + vendor** — optional vendor line (useful if selling multiple brands)
- [ ] c) **Title + price + "sold out" badge** — show availability status

I'd recommend (a) with a potential future enhancement for (c) if needed. Keep it minimal for now.

**Notes:**


### Q10: Hover Behavior on Mobile

The elliptical reveal and border color change are hover-based. On touch devices, hover doesn't really exist.

- [x] a) **No hover effects on mobile** — just show the primary image, tapping goes to product page
- [ ] b) **Tap-to-reveal** — first tap reveals the secondary image, second tap navigates. Can feel clunky.
- [ ] c) **Show secondary image by default on mobile** — always show the lifestyle image on mobile instead of the product shot

I'd recommend (a) — hover effects on mobile usually create more confusion than value. Clean tap-to-navigate.

**Notes:**

---

### Recommendations

Based on my analysis of the theme and the reference designs:

1. **New files needed:** `sections/section-collection-grid.liquid`, `snippets/card-collection-grid.liquid`, `assets/section-collection-grid.css`, `assets/collection-grid.js` (web component for hover animation).

2. **The concave corner shape** is the trickiest part visually. I'll prototype it during implementation and we can iterate on the exact curve. CSS clip-path gives us the shape; a pseudo-element or SVG overlay gives us the animated border.

3. **The elliptical reveal animation** (top-to-bottom) can be achieved with CSS `clip-path: ellipse()` transitioning on hover — no JS needed for the animation itself. JS is only needed if we want to control timing or add the secondary image loading logic.

4. **Stagger offset** is straightforward CSS — odd columns get a `margin-top` or `transform: translateY()`. We should define this as a CSS variable so it's easy to tune.

5. **The existing `pagination` snippet** can be reused directly if we go with standard pagination — no need to build custom pagination UI.

6. **Price rendering** — I'd recommend reusing the existing `price` snippet from the theme for consistent formatting (handles money formats, compare-at prices, etc.), just styled differently with the accent color.

## Extended Brief

Generated: 2026-04-05

### Chosen Approach

Fully custom section + custom card snippet (Approach A). Self-contained implementation with no modifications to existing theme components. The visual design is too different from the existing `card-product` snippet to justify reuse.

### Requirements

- Staggered 4-column product grid where alternating columns (2nd and 4th) are offset downward
- Custom card shape with a concave curved cutout on the bottom-right corner, implemented via CSS clip-path with a pseudo-element for the visible border
- Thin default border derived from the color scheme (`rgba(var(--color-foreground), 0.2)`); transitions to a configurable accent color on hover
- Secondary product image revealed on hover via an elliptical clip-path animation expanding from top to bottom
- Product info displayed below the card: product title + price (price rendered in the accent color)
- Collection source: section setting collection picker with fallback to the current page's collection object if none is selected
- Configurable products-per-page count with standard numbered pagination (reuses existing theme `pagination` snippet)
- Fixed 4 columns on desktop, fixed 2 columns on mobile with no stagger offset
- No hover effects on mobile — tap navigates directly to the product page
- Standard section settings: color scheme, padding top/bottom

### Where It Lives

New standalone section, available on any page via the theme customizer. Not a replacement for the existing `main-collection-product-grid` — this is an alternative presentation.

**New files:**
- `sections/section-collection-grid.liquid` — markup, schema, asset loading
- `snippets/card-collection-grid.liquid` — custom card component
- `assets/section-collection-grid.css` — all styles (grid, card shape, hover states, responsive)
- `assets/collection-grid.js` — web component for hover interaction (secondary image reveal)

### Data Sources

- **Products:** from the selected collection (section setting) or the current page's `collection` object as fallback
- **Primary image:** `product.featured_media`
- **Secondary image:** `product.media[1]` (second media item)
- **Price:** reuse existing theme `price` snippet, styled with the accent color via CSS

### User Interaction

- **Desktop hover:** border transitions to accent color + secondary image reveals via elliptical clip-path animation (top-to-bottom)
- **Desktop click:** navigates to product page
- **Mobile tap:** navigates directly to product page (no hover effects)
- **Pagination:** standard page navigation with numbered links and prev/next arrows

### Customizer Settings

- **Collection** — collection picker (type: `collection`)
- **Products per page** — range slider (min: 4, max: 24, step: 4, default: 8)
- **Accent color** — color picker for hover border and price text
- **Color scheme** — standard theme color scheme selector
- **Padding top** — range (0–100, step 4, default 36)
- **Padding bottom** — range (0–100, step 4, default 36)

### Decisions Made

1. **Custom card snippet over reusing `card-product`** — the existing snippet is 624 lines with badges, quick-add, bulk ordering, ratings, and volume pricing. The reference design uses none of these. A clean 50-line card snippet is more maintainable than overriding 95% of the existing one.
2. **CSS clip-path over SVG border** — cleaner implementation, no extra assets, scales well. The border line will be handled via a pseudo-element that matches the clip-path shape.
3. **Single accent color setting** — used for both hover border and price text. Keeps settings minimal while matching the reference where both elements share the same color.
4. **Default border derived from color scheme** — `rgba(var(--color-foreground), 0.2)` adapts automatically to different section backgrounds without a separate setting.
5. **Fixed column counts** — 4 desktop / 2 mobile. The staggered layout is designed around 4 columns; making it configurable would compromise the visual rhythm.
6. **No hover effects on mobile** — avoids the awkward tap-to-reveal-then-tap-again-to-navigate pattern.
7. **Standard pagination** — reuses the existing theme `pagination` snippet for consistency and zero additional code.

### Edge Cases to Handle

- **No secondary image** — card shows only the primary image; hover still triggers accent border but no image reveal animation
- **No images at all** — show a placeholder (theme's default placeholder SVG)
- **Empty collection** — display an empty state message
- **Fewer than 4 products** — grid works normally, just fewer items; stagger still applies to whichever columns exist
- **Sold out products** — still displayed in the grid (no badge or visual distinction per scope)
- **Very long product titles** — truncate or clamp to prevent layout breakage
- **Collection picker not set AND not on a collection page** — section shows nothing or an editor-friendly message

### Out of Scope

- Filtering and sorting (not in the reference design)
- Quick add to cart functionality
- Sale/sold-out badges
- Vendor display
- Product ratings
- Any modification to existing theme files (`card-product`, `main-collection-product-grid`, `global.js`, etc.)

### Dependencies

- Existing theme `pagination` snippet — for paginated navigation
- Existing theme `price` snippet — for consistent money formatting
- Theme color scheme system — for `color-{{ scheme }}` class pattern

### Notes

- The concave corner clip-path shape will need visual iteration during implementation to match the reference curve precisely
- The elliptical reveal animation timing/easing should feel smooth and intentional — will prototype and adjust during build

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# feature.md — v1.0

# AI Shopify Developer Bootcamp

# by Coding with Jan

# https://codingwithjan.com

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
