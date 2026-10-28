# Best Practices and Performance — Revision Notes (Intermediate)
> One-line mental model: good HTML is semantic, shallow, dimensioned, and non-blocking — it works before CSS and JS arrive.

## Core recall
- Start with the semantic element; `div` only when nothing fits.
- Div soup removes landmarks, weakens accessibility, and complicates CSS.
- Always set image `width`/`height` to reserve space and prevent CLS.
- Keep the hero/LCP image eager (`fetchpriority="high"`), lazy-load only offscreen images.
- `decoding="async"` avoids blocking decode; `srcset` serves the right resolution.
- `defer` app scripts; `async` independent widgets; no synchronous third-party scripts in head.
- Preload only critical fonts/CSS/LCP images — preload spam competes with first paint.
- Keep DOM depth low: fewer wrappers, simpler CSS, faster updates.
- Document outline: one `h1`, ordered `h2`/`h3`, no skipped levels.
- Progressive enhancement: content, links, and forms work without CSS/JS; add behavior on top.
- Name classes by role (`site-header`, `card__title`), not appearance.

## Must-remember code
```html
<img
  src="/img/hero.jpg"
  alt="Kakade's desk setup"
  width="1200"
  height="675"
  fetchpriority="high"
/>
<img
  src="/img/thumbnail.jpg"
  alt="HTML notes preview"
  width="320"
  height="180"
  loading="lazy"
  decoding="async"
/>
```
```html
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
<script src="/js/app.js" defer></script>
<script src="/js/analytics.js" async></script>
```
```html
<header>
  <nav aria-label="Primary">
    <a href="/notes">Notes</a>
  </nav>
</header>
```
```html
<noscript>
  <p>Search requires JavaScript. Browse the full index instead.</p>
</noscript>
```

## Interview traps
- Lazy-loading the LCP hero image.
- Missing width/height → layout shift.
- Preloading every asset.
- Synchronous scripts in head.
- Deep wrapper nesting for no semantic reason.
- Div + click handler instead of anchor/button.
- Appearance-based class names that break on redesign.
- Relying on JS for core content and navigation.

## 3 things to remember
1. Semantic, shallow markup first.
2. Dimensions + selective lazy loading protect CLS/LCP.
3. Defer JS and preload only what blocks first paint.
