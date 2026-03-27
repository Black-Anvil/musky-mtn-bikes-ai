# Theme Analysis — Dawn (Shopify)

Generated: 2026-03-26

## Summary

This is a **Shopify Dawn-based theme** — a modern, well-structured theme with 54 sections, 37 snippets, and 185 asset files. The codebase is clean and follows Shopify best practices consistently. Customization level is moderate — the core Dawn structure is intact with selective additions for B2B/bulk ordering and custom content. All user-facing text uses translation keys (`| t`), and the theme follows a web-components-first JavaScript architecture.

## File Structure Overview

```
assets/          185 files (87 SVG icons, 65 CSS, 32 JS, 1 GIF)
config/          2 files (settings_data.json, settings_schema.json)
layout/          2 files (theme.liquid, password.liquid)
locales/         51 files (25 languages + 25 schema files + 1 extra)
sections/        54 files (52 .liquid + 2 .json section groups)
snippets/        37 files (all .liquid)
templates/       20 files (13 JSON + 1 .liquid + 6 in customers/)
```

**Total: ~351 theme files.**

No `blocks/` directory — this theme does not use theme blocks (app blocks are handled via `@app` block type in sections).

**Notable custom files:**
- `sections/custom-liquid.liquid` — allows arbitrary Liquid/HTML
- `sections/bulk-quick-order-list.liquid` — B2B bulk ordering
- `sections/quick-order-list.liquid` — quick order functionality
- `sections/apps.liquid` — block-based app integration
- Section groups: `header-group.json`, `footer-group.json`

**Asset naming convention:**
- `component-*.css` (44 files) — reusable UI component styles
- `section-*.css` (12 files) — section-specific styles
- `template-*.css` (2 files) — page template styles
- `icon-*.svg` (87+ files) — icon library

## CSS Conventions

### Grid System

**Flexbox-based** with calculated percentage widths. No CSS Grid.

- Base class: `.grid` (`display: flex; flex-wrap: wrap`)
- Items: `.grid__item` with width calculations using CSS variables
- Column variants: `.grid--1-col`, `.grid--2-col`, `.grid--3-col`, `.grid--4-col`, `.grid--5-col-desktop`, `.grid--6-col-desktop`
- Tablet variants: `.grid--2-col-tablet`, `.grid--3-col-tablet`, `.grid--4-col-tablet`
- Mobile overrides: `.grid--1-col-tablet-down`, `.grid--2-col-tablet-down`

**Column width calculation:**
```css
.grid--3-col .grid__item {
  width: calc(33.33% - var(--grid-mobile-horizontal-spacing) * 2 / 3);
}
```

**Grid gap variables:**
- `--grid-mobile-horizontal-spacing`
- `--grid-mobile-vertical-spacing`
- `--grid-desktop-horizontal-spacing`
- `--grid-desktop-vertical-spacing`

### Breakpoints

Three main breakpoints used consistently:

| Name | Query | Use |
|------|-------|-----|
| Mobile | `max-width: 749px` | Phones, small screens |
| Tablet | `min-width: 750px` and `max-width: 989px` | Mid-size screens |
| Desktop | `min-width: 990px` | Large screens |

All queries use `screen and (min-width:)` or `screen and (max-width:)` syntax.

### Color Variables

Colors stored as **RGB values** and used with `rgb()` / `rgba()`:

```css
color: rgb(var(--color-foreground));
background-color: rgba(var(--color-foreground), 0.04);
```

**Base color variables:**
- `--color-foreground` — primary text
- `--color-background` — page background
- `--color-link` — link color
- `--color-button` — button background
- `--color-button-text` — button text
- `--color-secondary-button` / `--color-secondary-button-text`
- `--color-shadow` — shadow color

**Alpha variables:**
- `--alpha-button-background: 1`
- `--alpha-button-border: 1`
- `--alpha-link: 0.85`
- `--alpha-badge-border: 0.1`

**Component-specific variables:**
- `.product-card-wrapper .card` uses `--product-card-*`
- `.collection-card-wrapper .card` uses `--collection-card-*`
- `.article-card-wrapper .card` uses `--blog-card-*`
- `.content-container` uses `--text-boxes-*`
- `.global-media-settings` uses `--media-*`

### Naming Convention

**BEM (Block Element Modifier)** used consistently:

```css
.card                    /* Block */
.card__inner             /* Element */
.card__media             /* Element */
.card__content           /* Element */
.card__heading           /* Element */
.card--horizontal        /* Modifier */
.card--card              /* Modifier */
.card--standard          /* Modifier */
```

**Utility classes:**
- Visibility: `.hidden`, `.visually-hidden`, `.overflow-hidden`
- Responsive: `.small-hide`, `.medium-hide`, `.large-up-hide`
- Text: `.center`, `.left`, `.right`, `.uppercase`, `.light`
- Spacing: `.page-margin`, `.element-margin-top`

### Spacing Patterns

CSS custom properties for section-level spacing:
- `--spacing-sections-mobile`
- `--spacing-sections-desktop`
- `--page-width-margin`

Hardcoded values follow a consistent `rem` progression:
`0.5rem`, `0.6rem`, `1rem`, `1.5rem`, `2rem`, `2.5rem`, `3rem`, `4rem`, `4.5rem`, `5rem`, `7rem`

### Page Width / Containers

| Class | Max-width | Purpose |
|-------|-----------|---------|
| `.page-width` | `var(--page-width)` | Standard content container |
| `.page-width--narrow` | `72.6rem` (1160px) at 990px+ | Narrower content |
| `.page-width-desktop` | `var(--page-width)` at 990px+ | Desktop-only container |
| `.rte-width` | `82rem` (1312px) | Rich text content |

```css
.page-width {
  max-width: var(--page-width);
  margin: 0 auto;
  padding: 0 1.5rem;        /* Mobile */
}
@media screen and (min-width: 750px) {
  .page-width {
    padding: 0 5rem;         /* Tablet+ */
  }
}
```

## JavaScript Conventions

### Base Files (do not modify)

| File | Lines | Purpose |
|------|-------|---------|
| `constants.js` | 9 | `PUB_SUB_EVENTS` object and `ON_CHANGE_DEBOUNCE_TIMER` |
| `pubsub.js` | 25 | Pub/sub system: `subscribe()`, `publish()` |
| `global.js` | 1,332 | Core utilities + 11 custom elements |
| `animations.js` | 102 | Scroll-triggered animations via IntersectionObserver |
| `details-disclosure.js` | 53 | Collapsible details components |
| `details-modal.js` | 47 | Modal dialog implementation |
| `search-form.js` | 47 | Base search functionality |

**Feature-specific files (loaded conditionally):**
- `cart.js` (297 lines) — cart management
- `cart-drawer.js` (136 lines) — cart drawer UI
- `product-form.js` (142 lines) — product form submission
- `product-info.js` (429 lines) — variant selection, info swapping
- `facets.js` (365 lines) — filtering with history.pushState
- `quick-add.js` (122 lines) — quick add to cart modal

### Existing Components

**29 custom elements** registered across the codebase:

**global.js (11):**
`quantity-input`, `menu-drawer`, `header-drawer`, `modal-dialog`, `bulk-modal`, `modal-opener`, `deferred-media`, `slider-component`, `slideshow-component`, `variant-selects`, `product-recommendations`

**Other asset files (17):**
`cart-remove-button`, `cart-items`, `cart-drawer`, `cart-drawer-items`, `cart-notification`, `details-disclosure`, `header-menu`, `details-modal`, `facet-filters-form`, `price-range`, `facet-remove`, `main-search`, `password-modal`, `predictive-search`, `search-form`, `quick-add-modal`, `account-icon`

**Inline in sections (1):**
`sticky-header` (defined in `sections/header.liquid`)

### Event Patterns

**Pub/Sub system** (pubsub.js):
- `subscribe(eventName, callback)` — returns unsubscriber function
- `publish(eventName, data)` — returns `Promise.all()`

**Published events** (constants.js):
- `'cart-update'` — cart changed
- `'quantity-update'` — quantity changed
- `'option-value-selection-change'` — variant option changed
- `'variant-change'` — variant selected
- `'cart-error'` — cart operation failed

**DOM CustomEvents:**
- `product-info:loaded` — product info element ready
- `preventHeaderReveal` — prevents sticky header reveal
- `modalClosed` — dispatched on `document.body`

**Usage pattern:**
```js
// Subscribe in connectedCallback
this.unsubscriber = subscribe(PUB_SUB_EVENTS.optionValueSelectionChange, this.handleChange.bind(this));
// Unsubscribe in disconnectedCallback
this.unsubscriber();
```

