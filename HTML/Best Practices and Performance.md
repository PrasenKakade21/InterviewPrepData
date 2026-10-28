# Best Practices and Performance

Good HTML is boring in the best way: semantic, shallow, dimensioned, and loaded in the right order. Performance in HTML is mostly about what you *don't* do — no div soup, no missing dimensions, no blocking scripts, no preload spam.

The core idea:

```text
clean markup
    ├── meaning first
    ├── shallow DOM
    ├── reserved media space
    └── non-blocking loading
            ↓
faster render + easier maintenance + better a11y
```

Every rule here has a measurable payoff: CLS, LCP, accessibility, or review time.

---

# 1. Semantic-First Markup and Avoiding div Soup

Start with the element that describes the content; use `div` only when nothing semantic fits. Div soup makes CSS brittle and strips meaning from the accessibility tree.

```html
<!-- div soup -->
<div class="header">
  <div class="nav"><div class="link">Notes</div></div>
</div>

<!-- semantic -->
<header>
  <nav aria-label="Primary">
    <a href="/notes">Notes</a>
  </nav>
</header>
```

Output:

```text
div version: no landmarks, no link semantics
semantic version: banner + navigation landmarks, real links
```

Rule of thumb: if you are about to add a class named `header`, `nav`, `footer`, `article`, or `button` to a `div`, check whether the tag exists.

---

# 2. Image Dimensions and CLS, Lazy Loading

Set `width`/`height` on every image and lazy-load only offscreen ones. Reserve the hero image eagerly for LCP. Add `decoding="async"` so decode does not block rendering.

```html
<!-- Hero: eager, dimensioned -->
<img
  src="/img/hero.jpg"
  alt="Kakade's desk setup"
  width="1200"
  height="675"
  fetchpriority="high"
/>

<!-- Below the fold: lazy -->
<img
  src="/img/thumbnail.jpg"
  alt="HTML notes preview"
  width="320"
  height="180"
  loading="lazy"
  decoding="async"
/>
```

Output:

```text
Hero: fetched early, space reserved, no layout shift
Thumbnail: fetched when near viewport
```

Checklist:

- Dimensions on every `img`/`video`.
- `loading="lazy"` below the fold only.
- `fetchpriority="high"` for the LCP image.
- `decoding="async"` for non-critical images.
- `srcset` for large images across viewports.

---

# 3. Defer Scripts and Preload Critical Assets

Scripts should never block parsing unless they must run before paint (rare). Preload only the font/CSS/image that actually blocks first paint.

```html
<head>
  <link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
  <link rel="stylesheet" href="/css/app.css" />
  <script src="/js/app.js" defer></script>
  <script src="/js/analytics.js" async></script>
</head>
```

Output:

```text
Font starts early, CSS applied, app JS after parse,
analytics independent and non-blocking.
```

Avoid preloading icons, below-the-fold images, or every script — high priority for non-critical assets delays the real critical path.

---

# 4. Minimal DOM Depth and Document Outline

Every wrapper adds nodes to parse, style, lay out, and traverse. Fewer, meaningful elements are faster and easier to target. The heading outline should read like a table of contents.

```html
<main>
  <h1>Kakade's Notes</h1>

  <section aria-labelledby="html">
    <h2 id="html">HTML</h2>
    <h3>Text</h3>
    <h3>Forms</h3>
  </section>

  <section aria-labelledby="css">
    <h2 id="css">CSS</h2>
  </section>
</main>
```

Output:

```text
H1 Kakade's Notes
  H2 HTML
    H3 Text
    H3 Forms
  H2 CSS
```

Deep nesting (`div > div > div > section > div`) makes CSS specificity harder and slows large DOM updates. Flatten where possible.

---

# 5. Progressive Enhancement and Naming

Build the baseline in HTML so the page works without CSS/JS, then enhance. Name things by role (`site-header`, `card__title`) rather than appearance (`blue-box`, `big-text`).

```html
<form action="/search" method="get">
  <label for="q">Search notes</label>
  <input id="q" name="q" type="search" />
  <button type="submit">Search</button>
</form>

<noscript>
  <p>Search requires JavaScript. Browse the full index instead.</p>
</noscript>
```

Output:

```text
JS on: enhanced search behavior
JS off: form still submits, noscript explains the fallback
```

Progressive enhancement checklist:

- Content and navigation work without JS.
- Forms use real `action`/`method`.
- Links are anchors, not click handlers.
- Enhancements add behavior, not replace the baseline.
- Class names describe purpose, not pixels.

---

# Real-World Uses

A production page applies every rule: semantics, shallow structure, dimensioned images, deferred scripts, and a working no-JS baseline.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Kakade's Notes</title>

    <link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
    <link rel="stylesheet" href="/css/app.css" />
    <script src="/js/app.js" defer></script>
  </head>
  <body>
    <a href="#main" class="skip-link">Skip to content</a>

    <header>
      <h1>Kakade's Notes</h1>
      <nav aria-label="Primary"><a href="/notes">Notes</a></nav>
    </header>

    <main id="main">
      <article>
        <h2>Best Practices</h2>
        <img
          src="/img/notes.jpg"
          alt="HTML notes on a desk"
          width="800"
          height="450"
          loading="lazy"
        />
      </article>
    </main>
  </body>
</html>
```

Output:

```text
Fast first paint, semantic landmarks, lazy image with reserved space,
deferred JS, and a skip link.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Semantic-first** | Use the element that describes the content |
| **Avoid div soup** | `div` only when nothing semantic fits |
| **Image dimensions** | Always set width/height to prevent CLS |
| **Lazy loading** | Offscreen only; hero stays eager |
| **Defer scripts** | `defer` app code, `async` independent |
| **Preload critical assets** | Fonts/CSS/LCP only |
| **Minimal DOM depth** | Fewer wrappers, clearer structure |
| **Document outline** | Logical `h1` → `h2` → `h3` |
| **Progressive enhancement** | Works without CSS/JS, then improves |
| **Naming** | Role-based, not appearance-based |

## The Mental Model

```text
quality HTML
  │
  ├── structure → semantic, shallow, ordered headings
  ├── media → dimensions + lazy + srcset
  ├── loading → defer/async + selective preload
  ├── resilience → progressive enhancement
  └── maintenance → role-based names
```

### The 3 things to remember

**1. Dimensions prevent shift:**

```html
<img src="/notes.jpg" alt="HTML notes" width="800" height="450" loading="lazy" />
```

**2. Never block the first paint:**

```html
<script src="/js/app.js" defer></script>
```

**3. Build the baseline first:**

```html
<form action="/search" method="get">...</form>
```

> **Best-practice HTML is semantic, shallow, dimensioned, and non-blocking — it works before CSS and JS arrive.**
