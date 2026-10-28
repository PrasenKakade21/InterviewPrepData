# Entities, Encoding and Validation

Characters are bytes until an encoding says otherwise. Entities let you write reserved or invisible characters safely, and validation catches the markup mistakes browsers silently repair. This chapter is small but shows up in debugging questions constantly.

The core idea:

```text
text → bytes (UTF-8) → browser decodes → characters
special character → entity → guaranteed safe markup
```

If the encoding is wrong, every symbol after it is suspect.

---

# 1. Character Encoding and UTF-8

An encoding maps bytes to characters. `UTF-8` covers every Unicode character, is backward-compatible with ASCII, and is the required choice for modern HTML. Declare it in `head` and save the file as UTF-8.

```html
<head>
  <meta charset="UTF-8" />
</head>
<body>
  <p>Hello Kakade — ₹499 — café — 日本語</p>
</body>
```

Output:

```text
Hello Kakade — ₹499 — café — 日本語
```

Bad:

```text
Hello Kakade â€” â‚¹499 â€” cafÃ©
```

Mojibake means bytes were decoded with the wrong charset — usually UTF-8 bytes read as Windows-1252. Fix the `meta charset` and the file encoding; do not patch it with entities.

---

# 2. Named Entities

Named entities start with `&` and end with `;`. They represent characters that are hard to type or that collide with HTML syntax.

```html
<p>5 &lt; 10 &amp;&amp; 10 &gt; 5</p>
<p>&copy; 2026 Kakade &mdash; All rights reserved</p>
<p>&ldquo;Ship small&rdquo; &ndash; Kakade</p>
```

Output:

```text
5 < 10 && 10 > 5
© 2026 Kakade — All rights reserved
“Ship small” – Kakade
```

| Entity | Character | Use |
| ------ | --------- | --- |
| `&lt;` | `<` | Less-than in text |
| `&gt;` | `>` | Greater-than in text |
| `&amp;` | `&` | Ampersand |
| `&quot;` | `"` | Double quote in attributes |
| `&copy;` | © | Copyright |
| `&mdash;` | — | Em dash |
| `&nbsp;` | non-breaking space | Prevent a line break |

Modern HTML5 also allows many raw characters directly — `©`, `—`, and `₹` can be typed if the file is UTF-8. Entities remain necessary for `&`, `<`, and `>` in text.

---

# 3. Numeric Entities and Reserved Characters

Numeric entities use a code point: `&#169;` (decimal) or `&#xA9;` (hex). They are useful for characters your keyboard or editor cannot produce.

```html
<p>&#169; 2026 Kakade</p>
<p>&#x20B9; 499</p>
<p>&#128512;</p>
```

Output:

```text
© 2026 Kakade
₹ 499
😀
```

Reserved characters must always be escaped when they are content, not markup:

```html
<!-- Bad: parser thinks a tag starts -->
<p>Use <div> for layout</p>

<!-- Good -->
<p>Use &lt;div&gt; for layout</p>
```

Interview trap: an unescaped `&` usually still renders, but it can start an entity and produce odd output. Escape it as `&amp;`.

---

# 4. nbsp and Whitespace

HTML collapses runs of whitespace into one space. `&nbsp;` inserts a non-breaking space, and `<pre>` preserves whitespace entirely. Use CSS for spacing, not `&nbsp;` chains.

```html
<p>Kakade&nbsp;dev</p>
<p>Kakade&nbsp;&nbsp;&nbsp;dev</p>
<pre>Kakade    dev
  line two</pre>
```

Output:

```text
Kakade dev          ← stays together, no line break
Kakade   dev        ← multiple nbsp, still no wrapping
Kakade    dev
  line two
```

Interview trap: `&nbsp;` chains for indentation are a layout hack. They break responsive text, copy-paste, and screen readers (some announce them oddly). Use margins/padding.

---

# 5. Symbols and Emoji

Emoji and symbols are Unicode characters. With UTF-8 you can paste them directly; numeric entities are an alternative. Add `aria-label` or surrounding text when the symbol carries meaning.

