---
paths:
  - "src/pages/**"
  - "src/layouts/**"
  - "src/components/**"
description: Rules for Phase 5 — SEO, accessibility, and HTML quality
---

# Phase 5: SEO & Accessibility

## HTML Semantics

- One `<h1>` per page
- Logical heading hierarchy (h1 → h2 → h3, no skipping)
- `<main>` wraps primary content
- `<nav>` for navigation, `<footer>` for footer
- Lists use `<ul>`/`<ol>` not divs
- Links use `<a>`, buttons use `<button>` — never interchangeable

## Image Accessibility

- Every `<img>` has a descriptive `alt` attribute
- Decorative images use `alt=""` and `role="presentation"`
- Images have explicit `width` and `height`
- Use `loading="lazy"` below the fold

## Meta & SEO

- Unique `<title>` per page
- `<meta name="description">` per page
- Open Graph tags: `og:title`, `og:description`, `og:image`, `og:url`
- `<link rel="canonical">` on each page
- Proper `lang` attribute on `<html>`
- Favicon configured

## Accessibility (a11y)

- Color contrast meets WCAG AA (4.5:1 normal text, 3:1 large text)
- Visible focus states on all interactive elements
- `aria-label` on icon-only buttons/links
- Skip-to-content link present
- Form inputs have associated `<label>` elements
- Interactive elements are keyboard accessible
