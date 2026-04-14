# [Feature Name]

## Brief

We are going to build a video showcase section. As the section gets scrolled into view, the keyframes are displayed in succession with scroll action. When the video reaches the center of the viewport, scroll trap is enabled, and the scolling continues to advance the keyframes until the last is displayed, at which point normal page scrolling continues.

At different frames of the video, we are going to display "highlights" text on one side or the other of the video with indicator graphics that point out specific features of the product within.  The text and accompanying graphic fade in and out with the scroll action as the key frames advance.

As for mobile, the video will be replaced with a series of images representing a few keyframes from the video with associated highlight text and graphic indicators.  The user will scroll through these vertically in a more standard way

As for section/block schema, there will need to be a text setting for a section heading, one for a subheading, one for a tertiary heading that will appear below after the video following its final frame, and textarea setting for a paragraph just below that.  The highlights texts, and their positions will be established with section blocks.  The video will need to be a text setting, NOT a video picker that Shopify will reformat.  The mobile images (up to 4?) can presumably be added as image picker settings under a "mobile Layout" header.

I have included a .mov file in the reference folder that shows Apple's AirPods 4 as an inspiration for this feature.  Let me know what else you may need before diving in.

---

## Scoping Questions

Generated: 2026-04-13
Chosen approach: Custom Section + Web Component (Approach A)

### Q1: Video rendering method — `video.currentTime` or Canvas frame extraction?

Two proven approaches for scroll-driven video:

- **video.currentTime**: Set the video's current time directly based on scroll position. Simpler code, lower memory. Can stutter on some devices because video decoders aren't optimized for random seeking — this is minimized with high-keyframe-density MP4s.
- **Canvas**: Pre-extract frames from the video, draw them to a `<canvas>` on scroll. Buttery smooth, but requires loading all frames into memory (a 10-second video at 30fps = 300 images). Can use progressive loading to manage this.

My recommendation: Start with `video.currentTime` — it's simpler, works well with properly encoded MP4s, and avoids the memory overhead. We can always upgrade to canvas later if scrubbing quality isn't smooth enough.

- [x] a) `video.currentTime` scrubbing (simpler, start here)
- [ ] b) Canvas frame extraction (smoother, higher complexity + memory)

**Notes:**

### Q2: Scroll trap behavior — how should it feel?

The scroll trap locks page scrolling while the video advances. Key decisions:

- **Entry**: Should the trap engage when the video hits the center of the viewport, or when it reaches the top?
- **Exit**: Does it release after the last frame, or after holding on the last frame for a beat?
- **Speed mapping**: How much scroll distance = full video playback? (e.g., Apple uses roughly 3-4x the viewport height of scroll distance to play through their videos)

- [x] a) Trap at viewport center, release immediately after last frame, ~3x viewport scroll distance
- [ ] b) Trap at viewport top, release immediately after last frame, ~3x viewport scroll distance
- [ ] c) Trap at viewport center, hold last frame briefly then release, ~3x viewport scroll distance
- [ ] d) Other (describe in notes)

**Notes:**

### Q3: Highlight overlay positioning — how precise does it need to be?

The highlights point at specific product features in the video. As the product moves/rotates through the video, the pointer target moves too. Options:

- **Fixed positions**: Each highlight has a fixed x/y position relative to the video container. Simpler, but the pointer may drift from the product feature as the video plays through that highlight's active range.
- **Start/end positions**: Each highlight has a start position and end position, and the pointer interpolates between them as the video plays. More accurate tracking, more settings to configure.

My recommendation: Fixed positions. Apple's implementation uses fixed positions too — the highlights appear/disappear quickly enough that drift isn't noticeable. Interpolated positions would double the schema settings per block and add complexity for minimal visual gain.

- [x] a) Fixed x/y position per highlight (simpler, recommended)
- [ ] b) Start/end x/y positions with interpolation (more precise, more complex)

**Notes:**

### Q4: Highlight indicator graphic — what should it look like?

The "indicator graphic" that points from the text to the product feature. What style?

