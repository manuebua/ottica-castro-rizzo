---
description: Rules for Phase 1 — analyzing Figma designs, mapping site structure, and downloading images
---

# Phase 1: Figma Design Analysis

## 1.1 — Extract the Design

Use Figma MCP tools in this order:

1. `get_metadata` — file structure, page names, top-level frames
2. `get_design_context` — for each page/section, extract design code and context
3. `get_screenshot` — capture visual references for QA comparison

## 1.2 — Map the Site Structure

Produce `SITE_MAP.md` documenting:

- **Figma file key** (`fileKey`) at the top of the document — Phase 2 agents need this to call Figma MCP tools
- Pages identified (Home, About, Contact, etc.)
- Sections per page (Hero, Features, Testimonials, Footer, etc.) — **each section MUST include its `nodeId`** so Phase 2 agents can call `get_design_context` and `get_screenshot` directly
- Shared components (Navbar, Footer, Buttons, Cards, etc.)
- Design tokens observed (colors, spacing, typography, border radius, shadows)
- **Content max-width** from the top-level Figma frame (used for `--container-max`)

## 1.3 — Download and Store Images

The `get_design_context` tool returns asset URLs for all images in the design. These URLs are temporary (expire in 7 days).

**For every image returned:**

1. **Download ALL images using a Node.js script** — do NOT use `curl` or `wget` as these may be blocked by shell permissions. Use the following pattern to batch-download all images in a single `node -e` command:
   ```js
   node -e "
   const fs = require('fs');
   const https = require('https');
   const path = require('path');
   const dir = path.join(__dirname, 'public/assets/images');
   const images = [['filename.png', 'https://...'], ...];
   function download(name, url) {
     return new Promise((resolve) => {
       https.get(url, {headers: {'User-Agent': 'Mozilla/5.0'}}, (res) => {
         if (res.statusCode >= 300 && res.statusCode < 400 && res.headers.location) {
           const mod = res.headers.location.startsWith('https') ? require('https') : require('http');
           mod.get(res.headers.location, (res2) => {
             const chunks = [];
             res2.on('data', c => chunks.push(c));
             res2.on('end', () => { fs.writeFileSync(path.join(dir, name), Buffer.concat(chunks)); resolve({name, size: Buffer.concat(chunks).length, status: 'OK'}); });
           }).on('error', e => resolve({name, size: 0, status: 'FAILED: ' + e.message}));
         } else {
           const chunks = [];
           res.on('data', c => chunks.push(c));
           res.on('end', () => { fs.writeFileSync(path.join(dir, name), Buffer.concat(chunks)); resolve({name, size: Buffer.concat(chunks).length, status: 'OK'}); });
         }
       }).on('error', e => resolve({name, size: 0, status: 'FAILED: ' + e.message}));
     });
   }
   Promise.all(images.map(([n, u]) => download(n, u))).then(results => {
     results.forEach(r => console.log(r.status + ' | ' + r.name + ' | ' + r.size + ' bytes'));
   });
   "
   ```
2. **Verify the actual file type** after downloading — run `file <path>` to check. Figma exports may return PNGs even when saved as `.jpg`. Always rename to match the actual content type (e.g., if `file` reports PNG but extension is `.jpg`, rename to `.png`).
3. **Verify the file size is reasonable** — raster images (JPG/PNG) for content areas should generally be >5KB. Files under 1KB are likely SVG placeholders or failed downloads. If a "photo" image is tiny (<5KB), the Figma export returned a vector placeholder instead of the rasterized image — flag this in IMAGE_MANIFEST.md and notify the user.
4. Save to `public/assets/images/` with a descriptive filename derived from the Figma layer name (e.g., `hero-background.png`, `team-james-vile.png`, `icon-chair.svg`)
5. Record in `IMAGE_MANIFEST.md` with: filename, dimensions, actual file type, file size, description, download status

**If a download fails:**

- Log the failure in `IMAGE_MANIFEST.md` with status `FAILED` and the reason (expired URL, 404, network error, etc.)
- Use a placeholder (`/assets/images/placeholder.svg`) as a temporary fallback in the component
- **Immediately notify the user** which image failed, what component it belongs to, and why it failed
- Do NOT silently skip failed images — the user must always know

**Filename conventions:**
- Use kebab-case: `hero-background.jpg`, `logo-partner-vision.svg`
- Prefix with section name: `hero-`, `about-`, `team-`, `services-`, `footer-`
- Icons: `icon-{name}.svg`
- Logos: `logo-{name}.svg`
- Team/people: `team-{person-name}.jpg`

## 1.4 — Layout Analysis

For each section, determine:
- **Background treatment**: Does the background (color, image, gradient) span full viewport width?
- **Content width**: What is the max-width of the actual content within the section? (Usually matches the Figma frame width)
- **Decorative elements**: Do any decorative elements (grid lines, dividers, patterns) exist? What width do they span?

Document these findings in `SITE_MAP.md` so Phase 2 builds the correct layout structure.

## 1.5 — SITE_MAP.md Format (CRITICAL for Phase 2)

The SITE_MAP.md MUST include Figma references so Phase 2 agents can call Figma MCP tools directly. Without these, agents will build from text descriptions and produce inaccurate results.

**Required format:**

```markdown
# Site Map

**Figma File Key**: `<fileKey>`
**Source URL**: `<full Figma URL>`
**Container Max-Width**: `<frame width>px`

## Pages

### Page: Home
Node ID: `<pageNodeId>`

#### Sections

| Section | Node ID | Component | Background | Notes |
|---------|---------|-----------|------------|-------|
| Navbar  | `<id>`  | Navbar.astro | transparent | Shared |
| Hero    | `<id>`  | Hero.astro   | dark        | Full-width bg image |
| ...     | ...     | ...          | ...         | ... |

## Shared Components
| Component | Node ID | Description |
|-----------|---------|-------------|
| Button    | `<id>`  | Primary/secondary variants |
| ...       | ...     | ... |
```

Every section entry MUST have a `nodeId`. Phase 2 depends on these references to call `get_design_context` and `get_screenshot` per section.
