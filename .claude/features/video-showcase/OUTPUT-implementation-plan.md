# Implementation Plan: Video Showcase

Generated: 2026-04-13
Feature spec: `.claude/features/video-showcase/feature.md`

## Summary

A scroll-driven video scrubbing section where the video plays frame-by-frame as the user scrolls. The video pins to viewport center via `position: sticky` inside a tall scroll container (~3x viewport height). Highlight callouts with Apple-style line+dot indicators fade in/out at configured frame ranges, pointing to specific product features. Mobile replaces the video with stacked keyframe images and vertical indicators. Built as a self-contained web component with no modifications to existing theme files.

## Human-First Breakdown

### Admin Setup (human tasks in Shopify admin)

#### Prepare and upload the video file
The MP4 needs to be encoded with high keyframe density (GOP=1) for smooth frame-by-frame scrubbing. Standard videos will stutter when scrubbed. Once encoded, upload it to Shopify Files — the section needs the direct CDN URL, not the video picker.

- [ ] Encode the video with high keyframe density: `ffmpeg -i input.mov -c:v libx264 -g 1 -preset slow -crf 23 -an output.mp4`
- [ ] Go to Settings > Files in Shopify admin
- [ ] Upload the encoded MP4
- [ ] Copy the CDN URL (you'll paste this into the section's video URL setting)

#### Prepare mobile keyframe images
Mobile doesn't use the video — it shows static images representing key moments. Export up to 4 frames from the video that best represent the product highlights.

- [ ] Export 2-4 keyframe images from the video (PNG or JPG, high quality)
- [ ] Upload them to Shopify Files or have them ready to upload via the section's image picker

### Code Preparation (before any visitor touches the page)

1. Create the section file with a tall scroll container (~3x viewport height) — this provides the scroll distance without actual scroll hijacking
2. Place the video element inside a sticky container that pins to the viewport center as the user scrolls through the tall section
3. Add heading + subheading markup above the video area
4. Add tertiary heading + paragraph markup below the video area
5. Create highlight overlay containers positioned absolutely over the video — each has a text element, an SVG line, and a dot endpoint
6. Set all highlights to `opacity: 0` by default
7. Create the mobile layout: stacked full-width images, each with highlight text below and a vertical line+dot indicator pointing upward
8. Hide mobile layout on desktop, hide desktop layout on mobile

### Live Behavior — Desktop

1. User scrolls down the page
2. Section comes into view — heading and subheading fade in
3. Video's sticky container pins it to the viewport center as scrolling continues
4. The web component calculates scroll progress (0% to 100%) based on how far through the tall section the user has scrolled
5. Scroll progress maps directly to `video.currentTime` — at 0% progress the video is at frame 0, at 100% it's at the last frame
6. As progress passes each highlight's `start_percent`, that highlight fades in — text appears on its configured side, a line draws from the text toward the product, ending in a dot at the configured x/y position
7. As progress passes each highlight's `end_percent`, that highlight fades out
8. User reaches the bottom of the tall section — the video shows its final frame, the sticky container unpins, and normal scrolling resumes
9. Tertiary heading and paragraph reveal below the video area

### Live Behavior — Mobile

1. User scrolls down the page — heading and subheading appear
2. Full-width keyframe images scroll into view vertically (standard scroll, no trapping)
3. Each image has highlight text below it and a vertical line+dot indicator pointing up to the relevant feature in the image
4. Tertiary heading and paragraph appear below the last image

## Files

### New Files
- `sections/section-video-showcase.liquid` — section markup, schema, asset loading
- `assets/section-video-showcase.css` — all component styles (desktop + mobile)
- `assets/video-showcase.js` — `<video-showcase>` web component (scroll observation, video time control, highlight visibility)

### Modified Files
- None — fully self-contained

### Theme Components Reused
- `page-width` wrapper class for heading/text areas
- `color-{{ section.settings.color_scheme }} gradient` outer wrapper pattern
- `section-{{ section.id }}-padding` responsive padding pattern (0.75x mobile multiplier)
- Standard `h2` / `h3` / body text typography from the theme

## Build Steps

### Step 1: Section skeleton + scroll container structure

**Do:** Create the section file with the fundamental layout: a tall scroll container with a sticky video area inside it. No video yet — just the structural divs with background colors so we can verify the sticky behavior works.

**Files:** `sections/section-video-showcase.liquid`, `assets/section-video-showcase.css`

**Details:**
- Outer wrapper: `color-{{ section.settings.color_scheme }} gradient` pattern
- Tall scroll container: `height: 300vh` (3x viewport = scroll distance)
- Sticky video container: `position: sticky; top: 50%; transform: translateY(-50%)` to pin at viewport center
- Give the video container a 16:9 aspect ratio placeholder (e.g., `aspect-ratio: 16/9; max-width: 100%; background: #333`)
- Add minimal schema: just `color_scheme` and padding settings + an empty preset
- Load CSS via `stylesheet_tag`

