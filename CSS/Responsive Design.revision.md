# Responsive Design — Revision Notes (Intermediate)
> One-line mental model: responsive design is layers — viewport, fluid base, viewport queries for pages, container queries for components.

## Core recall
- Viewport meta `width=device-width, initial-scale=1` makes CSS pixels match device width; never block zoom.
- Media queries apply CSS by viewport features; mobile-first uses `min-width` on a small-screen base.
- Breakpoints should follow content needs (common 640/768/1024/1280), not device names.
- `min-width` and `max-width` at the same value overlap; use disjoint ranges or mobile-first only.
- Container queries need `container-type` on an ancestor; they respond to container size, not viewport.
- `clamp()` gives fluid type/spacing with min/max bounds, reducing breakpoints.
- Responsive images: `srcset` + `sizes`, plus width/height to prevent CLS.
- Print styles hide chrome, show URLs, and force readable colors.
- Test 320px → 1440px, 200% zoom, long content, landscape, real devices, reduced motion, dark mode.

## Must-remember code
```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```
```css
.cards {
  display: grid;
  gap: 1rem;
  grid-template-columns: 1fr;
}
@media (min-width: 640px) {
  .cards { grid-template-columns: repeat(2, 1fr); }
}
@media (min-width: 1024px) {
  .cards { grid-template-columns: repeat(3, 1fr); }
}
```
```css
.wrap {
  container-type: inline-size;
  container-name: card;
}
@container card (min-width: 480px) {
  .card { grid-template-columns: 160px 1fr; }
}
```
```css
.hero__title {
  font-size: clamp(2rem, 5vw + 1rem, 4.5rem);
}
```

## Interview traps
- Missing viewport meta → media queries see a desktop-width viewport.
- Overlapping min/max breakpoints.
- Chasing device-specific breakpoints instead of content.
- Container queries without `container-type`.
- Fluid type without min/max bounds.
- Hiding content on mobile instead of reflowing it.
- Blocking zoom with `user-scalable=no`.

## 3 things to remember
1. Mobile-first `min-width` queries.
2. `clamp()` for fluid type and spacing.
3. Media queries for pages, container queries for components.
