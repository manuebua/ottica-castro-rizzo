---
description: Rules for Phase 0 — auto-generating the project brief from the first Figma page
---

# Phase 0: Project Brief (Auto-Generated from Figma)

The project brief is always extracted from the **first page** of the Figma file. Do NOT ask the user for brief details manually — parse them from the design.

## Process

1. Use `get_metadata` on the Figma URL to identify the first page and its frames
2. Use `get_design_context` on the first page to extract all design information
3. Use `get_screenshot` on the first page for visual reference

## What to Extract

Parse the first Figma page and derive:

1. **Project name** — from the file name or any title/heading text on the first page
2. **Brand colors** — extract all unique colors used (backgrounds, text, accents, borders) as hex values. Categorize as primary, secondary, accent, neutrals
3. **Typography** — identify all font families, weights, sizes, and line heights used. Note the font source if identifiable (Google Fonts, system fonts, etc.)
4. **Spacing system** — identify the spacing scale used (gaps, margins, padding patterns)
5. **Border radius values** — extract all unique border radius values
6. **Shadows** — extract all unique box shadow values
7. **Design frame width** — read the top-level frame width (e.g., 1440px). This becomes `--size-container-ideal` in the fluid scaling system
8. **Special interactions** — note any elements that imply animations, sliders, modals, hover states, or scroll effects based on the design structure
9. **Component patterns** — identify reusable UI patterns (buttons, cards, inputs, badges, etc.)
10. **Decorative patterns** — identify repeating decorative elements (grid lines, dividers, background patterns) and note their width constraints

## Breakpoints

The fluid scaling system uses these breakpoints (aligned with the scaling system in global.css):
- Desktop: 992px+ (ideal: Figma frame width, max: 1440px)
- Tablet: 768px–991px (ideal: 834)
- Mobile Landscape: 480px–767px (ideal: 550)
- Mobile Portrait: 320px–479px (ideal: 390)

## Units

All design token values in `global.css` MUST use `em`. Convert Figma px values to em (1em = 16px at the design's ideal viewport).

**Exceptions that use px**:
- `1px` borders
- box-shadow values
- scaling system breakpoint values (`--size-container-min`, `--size-container-max`)
- **letter-spacing** — always keep the exact px value from Figma (e.g., `-1.92px`). Never convert to em because em compounds with the element's own font-size, causing unreadable text on headings.

**line-height** — always use **unitless ratios** (Figma line-height ÷ font-size). Example: 56px line-height on 48px font → `1.167`. Never use em for line-height.

Example conversions: 4px → 0.25em, 12px → 0.75em, 24px → 1.5em, 48px → 3em, 58px → 3.625em.

## Output

Save the extracted brief to `PROJECT_BRIEF.md` at the project root with all findings organized clearly.

After saving, immediately update `src/styles/global.css`:
- Set `--size-container-ideal` to the Figma frame width (e.g., 1280 for a 1280px design — no px unit)
- Set `--size-container-max` to `1440px` on desktop — this caps the font scaling so text doesn't get too large on wide screens
- Update design tokens: colors, typography, spacing, shadows, border radius values (all in em)
- Add `--container-padding` in the **scaling system `:root` block** (NOT the design tokens block) so the media query overrides below it take effect. CSS cascade: later declarations at the same specificity win, so if `--container-padding` is in the design tokens block (after the media queries), the overrides will be ignored.
  - Desktop (in scaling `:root`): value from Figma (e.g., `3.25em`)
  - Tablet (≤991px media query): `1.5em`
  - Mobile (≤767px media query): `1em`
- The fluid scaling system handles container sizing automatically — do NOT add a separate `--container-max`