**Verify:** Add the section to a page in the customizer. Scroll through it — the dark placeholder rectangle should stick to the viewport center while you scroll through ~3 viewport heights of space, then unpin and scroll away.

### Step 2: Video element + scroll-driven playback

**Do:** Add the `<video>` element and the `<video-showcase>` web component. The component calculates scroll progress through the section and sets `video.currentTime` accordingly.

**Files:** `sections/section-video-showcase.liquid`, `assets/video-showcase.js`

**Details:**
- Add `<video>` element inside the sticky container: `playsinline`, `muted`, `preload="auto"`, no `controls`, no `autoplay`
- Hardcode a test video URL for now (or use a text setting right away since it's just a URL field)
- Web component `<video-showcase>`:
  - `connectedCallback`: query the video element, the scroll container, set up a scroll listener
  - Scroll handler: calculate progress = (how far the scroll container's top has passed above viewport center) / (container height - viewport height). Clamp to 0–1.
  - Map progress to `video.currentTime = progress * video.duration` (only after `loadedmetadata` fires)
  - Use `requestAnimationFrame` to throttle scroll updates
  - `disconnectedCallback`: remove scroll listener
- Load JS with `defer="defer"`
- Wrap in `if (!customElements.get('video-showcase'))` guard

**Verify:** Paste a video URL into the setting, scroll through the section. The video should scrub smoothly forward and backward as you scroll.

### Step 3: Heading areas (above + below)

**Do:** Add the heading/subheading above the scroll container, and the tertiary heading + paragraph below it. Hardcode text for now.

**Files:** `sections/section-video-showcase.liquid`, `assets/section-video-showcase.css`

**Details:**
- Above: `<div class="page-width">` with `<h2>` for heading, `<p>` for subheading, centered text
- Below: `<div class="page-width">` with `<h3>` for tertiary heading, `<p>` for paragraph, centered text
- Heading fade-in: use CSS `opacity: 0; transform: translateY(20px)` by default, transition to visible. The web component adds a class when the section enters viewport (IntersectionObserver)
- Tertiary reveal: similar approach — add a class when scroll progress reaches ~95-100%, or use a second IntersectionObserver on the below area

**Verify:** Scroll the section into view — heading/subheading should fade in. Scroll through the video to the end — tertiary heading and paragraph should appear below.

### Step 4: Highlight overlays — markup + positioning

**Do:** Create the highlight overlay system. Each highlight is an absolutely positioned container over the video with a text element and an indicator dot. Hardcode 2-3 test highlights.

**Files:** `sections/section-video-showcase.liquid`, `assets/section-video-showcase.css`

**Details:**
- Highlight container: `position: absolute; inset: 0; pointer-events: none` overlaying the video
- Each highlight: `position: absolute` within the overlay
- Dot: small circle (`width: 12px; height: 12px; border-radius: 50%; background: currentColor`) positioned at `left: x%; top: y%`
- Text: positioned on the configured side — `left: 0` or `right: 0`, vertically aligned near the dot's y position, with max-width ~35% so it doesn't cover the video
- All highlights start at `opacity: 0; transition: opacity 0.4s ease`
- Hardcode 2-3 test positions to verify the positioning system works

**Verify:** Temporarily set a test highlight to `opacity: 1`. The dot should appear at the correct position over the video, and the text should be on the correct side.

### Step 5: Indicator line (connecting text to dot)

**Do:** Draw the line connecting each highlight's text area to its dot. This completes the Apple-style callout graphic.

**Files:** `assets/section-video-showcase.css`, `sections/section-video-showcase.liquid`

**Details:**
- Use an SVG `<line>` element within each highlight container — its coordinates go from the text edge to the dot center
- SVG takes up the full overlay area (`width: 100%; height: 100%`) with `pointer-events: none`
- Line style: thin (1-2px), semi-transparent, uses `currentColor` to inherit from color scheme
- Dot circle can be an SVG `<circle>` in the same SVG, or remain a CSS element
- The SVG viewBox should match the container's percentage coordinates so `x1/y1/x2/y2` can use percentage-based values
- Alternative simpler approach: use CSS with a pseudo-element rotated/stretched to form the line — but SVG is more precise for arbitrary angles

**Verify:** Set test highlights to visible. Lines should connect from the text area to the dot on the video. Lines should look clean at different viewport sizes.

### Step 6: Highlight timing (fade in/out based on scroll progress)

**Do:** Wire up highlight visibility to scroll progress. Each highlight has a start and end percentage — it fades in when progress >= start and fades out when progress >= end.

**Files:** `assets/video-showcase.js`

**Details:**
- Store highlight elements with their start/end percentages (hardcoded for now, read from data attributes later)
- In the scroll handler (after updating `video.currentTime`), loop through highlights:
  - If `progress >= start && progress < end`: add `.is-visible` class
  - Otherwise: remove `.is-visible` class
- CSS: `.video-showcase__highlight.is-visible { opacity: 1 }`
- Consider adding a small easing buffer — e.g., start fading in slightly before `start_percent` — but keep it simple for v1

**Verify:** Scroll through the video. Highlights should fade in and out at the correct moments. Scrolling backward should reverse them correctly.

### Step 7: Mobile layout

**Do:** Build the mobile experience — stacked keyframe images with highlight text and vertical indicator graphics. Hide desktop layout on mobile, show this instead.

**Files:** `sections/section-video-showcase.liquid`, `assets/section-video-showcase.css`

**Details:**
- Mobile container: visible only below 990px (media query)
- Desktop container: hidden below 990px (media query)
- Each mobile item: full-width image (use `{{ image | image_url }}` with responsive widths), highlight text below, vertical line+dot indicator pointing upward from the text to the feature in the image
- Hardcode 2-3 test images for now
- No scroll trap, no JS interaction on mobile — pure static/scroll layout
- The vertical indicator: a thin vertical line with a dot at the top end, centered above the text
- Mobile images use `loading: 'lazy'`
- Video element should not load on mobile: use JS to conditionally set the `src` only on desktop, or use a `<source>` element with `media="(min-width: 990px)"`

**Verify:** Resize to mobile width. The video/scroll container should disappear. Stacked images with text and upward-pointing indicators should appear. Scrolling should be standard.

### Step 8: Edge cases + accessibility

**Do:** Handle `prefers-reduced-motion`, theme editor context, empty states, and keyboard accessibility.

**Files:** `assets/video-showcase.js`, `assets/section-video-showcase.css`, `sections/section-video-showcase.liquid`

**Details:**
- **`prefers-reduced-motion`:** Via CSS media query, collapse the tall scroll container to auto height. Show the video at a static frame (poster or mid-point). Show all highlights at once. Disable scroll-driven behavior in JS by checking `window.matchMedia('(prefers-reduced-motion: reduce)')`.
- **Theme editor:** Check `Shopify.designMode`. If true, show video at mid-point, display all highlights, skip scroll logic. This lets the store owner see and configure highlights.
- **No video URL:** Show a placeholder message ("Add a video URL in section settings") in the editor. Hide the section entirely on the storefront.
- **No mobile images:** Only render mobile items for images that exist.
- **Skip link:** Add a visually hidden "Skip video showcase" link at the top of the section that jumps past it — critical for keyboard users who'd otherwise have to scroll through 3x viewport height of content.

**Verify:** Enable `prefers-reduced-motion` in browser dev tools — section should show static view with all highlights. Open in theme editor — should see static mid-point frame with highlights. Remove video URL — should see placeholder.

### Step 9: Schema settings (replace hardcoded values)

**Do:** Replace all hardcoded content with proper schema settings. Add highlight blocks with position/timing settings.

**Files:** `sections/section-video-showcase.liquid`

**Details:**
- Section settings: `heading`, `subheading`, `tertiary_heading` (text), `paragraph` (textarea), `video_url` (text), `mobile_image_1` through `mobile_image_4` (image_picker), `color_scheme`, `padding_top`, `padding_bottom`
- Block type `"highlight"`: `text` (textarea), `position_x` (range 0-100, step 1), `position_y` (range 0-100, step 1), `side` (select: left/right), `start_percent` (range 0-100, step 1), `end_percent` (range 0-100, step 1)
- Update all Liquid references to use `section.settings.*` and `block.settings.*`
- Add `{{ block.shopify_attributes }}` to each highlight block for editor functionality
- Preset: `{ "name": "Video Showcase" }`
- Schema organization: content settings first, then mobile images under a header, then color/padding last
- Read highlight data from data attributes in JS: `data-start`, `data-end`, `data-x`, `data-y`

**Verify:** Open in customizer. Change heading text — it should update. Add/remove highlight blocks — they should appear/disappear. Change a highlight's position — the dot and line should move. Change side — text should flip.

## Risks & Considerations

1. **Video encoding is critical.** If the client provides a standard MP4 (GOP=30+), scrubbing will stutter badly. We should provide clear encoding instructions and possibly a sample video for testing.
2. **`video.currentTime` browser behavior varies.** Safari and Chrome handle seeking differently. Safari is generally smoother with `currentTime` scrubbing. Chrome may need `requestVideoFrameCallback` for smoothest results. Test both.
3. **Video preloading.** `preload="auto"` is necessary for scrubbing (need the full video buffered) but means the full file downloads on page load. Keep the video file size reasonable (under 10MB ideally).
4. **Sticky positioning + scroll calculation.** The math for mapping scroll position to progress needs to account for the section's offset from the top of the page. Using `getBoundingClientRect()` on each scroll frame ensures accuracy regardless of content above.
5. **Indicator line coordinates.** SVG lines between text and dot need to update if the viewport resizes. Add a resize observer or recalculate on window resize.
6. **Mobile detection.** Using CSS to hide/show is fine for layout, but we also need to prevent the video from loading on mobile. Either use JS to conditionally set the `src`, or use a `<source>` element with a `media` attribute.

## Open Questions

- None — all decisions were resolved during scoping.