```html
<p>Status: ✅ Done</p>
<p>Price: &#8377;499</p>
<p><span role="img" aria-label="Warning">⚠️</span> Check the console</p>
```

Output:

```text
Status: ✅ Done
Price: ₹499
⚠️ Check the console
```

Interview trap: emoji in `alt`, labels, or button text can be read aloud as their Unicode name ("white heavy check mark"). Use text or `aria-label` for anything functional.

---

# 6. W3C Validation and Common Errors

The W3C validator checks your HTML against the spec. Browsers recover from most errors, which is exactly why validation is useful — it exposes mistakes the browser hides.

```html
<!-- Invalid: li outside a list -->
<li>Kakade</li>

<!-- Invalid: duplicate id -->
<p id="note">One</p>
<p id="note">Two</p>
```

Output:

```text
Validator: "Element li not allowed as child of body"
Validator: "Duplicate ID note"
```

Common errors:

- Unclosed or misnested tags.
- Block elements inside inline elements (`div` inside `span`).
- Duplicate `id`s.
- `img` without `alt`.
- `label for` with no matching `id`.
- Invalid attribute values (`tabindex="abc"`).

---

# 7. Quirks Mode vs Standards Mode

The doctype selects the rendering mode. Standards mode follows modern CSS/HTML rules; quirks mode emulates legacy browser behavior, changing box sizing, line heights, and table layout. No doctype means quirks mode.

```html
<!-- Standards mode -->
<!DOCTYPE html>
<html>
  <body><p>Kakade</p></body>
</html>
```

Output:

```text
Standards mode: modern box model and layout rules
Quirks mode: legacy box model, different sizing
```

You can check the mode in DevTools with `document.compatMode`:

```js
console.log(document.compatMode); // "CSS1Compat" = standards
```

Interview trap: quirks mode is not just "old CSS" — it can change the width calculation of elements, which makes a pixel-perfect layout look broken for no obvious reason.

---

# Real-World Uses

A documentation page declares UTF-8, escapes code examples, uses entities for symbols, and validates cleanly.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Kakade — HTML Entities</title>
  </head>
  <body>
    <h1>Entities &amp; Encoding</h1>

    <p>Write <code>&lt;div&gt;</code> to create a container.</p>
    <p>Price: &#8377;499 &mdash; &copy; 2026 Kakade</p>

    <pre><code>&lt;p&gt;Hello Kakade&lt;/p&gt;</code></pre>
  </body>
</html>
```

Output:

```text
Entities & Encoding
Write <div> to create a container.
Price: ₹499 — © 2026 Kakade
<p>Hello Kakade</p>   (shown as code, not rendered)
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Character encoding** | Maps bytes to characters |
| **UTF-8** | Universal, ASCII-compatible, required default |
| **Named entities** | `&lt;`, `&amp;`, `&copy;`, `&mdash;` |
| **Numeric entities** | `&#169;` decimal, `&#xA9;` hex |
| **Reserved characters** | Escape `<`, `>`, `&` in text |
| **nbsp and whitespace** | HTML collapses spaces; `&nbsp;` prevents breaks |
| **Symbols and emoji** | Unicode; label meaning, not decoration |
| **W3C validator** | Catches errors browsers silently fix |
| **Common errors** | Misnesting, duplicate ids, missing alt |
| **Quirks vs standards** | Doctype controls rendering mode |

## The Mental Model

```text
bytes
  ↓ charset=UTF-8
characters
  ↓ markup parsing
  ├── &lt; &gt; &amp; → literal characters
  ├── &#8377; → ₹
  ├── &nbsp; → no line break
  └── raw emoji → Unicode symbol
        │
        └── validator → spec compliance → standards mode
```

### The 3 things to remember

**1. UTF-8 first:**

```html
<meta charset="UTF-8" />
```

**2. Escape reserved characters:**

```html
<p>Use &lt;div&gt; for layout &amp; structure.</p>
```

**3. Doctype keeps standards mode:**

```html
<!DOCTYPE html>
```

> **Encoding decides whether characters exist, entities keep markup safe, and the doctype decides which rules the browser follows.**
