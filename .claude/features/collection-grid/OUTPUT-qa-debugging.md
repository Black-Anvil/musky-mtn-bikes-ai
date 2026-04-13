# QA: Collection Grid

Feature spec: `.claude/features/collection-grid/feature.md`
Implementation plan: `.claude/features/collection-grid/OUTPUT-implementation-plan.md`

## How to Use This File

1. Run through the checklist after implementation — check items that pass, add notes on items that fail
2. Tell AI to read this file — it will fix the issues and reset the checklist for another round
3. Repeat until everything passes
4. Previous rounds are kept as a log below the current round

## Current Round (Round 1)

Status: **Passed**

### Accepted trade-offs
- Hover reveal uses `clip-path: inset()` with `opacity` masking to hide curve deformation at animation start/end — works well but is a workaround, not a pixel-perfect solution
- Mobile tab notch uses a separate SVG clipPath/border from desktop (two paths to maintain)
- Price strips `.00` via Liquid `replace` — works for whole-dollar prices but prices like `$19.50` would keep their decimals (acceptable for this store's pricing)

### Core behavior
- [x] Section appears in the theme customizer under "Collection Grid"
- [x] Selecting a collection in the section settings populates the grid with products
- [x] Products display with their primary image, title, and price
- [x] Grid shows 4 columns on desktop with staggered layout (columns 2 and 4 offset downward)
- [x] Grid shows 2 columns on mobile with no stagger
- [x] Pagination appears when products exceed the "products per page" setting
- [x] Clicking a pagination link loads the correct page of products
- [x] Clicking a product card navigates to the product page

### Card shape
- [x] Cards have the concave curved cutout on the bottom-right corner
- [x] A thin, subtle border outlines the card shape in the default state
- [x] The concave curve shape looks close to the reference screenshots

### Hover effects (desktop only)
- [x] Hovering a card transitions the border to the accent color
- [x] Moving away transitions the border back to the default color
- [x] Hovering a card with a secondary image triggers the elliptical reveal animation (top-to-bottom)
- [x] The elliptical reveal animation reverses smoothly on mouse leave
- [x] Cards with only one image show the border color change but no image reveal
- [x] Hover effects do NOT trigger on mobile/touch devices

### Styling
- [x] Product title is readable and truncates gracefully for long names
- [x] Price displays in the accent color
- [x] Title and price are positioned correctly (title left, price right)
- [x] The accent color setting in the customizer updates both the hover border and the price color
- [x] The color scheme setting changes the section background/text appropriately
- [x] Padding top/bottom settings adjust spacing correctly

### Collection source
- [x] Section works with a selected collection on any page (homepage, landing page)
- [x] Section falls back to the current collection on collection template pages when no collection is selected
- [x] When no collection is available (not selected, not on collection page), section handles gracefully

### Edge cases
- [x] Empty collection shows an appropriate message
- [x] Products without any images show a placeholder
- [x] Products with only 1 image render correctly (no broken hover state)
- [x] Grid handles fewer than 4 products without layout issues

### Responsive
- [x] Desktop (990px+): 4-column staggered grid
- [x] Tablet (750px–989px): 2-column grid, no stagger
- [x] Mobile (below 750px): 2-column grid, no stagger
- [x] Spacing and card sizes feel appropriate at all breakpoints

### Accessibility
- [x] Cards are keyboard-navigable (can tab to each card link)
- [x] Focus outline is visible on card links
- [x] Images have meaningful alt text
- [x] Pagination is keyboard accessible

## Previous Rounds

[Empty — no previous rounds yet.]