### Third-Party Libraries

**None.** All code is vanilla JS. Only Shopify-native APIs are used:
- `window.Shopify.PaymentButton.init()`
- `Shopify.CountryProvinceSelector()`
- `window.ProductModel.loadShopifyXR()`

### Script Loading

All scripts use `defer="defer"`:
```html
<script src="{{ 'global.js' | asset_url }}" defer="defer"></script>
```

Core scripts loaded in `layout/theme.liquid`. Feature scripts loaded conditionally in their sections.

One exception: `sticky-header` is defined inline in `sections/header.liquid` using `{% javascript %}` block.

## Liquid Conventions

### Section Wrapper Pattern

Every section follows a **3-layer wrapper**:

```liquid
<div class="color-{{ section.settings.color_scheme }} gradient">
  <div class="page-width section-{{ section.id }}-padding">
    <!-- Content -->
  </div>
</div>
```

1. **Outer div** — color scheme + gradient
2. **Middle div** — page-width container + section-specific padding
3. **Content** — section-specific markup

Some sections add `isolate` class to prevent z-index stacking issues.

### Section Padding Approach

**All sections use the same responsive padding pattern:**

```liquid
{%- style -%}
  .section-{{ section.id }}-padding {
    padding-top: {{ section.settings.padding_top | times: 0.75 | round: 0 }}px;
    padding-bottom: {{ section.settings.padding_bottom | times: 0.75 | round: 0 }}px;
  }
  @media screen and (min-width: 750px) {
    .section-{{ section.id }}-padding {
      padding-top: {{ section.settings.padding_top }}px;
      padding-bottom: {{ section.settings.padding_bottom }}px;
    }
  }
{%- endstyle -%}
```

**Important:** Mobile gets **75% of the desktop value** (multiplied by `0.75`). The CLAUDE.md template shows `0.5` — **always use `0.75` to match the theme's actual convention**.

### Standard Schema Settings

Settings that appear across most/all sections:

1. `color_scheme` — type: `color_scheme`, default: `"scheme-1"`
2. `padding_top` — type: `range`, step: 4, unit: "px"
3. `padding_bottom` — type: `range`, step: 4, unit: "px"

### Section Structure

Common section file anatomy:

```liquid
{%- comment -%} 1. Asset loading {%- endcomment -%}
{{ 'section-name.css' | asset_url | stylesheet_tag }}
{{ 'component-name.css' | asset_url | stylesheet_tag }}

{%- comment -%} 2. Dynamic styles {%- endcomment -%}
{%- style -%}
  .section-{{ section.id }}-padding { ... }
{%- endstyle -%}

{%- comment -%} 3. Markup {%- endcomment -%}
<div class="color-{{ section.settings.color_scheme }} gradient">
  <div class="page-width section-{{ section.id }}-padding">
    ...
  </div>
</div>

{%- comment -%} 4. Conditional JS loading {%- endcomment -%}
<script src="{{ 'feature.js' | asset_url }}" defer="defer"></script>

{%- comment -%} 5. Schema {%- endcomment -%}
{% schema %} ... {% endschema %}
```

### Snippet Patterns

**Most reused snippets:**
- `card-product` — used in 6+ sections
- `card-collection` — collection cards
- `product-media-gallery` — product media
- `price` — product pricing
- `product-variant-options` / `product-variant-picker`
- `icon-accordion`

**Naming convention:** hyphen-separated, lowercase, prefixed by component type (`card-`, `component-`, `snippet-`)

**Render pattern:**
```liquid
{% render 'card-product',
  card_product: product,
  media_aspect_ratio: section.settings.image_ratio,
  show_secondary_image: section.settings.show_secondary_image
%}
```

### Translation Approach

**100% translation coverage.** No hardcoded English text found. All user-facing strings use `| t` filter:

```liquid
{{ 'sections.featured_collection.view_all' | t }}
aria-label="{{ 'general.slider.previous_slide' | t }}"
```

Schema labels use `t:` prefix:
```json
"label": "t:sections.featured-collection.settings.title.label"
```

### Block Patterns

**Common block types across sections:**
- `heading` — large text display (often limited to 3)
- `text` — body/descriptive text
- `caption` — smaller styled text
- `button` — CTA buttons (often limited to 2)
- `column` — multicolumn containers
- `image` — image display
- `@app` — app integration blocks

