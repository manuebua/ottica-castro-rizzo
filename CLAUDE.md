# Figma-to-Astro Orchestration Flow

## Commands
```
npm run dev          # Start dev server
npm run build        # Production build
npm run preview      # Preview production build
```

## Architecture
- Astro v6 static site generator
- TypeScript strict mode
- CSS custom properties for design tokens (no CSS framework by default)
- Scoped styles in `.astro` components
- Desktop-first responsive (media queries use `max-width`)
- **GSAP 3.12 + ScrollTrigger** loaded via CDN in `BaseLayout.astro` for animations

## Project Structure
```
src/
├── layouts/       # BaseLayout.astro (head, meta, fonts, global styles)
├── components/    # Flat structure, PascalCase.astro
├── pages/         # File-based routing
├── styles/        # global.css with design tokens and reset
└── assets/        # Images processed by Astro, local fonts
public/assets/     # Static images served as-is
```

## Orchestration Workflow
This project uses a phased agent pipeline to convert Figma designs into production Astro sites. The phases are defined in `.claude/rules/` and triggered via custom commands:

1. `/project:brief [figma-url]` — Parse the first Figma page to auto-generate the project brief
2. `/project:build [figma-url]` — Full pipeline: brief → Figma analysis → desktop build → QA → responsive → SEO
3. `/project:qa` — Re-run desktop QA and auto-correction on current build

## Prerequisites
- **Figma MCP must be installed and connected** — the pipeline depends on it
- Only **desktop designs** are provided in Figma — responsive is derived

## Layout Model (CRITICAL)
Every section follows this pattern:
- **Outer wrapper**: full viewport width. Background colors/images stretch edge to edge.
- **Inner content**: constrained to `var(--size-container)` (fluid, derived from Osmo scaling) and centered with `margin-inline: auto`.
- **Decorative elements** (grid lines, vertical rules, etc.): also constrained to `var(--size-container)` and centered — they do NOT span the full viewport.
- The `--size-container-ideal` is set to the Figma frame width during Phase 0 (e.g., 1440 for a 1440px design).

## Image Handling
- **Download images from Figma** using the asset URLs returned by `get_design_context`
- Store downloaded images in `public/assets/images/` with descriptive filenames
- Connect images to components with correct `src`, `alt`, `width`, `height`
- If an image cannot be downloaded (expired URL, network error, etc.), immediately notify the user with the reason and which component is affected
- Track all images in `IMAGE_MANIFEST.md` with: filename, dimensions, description, download status

## Scaling System (CRITICAL) — Fluid Viewport Scaling
This project uses a fluid scaling system for viewport-based sizing. The `<body>` font-size is set to `var(--size-font)`, which scales proportionally with the viewport. All sizing uses **`em`** so everything scales automatically.

### How it works
- `--size-container-ideal` = the Figma design width (no px). Set during Phase 0.
- `--size-font` = `container / (ideal / unit)` — fluid body font-size
- `--size-container` = `clamp(min, 100vw, max)` — fluid container width
- `--size-container-max` = **1440px** on desktop — caps the maximum container width so font-size doesn't grow excessively on large screens
- Each breakpoint redefines `--size-container-ideal`, `--size-container-min`, `--size-container-max`

### Units
- **Use `em` for most sizing** — font-size, padding, margin, gap, width, height, border-radius
- Because body `font-size` is fluid, `em` values scale proportionally across all viewports
- **letter-spacing: use `px` for normal text** — keep the exact px value from Figma (e.g., `-1.92px`). Never convert letter-spacing to em because em is relative to the element's own font-size, causing compounding on headings (a 3em heading with `-0.12em` letter-spacing gets 3× the intended tightening, making text unreadable)
- **letter-spacing exception for giant decorative text (font-size > 10em)**: Use `em` so it scales proportionally. Convert: `px-value / font-size-px` (e.g., `-22px / 367px = -0.06em`). Fixed px on giant text breaks at smaller viewports.
- **line-height: always use unitless ratios** — divide Figma line-height by font-size (e.g., Figma says 56px line-height on 48px font → `56/48 = 1.167`). Unitless line-height is correctly relative to the element's font-size. Never use em for line-height.
- **Only exceptions for px**: `1px` borders, box-shadows, and `letter-spacing`
- Base: `1em = 16px` at the design's ideal viewport. Convert Figma px values to em (e.g., 48px → 3em, 24px → 1.5em, 12px → 0.75em)
- Design tokens in `global.css` must also use em (spacing, radius, container-padding)

### Container
- `.container` uses `max-width: var(--size-container)` — NOT a fixed px value
- `.container--medium` = 85% of container, `.container--small` = 70%
- The old `--container-max` is replaced by the Osmo `--size-container` system

### Container Padding (Responsive)
- Desktop: `--container-padding` from Figma (e.g., 3.25em for 52px)
- Tablet (≤991px): reduce to `1.5em`
- Mobile (≤767px): reduce to `1em`
- Phase 0 must add these media query overrides to `global.css`

### Phase 0 setup
During Phase 0, set `--size-container-ideal` to the Figma frame width (e.g., 1440 for a 1440px design) and `--size-container-max` to `1440px`. The scaling system handles the rest.

## Responsive (CRITICAL)
- **Build responsive from the start** — every component must include media queries for tablet and mobile during Phase 2, not deferred to Phase 4
- The Osmo scaling system handles most sizing automatically across viewports — explicit font-size overrides in media queries are rarely needed
- Breakpoints: tablet ≤991px, mobile landscape ≤767px, mobile portrait ≤479px (aligned with Osmo breakpoints)
- Key responsive patterns:
  - Multi-column grids → fewer columns → single column
  - Large headings scale down (e.g., 3rem desktop → 2rem mobile)
  - Horizontal layouts stack vertically
  - Padding/gaps reduce on smaller screens
  - Navigation collapses to hamburger menu at tablet (≤991px)
  - No horizontal overflow at any breakpoint
  - Min 44px (2.75rem) touch targets on mobile

## Conventions
- Components: `PascalCase.astro`
- CSS vars: `--color-primary`, `--font-heading`, `--spacing-lg`
- CSS classes: `kebab-case`
- All colors via CSS custom properties — no hardcoded hex values in components
- Semantic HTML: `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>`
- Every `<img>` gets `alt`, `width`, `height`
- One `<h1>` per page, logical heading hierarchy

## GSAP Animations
- GSAP and ScrollTrigger are loaded via CDN `<script is:inline>` tags in `BaseLayout.astro`
- Site-wide scroll-triggered animations (fade-ups, card staggers, image reveals, stat count-ups) are defined in an inline script in `BaseLayout.astro`
- Animation style: soft, slow, professional — `power2.out` ease, 0.8–1.2s durations, `once: true` triggers
- Respects `prefers-reduced-motion: reduce` — all animations are skipped if enabled

### Astro Script Rule (CRITICAL)
- **All `<script>` tags that reference CDN globals (like `gsap`) MUST use `is:inline`**
- Without `is:inline`, Astro bundles scripts as ES modules — CDN globals are not accessible in module scope
- This applies to both the CDN `<script src="...">` tags AND any inline `<script>` that calls `gsap`
- Component-level scripts that use GSAP must also use `<script is:inline>`

## Watch Out For
- Build must pass (`npm run build`) before any phase is considered complete
- The PROJECT_BRIEF.md is auto-generated from the first Figma page — it must exist before building
- Desktop QA must fully pass before starting responsive migration
- Every section: full-width bg, max-width centered content — never let content stretch to viewport edges
