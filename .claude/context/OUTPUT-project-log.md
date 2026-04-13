[Do not write this manually. AI maintains this file automatically after every meaningful action.]

This is the project's persistent memory. AI reads it at the start of every session to restore context. It will contain:

- **Timestamped entries** — What was done and when
- **Decisions made** — What was decided and why, including alternatives that were considered
- **Architectural choices** — Technical decisions that affect the project long-term
- **Risks & open questions** — Things to watch out for in future development

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# OUTPUT-project-log.md — v1.0

# AI Shopify Developer Bootcamp

# by Coding with Jan

# https://codingwithjan.com

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 2026-03-26 — Theme Onboarding

**Action:** Ran initial theme analysis via `/onboard-theme` skill. Output written to `.claude/context/OUTPUT-initial-theme-analysis.md`.

**Key findings:**
- Dawn-based theme, ~351 files, moderate customization level
- 29 web components, pub/sub event system, 100% translation coverage
- Flexbox grid, 3 breakpoints (749px / 750px / 990px), BEM naming
- Mobile padding uses `0.75` multiplier (not `0.5` as shown in CLAUDE.md template)
- B2B bulk ordering features present — watch for interactions with cart/product work
- No external JS dependencies, no blocks/ directory

**Decisions:** None yet — this is the project baseline.

**Risks/Open questions:**
- Shopify Dev MCP not yet configured — recommend connecting for Liquid validation

## 2026-04-05 — Collection Grid Feature: Scoped, Planned & Built

**Action:** Scoped, planned, and implemented the Collection Grid feature — a custom staggered product grid section.

**Files created:**
- `sections/section-collection-grid.liquid` — section with schema, SVG clipPath definition, paginate loop
- `snippets/card-collection-grid.liquid` — custom card with dual images, SVG border outline
- `assets/section-collection-grid.css` — grid layout, clip-path shape, hover effects, responsive
- `assets/collection-grid.js` — web component shell (hover animation is pure CSS)

**Key decisions:**
- **Approach A (custom section + custom card)** over reusing `card-product` snippet — the existing snippet is 624 lines with features this design doesn't use. Clean separation avoids conflicts.
- **SVG clipPath + SVG stroke border** instead of pure CSS clip-path — `clip-path: path()` doesn't support percentages/scaling. SVG `clipPathUnits="objectBoundingBox"` scales correctly. Border is an SVG `<path>` element outside the clipped container to avoid stroke clipping.
- **Pure CSS hover animation** over JS-driven — the elliptical clip-path reveal and border transition are both CSS-only via `@media (hover: hover)`. The web component is structural only. Better performance, simpler code.
- **Single accent color setting** for both hover border and price text — matches the reference design, keeps settings minimal.
- **Default border derived from color scheme** — `rgba(var(--color-foreground), 0.2)` adapts automatically to different backgrounds.

**Architecture notes:**
- The SVG `<clipPath>` definition (id: `collection-grid-clip`) is rendered once per section instance. If multiple instances exist on the same page, they share the same clip shape — this is correct behavior.
- Collection source uses `section.settings.collection | default: collection` for picker-with-fallback pattern.

**Status:** Built, QA Round 1 in progress.

**Risks:**
- The concave corner curve values may need visual tuning once previewed in the theme
- The `--stagger-offset: 60px` may need adjustment to match the reference screenshots exactly

## 2026-04-07 — Collection Grid: Shape & Reveal Revisions

**Action:** Revised card shape, reveal animation, and removed unused JS based on QA feedback.

**Changes:**
1. **Removed `collection-grid.js`** from section — hover animation is pure CSS, JS file was empty/unused
2. **Revised card shape** — the concave curve was too shallow. Updated to a deeper scoop (`C 0.75 0.75, 0.55 0.85, 0.55 1`) creating a more pronounced "file folder tab" notch at bottom-right where product info sits
3. **Revised reveal animation** — changed from `clip-path: ellipse()` (radiated from center point at top) to `clip-path: inset()` with rounded bottom corners (uniform horizontal wipe from top to bottom). More faithful to the reference design.
4. **Repositioned product info** — text now right-aligned, tucked into the tab notch area instead of spanning full width

**Decisions:**
- Kept `collection-grid.js` file in assets (not deleted) in case JS is needed later — just removed the `<script>` tag from the section
- Used `inset(0 0 100% 0 round 0 0 35% 35%)` for the reveal hidden state — the `35%` rounded bottom corners give the wipe a curved leading edge

## 2026-04-12 — Collection Grid: QA Complete

**Action:** Extensive visual refinement and QA pass. All checklist items passed.

**Changes since last entry:**
1. **Card shape finalized** — iterated through ~8 shape revisions. Final shape: desktop tab at 92% height, mobile tab at 90% (deeper/wider for text). Diagonal transition is a straight line with small radiused transitions at each end.
2. **Responsive tab notch** — separate SVG clipPaths and border SVGs for mobile vs desktop, swapped via CSS media queries at 990px. Mobile tab is wider (~50% from right) to accommodate title text.
3. **Reveal animation reworked** — `clip-path: inset()` with `round` using `vw` units for constant curve depth. Added `opacity` layer (0.15s fade-in, 0.25s fade-out with 0.3s delay) to mask curve deformation at small rectangle sizes. Asymmetric easing: `ease-out` on hover-in, `ease-in-out` on hover-out.
4. **Price added** — `$` prefix, no currency code, `.00` stripped. Accent color. Hidden on mobile. Title truncates with ellipsis when space is tight (flex layout, `min-width: 0`).
5. **Primary image** set to `object-fit: contain`, secondary stays `cover`.

**Key decisions:**
- **Two SVG clipPaths** (mobile + desktop) rather than one responsive shape — SVG clipPath values can't use media queries, so CSS swaps which `clip-path: url()` reference is active.
- **Opacity masking for clip-path deformation** — the `inset()` `round` values deform when the visible rectangle is smaller than the radius. Opacity hides those frames rather than trying to solve the underlying CSS limitation.
- **`vw` units for reveal curve radius** — keeps curve depth constant as the reveal animates, since viewport width doesn't change during the transition.

**Files changed:**
- `sections/section-collection-grid.liquid` — added mobile clipPath, refined desktop clipPath
- `snippets/card-collection-grid.liquid` — added price, dual border SVGs (mobile/desktop)
- `assets/section-collection-grid.css` — responsive clip-path swap, reveal animation, price styling, info positioning

**Status:** QA Round 1 passed. Feature complete pending commit.

**Risks:**
- Two separate SVG paths (mobile/desktop) means shape changes require updating both — document if handing off
- Price `.00` stripping uses simple Liquid `replace` — prices like `$19.50` keep decimals (fine for this store)
