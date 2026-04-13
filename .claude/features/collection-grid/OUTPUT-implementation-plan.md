# Implementation Plan: Collection Grid

Generated: 2026-04-05
Feature spec: `.claude/features/collection-grid/feature.md`

## Summary

A custom staggered product grid section with a distinctive visual style: 4-column layout with alternating column offsets, custom concave-corner card shapes via CSS clip-path, accent-colored hover borders, and a secondary image reveal via elliptical clip-path animation (top-to-bottom). Products are sourced from a collection picker with fallback to the current page's collection. Includes standard pagination and a minimal product info display (title + accent-colored price).

## Human-First Breakdown

### Admin Setup (human tasks in Shopify admin)

No admin setup required — this feature works entirely with theme code and section settings.

**Recommendation:** Products should have at least 2 images (primary product shot + lifestyle/secondary) to take full advantage of the hover reveal effect. Products with only 1 image will still work — they just won't get the reveal animation.

### Code Preparation (before any visitor touches the page)

1. Create the section file with a schema that includes: collection picker, products per page range, accent color picker, color scheme, and padding settings
2. Create a card snippet that renders each product — a card wrapper with the concave-corner clip-path shape, the product's primary image inside, and the secondary image positioned on top (hidden by default)
3. Create CSS for the staggered 4-column grid — all cards the same size, columns 2 and 4 pushed down with a top margin
4. Style the card border as a pseudo-element matching the clip-path shape — thin, low-opacity foreground color by default
5. Style the product info area below each card — product title as a link, price rendered in the accent color
6. Create a web component that handles the hover interaction — on mouseenter, triggers the elliptical clip-path animation on the secondary image (expanding from top to bottom); on mouseleave, reverses it
7. Wire up pagination using the existing `pagination` snippet inside a `paginate` block

### Live Behavior (when a user interacts)

