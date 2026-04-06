# QA: Collection Grid

Feature spec: `.claude/features/collection-grid/feature.md`
Implementation plan: `.claude/features/collection-grid/OUTPUT-implementation-plan.md`

## How to Use This File

1. Run through the checklist after implementation — check items that pass, add notes on items that fail
2. Tell AI to read this file — it will fix the issues and reset the checklist for another round
3. Repeat until everything passes
4. Previous rounds are kept as a log below the current round

## Current Round (Round 1)

Status: Testing

### Core behavior
- [ ] Section appears in the theme customizer under "Collection Grid"
- [ ] Selecting a collection in the section settings populates the grid with products
- [ ] Products display with their primary image, title, and price
- [ ] Grid shows 4 columns on desktop with staggered layout (columns 2 and 4 offset downward)
- [ ] Grid shows 2 columns on mobile with no stagger
- [ ] Pagination appears when products exceed the "products per page" setting
- [ ] Clicking a pagination link loads the correct page of products
- [ ] Clicking a product card navigates to the product page

### Card shape
- [ ] Cards have the concave curved cutout on the bottom-right corner
- [ ] A thin, subtle border outlines the card shape in the default state
- [ ] The concave curve shape looks close to the reference screenshots

### Hover effects (desktop only)
- [ ] Hovering a card transitions the border to the accent color
- [ ] Moving away transitions the border back to the default color
- [ ] Hovering a card with a secondary image triggers the elliptical reveal animation (top-to-bottom)
- [ ] The elliptical reveal animation reverses smoothly on mouse leave
- [ ] Cards with only one image show the border color change but no image reveal
- [ ] Hover effects do NOT trigger on mobile/touch devices

### Styling
- [ ] Product title is readable and truncates gracefully for long names
- [ ] Price displays in the accent color
- [ ] Title and price are positioned correctly (title left, price right)
- [ ] The accent color setting in the customizer updates both the hover border and the price color
- [ ] The color scheme setting changes the section background/text appropriately
- [ ] Padding top/bottom settings adjust spacing correctly

### Collection source
- [ ] Section works with a selected collection on any page (homepage, landing page)
- [ ] Section falls back to the current collection on collection template pages when no collection is selected
- [ ] When no collection is available (not selected, not on collection page), section handles gracefully

### Edge cases
- [ ] Empty collection shows an appropriate message
- [ ] Products without any images show a placeholder
- [ ] Products with only 1 image render correctly (no broken hover state)
- [ ] Grid handles fewer than 4 products without layout issues

### Responsive
- [ ] Desktop (990px+): 4-column staggered grid
- [ ] Tablet (750px–989px): 2-column grid, no stagger
- [ ] Mobile (below 750px): 2-column grid, no stagger
- [ ] Spacing and card sizes feel appropriate at all breakpoints

### Accessibility
- [ ] Cards are keyboard-navigable (can tab to each card link)
- [ ] Focus outline is visible on card links
- [ ] Images have meaningful alt text
- [ ] Pagination is keyboard accessible

## Previous Rounds

[Empty — no previous rounds yet.]
