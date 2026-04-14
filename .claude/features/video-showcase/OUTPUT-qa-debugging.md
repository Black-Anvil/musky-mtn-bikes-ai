# QA: Video Showcase

Feature spec: `.claude/features/video-showcase/feature.md`
Implementation plan: `.claude/features/video-showcase/OUTPUT-implementation-plan.md`

## How to Use This File

1. Run through the checklist after implementation — check items that pass, add notes on items that fail
2. Tell AI to read this file — it will fix the issues and reset the checklist for another round
3. Repeat until everything passes
4. Previous rounds are kept as a log below the current round

## Current Round (Round 1)

Status: Testing

### Core behavior — Desktop
- [ ] Section appears in the theme customizer when added to a page
- [ ] Heading and subheading fade in as the section scrolls into view
- [ ] Video scrubs forward smoothly when scrolling down
- [ ] Video scrubs backward smoothly when scrolling up
- [ ] Video stays pinned at viewport center during scroll (sticky behavior)
- [ ] Video unpins and scrolls away after reaching the final frame
- [ ] Scroll distance feels natural (~3x viewport height to play through the full video)
- [ ] Tertiary heading and paragraph reveal below the video after the final frame

### Highlights — Desktop
- [ ] Highlights fade in at the correct scroll progress (matching their start_percent)
- [ ] Highlights fade out at the correct scroll progress (matching their end_percent)
- [ ] Highlight text appears on the correct side (left or right)
- [ ] Indicator dot appears at the correct x/y position over the video
- [ ] Indicator line connects from the text to the dot cleanly
- [ ] Multiple highlights can appear and disappear independently
- [ ] Scrolling backward correctly reverses highlight visibility

### Mobile
- [ ] Video and scroll container are hidden on mobile (below 990px)
- [ ] Mobile keyframe images display full-width, stacked vertically
- [ ] Each mobile image has associated highlight text
- [ ] Vertical line+dot indicator points upward from text to the image
- [ ] Standard page scrolling (no scroll trap on mobile)
- [ ] Video does NOT load on mobile (check network tab)

### Responsive
- [ ] Section looks correct at desktop widths (1200px+)
- [ ] Section looks correct at tablet widths (750px–989px) — should show mobile layout
- [ ] Section looks correct at mobile widths (375px)
- [ ] Resizing the browser window doesn't break indicator line positioning

### Edge cases
- [ ] `prefers-reduced-motion`: scroll trap disabled, video shows static frame, all highlights visible
- [ ] Theme editor: video shows mid-point frame, all highlights visible, no scroll trap
- [ ] No video URL: section hidden on storefront, placeholder shown in editor
- [ ] Fewer than 4 mobile images: only provided images render, no empty slots

### Accessibility
- [ ] Skip link present and functional (keyboard users can skip past the section)
- [ ] Highlight text is readable by screen readers
- [ ] No content is lost when JavaScript is disabled (graceful degradation)

### Schema / Customizer
- [ ] Heading, subheading, tertiary heading, and paragraph settings work
- [ ] Video URL text setting accepts and displays the video
- [ ] Mobile image pickers work (up to 4)
- [ ] Color scheme setting applies correctly
- [ ] Padding top/bottom settings work (mobile uses 0.75x multiplier)
- [ ] Highlight blocks can be added/removed in the customizer
- [ ] Highlight block settings (text, x, y, side, start%, end%) all function correctly

## Previous Rounds

[Empty — no previous rounds yet.]
