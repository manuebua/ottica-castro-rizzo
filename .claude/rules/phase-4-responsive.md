---
paths:
  - "src/components/**"
  - "src/styles/**"
description: Rules for Phase 4 — responsive migration from desktop to tablet and mobile
---

# Phase 4: Responsive Migration

Migrate desktop-down. Responsive layouts are derived from desktop using best practices.

## Tablet (768px–1279px)

- Reduce multi-column grids (e.g., 4-col → 2-col)
- Scale typography ~90-95% of desktop
- Maintain visual hierarchy and reading order
- Reduce horizontal padding
- Stack side-by-side layouts if cramped
- Navigation may need hamburger menu

## Mobile Landscape (480px–767px)

- Most layouts become single column
- Typography ~85-90% of desktop
- Navigation must be mobile menu (hamburger/slide-out)
- Images full-width or proportionally scaled
- Min 44px touch targets on all interactive elements
- Reduce padding further

## Mobile Portrait (<480px)

- Everything single column
- Min 16px body text for readability
- Full-width buttons where appropriate
- Compact but breathable spacing
- All interactive elements meet 44px min touch target
- No horizontal scrolling

## Responsive QA

For each breakpoint verify:
- No horizontal overflow/scrolling
- Text readable (not too small, not overflowing)
- Touch targets adequate (44px min)
- Images scale properly
- Layout stacks logically
- No content hidden or cut off
