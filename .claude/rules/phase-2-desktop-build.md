---
paths:
  - "src/components/**"
  - "src/pages/**"
  - "src/layouts/**"
  - "src/styles/**"
description: Rules for Phase 2 — building the desktop version from Figma designs
---

# Phase 2: Desktop Build (Responsive from the Start)

Build desktop-first but **include responsive styles in every component**. Do not defer responsive to a later phase — each component must work from desktop down to mobile (320px) when it's built.

## Figma MCP Access (CRITICAL — do this FIRST)

Every component MUST be built from live Figma data, not from text descriptions or summaries in SITE_MAP.md.

**Required workflow for each section:**

1. **Read SITE_MAP.md** — get the `fileKey` and `nodeId` for the section you're building
2. **Call `get_design_context`** with the fileKey and nodeId — extract exact text, colors, typography, spacing, layout structure, and image references
3. **Call `get_screenshot`** with the fileKey and nodeId — use as visual reference for layout, alignment, column count, and positioning
4. **Build the component** using ONLY the data from steps 2 and 3

**EXACT TEXT**: Use character-for-character text from `get_design_context`. Never paraphrase, rewrite, shorten, or invent copy. If the design says "Transform your business" do NOT write "Transforming businesses" or any variation.

**NO INVENTED FEATURES**: Do not add animations, scroll effects, parallax, hover transitions, gradient overlays, or any visual/interactive behavior unless it is explicitly present in the Figma design or specifically requested by the user. Simple, static HTML/CSS that matches the design is always correct.

**LAYOUT FROM SCREENSHOT**: Column count, flex direction, alignment, spacing, and content positioning must match the screenshot. Do not guess or assume layout — verify against the visual reference.

**IMAGE TREATMENTS FROM SCREENSHOT**: Match the exact image presentation — aspect ratio, cropping, border-radius, overlaps, and sizing as shown in the screenshot. Do not add effects (shadows, overlays, shape masks) that aren't in the design.

## Global Setup

1. Update `src/styles/global.css` with design tokens from the brief and Figma analysis
2. Update `src/layouts/BaseLayout.astro` with project-specific meta, fonts, and title
3. Set `--size-container-ideal` to the Figma frame width (e.g., 1440 for a 1440px design — no px unit)

## Units (CRITICAL) — Fluid Scaling with em

