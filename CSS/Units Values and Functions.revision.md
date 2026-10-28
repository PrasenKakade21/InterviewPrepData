# Units, Values and Functions — Revision Notes (Beginner)
> One-line mental model: units carry context — px for fixed edges, rem for scale, em for components, clamp() for fluid ranges.

## Core recall
- Absolute units: `px` (practical), `in` (96px), `cm`, `mm`, `pt`, `pc`.
- `px` is a CSS reference pixel, not a device pixel; it scales with zoom/DPR.
- `em` is relative to the current font size and compounds when nested; `rem` is always root-relative.
- `%` resolves against the containing block or property-specific base, not always the parent width.
- Viewport units: `vw`/`vh` (large), `dvh` (dynamic), `svh` (smallest) — use dvh/svh for mobile chrome.
- `ch` = width of "0"; useful for line length limits.
- `calc()` mixes units; spaces required around `+` and `-`.
- `clamp(min, preferred, max)` bounds fluid values; `min()`/`max()` pick a bound.
- Color formats: named, hex, `rgb()`, `hsl()`, `oklch()`; alpha applies to color, `opacity` to the whole element.
- Global keywords: `inherit`, `initial`, `unset`, `revert`; `auto` behavior is property-specific.

## Must-remember code
```css
.hero {
  min-height: 100dvh;
  padding: clamp(2rem, 6vw, 6rem);
}

.hero__title {
  font-size: clamp(2rem, 5vw + 1rem, 4.5rem);
  max-width: min(30ch, 90%);
  margin-inline: auto;
}
```
```css
.card {
  width: calc(100% - 2rem);
  padding: 1.5rem;
}
```
```css
.card {
  color: #0f172a;
  background: rgb(255 255 255 / 0.9);
  accent-color: oklch(0.4 0.1 250);
}
```

## Interview traps
- Using `em` for everything and getting compounding sizes.
- `100vh` overflow on mobile → use `dvh`.
- Missing spaces in `calc(100% - 2rem)`.
- Thinking `%` always means parent width.
- Confusing `opacity` with alpha colors.
- Hardcoding font sizes in px instead of scaling from root.

## 3 things to remember
1. rem for global scale, em for component-relative.
2. clamp() for fluid, min()/max() for bounds.
3. dvh for honest mobile viewport height.
