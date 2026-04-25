---
description: Run the full Figma-to-Astro build pipeline
argument-hint: [figma-url]
---

Run the complete orchestration pipeline for: $ARGUMENTS

## Pre-flight checks
1. Verify Figma MCP is connected (try `get_metadata` on the URL)
2. Verify the Figma URL is valid

## Execute phases in order
1. **Phase 0** — Parse the first Figma page to auto-generate `PROJECT_BRIEF.md`, set `--size-container-ideal` from frame width, and update design tokens. All token values in em. Set `--size-container-max: 1440px` on desktop. Add responsive `--container-padding` overrides (tablet: 1.5em, mobile: 1em). Skip if brief already exists.
2. **Phase 1** — Analyze remaining Figma pages: extract structure, download all images to `public/assets/images/`, map components. Report any failed image downloads to the user immediately. **SITE_MAP.md must include `fileKey` at the top and `nodeId` for every section.**
3. **Pre-Phase-2 validation** — Before starting Phase 2, verify that SITE_MAP.md contains:
   - A `fileKey` value at the top
   - A `nodeId` for every section entry
   - If either is missing, go back and fix SITE_MAP.md before proceeding. Phase 2 agents CANNOT build accurately without Figma references.
4. **Phase 2** — Build all components. For EACH section:
   a. Read SITE_MAP.md to get the `fileKey` and `nodeId`
   b. Call `get_design_context` with the fileKey and nodeId to get exact text, colors, spacing, layout
   c. Call `get_screenshot` with the fileKey and nodeId for visual reference
   d. Build the component from the Figma MCP data — not from text descriptions or summaries
   e. Use em for sizing, px for letter-spacing, unitless for line-height
   f. Every section uses full-width bg + max-width centered content
   g. Include responsive styles from the start
   h. Wire up downloaded images from `public/assets/images/`
   - **Do NOT add animations, scroll effects, parallax, or hover transitions unless they are in the Figma design or explicitly requested by the user**
5. **Phase 3** — QA: Figma source-of-truth check (text accuracy, layout verification against screenshot, no invented features), visual fidelity check, code quality check (em usage, letter-spacing in px, line-height unitless, container-max ≤1440px, responsive padding overrides, responsive media queries present), auto-correction loop
6. **Phase 5** — SEO & accessibility pass
7. **Phase 6** — Final review: `npm run build` must pass, verify all breakpoints
8. **Phase 7** — Launch: Start the dev server with `npm run dev` so the user can preview the site immediately

Between each phase, confirm the build still compiles with `npm run build`.

Report progress at each phase transition.
