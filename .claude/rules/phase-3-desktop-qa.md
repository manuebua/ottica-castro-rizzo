---
paths:
  - "src/components/**"
  - "src/pages/**"
  - "src/layouts/**"
description: Rules for Phase 3 — desktop QA checklist and auto-correction loop
---

# Phase 3: Desktop QA & Auto-Correction

After building each section, run this QA checklist. Fix failures immediately before proceeding.

## Layout Structure Check (CRITICAL — check first)

For each section verify:
- Background (color/image/gradient) spans full viewport width
- Content is constrained to `var(--size-container)` and centered with `margin-inline: auto`
- Decorative elements (grid lines, dividers, patterns) are also constrained to `var(--size-container)` and centered — NOT spanning full viewport
- The two-layer pattern is used: outer wrapper (full width) + inner container (max-width)

## Image Check

For each image in the section:
- The downloaded image file exists in `public/assets/images/`
- The `src` attribute points to the correct file
- **File extension matches actual content type** — run `file <path>` to verify. SVGs saved as `.jpg`/`.png` will show as broken images in the browser. If mismatched, rename the file and update the `src` attribute.
- **File size is reasonable** — raster images (JPG/PNG) for content areas should be >5KB. Files under 1KB are likely SVG placeholders or failed exports from Figma. Flag these and notify the user.
- The image renders (not broken)
- If an image is using a placeholder, there should be a comment explaining why and a FAILED entry in IMAGE_MANIFEST.md

## SVG & Image Visibility Check (CRITICAL — catches invisible or mismatched images)

Figma MCP exports SVGs with fill values from the source component. The fills are correct as-is — do NOT change them. The problem is usually that the agent invents wrapper/card backgrounds that don't exist in the design, making the SVGs invisible.

**For every SVG/image in the section:**
1. Compare against the Figma screenshot — how does the image appear in the original design?
2. Check the parent container's background in the component CSS
3. **If the image is invisible**, the fix is almost always to **remove an invented background** from the container, NOT to change the SVG fills. The SVG fills from Figma are correct — the wrapper styling is what's wrong.
4. Only as a last resort, if the Figma screenshot clearly shows a different fill than what the SVG has, adjust the SVG or use CSS overrides.
5. **Key principle**: Trust the SVG fills from Figma. Verify the container styling matches the design.

## Visual Fidelity Check

For each section:
1. Use Figma MCP `get_screenshot` for the original design reference
2. Compare the built section against the Figma design
3. Verify:
   - Spacing matches (margins, padding, gaps)
   - Typography matches (size, weight, line-height, color)
   - Colors match (backgrounds, text, borders, shadows)
   - Layout structure matches (grid columns, flex direction, alignment)
   - Border radius and visual treatments match
   - Content/copy matches the design

## Code Quality Check

- Semantic HTML used correctly
- No unnecessary wrapper `<div>` elements
- CSS custom properties used consistently (no hardcoded hex)
- **Sizing in `em`** for font-size, padding, margin, gap, width, height, border-radius. No px values for these except 1px borders and box-shadows.
- **letter-spacing MUST be in `px`** — flag any letter-spacing using em units. em letter-spacing compounds with the element's font-size, causing unreadable text on headings.
- **line-height MUST be unitless** (a ratio like `1.167`, not `3.5em`). Flag any line-height using em units. em line-height compounds with the element's font-size, creating enormous vertical gaps.
- **`--size-container-max` must be `1440px`** on desktop — not 1920px. Flag if larger.
- **`--container-padding` must reduce on mobile** — verify media query overrides exist (≤991px: `1.5em`, ≤767px: `1em`)
- No unused CSS
- Component props properly typed
- No inline styles unless absolutely necessary

## Responsive Check

Every component MUST include media queries. Verify:
- Has `@media (max-width: 991px)` tablet styles
- Has `@media (max-width: 767px)` mobile styles
- No horizontal overflow at any viewport width down to 320px
- Grids reduce columns on smaller screens
- Fixed widths become fluid on mobile
- Touch targets are at least 2.75em on mobile

## Figma Source-of-Truth Check (CRITICAL — run before visual fidelity)

For each section, read the `fileKey` and `nodeId` from SITE_MAP.md, then:

1. **Text accuracy**: Call `get_design_context` and compare every piece of text in the built component character-for-character against the Figma data. Flag any paraphrased, rewritten, shortened, or invented copy.
2. **Layout structure**: Call `get_screenshot` and verify column count, flex direction, content positioning, and alignment match the screenshot exactly. Flag any layout that doesn't match.
3. **Content positioning**: Verify element order, alignment (left/center/right), and relative positioning match the screenshot.
4. **No invented features**: Scan the component CSS for `animation`, `@keyframes`, `transition`, `transform` (used for motion), `scroll-behavior`, or `parallax`-related styles that are NOT present in the Figma design. Flag and remove any that were added without basis in the design.
5. **No extra elements**: Verify every HTML element and visual treatment in the component corresponds to something in the Figma design. Flag decorative elements, overlays, gradients, or shapes that don't exist in the design.
6. **Image treatments**: Compare image presentation (aspect ratio, border-radius, shadows, overlaps) against the screenshot. Flag any effects not in the design.

## Auto-Correction Loop

If issues are found:
1. Identify the discrepancy
2. Fix the code
3. Re-run checks for that section
4. Repeat until all checks pass

**Only proceed to Phase 4 once ALL desktop sections pass QA.**