1. Page loads — the grid appears with products from the selected collection (or the current page's collection). Primary product images are visible. The stagger offset is visible on desktop (columns 2 and 4 shifted down).
2. On desktop, user hovers over a card — the border transitions from the subtle default color to the accent color
3. If the product has a secondary image, it's revealed via an elliptical clip-path that expands from the top edge downward, creating a smooth reveal effect
4. User moves the mouse away — the accent border fades back to the default color, and the secondary image clip-path animates back to hidden (ellipse shrinks back up)
5. User clicks anywhere on the card — navigates to the product page
6. On mobile — the grid collapses to 2 columns with no stagger offset. No hover effects. Tapping a card navigates directly to the product page.
7. If there are more products than the "products per page" setting — pagination links appear below the grid. Clicking a page number reloads with the next set.

## Files

### New Files
- `sections/section-collection-grid.liquid` — section markup, schema, asset loading, paginate block
- `snippets/card-collection-grid.liquid` — custom card component (clip-path shape, dual images, product info)
- `assets/section-collection-grid.css` — all styles (grid layout, stagger, card shape, hover states, responsive)
- `assets/collection-grid.js` — web component for hover interaction (secondary image elliptical reveal)

### Modified Files
None — fully self-contained.

### Theme Components Reused
- `snippets/pagination.liquid` — standard paginated navigation with numbered links and prev/next arrows
- `snippets/price.liquid` — consistent money formatting (handles currency codes, sale prices, price ranges)
- `component-price.css` — price styling (loaded in the section file)
- `component-pagination.css` — pagination styling (loaded automatically by the pagination snippet)
- `.page-width` wrapper — standard content container from the theme
- `color-{{ scheme }} gradient` — standard color scheme class pattern

## Build Steps

### Step 1: Section Skeleton + Schema

**Do:** Create the section file with the full schema, asset loading, and basic wrapper markup. No card rendering yet — just the outer structure with a placeholder message so we can verify the section appears in the customizer.

**Files:** `sections/section-collection-grid.liquid`

**Details:**
- Load CSS: `section-collection-grid.css` via `stylesheet_tag`
- Load CSS: `component-price.css` via `stylesheet_tag`
- Load JS: `collection-grid.js` with `defer="defer"`
- Dynamic style block for section padding using `times: 0.75` mobile multiplier (theme convention)
- Set `--accent-color` CSS variable from `section.settings.accent_color`
- Outer wrapper: `color-{{ section.settings.color_scheme }} gradient`
- Inner wrapper: `page-width section-{{ section.id }}-padding`
- Schema settings in this order:
  1. `collection` — type: `collection`
  2. `products_per_page` — type: `range`, min: 4, max: 24, step: 4, default: 8
  3. Header: "Colors"
  4. `accent_color` — type: `color`, default: `#CCFF00` (neon yellow-green from reference)
  5. `color_scheme` — type: `color_scheme`, default: `scheme-1`
  6. Header: "Section padding"
  7. `padding_top` — type: `range`, min: 0, max: 100, step: 4, default: 36
  8. `padding_bottom` — type: `range`, min: 0, max: 100, step: 4, default: 36
- Preset: `{ "name": "Collection Grid" }`
- Inside the wrapper, add temporary text: "Collection Grid section — cards coming next"

**Verify:** Add the section to a page in the theme customizer. It should appear with the placeholder text. Settings should all be visible and functional (collection picker, accent color, padding sliders).

### Step 2: Empty CSS + JS Files

**Do:** Create the CSS and JS files with basic structure so the section loads without errors.

**Files:** `assets/section-collection-grid.css`, `assets/collection-grid.js`

**Details:**
- CSS: Add a comment header and the root `.collection-grid` class as empty placeholder
- JS: Set up the web component shell — `collection-grid-card` custom element with `connectedCallback` and `disconnectedCallback` stubs, wrapped in the `if (!customElements.get(...))` guard

**Verify:** Section still loads without console errors. No visual changes yet.

### Step 3: Card Snippet + Grid Loop

**Do:** Create the card snippet with the product image, product title, and price. Wire it into the section with the paginate block and product loop.

**Files:** `snippets/card-collection-grid.liquid`, `sections/section-collection-grid.liquid`

**Details:**
- Snippet accepts: `card_product`, `accent_color`, `section_id`, `lazy_load`
- Outer wrapper: `<collection-grid-card>` custom element with class `collection-grid-card`
- Inside: `<a href="{{ card_product.url }}">` wrapping the full card for navigation
- Card media container (`.collection-grid-card__media`) with primary image using responsive `srcset` (follow the pattern from `card-product.liquid` — widths: 165, 360, 533, 720, 940, 1066)
- If `card_product.media[1]` exists, render the secondary image inside the same media container, with class `.collection-grid-card__media-secondary`
- Below the media: product info container (`.collection-grid-card__info`) with:
  - Product title: `{{ card_product.title | escape }}`
  - Price: `{% render 'price', product: card_product, price_class: 'collection-grid-card__price', show_compare_at_price: true %}`
- Section file updates:
  - Replace placeholder text with collection logic: `assign grid_collection = section.settings.collection | default: collection`
  - Wrap in `{%- paginate grid_collection.products by section.settings.products_per_page -%}`
  - Product loop rendering `card-collection-grid` snippet for each product
  - Grid wrapper: `<ul class="collection-grid">` with `<li class="collection-grid__item">` for each product
  - After the grid: `{% render 'pagination', paginate: paginate %}`
  - Empty state: if no products, show a message

**Verify:** Products from the selected collection render as simple rectangular cards with images, titles, and prices. Pagination appears if needed. Selecting a different collection in the customizer updates the grid.

### Step 4: Grid Layout — Staggered 4-Column

**Do:** Style the grid as a 4-column layout on desktop with alternating column stagger. 2 columns on mobile with no stagger.

**Files:** `assets/section-collection-grid.css`

**Details:**
- `.collection-grid`: `display: grid`, `grid-template-columns: repeat(4, 1fr)`, `gap: 2rem` (tune to reference)
- Stagger offset via CSS variable: `--stagger-offset: 60px` (tune to reference)
- Apply offset to even columns: `.collection-grid__item:nth-child(4n+2), .collection-grid__item:nth-child(4n+4)` get `margin-top: var(--stagger-offset)`
- Reset list styles: `list-style: none`, `padding: 0`, `margin: 0`
- Card images: `width: 100%`, `aspect-ratio: 1 / 1`, `object-fit: cover`, `display: block`
- Product info: flexbox row, `justify-content: space-between`, title and price side by side
- Mobile (`max-width: 749px`): `grid-template-columns: repeat(2, 1fr)`, stagger offset to `0`, reduce gap
- Tablet (`min-width: 750px` and `max-width: 989px`): keep 2 columns, no stagger (safe default)

**Verify:** Desktop shows a 4-column staggered grid. Mobile/tablet shows a clean 2-column grid with no offset. Products are evenly sized. Spacing looks reasonable.

### Step 5: Card Shape — Concave Corner Clip-Path

**Do:** Apply the concave corner cutout shape to cards using CSS clip-path, and add the visible border via a pseudo-element.

**Files:** `assets/section-collection-grid.css`

**Details:**
- The clip-path shape on `.collection-grid-card__media`: a rectangle with a concave curve scooped out of the bottom-right corner. Use `clip-path: path()` with an SVG-style path string. The curve starts roughly 65% along the bottom edge and curves up to about 75% of the right edge height.
- For the visible border, use a layered pseudo-element approach:
  - `.collection-grid-card__media::before` — positioned absolute, same dimensions, same clip-path but slightly larger (inset: -1px or -2px), background color is the border color
  - The actual media content clips on top, leaving the pseudo-element's edge visible as a "border"
  - Default border color: `rgba(var(--color-foreground), 0.2)`
- Apply `position: relative` and `overflow: visible` to the media container
- Both primary and secondary images need to be clipped to the same shape

**Verify:** Cards show the concave corner cutout shape. A thin, subtle border outlines the card media area. The shape approximates the reference screenshots.

### Step 6: Hover — Accent Border

**Do:** Add the hover state that transitions the card border to the accent color.

**Files:** `assets/section-collection-grid.css`

**Details:**
- On `.collection-grid-card:hover .collection-grid-card__media::before`, change the background color to `var(--accent-color)`
- Add `transition: background-color 0.3s ease` to the `::before` pseudo-element
- Wrap hover styles in `@media (hover: hover)` to prevent sticky hover on touch devices
- The `--accent-color` variable is set on the section wrapper from the dynamic style block

**Verify:** Hovering a card on desktop transitions the border from subtle default to accent color. Moving away transitions back. No hover effects on touch devices.

### Step 7: Hover — Secondary Image Elliptical Reveal

**Do:** Implement the secondary image reveal animation using an elliptical clip-path that expands from the top downward on hover.

**Files:** `assets/section-collection-grid.css`, `assets/collection-grid.js`

**Details:**
- CSS:
  - `.collection-grid-card__media-secondary`: positioned absolute over the primary image, full width/height, `object-fit: cover`
  - Default state: `clip-path: ellipse(0% 0% at 50% 0%)` — fully hidden, zero-size ellipse at top center
  - Hover state (via `.collection-grid-card:hover .collection-grid-card__media-secondary`): `clip-path: ellipse(150% 100% at 50% 50%)` — oversized ellipse covering the entire image
  - Transition: `transition: clip-path 0.6s ease-in-out`
  - Wrap in `@media (hover: hover)`
- JS web component (`collection-grid-card`):
  - In `connectedCallback`: find the secondary image element; if it doesn't exist, return early (no-op)
  - The CSS handles the animation via `:hover` — the JS component is mainly structural (the custom element wrapper) and could handle future enhancements
  - In `disconnectedCallback`: clean up any bound listeners

**Verify:** Hovering a card with a secondary image shows the elliptical reveal animation expanding from the top. Animation reverses on mouse leave. Cards without a secondary image just show the border color change. Smooth performance, no jank.

### Step 8: Product Info Styling

**Do:** Polish the product title and price display below each card.

**Files:** `assets/section-collection-grid.css`

**Details:**
- `.collection-grid-card__info`: `display: flex`, `justify-content: space-between`, `align-items: baseline`, `padding-top: 1rem`, `gap: 1rem`
- Product title: inherit font from theme, appropriate size, `color: rgb(var(--color-foreground))`, no text decoration
- Truncate long titles: `overflow: hidden`, `text-overflow: ellipsis`, `white-space: nowrap`, with `min-width: 0` on the flex child
- Price text: `color: var(--accent-color)` applied to `.collection-grid-card__price .price-item`
- The `<a>` wrapper covering the whole card: `text-decoration: none`, `color: inherit`, `display: block`
- Ensure the price snippet's default styles don't conflict — override specificity as needed

**Verify:** Product info looks clean below each card. Title is readable, price is in the accent color. Long titles truncate. Layout matches the reference.

### Step 9: Empty States + Edge Cases

**Do:** Handle edge cases — empty collection, no images, collection not selected on non-collection pages.

**Files:** `sections/section-collection-grid.liquid`, `snippets/card-collection-grid.liquid`

**Details:**
- **Empty collection / no products:** Show a centered message within the grid area. Use a translation-friendly approach (hardcode for now with a note to add translation keys if the theme goes multilingual).
- **No collection available:** If `section.settings.collection` is blank AND `collection` object doesn't exist:
  - In the theme editor (`request.design_mode`): show a placeholder message: "Select a collection in the section settings"
  - On the storefront: render nothing (hide the section)
- **Product has no image:** Render placeholder SVG via `{{ 'product-apparel-2' | placeholder_svg_tag: 'placeholder-svg' }}`. No secondary image reveal.
- **Product has only 1 image:** Card renders normally with primary image. Hover still triggers accent border, but no image reveal.

**Verify:** Test with an empty collection — message appears. Test with a product that has no images — placeholder renders. Test without selecting a collection on a non-collection page — appropriate behavior in editor vs. storefront.

### Step 10: Final Polish

**Do:** Review all styles against the reference screenshots, tune spacing, stagger offset, animation timing, and clip-path curve. Ensure consistency with theme conventions.

**Files:** `assets/section-collection-grid.css`, potentially minor tweaks to other files

**Details:**
- Compare side-by-side with reference screenshots
- Tune `--stagger-offset` value to match the visual rhythm in the reference
- Tune the concave corner curve in the clip-path to match the reference shape
- Tune animation timing (duration, easing) for the elliptical reveal
- Verify pagination styling integrates well with the section's color scheme
- Check the accent color default (`#CCFF00`) renders correctly
- Ensure spacing feels right at various viewport widths
- Tablet breakpoint (750px–989px): verify 2 columns looks good
- Check that the full-card link is accessible (focus outline visible, meaningful aria-label)

**Verify:** Section matches the reference screenshots closely on desktop and mobile. All interactions feel smooth. No visual regressions at different viewport widths.

## Risks & Considerations

- **Clip-path border trick:** Getting a visible "border" on a clip-path shape is the trickiest CSS challenge. The pseudo-element approach (slightly larger background behind the clipped content) is the most reliable method. If it proves problematic, we can fall back to an SVG overlay approach.
- **Clip-path browser support:** `clip-path: path()` and `clip-path: ellipse()` have excellent modern browser support. No IE11 support, which is fine for a modern Shopify store.
- **Performance:** CSS clip-path transitions are GPU-accelerated in modern browsers, so animation performance should be good. Lazy loading secondary images prevents unnecessary network requests.
- **Pagination on non-collection pages:** Shopify's `paginate` tag works with any collection object via the collection picker. Pagination URLs use query parameters (`?page=2`), which is standard.
- **Color scheme interaction:** The accent color is a fixed color picker, not tied to the color scheme. If the store owner changes the section's color scheme, they may need to update the accent color to maintain contrast. This is intentional — it's a design choice.

## Open Questions

None — all decisions resolved during scoping.