**Block rendering pattern:**
```liquid
{%- for block in section.blocks -%}
  {%- case block.type -%}
    {%- when 'heading' -%}
      <h2 {{ block.shopify_attributes }}>{{ block.settings.heading }}</h2>
    {%- when 'text' -%}
      <div {{ block.shopify_attributes }}>{{ block.settings.text }}</div>
  {%- endcase -%}
{%- endfor -%}
```

`{{ block.shopify_attributes }}` is **always included** for theme editor functionality.

## Schema Conventions

### Common Settings

Every section typically includes:

| Setting | Type | Default | Notes |
|---------|------|---------|-------|
| `color_scheme` | `color_scheme` | `"scheme-1"` | Applied as `color-{{ value }}` CSS class |
| `padding_top` | `range` | 36 or 40 | Step: 4, unit: px |
| `padding_bottom` | `range` | 36 or 52 | Step: 4, unit: px |

**Schema organization order:** primary input > content/text > layout > design/color > mobile settings > spacing (padding always last).

Headers (`"type": "header"`) separate each group within the schema.

### Color Scheme Handling

Applied via CSS class binding at the wrapper level:
```liquid
<div class="color-{{ section.settings.color_scheme }} gradient">
```

Some sections support multiple color schemes (e.g., header has `color_scheme` + `menu_color_scheme`). Some have `container_color_scheme` for nested elements.

### Padding / Spacing Approach

**Standard range settings:**
- Content sections: min 0, max 100, step 4, unit "px"
- Defaults vary: 36px, 40px, or 52px depending on section

**Responsive scaling:** Mobile gets 75% of the setting value via `| times: 0.75`.

**Between-section spacing** handled via CSS variables:
```css
.section + .section {
  margin-top: var(--spacing-sections-desktop);
}
```

### Preset Patterns

Single preset per section, with default block content when applicable:
```json
"presets": [{
  "name": "t:sections.multicolumn.presets.name",
  "blocks": [
    { "type": "column" },
    { "type": "column" },
    { "type": "column" }
  ]
}]
```

## Asset Loading Patterns

### CSS Loading

**Standard (render-critical):**
```liquid
{{ 'section-footer.css' | asset_url | stylesheet_tag }}
```

**Deferred (non-critical, used in header):**
```liquid
<link rel="stylesheet" href="{{ 'component-list-menu.css' | asset_url }}" media="print" onload="this.media='all'">
```

**Conditional:**
```liquid
{%- if section.settings.image_shape == 'blob' -%}
  {{ 'mask-blobs.css' | asset_url | stylesheet_tag }}
{%- endif -%}
```

### JS Loading

All scripts use `defer="defer"`. No `type="module"` usage.

### Image Loading

- First section images: `fetchpriority="high"`
- Below-fold images: `loading: 'lazy'`
- Responsive widths: `'375, 550, 750, 1100, 1500, 1780, 2000, 3000, 3840'`

## Visual Analysis

No reference images found in `.claude/context/reference/`. Upload screenshots or design files there for visual analysis in future sessions.

## Recommendations

1. **Padding multiplier mismatch:** The CLAUDE.md template shows `times: 0.5` for mobile padding, but the actual theme uses `times: 0.75`. **Always use 0.75** to match the theme's convention.

2. **Section spacing consistency:** Between-section spacing is handled via CSS custom properties (`--spacing-sections-mobile`, `--spacing-sections-desktop`). New sections inherit this automatically when using the `section` class.

3. **Color scheme pattern is mandatory:** Every new section should include a `color_scheme` setting and apply it via `color-{{ section.settings.color_scheme }} gradient` on the outer wrapper.

4. **No blocks/ directory:** This theme doesn't use Shopify's newer theme blocks system. All block logic lives inside section schemas.

5. **Pub/sub is the cross-component communication standard.** Use `subscribe()` / `publish()` from pubsub.js rather than custom DOM events for component-to-component communication.

6. **B2B features present:** The theme includes bulk ordering features (`bulk-quick-order-list`, `bulk-modal`, `bulk-add`). Be aware these exist when working on cart or product-related features.

7. **No external dependencies.** The theme is 100% vanilla JS. Any new library needs explicit approval and must be added to `assets/` directly.
