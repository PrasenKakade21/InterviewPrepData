# Document and Metadata Tags

These tags define the document itself: its mode, root, metadata, and resources. They are mostly invisible, but they decide encoding, tab title, mobile scaling, search snippets, and when CSS/JS load.

The core idea:

```text
<!DOCTYPE html>
    ↓
<html> → <head> (metadata) + <body> (content)
                ↓
        title, meta, link, base, style, script
```

Every page needs the skeleton; the rest of this file explains each piece.

---

# `<!DOCTYPE html>`

Declares HTML5 and puts the browser in standards mode. It is not an element and has no closing tag.

```html
<!DOCTYPE html>
<html lang="en">
  <body><p>Kakade</p></body>
</html>
```

Output:

```text
Standards mode; document.compatMode === "CSS1Compat"
```

Gotchas: missing doctype → quirks mode; legacy long doctypes still work but are unnecessary.

---

# `<html>`

The root element. Carries `lang` (required for accessibility) and optionally `dir`.

| Attribute | Purpose | Example |
| --------- | ------- | ------- |
| `lang` | Content language | `lang="en"` |
| `dir` | Text direction | `dir="rtl"` |

```html
<html lang="en">
  <head>...</head>
  <body>...</body>
</html>
```

Output:

```text
Screen readers use lang for pronunciation; search engines use it for targeting.
```

Gotchas: only one `<html>` per document; `lang` on `<html>` is inherited by everything.

---

# `<head>`

Container for machine-readable metadata. Not rendered.

```html
<head>
  <meta charset="UTF-8" />
  <title>Kakade's Notes</title>
  <link rel="stylesheet" href="/css/app.css" />
</head>
```

Output:

```text
No visible output; title appears in the tab.
```

Gotchas: exactly one per document; visible content placed here is moved or dropped by the parser.

---

# `<body>`

Container for everything the user sees and interacts with.

```html
<body>
  <h1>Kakade</h1>
  <p>Frontend learner.</p>
</body>
```

Output:

```text
Kakade
Frontend learner.
```

Gotchas: one per document; scripts at the end of body still block parsing while they execute, so prefer `defer`.

---

# `<title>`

Sets the document title: browser tab, bookmark name, and default search-result heading.

```html
<title>HTML Forms for Interviews — Kakade.dev</title>
```

Output:

```text
Tab: HTML Forms for Interviews — Kakade.dev
```

Gotchas: required for valid documents; keep it unique and ~50–60 characters; it is not an `h1`.

---

# `<meta>`

Provides metadata as name/value or charset/http-equiv pairs. Void element.

| Form | Example | Purpose |
| ---- | ------- | ------- |
| charset | `<meta charset="UTF-8">` | Byte decoding |
| viewport | `<meta name="viewport" content="width=device-width, initial-scale=1">` | Mobile scaling |
| description | `<meta name="description" content="...">` | Search snippet |
| robots | `<meta name="robots" content="index, follow">` | Crawling policy |
| Open Graph | `<meta property="og:title" content="...">` | Social preview |
| http-equiv | `<meta http-equiv="refresh" content="5;url=/home">` | Legacy redirect |

```html
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<meta name="description" content="Kakade's HTML interview notes." />
<meta property="og:title" content="Kakade's HTML Notes" />
```

Output:

```text
Correct decoding, mobile scaling, search snippet, social title.
```

Gotchas: charset must be within the first 1024 bytes; Open Graph uses `property`, standard meta uses `name`.

---

# `<link>`

Links external resources and declares relationships. Void element; most commonly used for stylesheets and icons.

| `rel` | Purpose |
| ----- | ------- |
| `stylesheet` | Load CSS |
| `icon` | Favicon |
| `canonical` | Preferred URL |
| `preload` | Fetch critical resource early |
| `prefetch` | Fetch likely next page |
| `preconnect` | Warm up an origin |
| `alternate` | RSS, translations, print |

```html
<link rel="stylesheet" href="/css/app.css" />
<link rel="icon" href="/favicon.ico" />
<link rel="canonical" href="https://kakade.dev/html" />
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
```

Output:

```text
CSS applied; icon in tab; canonical consolidated; font fetched early.
```

Gotchas: `preload` needs a correct `as` value; font preloads need `crossorigin`; preloading everything hurts performance.

---

# `<base>`

Sets a default URL and/or default target for all relative links in the document.

```html
<head>
  <base href="https://kakade.dev/" target="_blank" />
</head>
<body>
  <a href="notes">Notes</a>
</body>
```

Output:

```text
Link resolves to https://kakade.dev/notes and opens in a new tab.
```

Gotchas: only the first `<base>` is honored; it also affects fragment links and form actions — use sparingly.

---

# `<style>`

Embeds CSS directly in the document. Render-blocking like an external stylesheet, but with no extra request.

```html
<style>
  .hero {
    padding: 2rem;
    background: #0f172a;
    color: #fff;
  }
</style>
```

Output:

```text
Hero styled before first paint (critical CSS pattern).
```

Gotchas: applies globally; `scoped` was removed from the spec. Use it for critical CSS, not the whole design system.

---

# `<script>`

Embeds or loads JavaScript. Behavior depends on placement and attributes.

| Attribute | Effect |
| --------- | ------ |
| `src` | Load external file |
| `defer` | Parallel download, ordered execution after parse |
| `async` | Parallel download, execute when ready |
| `type="module"` | ES module, deferred by default |
| `integrity` | Subresource hash check |
| `crossorigin` | CORS mode for the request |
| `nomodule` | Fallback for legacy browsers |

```html
<script src="/js/app.js" defer></script>
<script type="module" src="/js/search.js"></script>
```

Output:

```text
app.js executes after parsing, in order.
search.js executes as a deferred module.
```

Gotchas: inline scripts cannot use `defer`/`async`; module scripts are always strict and scoped.

---

# `<noscript>`

Shows fallback content when scripting is disabled or unsupported.

```html
<noscript>
  <p>Search needs JavaScript. Browse the full index instead.</p>
</noscript>
```

Output:

```text
JS on: nothing shown
JS off: fallback paragraph visible
```

Gotchas: with JS enabled, its contents are not rendered (and in some parsers, not even loaded). Do not put essential content only in `noscript`.

---

# Quick Reference

| Tag | Void | Purpose |
| --- | ---- | ------- |
| `doctype` | — | Standards mode |
| `html` | No | Root + `lang` |
| `head` | No | Metadata container |
| `body` | No | Visible content |
| `title` | No | Tab/search title |
| `meta` | Yes | Charset, viewport, SEO, social |
| `link` | Yes | Stylesheets, icons, relationships |
| `base` | Yes | Default URL/target |
| `style` | No | Embedded CSS |
| `script` | No | JavaScript |
| `noscript` | No | No-JS fallback |

> **The head is the control panel — one doctype, one root, one head, one body, and metadata that tells the browser exactly how to render.**
