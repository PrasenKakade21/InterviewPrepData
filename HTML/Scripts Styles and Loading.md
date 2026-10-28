# Scripts, Styles and Loading

How CSS and JavaScript enter the page decides how fast it renders. A blocking stylesheet or a synchronous script in `head` can freeze the first paint; `defer`, `async`, and `preload` exist to fix that.

The core idea:

```text
HTML parse
    ├── <link rel=stylesheet> → blocks render until CSSOM ready
    ├── <script>              → blocks parse until executed
    ├── <script defer>        → after parse, in order
    ├── <script async>        → whenever downloaded
    └── <script type=module>  → deferred by default
```

Where a tag goes is a performance decision, not a style preference.

---

# 1. link rel="stylesheet" and the style Element

`link rel="stylesheet"` loads an external stylesheet; `style` embeds CSS directly in the document. Both block rendering until CSS is ready, because the browser will not paint unstyled content.

```html
<head>
  <link rel="stylesheet" href="/css/style.css" />
  <style>
    .hero {
      padding: 2rem;
      background: #0f172a;
      color: #fff;
    }
  </style>
</head>
```

Output:

```text
External rules + inline critical styles applied before first paint.
```

Think:

```text
link → cached, reusable, separate file
style → no request, but shipped on every page
```

Interview trap: CSS is render-blocking by design. Large stylesheets delay first paint; inline only the critical above-the-fold CSS and load the rest asynchronously.

---

# 2. script src, defer and async

A plain `<script>` blocks HTML parsing while it downloads and executes. `defer` downloads in parallel and runs after parsing, in document order. `async` downloads in parallel and runs as soon as it is ready, in unpredictable order.

```html
<script src="/js/analytics.js" async></script>
<script src="/js/app.js" defer></script>
<script src="/js/vendor.js" defer></script>
```

Output:

```text
analytics.js runs whenever it finishes (order not guaranteed)
vendor.js then app.js run in order after parsing
```

| Attribute | Downloads | Executes | Order |
| --------- | --------- | -------- | ----- |
| none | Blocking | Immediately | Document |
| `defer` | Parallel | After parsing | Document |
| `async` | Parallel | When ready | Unpredictable |
| `type="module"` | Parallel | After parsing | Document |

Use `defer` for app code that depends on the DOM or other scripts, `async` for independent widgets (analytics, ads).

---

# 3. type="module" and noscript

`type="module"` loads an ES module: deferred by default, strict mode, own scope, and `import`/`export` supported. `noscript` provides fallback content when JavaScript is disabled or blocked.

```html
<script type="module" src="/js/app.js"></script>

<noscript>
  <p>Search needs JavaScript. All articles are still readable.</p>
</noscript>
```

Output:

```text
Module executes after parsing, in strict mode.
With JS off, the noscript paragraph appears.
```

Interview trap: modules are deferred, not async — they still preserve order. A module with a top-level `await` delays its own execution but not HTML parsing.

---

# 4. preload, prefetch, integrity and crossorigin

`rel="preload"` fetches a critical resource early with high priority; `prefetch` fetches a likely future navigation at low priority. `integrity` verifies a file hash for CDN assets, and `crossorigin` is required for integrity checks and CORS fonts.

```html
<link rel="preload" href="/fonts/kakade.woff2" as="font" type="font/woff2" crossorigin />
<link rel="preload" href="/css/critical.css" as="style" />
<link rel="prefetch" href="/notes/html" as="document" />

<script
  src="https://cdn.kakade.dev/lib.js"
  integrity="sha384-abc123..."
  crossorigin="anonymous"
  defer
></script>
```

Output:

```text
Font and critical CSS start downloading immediately.
Next-page HTML fetched at idle priority.
CDN script rejected if its hash does not match.
```

| Hint | Priority | Use |
| ---- | -------- | --- |
| `preload` | High | Critical font/CSS/script |
| `prefetch` | Low | Next likely page |
| `preconnect` | — | Warm up a third-party origin |

Interview trap: preloading everything cancels the benefit — it competes with the resources that actually block first paint.

---

# 5. The Critical Rendering Path

The critical rendering path is the minimum work to show above-the-fold content: fetch HTML, build DOM, fetch CSS, build CSSOM, render, layout, paint. Scripts and stylesheets in `head` shape that path.

```html
<head>
  <link rel="preload" href="/fonts/kakade.woff2" as="font" type="font/woff2" crossorigin />
  <style>/* critical above-the-fold CSS */</style>
  <link rel="stylesheet" href="/css/app.css" media="print" onload="this.media='all'" />
  <script src="/js/app.js" defer></script>
</head>
```

Output:

```text
Critical CSS inline → first paint not blocked by app.css
Font preloaded → less text flash
App JS deferred → parsing never blocked
```

Optimization checklist:

- Inline critical CSS, defer the rest.
- `defer` app scripts, `async` independent widgets.
- Preload only the hero font/image.
- Avoid synchronous third-party scripts in `head`.
- Keep module graphs small; split non-critical code.

---

# Real-World Uses

A production head preloads the hero font, inlines critical CSS, defers app JS, and loads analytics asynchronously.

```html
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />

  <link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />

  <style>
    body { margin: 0; font-family: Inter, sans-serif; }
  </style>

  <link rel="stylesheet" href="/css/app.css" />

  <script src="/js/analytics.js" async></script>
  <script src="/js/app.js" defer></script>
</head>
```

Output:

```text
Hero font fetched early, critical CSS inline, app CSS applied,
analytics non-blocking, app JS after parsing.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **link rel stylesheet** | External CSS; render-blocking |
| **style** | Inline CSS; no request, shipped per page |
| **script src** | Plain script blocks parsing |
| **defer** | Parallel download, ordered after parse |
| **async** | Parallel download, runs when ready |
| **type module** | Deferred by default, strict, scoped |
| **noscript** | Fallback when JS is off |
| **preload and prefetch** | Critical now vs likely later |
| **integrity and crossorigin** | Hash verification; CORS for fonts |
| **Critical rendering path** | DOM + CSSOM → render → paint |

## The Mental Model

```text
head loading
  │
  ├── CSS
  │     ├── critical → inline in <style>
  │     └── rest → link stylesheet (or async load)
  │
  ├── JS
  │     ├── independent → async
  │     ├── app/DOM → defer
  │     └── ES modules → type=module (deferred)
  │
  └── hints
        ├── preload → critical font/image
        ├── prefetch → next page
        └── preconnect → third-party origin
```

### The 3 things to remember

**1. Defer app scripts:**

```html
<script src="/js/app.js" defer></script>
```

**2. Async only independent code:**

```html
<script src="/js/analytics.js" async></script>
```

**3. Preload selectively:**

```html
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
```

> **Loading is a priority problem — inline critical CSS, defer app JS, and preload only what blocks the first paint.**
