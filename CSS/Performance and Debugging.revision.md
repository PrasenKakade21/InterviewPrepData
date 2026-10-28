# Performance and Debugging — Revision Notes (Advanced)
> One-line mental model: performance is delivery + rendering discipline — critical CSS, composited animation, fonts that never block text.

## Core recall
- CSS blocks rendering because the CSSOM must exist before paint; inline critical above-the-fold CSS and defer the rest.
- Use DevTools Coverage to find unused CSS; split by route/component; minify.
- Purge tools can remove dynamically constructed classes — safelist them.
- `will-change` promotes an element but costs memory; use temporarily.
- `contain: layout paint` limits layout/paint scope; can affect sticky/overflow.
- `content-visibility: auto` skips offscreen rendering; pair with `contain-intrinsic-size`.
- Animate `transform`/`opacity` (composite) not `width`/`height`/`top`/`left` (layout).
- Layout thrash = read/write interleaving; batch reads then writes or use rAF.
- Fonts: subset, preload, `font-display: swap`, `unicode-range`.
- DevTools: Styles, Computed, Layout, Rendering (paint flashing), Coverage, Performance.

## Must-remember code
```html
<style>
  .site-header { display: flex; gap: 1rem; }
  .hero { min-height: 60dvh; }
</style>
<link rel="stylesheet" href="/css/app.css" media="print" onload="this.media='all'" />
<noscript><link rel="stylesheet" href="/css/app.css" /></noscript>
```
```css
@font-face {
  font-family: "Kakade Sans";
  src: url("/fonts/kakade-sans.woff2") format("woff2");
  font-display: swap;
  unicode-range: U+0000-00FF;
}
```
```css
.toast {
  transition: transform 200ms ease, opacity 200ms ease;
}
.section {
  content-visibility: auto;
  contain-intrinsic-size: 600px;
}
```

## Interview traps
- Inlining the entire stylesheet as "critical CSS".
- Permanent `will-change`.
- Animating layout properties.
- Forgetting `contain-intrinsic-size` with content-visibility.
- Purging dynamic class names.
- Blocking text on web fonts.
- Ignoring layout thrash in scroll handlers.

## 3 things to remember
1. Critical CSS + deferred rest.
2. Composite-only animation.
3. Measure with DevTools, not guesses.