- [ ] a) Simple line/connector — a thin line from text to the point on the product (clean, minimal)
- [x] b) Line with dot endpoint — line with a small circle at the product feature (Apple style)
- [ ] c) Custom SVG — allow the store owner to upload a custom indicator graphic per highlight
- [ ] d) Other (describe in notes)

**Notes:**

### Q5: Highlight text positioning — which side of the video?

In the brief you mentioned text appears "on one side or the other." How should this be controlled?

- [x] a) Per-block setting — each highlight block has a "side" setting (left/right) so the store owner decides per highlight
- [ ] b) Automatic alternation — highlights alternate left/right automatically
- [ ] c) All on one side — text always appears on the same side, indicator lines extend to the product
- [ ] d) Other (describe in notes)

**Notes:**

### Q6: Mobile layout — how many keyframe images, and how should highlights work?

You mentioned up to 4 images for mobile. Questions:

- Should each mobile image have its own highlight text, or should highlights be separate from the images?
- Should the images be full-width stacked, or in a different layout?
- Should the indicator graphics appear on mobile, or just the text?

- [ ] a) Full-width stacked images, each with its own highlight text below/overlaid, no indicator graphics
- [ ] b) Full-width stacked images with indicator graphics and highlight text overlaid
- [x] c) Other (describe in notes)

**Notes:**
Option A, but with indicator graphic pointing vertically up to the feature.

### Q7: Video file hosting — where will the MP4 live?

Shopify's video picker re-encodes videos (which you want to avoid). Options for the raw MP4 URL:

- [x] a) Shopify Files — upload to Settings > Files, paste the CDN URL into a text setting
- [ ] b) External hosting — user provides a URL from their own CDN/S3/etc.
- [ ] c) Either — just a URL text field, doesn't matter where it's hosted

**Notes:**

### Q8: Section heading area — layout and styling?

The brief mentions a section heading, subheading, and a tertiary heading + paragraph that appears below the video after the final frame. Questions:

- Should the heading/subheading appear above the video before scrolling begins?
- Should the tertiary heading + paragraph appear as a reveal after the scroll trap releases, or just be statically placed below?
- Any specific typography hierarchy in mind? (The theme uses `h1`–`h5` with standard Dawn sizing)

- [ ] a) Heading + subheading above video (static), tertiary heading + paragraph below (static)
- [x] b) Heading + subheading above video (fade in on scroll), tertiary heading + paragraph below (reveal after trap releases)
- [ ] c) Other (describe in notes)

**Notes:**

### Q9: Color scheme and background — how should it look?

Apple's version uses a dark/black background with white text to make the product pop. Should this section:

- [x] a) Use the theme's color scheme setting — let the store owner pick from existing color schemes
- [ ] b) Default to dark background (but still use color scheme setting for flexibility)
- [ ] c) Hardcode a dark background — this section always looks best dark
- [ ] d) Other (describe in notes)

**Notes:**

### Q10: Settings approach — hardcoded first or schema settings from the start?

Per our workflow convention, I should ask: do you want to prototype with hardcoded values first (faster iteration, add settings later), or set up all schema settings from the start?

Given the complexity of this feature, I'd recommend hardcoded values for the first pass — get the scroll behavior and video timing right, then layer in the customizer settings. The interaction mechanics are the hard part, not the settings.

- [x] a) Hardcoded first, add settings later (recommended for complex features)
- [ ] b) Full schema settings from the start

**Notes:**

---

### Recommendations

1. **Video encoding is critical.** The MP4 needs to be encoded with a very low GOP (Group of Pictures) — ideally every frame is a keyframe (GOP=1). Standard video encoding uses keyframes every 30-60 frames, which means seeking to intermediate frames requires decoding from the nearest keyframe, causing stutter. Recommend providing the client with encoding specs or a sample ffmpeg command.

2. **Aspect ratio handling.** The video container needs a defined aspect ratio to prevent layout shifts. I'd suggest a 16:9 default with the video set to `object-fit: cover` or `contain` depending on the content. This should be a setting.

