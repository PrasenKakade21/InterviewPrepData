# Document Structure and Metadata

A web page is a tree: `html` is the root, `head` describes the document, and `body` shows it. Metadata in the head is invisible, yet it controls the tab title, character decoding, mobile scaling, search snippets, and which CSS files load.

The core idea:

```text
<html lang="en">
    ├── <head>  → title, charset, viewport, description, links
    └── <body>  → headings, text, images, forms
```

Get the head right once, and encoding bugs, mobile zoom-out, and broken styles disappear.

---

# 1. The html, head and body Elements

`html` wraps the entire document and carries `lang`. `head` contains machine-readable metadata. `body` contains user-visible content. The browser tolerates missing tags, but explicit structure is required for maintainability and accessibility.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Kakade Notes</title>
  </head>
  <body>
    <h1>Kakade Notes</h1>
  </body>
</html>
```

Output:

```text
Tab title:  Kakade Notes
Page:       Kakade Notes
```

Think:

```text
html
 ├── head → instructions for the browser
 └── body → content for the user
```

Interview trap: there is only one `head` and one `body` per document. Duplicate tags are merged unpredictably by the parser.

---

# 2. The title Element and meta charset

`title` sets the browser tab text and the default search-result title. `meta charset="UTF-8"` tells the browser how to decode bytes into characters — without it, symbols like `₹`, `é`, or emoji can become mojibake.

```html
<head>
  <meta charset="UTF-8" />
  <title>Kakade — Frontend Interview Notes</title>
</head>
```

Output:

```text
Tab: Kakade — Frontend Interview Notes
Search result heading: Kakade — Frontend Interview Notes
```

Bad:

```html
<!-- Missing charset: curl output shows broken symbols -->
<p>Price: ₹499</p>
```

Good:

```html
<meta charset="UTF-8" />
<p>Price: ₹499</p>
```

Notice: `charset` must appear within the first 1024 bytes of the document for the browser to apply it reliably — put it first in `head`.

---

# 3. meta viewport and meta description

`meta viewport` makes the page scale to the device instead of rendering at a fake desktop width. `meta description` is the snippet search engines and social previews often show.

```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
<meta
  name="description"
  content="Kakade's HTML, CSS and JavaScript interview notes."
/>
```

Output:

```text
Mobile: page width matches the device, no zoomed-out desktop layout
Search snippet: Kakade's HTML, CSS and JavaScript interview notes.
```

Compare:

| Meta | Purpose | Common value |
| ---- | ------- | ------------ |
| `viewport` | Mobile scaling | `width=device-width, initial-scale=1` |
| `description` | Search snippet | 120–160 characters |

Interview trap: `user-scalable=no` and `maximum-scale=1` block pinch-zoom and are an accessibility failure. Use the plain viewport tag.

---

# 4. The link Element and base Element

`link` connects external resources — most commonly a stylesheet — and declares relationships like canonical URLs. `base` sets a default URL or target for every relative link on the page.

```html
<head>
  <link rel="stylesheet" href="/css/style.css" />
  <link rel="icon" href="/favicon.ico" />
  <base href="https://kakade.dev/" />
</head>
<body>
  <a href="notes">Notes</a>
</body>
```

Output:

```text
Stylesheet loads from /css/style.css
Link resolves to https://kakade.dev/notes
```

When `base` is absent, `href="notes"` resolves relative to the current page — usually the safer default. `link` is not only for CSS: `rel="canonical"`, `rel="preload"`, and `rel="alternate"` all use it.

| `rel` | Meaning |
| ----- | ------- |
| `stylesheet` | Load CSS |
| `icon` | Favicon |
| `canonical` | Preferred URL for SEO |
| `preload` | Fetch a critical resource early |

---

# 5. lang and dir Attributes

`lang` declares the content language for screen readers, spellcheck, and search engines. `dir` sets text direction: `ltr` for English, `rtl` for Arabic or Hebrew.

```html
<html lang="en" dir="ltr">
  <body>
    <p>Hello Kakade</p>
    <p lang="hi">नमस्ते काकड़े</p>
  </body>
</html>
```

Output:

```text
Hello Kakade
नमस्ते काकड़े
```

Think:

```text
lang → pronunciation + translation hints
dir  → left-to-right or right-to-left layout
```

Interview trap: `lang` on `<html>` is inherited. Set it on an inline element only when that specific content switches language.

---

# Real-World Uses

A production head for Kakade's blog combines encoding, mobile scaling, search metadata, canonical URL, and a stylesheet in a predictable order.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Kakade — HTML Notes</title>
    <meta name="description" content="Beginner-friendly HTML interview notes." />
    <link rel="canonical" href="https://kakade.dev/html" />
    <link rel="stylesheet" href="/css/style.css" />
  </head>
  <body>
    <h1>HTML Notes</h1>
  </body>
</html>
```

Output:

```text
Tab: Kakade — HTML Notes
Mobile: correctly scaled
Snippet: Beginner-friendly HTML interview notes.
Stylesheet: loaded
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **html element** | Root; carries `lang` and `dir` |
| **head element** | Metadata, never visible |
| **body element** | All rendered content |
| **title** | Tab text and search-result heading |
| **meta charset** | UTF-8, placed first in `head` |
| **meta viewport** | `width=device-width, initial-scale=1` |
| **meta description** | 120–160 character search snippet |
| **link element** | Stylesheets, icons, canonical, preload |
| **base element** | Default URL/target for relative links |
| **lang and dir** | Language and text direction |

## The Mental Model

```text
document
  │
  ├── html lang="en"
  │     ├── head
  │     │    ├── charset → decode bytes
  │     │    ├── viewport → scale to device
  │     │    ├── title → tab + search
  │     │    ├── description → snippet
  │     │    └── link → CSS/icons/canonical
  │     │
  │     └── body
  │          └── visible content
```

### The 3 things to remember

**1. Always start with encoding:**

```html
<meta charset="UTF-8" />
```

**2. Mobile needs viewport:**

```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

**3. One root, one head, one body:**

```html
<html lang="en">
  <head><title>Kakade</title></head>
  <body><h1>Kakade</h1></body>
</html>
```

> **The head is the control panel: encoding, scaling, title, description, and links — set it correctly and the body behaves.**
