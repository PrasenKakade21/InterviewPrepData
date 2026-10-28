# HTML Fundamentals

HTML clicks once you see the chain: `tags` → `DOM tree` → `browser render`. It is not a programming language — it is the markup that tells the browser what each piece of content *is*, and everything else (CSS, JS) hangs off that structure.

The core idea:

```text
HTML source (.html)
    ↓
browser parses tags
    ↓
DOM tree (nodes)
    ↓
render tree + layout + paint
    ↓
page Kakade sees
```

HTML gives meaning, CSS gives style, JavaScript gives behavior. Get the markup right and the other two become easier.

---

# 1. What is HTML and the HTML5 Overview

`HTML` (HyperText Markup Language) describes content with elements: `<h1>` is a heading, `<p>` is a paragraph, `<a>` is a link. `HTML5` is the current living standard — it added semantic tags, native media, form validation, and browser APIs on top of the old HTML 4 markup.

```html
<h1>Kakade</h1>
<p>Learning HTML for interviews.</p>
<a href="/notes">Read notes</a>
```

Output:

```text
Kakade
Learning HTML for interviews.
Read notes (clickable link)
```

Markup is not code — there are no variables or loops. You declare meaning, and the browser decides the default presentation.

Think:

```text
HTML → structure ("what is it?")
CSS  → presentation ("how does it look?")
JS   → behavior ("what happens?")
```

Interview trap: saying "HTML is a programming language" is an instant red flag. HTML is a markup language, and it is not compiled — it is parsed.

---

# 2. Document Anatomy and the Doctype

Every page needs the same skeleton: `<!DOCTYPE html>`, a single `<html>` root, a `<head>` for metadata, and a `<body>` for visible content. The `doctype` tells the browser to use standards mode instead of quirks mode.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Kakade Portfolio</title>
  </head>
  <body>
    <h1>Hello Kakade</h1>
  </body>
</html>
```

Output:

```text
Browser tab: Kakade Portfolio
Page:        Hello Kakade
```

Anatomy:

```text
<!DOCTYPE html>     ← standards mode
<html lang="en">    ← root + language
  <head> ... </head> ← metadata, not visible
  <body> ... </body> ← visible content
</html>
```

Notice: the old `<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">` is gone. HTML5 uses the short form, and a missing doctype triggers quirks mode.

---

# 3. head vs body

`head` holds information *about* the page — title, charset, viewport, styles, scripts. `body` holds the content the user sees. Only one `head` and one `body` are allowed.

```html
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Kakade Notes</title>
  <link rel="stylesheet" href="/style.css" />
</head>
<body>
  <h1>Kakade Notes</h1>
  <p>Seven topics, zero fluff.</p>
</body>
```

Output:

```text
Tab title: Kakade Notes
Visible:   Kakade Notes
           Seven topics, zero fluff.
```

| Element | Visible? | Contains |
| ------- | -------- | -------- |
| `head` | No | `title`, `meta`, `link`, `style`, `script` |
| `body` | Yes | Headings, text, images, forms, sections |

Interview trap: text placed directly in `head` is not rendered — browsers move stray content into `body` or drop it, which is why a broken layout often starts with a misplaced tag.

---

# 4. Comments and Case-Insensitivity

HTML comments use `<!-- ... -->` and never appear on the page. Tag and attribute names are case-insensitive, but lowercase is the convention and the only style that looks right next to modern tooling.

```html
<!-- Kakade's profile card starts here -->
<section>
  <H2 Class="name">Kakade</H2>
  <p>Frontend learner</p>
</section>
```

Output:

```text
Kakade
Frontend learner
```

Both `<H2 Class="name">` and `<h2 class="name">` work — the browser normalizes them to lowercase in the DOM. Attribute *values* are not all case-insensitive (`class` names and `id` values are case-sensitive).

Bad:

```html
<!-- Don't hide real content in comments -->
<!-- <h1>Kakade</h1> -->
```

Good:

