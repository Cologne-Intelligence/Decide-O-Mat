# User Story: Design System Foundation

**As a** user
**I want** a modern, visually consistent dark-themed interface
**So that** the application feels professional and cohesive across all pages.

## Acceptance Criteria
1. **Design Tokens**: A complete set of CSS custom properties extracted from the Figma designs (colors, spacing, typography, shadows, radii).
2. **Dark Theme**: The application renders with the dark color palette as defined in Figma (dark backgrounds, light text, purple/lavender accents, lime green CTAs).
3. **CSS Architecture**: All components use a consistent styling approach (CSS Modules, global CSS, or utility classes — to be decided).
4. **Theme Toggle** (Optional): If decided, a light/dark mode toggle is available in the navigation bar, with the dark theme as default.
5. **Typography**: Font family and weight hierarchy matches the Figma designs (bold display font for headings, regular for body).
6. **No Visual Regression**: Existing functionality continues to work correctly after the token swap.

## Decisions Required
- **Resolved — Theming strategy**: Dark-only was chosen. No light/dark toggle exists or is planned, despite the toggle icon shown in Figma.
- **Resolved — CSS architecture**: CSS Modules (co-located `.module.css` files per component) was chosen over Tailwind, global CSS with BEM, and CSS-in-JS — confirmed by the `*.module.css` files present alongside components such as `Header.jsx`, `Decision.jsx`, `ElectionHero.jsx`, `FloatingArgumentInput.jsx`, and `ContextMenu.jsx`.

## Technical Notes
- Current tokens live in `frontend/src/index.css` `:root` block.
- Legacy custom-property names (`--color-primary`, `--color-background`, etc.) are remapped to the new dark-theme Figma token values inside `frontend/src/index.css`, so any remaining inline-styled legacy code still renders correctly without needing immediate migration.
- The Figma design file key is `ac7nNE0NZCX5iJy25dxDkY`.

## Figma Reference
- All screens use the same token set (consistent dark backgrounds, accent colors, spacing).

## Implementation Plan
- [IMP-028-Design-System-Foundation](../plans/IMP-028-Design-System-Foundation.md)
