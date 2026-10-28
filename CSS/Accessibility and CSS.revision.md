# Accessibility and CSS — Revision Notes (Intermediate)
> One-line mental model: accessible CSS is defaults you keep — focus, contrast, honest hiding, reflow, and user preferences.

## Core recall
- Never use `outline: none` without a visible replacement; use `:focus-visible`.
- WCAG AA contrast: 4.5:1 normal text, 3:1 large text (≥24px / ≥18.66px bold) and UI/focus indicators.
- `prefers-reduced-motion` → disable decorative motion and smooth scrolling.
- `prefers-color-scheme` + `color-scheme` → theme and native controls.
- Hiding: `display:none` for irrelevant content; `.sr-only` for screen-reader-only content.
- Skip links must become visible on focus, not be `display:none`.
- Logical properties (`margin-inline`, `padding-block`, `border-inline-start`) support RTL.
- Reflow: no horizontal scroll at 320px and 400% zoom; avoid fixed heights that clip text.
- Target size: 24×24 CSS px minimum, 44×44 recommended.
- Forced-colors mode: use system colors; `forced-color-adjust:none` only when necessary.

## Must-remember code
```css
:where(a, button, input, select, textarea, [tabindex]):focus-visible {
  outline: 3px solid #0f172a;
  outline-offset: 2px;
  border-radius: 4px;
}
```
```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
  border: 0;
}
```
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```
```css
.icon-button {
  min-width: 44px;
  min-height: 44px;
  display: inline-grid;
  place-items: center;
}
```

## Interview traps
- Removing focus outlines.
- Low-contrast placeholder or muted text.
- `display:none` on skip links or screen-reader labels.
- Ignoring reduced motion.
- Fixed heights that clip at 200% zoom.
- Physical left/right only, breaking RTL.
- Tiny touch targets.
- `forced-color-adjust:none` used casually.

## 3 things to remember
1. Focus ring + contrast are non-negotiable.
2. sr-only for hidden-but-available content.
3. Reflow, target size, and user preferences are CSS responsibilities.
