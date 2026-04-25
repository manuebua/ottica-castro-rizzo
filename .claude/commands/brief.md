---
description: Parse the first Figma page to auto-generate the project brief
argument-hint: [figma-url]
---

Run Phase 0: Auto-generate project brief from Figma.

Figma URL: $ARGUMENTS

1. Use `get_metadata` to identify the first page in the Figma file
2. Use `get_design_context` on the first page to extract all design details
3. Use `get_screenshot` on the first page for visual reference

From the first page, extract:
- Project name (from file name or heading text)
- Brand colors (all unique hex values, categorized: primary, secondary, accent, neutrals)
- Typography (font families, weights, sizes, line heights, source)
- Spacing system (recurring gap/margin/padding values)
- Border radius values
- Shadow values
- **Content max-width** (top-level frame width → becomes `--container-max`)
- Decorative patterns (grid lines, dividers) and their width constraints
- Special interactions implied by the design (sliders, modals, hover states)
- Reusable component patterns (buttons, cards, inputs, etc.)

Save everything to `PROJECT_BRIEF.md` and update `src/styles/global.css` design tokens to match.

If `PROJECT_BRIEF.md` already exists, show its contents and ask if the user wants to regenerate it from the Figma file.
