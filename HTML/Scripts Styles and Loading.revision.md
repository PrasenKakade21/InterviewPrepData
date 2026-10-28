# Scripts, Styles and Loading — Revision Notes (Intermediate)
> One-line mental model: loading is a priority problem — inline critical CSS, defer app JS, preload only what blocks first paint.

## Core recall
- `link rel="stylesheet"` is render-blocking because paint waits for the CSSOM.
- `style` embeds CSS with no extra request, ideal for critical above-the-fold rules.
- Plain `<script>` blocks parsing while downloading and executing.
- `defer` = parallel download, ordered execution after parsing; use for app code.
- `async` = parallel download, execution when ready, order not guaranteed; use for independent widgets.
- `type="module"` = deferred by default, strict mode, own scope, `import`/`export`.
- `noscript` shows fallback content when JS is unavailable.
- `preload` = critical current-page resource at high priority; `prefetch` = likely future page; `preconnect` = warm a third-party origin.
- `integrity` + `crossorigin` verify CDN assets; `crossorigin` is also required for CORS fonts.
- Critical rendering path = HTML → DOM, CSS → CSSOM, render tree → layout → paint.

## Must-remember code
```html
<link rel="stylesheet" href="/css/app.css" />
<style>
  body { margin: 0; font-family: Inter, sans-serif; }
</style>
```
```html
<script src="/js/analytics.js" async></script>
<script src="/js/app.js" defer></script>
```
```html
<script type="module" src="/js/app.js"></script>
<noscript><p>Search needs JavaScript.</p></noscript>
```
```html
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
<link rel="prefetch" href="/notes/html" as="document" />
```
```html
<script
  src="https://cdn.kakade.dev/lib.js"
  integrity="sha384-abc123..."
  crossorigin="anonymous"
  defer
></script>
```

## Interview traps
- Synchronous third-party scripts in `head`.
- Expecting `async` scripts to run in order.
- Treating modules as async (they are deferred, ordered).
- Preloading every asset, delaying the real critical path.
- Missing `crossorigin` on font preload or integrity checks.
- Loading app CSS with a huge file instead of critical inline CSS.
- Forgetting `noscript` for essential no-JS fallback.

## 3 things to remember
1. Critical CSS inline, rest deferred.
2. `defer` for app code, `async` for independent widgets.
3. Preload only the hero font/image and critical CSS.
