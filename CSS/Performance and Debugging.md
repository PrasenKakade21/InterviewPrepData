# Performance and Debugging

CSS can block rendering, trigger layout, and cause jank. Most performance work is avoiding layout-affecting animation, keeping CSS small, and loading fonts sensibly. Debugging is about finding the rule that actually won and the layer that actually painted.

The core idea:

```text
CSS performance
    ├── delivery → critical CSS, size, caching
    ├── rendering → avoid layout/paint storms
    ├── animation → transform/opacity only
    └── debugging → DevTools + isolation
```

---

# 1. Critical CSS and Render-Blocking CSS

Stylesheets in `<head>` block rendering until loaded and parsed. Critical CSS inlines the above-the-fold styles and defers the rest.

```html
<style>
  /* critical: header, hero */
  .site-header { display: flex; gap: 1rem; }
  .hero { min-height: 60dvh; }
</style>

<link rel="stylesheet" href="/css/app.css" media="print" onload="this.media='all'" />
<noscript><link rel="stylesheet" href="/css/app.css" /></noscript>
```

Output:

```text
Above-the-fold renders immediately; the full stylesheet loads
without blocking first paint.
```

Gotchas: inlining too much CSS harms caching and HTML size; measure with Lighthouse and real network conditions.

---

# 2. Unused CSS and Minification

Unused rules still download and parse. Audit with coverage tools, split CSS by route/component, and minify for production.

```text
styles/
  base.css       → global, small
  home.css       → route-specific
  dashboard.css  → route-specific
```

Output:

```text
Each page loads only the CSS it needs; unused rules are removed.
```

Gotchas: purge tools can remove dynamically constructed class names; safelist those patterns. Minification saves bytes but does not fix architectural bloat.

---

# 3. will-change, contain and content-visibility

These hints can improve rendering but are easy to misuse. `will-change` promotes an element ahead of time; `contain` limits layout/paint scope; `content-visibility: auto` skips offscreen rendering.

```css
.drawer {
  will-change: transform;
}

.card {
  contain: layout paint;
}

.section {
  content-visibility: auto;
  contain-intrinsic-size: 600px;
}
```

Output:

```text
Drawer is promoted for smooth sliding; card isolates layout/paint;
offscreen sections skip rendering until near the viewport.
```

Gotchas: permanent `will-change` wastes memory; `contain` can break sticky/overflow behavior; `content-visibility` needs intrinsic sizes to avoid scroll jumps.

---

# 4. Animating Transform and Opacity, Avoiding Layout Thrash

The rendering pipeline is layout → paint → composite. Animating `width`, `height`, `top`, or `left` forces layout every frame. `transform`/`opacity` can skip to composite.

```css
/* Good */
.toast {
  transition: transform 200ms ease, opacity 200ms ease;
}

/* Bad */
.toast-bad {
  transition: height 200ms ease, top 200ms ease;
}
```

Output:

```text
Transform/opacity animations stay smooth;
height/top animations cause layout and jank.
```

Layout thrash in JS: reading layout (`offsetHeight`) then writing styles repeatedly. Batch reads, then writes, or use `requestAnimationFrame`.

---

# 5. DevTools Layout Panels and Font Loading

DevTools shows matched rules, computed values, layout overlays, paint flashing, and coverage. Fonts should be subset, preloaded, and use `font-display: swap`.

```css
@font-face {
  font-family: "Kakade Sans";
  src: url("/fonts/kakade-sans.woff2") format("woff2");
  font-display: swap;
  unicode-range: U+0000-00FF;
}
```

```html
<link rel="preload" href="/fonts/kakade-sans.woff2" as="font" type="font/woff2" crossorigin />
```

Output:

```text
Only needed glyphs download; text renders with fallback immediately;
font swaps when ready.
```

Debug checklist:

- **Styles**: which rule won and what was overridden.
- **Computed**: final values.
- **Layout**: grid/flex overlays, box model.
- **Rendering**: paint flashing, layer borders.
- **Coverage**: unused CSS.
- **Performance**: long tasks, layout shifts.

---

# Real-World Uses

A landing page inlines critical CSS, defers the rest, swaps fonts, and animates only transforms.

```html
<style>
  .site-header { display: flex; align-items: center; gap: 1rem; }
  .hero { min-height: 60dvh; display: grid; place-items: center; }
</style>
<link rel="preload" href="/fonts/kakade-sans.woff2" as="font" type="font/woff2" crossorigin />
<link rel="stylesheet" href="/css/app.css" media="print" onload="this.media='all'" />
```

```css
.hero__cta {
  transition: transform 180ms ease, opacity 180ms ease;
}

.hero__cta:hover {
  transform: translateY(-2px);
}
```

Output:

```text
Fast first paint, no font blocking, smooth CTA interaction.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Critical CSS** | Inline above-the-fold styles |
| **Render-blocking CSS** | `<link>` in head blocks paint |
| **Unused CSS** | Coverage audit + route splitting |
| **will-change** | Promote sparingly, remove after |
| **contain** | Limit layout/paint scope |
| **content-visibility** | Skip offscreen rendering |
| **Transform/opacity** | Composite-friendly animation |
| **Layout thrash** | Batch reads/writes |
| **DevTools** | Styles, computed, layout, coverage |
| **Font loading** | Subset, preload, swap |

## The Mental Model

```text
performance
  ├── delivery → critical CSS, split, minify, cache
  ├── rendering → contain, content-visibility, will-change
  ├── animation → transform/opacity
  └── diagnosis → DevTools panels + coverage
```

### The 3 things to remember

**1. Inline critical CSS:**

```html
<style>.hero { min-height: 60dvh; }</style>
```

**2. Animate transform/opacity:**

```css
.toast { transition: transform 200ms ease, opacity 200ms ease; }
```

**3. Fonts swap + preload:**

```css
@font-face {
  font-family: "Kakade Sans";
  src: url("/fonts/kakade-sans.woff2") format("woff2");
  font-display: swap;
}
```

> **CSS performance is delivery + rendering discipline — critical CSS, small files, composited animation, and fonts that never block text.**