```html
<!-- Section: profile header -->
<h1>Kakade</h1>
```

Comments are for future humans, not for disabling code in production.

---

# 5. Markup to DOM to Render

The browser reads HTML top to bottom, builds the DOM, combines it with CSS into a render tree, then lays out and paints pixels. Understanding this chain explains why blocking scripts and late images hurt performance.

```html
<body>
  <h1>Kakade</h1>
  <img src="/kakade.png" alt="Kakade" width="120" height="120" />
</body>
```

Output:

```text
Kakade
[120 x 120 image of Kakade]
```

Think:

```text
HTML bytes
    ↓ parse
DOM tree
    ↓ + CSSOM
Render tree
    ↓ layout
Boxes with sizes
    ↓ paint
Pixels on screen
```

- Parsing can pause for synchronous `<script>` tags.
- Missing image dimensions cause layout shift when the image arrives.
- `defer` scripts run after parsing, which keeps first paint fast.

---

# 6. HTML vs XHTML and MIME Types

`XHTML` is HTML written with XML rules — every tag closed, attributes quoted, case-sensitive. Modern HTML5 is forgiving and served as `text/html`; XHTML was served as `application/xhtml+xml`. In practice, HTML5 won because it renders even when authors make small mistakes.

| Rule | HTML5 | XHTML |
| ---- | ----- | ----- |
| Void tags | `<br>` is fine | Must be `<br />` |
| Case | Insensitive | Lowercase required |
| Errors | Browser recovers | Parse error / no render |
| MIME type | `text/html` | `application/xhtml+xml` |

```html
<!-- Valid HTML5 -->
<img src="/kakade.png" alt="Kakade" />
<input type="text" name="name" />
```

Output:

```text
Image and text input render normally.
```

Interview trap: self-closing slashes are optional in HTML5 (except for foreign content like SVG). `<img />` is accepted, but the `/` is ignored by the HTML parser.

---

# Real-World Uses

A single profile card uses the whole chapter: doctype, head metadata, semantic body markup, a comment, and correctly closed void elements.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Kakade — Profile</title>
  </head>
  <body>
    <!-- Profile card -->
    <main>
      <h1>Kakade</h1>
      <p>Frontend learner at Kakade.dev</p>
      <img src="/kakade.png" alt="Kakade" width="120" height="120" />
    </main>
  </body>
</html>
```

Output:

```text
Tab: Kakade — Profile
Page: Kakade
      Frontend learner at Kakade.dev
      [profile image]
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **What is HTML** | Markup language describing content structure |
| **HTML5** | Living standard: semantics, media, validation, APIs |
| **Document anatomy** | doctype → html → head + body |
| **Doctype** | `<!DOCTYPE html>` enables standards mode |
| **head vs body** | Metadata vs visible content |
| **Comments** | `<!-- ... -->`, never rendered |
| **Case-insensitivity** | Tags/attributes are case-insensitive; use lowercase |
| **Markup to render** | DOM → CSSOM → render tree → layout → paint |
| **HTML vs XHTML** | Forgiving `text/html` vs strict XML rules |
| **MIME types** | HTML5 is served as `text/html` |

## The Mental Model

```text
HTML file
  │
  ├── doctype → standards mode
  ├── head → metadata the user never sees
  ├── body → content the user sees
  │      │
  │      ├── tags → DOM nodes
  │      └── comments → ignored
  │
  └── parser → DOM + CSSOM → render → paint
```

### The 3 things to remember

**1. Start with the skeleton:**

```html
<!DOCTYPE html>
<html lang="en">
  <head><title>Kakade</title></head>
  <body><h1>Hello Kakade</h1></body>
</html>
```

**2. Meaning first:**

```html
<main>
  <h1>Kakade</h1>
  <p>Frontend learner</p>
</main>
```

**3. Close and quote:**

```html
<img src="/kakade.png" alt="Kakade" width="120" height="120" />
```

> **HTML describes meaning — doctype, head, body, and clean tags are the foundation everything else renders on.**