3. **Accessibility.** Scroll hijacking is problematic for keyboard users and screen readers. We should include: (a) a `prefers-reduced-motion` media query that disables the scroll trap and shows a static fallback, (b) proper ARIA labels on the highlight text, (c) a way to skip past the section.

4. **Performance.** The video should not autoload on mobile (since mobile uses images instead). Use `preload="auto"` only on desktop via media query or JS detection. Mobile images should use `loading="lazy"`.

5. **Theme editor compatibility.** Scroll-driven effects don't work in Shopify's theme editor iframe. We'll need a static fallback for the editor context — show the video at a mid-point frame with all highlights visible so the store owner can configure settings.

## Extended Brief

Generated: 2026-04-13

### Chosen Approach

Custom Section + Web Component (Approach A) — a fully custom `section-video-showcase.liquid` with a `<video-showcase>` web component that ties `video.currentTime` to scroll position. This is the only approach that delivers the Apple-style scroll-driven video scrubbing with synchronized highlight overlays.

### Requirements

- Video plays frame-by-frame as the user scrolls, synced to scroll position via `video.currentTime`
- Scroll trap engages when the video reaches viewport center — continued scrolling advances the video instead of the page
- Scroll trap uses ~3x viewport height of scroll distance to play through the full video
- Scroll trap releases immediately after the last frame — no hold
- Highlight callouts appear/disappear at configurable frame ranges during video playback
- Each highlight has: text content, fixed x/y position (% based), left/right side, line-with-dot indicator pointing to the product feature
- Heading + subheading fade in above the video as the section scrolls into view
- Tertiary heading + paragraph reveal below the video after the scroll trap releases
- Mobile: no video, replaced with up to 4 full-width stacked keyframe images with highlight text and vertical upward-pointing indicator graphics
- No scroll trap on mobile — standard vertical scrolling

### Where It Lives

Available on any page via theme customizer preset. Not template-specific. Files:

- `sections/section-video-showcase.liquid` — markup, schema, asset loading
- `assets/section-video-showcase.css` — component styles
- `assets/video-showcase.js` — web component (scroll logic, video control, highlight timing)

### Data Sources

- **Video:** MP4 URL pasted into a text setting (uploaded to Shopify Files > CDN URL). Not the Shopify video picker (which re-encodes).
- **Mobile images:** Up to 4 image picker settings in the section schema
- **Highlights:** Section blocks with text, position, timing, and side settings
- **Headings/text:** Section settings (heading, subheading, tertiary heading, paragraph)

### User Interaction

**Desktop:**
1. User scrolls down the page → heading + subheading fade in
2. Video section reaches viewport center → scroll trap engages
3. Continued scrolling advances `video.currentTime` proportionally across ~3x viewport height of scroll distance
4. Highlights (text + line-with-dot indicator) fade in/out at their configured frame ranges
5. Video reaches final frame → scroll trap releases immediately, page scroll resumes
6. Tertiary heading + paragraph reveal below the video

**Mobile:**
1. User scrolls through stacked keyframe images vertically (standard scroll)
2. Each image has associated highlight text and a vertical indicator line+dot pointing up to the feature
3. Heading/subheading and tertiary heading/paragraph appear statically or with simple scroll-reveal

### Customizer Settings

**Section settings (first pass will be hardcoded, settings added after interaction is solid):**
- `heading` — text, section heading
- `subheading` — text, section subheading
- `tertiary_heading` — text, heading below the video
- `paragraph` — textarea, paragraph below the video
- `video_url` — text (URL), MP4 video source
- `mobile_image_1` through `mobile_image_4` — image pickers
- `color_scheme` — standard theme color scheme selector
- `padding_top` / `padding_bottom` — standard range settings

**Block settings (type: "highlight"):**
- `text` — textarea, highlight callout text
- `position_x` — range (0-100%), horizontal position of the indicator dot
- `position_y` — range (0-100%), vertical position of the indicator dot
- `side` — select (left/right), which side the text appears on
- `start_percent` — range (0-100%), when the highlight fades in (% of video duration)
- `end_percent` — range (0-100%), when the highlight fades out (% of video duration)