This project uses a fluid scaling system. The `<body>` font-size is `var(--size-font)` which scales proportionally with the viewport. **Use `em` for most sizing values.** Convert Figma px values to em (1em = 16px at the design's ideal viewport):

| Figma px | em       |
|----------|----------|
| 4px      | 0.25em   |
| 8px      | 0.5em    |
| 12px     | 0.75em   |
| 14px     | 0.875em  |
| 16px     | 1em      |
| 20px     | 1.25em   |
| 24px     | 1.5em    |
| 32px     | 2em      |
| 48px     | 3em      |
| 56px     | 3.5em    |
| 64px     | 4em      |
| 68px     | 4.25em   |
| 80px     | 5em      |

This applies to: font-size, padding, margin, gap, width, height, border-radius, max-width.

### letter-spacing — use px for normal text, em for giant decorative text (CRITICAL)

**For normal text (headings, body, labels, buttons):** Keep the exact px value from Figma. Never convert to em.

Why: `em` in letter-spacing is relative to the *element's own font-size*, not the root. A heading with `font-size: 3em` and `letter-spacing: -0.12em` gets `-0.12 × 48px = -5.76px` instead of the intended `-1.92px`. This makes text unreadable — letters overlap and collide.

Examples:
- Figma says `-1.92px` → use `letter-spacing: -1.92px`
- Figma says `1.68px` → use `letter-spacing: 1.68px`
- Figma says `-0.32px` → use `letter-spacing: -0.32px`

**Exception — giant decorative text (font-size > 10em):** Use `em` for letter-spacing. For very large display text (e.g., a 367px brand name in the footer), a fixed px letter-spacing like `-22px` won't scale when the font-size shrinks at smaller viewports, causing letters to collide. Convert by dividing: `-22px / 367px = -0.06em`. This keeps the letter-spacing proportional to the font-size across all breakpoints.

### line-height — ALWAYS use unitless ratios (CRITICAL)

**Never use em for line-height.** Use the unitless ratio: `Figma line-height ÷ Figma font-size`.

Why: `em` in line-height is relative to the *element's own font-size*. A heading with `font-size: 3em` and `line-height: 3.5em` gets `3.5 × 48px = 168px` instead of the intended 56px. This creates enormous vertical gaps between lines.

Examples:
- Figma: 56px line-height on 48px font → `line-height: 1.167` (56 ÷ 48)
- Figma: 32px line-height on 24px font → `line-height: 1.333` (32 ÷ 24)
- Figma: 24px line-height on 16px font → `line-height: 1.5` (24 ÷ 16)
- Figma: 20px line-height on 14px font → `line-height: 1.429` (20 ÷ 14)

### px exceptions

**Use px for**: `1px` borders, box-shadow offsets/blur, and **letter-spacing**.

Because the body font-size is fluid, all `em` values scale proportionally across viewports. This means most elements will already look correct at smaller screens without explicit media query overrides — but layout changes (column stacking, flex-direction, etc.) still need media queries.

Design tokens in `global.css` (spacing, radius, container-padding) must use em.

## Layout Model (CRITICAL)

Every section MUST follow this two-layer pattern:

```
<section class="section-name">                                ← full viewport width
  <!-- background: color/image/gradient here -->               ← stretches edge to edge
  <div class="section-name__inner container container--padded"> ← fluid max-width centered + padded
    <!-- actual content here -->
  </div>
</section>
```

Rules:
- **Outer wrapper** (`<section>`): always `width: 100%`. Background colors, background images, and gradients go here — they stretch to the full viewport width.
- **Inner content** (`.container`): constrained to `max-width: var(--size-container)` and centered with `margin-inline: auto`. Add `.container--padded` for `padding-inline: var(--container-padding)`.
- **Padding rule**: EVERY section's content container MUST have `padding-inline: var(--container-padding)` — either via `.container--padded` or via component CSS. Content must NEVER touch the container edges without padding. This applies to ALL sections — no exceptions.
- **Decorative elements** (grid lines, vertical rules, dividers): constrained to `var(--size-container)` and centered. They span the full container width (no padding), NOT the full viewport.
- **Images used as section backgrounds**: go on the outer wrapper, covering full width.
- **Images used as content** (photos, illustrations in the layout): go inside the inner content container.
- **No double padding**: If the inner container already has `container--padded`, do NOT add extra `padding-inline` on child elements. The container padding alone provides the left/right inset.

## Component Build Order

Build bottom-up to resolve dependencies:

1. **Atoms** — buttons, links, icons, badges, tags
2. **Molecules** — cards, form groups, nav items
3. **Organisms** — navbar, hero, sections, footer
4. **Pages** — compose organisms into full pages

## Component Rules

- Live in `src/components/` (flat structure unless complexity demands folders)
- Use scoped `<style>` tags in `.astro` files
- Receive data via Astro props with TypeScript interfaces
- Use semantic HTML (`<nav>`, `<main>`, `<section>`, `<article>`, `<footer>`)

## Image Handling in Components

- Use the downloaded images from `public/assets/images/`
- Reference with `/assets/images/{filename}` in `src` attributes
- Always include `alt`, `width`, `height` attributes
- If an image was marked as FAILED in `IMAGE_MANIFEST.md`, use the placeholder and add a comment: `<!-- TODO: Replace placeholder - image download failed -->`

## Responsive Approach (CRITICAL)

The fluid scaling system handles most sizing automatically — font-sizes, padding, gaps, and spacing all scale with the viewport. However, **layout changes** still require media queries.

**Breakpoints** (desktop-first, use `max-width` — aligned with scaling system):
```css
@media (max-width: 991px)  { /* Tablet */ }
@media (max-width: 767px)  { /* Mobile landscape */ }
@media (max-width: 479px)  { /* Mobile portrait */ }
```

**What the scaling system handles automatically** (no media queries needed):
- Font sizes
- Spacing (padding, margin, gap) when using em
- Border radius
- Element dimensions specified in em

**What still needs media queries** (layout changes):
- **Grids**: 3-col → 2-col (tablet) → 1-col (mobile)
- **Side-by-side layouts**: Stack vertically on mobile
- **Fixed widths**: Replace with `width: 100%` or `max-width` on mobile
- **Navigation**: Collapse to hamburger/mobile menu at ≤991px (tablet breakpoint, not mobile)
- **Buttons**: Full-width on mobile where appropriate
- **No horizontal overflow**: At ANY breakpoint

## Typography Fidelity (CRITICAL)

- **Use the exact font-weight from Figma** — do NOT assume headings should be bold. Many designs use `font-weight: 400` (Regular) for headings. Check the Figma style: "Regular" = 400, "Medium" = 500, "SemiBold" = 600, "Bold" = 700.
- The global CSS reset already sets headings to use `var(--font-heading)` and `line-height: 1.2`, but it does NOT set font-weight — the browser default bold must be explicitly overridden when Figma specifies Regular (400).
- When building headings, always set `font-weight` explicitly to match Figma. Never rely on browser defaults.

## Desktop Fidelity (Source of Truth: Figma MCP)

All fidelity checks reference data from `get_design_context` and `get_screenshot` — not from memory, summaries, or SITE_MAP.md descriptions.

- **Spacing**: Match exact values from `get_design_context` (converted to em)
- **Typography**: Match exact font-size (em), font-weight, line-height (unitless ratio), letter-spacing (px) from `get_design_context`. Never assume headings are bold — use the actual weight from Figma.
- **Colors**: Match exact hex values from `get_design_context`, mapped to CSS custom properties
- **Layout**: Column count, flex direction, alignment must match `get_screenshot`
- **Border radius, shadows, visual treatments**: Match exact values from `get_design_context`
- **Text content**: Character-for-character match from `get_design_context` — no lorem ipsum, no rewriting
- **Images**: Use downloaded images from `public/assets/images/`, presented exactly as shown in `get_screenshot`
- **No additions**: Do NOT add animations, transitions, hover effects, or decorative elements that don't appear in the Figma design