**Not configurable:** indicator graphic style (always line+dot), scroll trap distance multiplier, animation easing curves.

### Decisions Made

| Decision | Choice | Reasoning |
|----------|--------|-----------|
| Video rendering | `video.currentTime` scrubbing | Simpler, lower memory, works well with high-keyframe MP4s. Canvas is a future upgrade path if needed. |
| Scroll trap entry | Viewport center | Feels natural — user sees the video centered before trap engages |
| Scroll trap exit | Immediate release | No artificial hold — clean transition back to normal scrolling |
| Scroll distance | ~3x viewport height | Matches Apple's pacing — enough scroll distance for smooth frame progression without feeling sluggish |
| Highlight positioning | Fixed x/y per block | Apple uses this too. Highlights appear/disappear quickly enough that drift isn't noticeable. Avoids doubling schema settings. |
| Indicator style | Line with dot endpoint | Apple style — clean, minimal, communicates the connection clearly |
| Text side | Per-block setting | Gives store owner full control over layout composition per highlight |
| Mobile approach | Stacked images + text + vertical indicator | No scroll trap on mobile (problematic UX). Vertical indicator points up to the feature. |
| Video hosting | Shopify Files CDN URL in text setting | Avoids Shopify's video picker re-encoding which would destroy keyframe density |
| Headings | Fade-in on scroll (top), reveal after trap (bottom) | Adds polish, ties the headings to the scroll-driven experience |
| Color scheme | Theme's standard color scheme setting | Flexibility for the store owner, consistent with other sections |
| Dev approach | Hardcoded first | Get the interaction mechanics right before layering in settings complexity |

### Edge Cases to Handle

- **Video not loaded yet:** Show a loading state / placeholder to prevent layout shift. Define a fixed aspect ratio container.
- **`prefers-reduced-motion`:** Disable scroll trap entirely. Show video at a mid-point frame (or poster image) with all highlights visible statically.
- **Theme editor context:** Static fallback — show video at mid-point, all highlights visible, no scroll trap (editor iframe doesn't support scroll hijacking).
- **No video URL provided:** Hide the desktop video area gracefully, show a placeholder message in the editor.
- **No mobile images provided:** Hide the mobile section or show a message in the editor.
- **Fewer than 4 mobile images:** Only render the images that are provided.
- **Highlight with start_percent >= end_percent:** Ignore / don't display that highlight.
- **Very short video:** Scroll distance is proportional, so it still works — but may feel fast. The ~3x multiplier should handle most cases.
- **Browser without IntersectionObserver:** Extremely unlikely (98%+ support), but could gracefully degrade to autoplay.

### Out of Scope

- Canvas frame extraction (potential future upgrade if `video.currentTime` scrubbing quality is insufficient)
- Multiple videos or video switching within the section
- Autoplay mode (non-scroll-driven playback)
- Audio playback
- Video controls (play/pause/seek bar)
- Per-highlight mobile image mapping (mobile images are independent of desktop highlights)

### Dependencies

- None — fully self-contained section. No modifications to existing theme files.
- Requires a properly encoded MP4 with high keyframe density (low GOP) for smooth scrubbing.

### Notes

- **Video encoding spec for the client:** The MP4 should be encoded with GOP=1 (every frame is a keyframe) for optimal scrubbing. Sample ffmpeg command: `ffmpeg -i input.mov -c:v libx264 -g 1 -preset slow -crf 23 -an output.mp4`. The `-an` flag strips audio (not needed). `-g 1` sets every frame as a keyframe.
- **First pass will be hardcoded.** All text, video URL, and highlight positions will be hardcoded in the Liquid/JS. Schema settings will be added in a second pass after the scroll interaction is working correctly.
- **Performance:** Video should only load on desktop. Mobile loads images only. Use `preload="auto"` on desktop for smooth scrubbing (the entire video needs to be buffered for random access).

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# feature.md — v1.0

# AI Shopify Developer Bootcamp

# by Coding with Jan

# https://codingwithjan.com

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
