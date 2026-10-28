# HTML Interview Traps

Most HTML interview questions are not about obscure tags — they are about the distinctions beginners blur: `div` vs `section`, `strong` vs `b`, `id` vs `class`, `alt` vs `title`, GET vs POST. This chapter collects the traps and the one-line answers.

The core idea:

```text
interview question
    ↓
two similar things
    ↓
difference = meaning + behavior, not looks
```

If you can explain *why* each pair exists, you can answer any variant.

---

# 1. div vs section vs article

All three are block containers, but only two carry meaning. `div` is generic, `section` is a thematic group (usually with a heading), and `article` is self-contained and syndicatable.

```html
<div class="layout">
  <section aria-labelledby="posts">
    <h2 id="posts">Posts</h2>
    <article>
      <h3>Why HTML Matters</h3>
      <p>By Kakade</p>
    </article>
  </section>
</div>
```

Output:

```text
div → layout wrapper, no semantics
section → thematic region "Posts"
article → portable post
```

Interview answer: "`div` when there is no meaning, `section` for a named thematic part of the page, `article` when the content stands alone."

---

# 2. b vs strong and i vs em

`strong` and `em` carry meaning (importance, stress); `b` and `i` are visual offsets with no extra semantics. Screen readers may stress `strong`/`em` but not `b`/`i`.

```html
<p><strong>Warning:</strong> deploy is <em>today</em>.</p>
<p>The <b>Kakade</b> brand and the <i>Kakade.dev</i> title.</p>
```

Output:

```text
Warning: deploy is today.   ← meaning conveyed
The Kakade brand and the Kakade.dev title.   ← style only
```

Interview answer: "Use `strong`/`em` when the emphasis is part of the meaning; use `b`/`i` for styling-only offsets."

---

# 3. id vs class

`id` is unique, used for anchors and programmatic references. `class` is reusable, used for styling and grouping. Duplicate ids break `label for`, fragment links, and ARIA references.

```html
<h1 id="page-title" class="title title--xl">Kakade</h1>
<p class="title">Notes</p>
```

Output:

```text
#page-title → exactly one element
.title → both elements
```

Interview answer: "`id` identifies one element; `class` groups many."

---

# 4. alt vs title

`alt` is the accessible name for an image and is required. `title` is advisory tooltip text and is unreliable on touch, keyboard, and screen readers. `title` never replaces `alt` or a label.

```html
<img src="/kakade.png" alt="Kakade presenting HTML notes" title="Kakade at DevConf 2026" />
```

Output:

```text
Screen reader: "Kakade presenting HTML notes, image"
Hover: tooltip "Kakade at DevConf 2026"
```

Interview answer: "`alt` describes the image for accessibility; `title` is optional advisory text."

---

# 5. GET vs POST

GET puts data in the URL and is for retrieval; POST puts data in the body and is for changes. POST is not encryption — HTTPS is.

```html
<form action="/search" method="get">
  <input name="q" />
</form>

<form action="/signup" method="post">
  <input name="email" type="email" />
</form>
```

Output:

```text
GET  → /search?q=html
POST → /signup  body: email=...
```

Interview answer: "GET is safe, bookmarkable retrieval; POST is a state-changing body request. Neither replaces HTTPS."

---

# 6. label for and Void Elements

`label for` must match an input's `id`. Void elements (`img`, `br`, `hr`, `input`, `meta`, `link`) have no children and no closing tag.

```html
<label for="email">Email</label>
<input id="email" name="email" type="email" />

<img src="/kakade.png" alt="Kakade" width="80" height="80" />
<br />
<hr />
```

Output:

```text
Clicking "Email" focuses the input.
Void elements render without closing tags.
```

Interview answer: "`for` points to `id`, not `name`; void elements cannot contain content."

---

# 7. Block vs Inline and Boolean Attributes

Block elements start on a new line and take full width; inline elements flow with text. Boolean attributes are true when present, regardless of value.

```html
<p>Kakade <span>writes</span> HTML.</p>

<input required />
<input required="false" />
```

Output:

```text
p starts a new block; span stays inline.
Both inputs are required — presence wins.
```

Interview answer: "Block vs inline is default display, changeable with CSS; boolean attributes are true by presence."

---

# 8. Quirks Mode and Table vs Div Layout

A missing doctype triggers quirks mode with legacy sizing. Tables are for data; page layout belongs to CSS grid/flexbox.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <table>
      <caption>Kakade's scores</caption>
      <tr><th scope="col">Topic</th><th scope="col">Score</th></tr>
      <tr><td>HTML</td><td>92</td></tr>
    </table>
  </body>
</html>
```

Output:

```text
Standards mode with a data table.
Layout still handled by CSS, not the table.
```

Interview answer: "Doctype controls the rendering mode; tables encode data relationships, not page structure."

---

# Real-World Uses

A code review checklist catches every trap in one pass.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Kakade's Review Checklist</title>
  </head>
  <body>
    <main>
      <h1>Review Checklist</h1>

      <section aria-labelledby="structure">
        <h2 id="structure">Structure</h2>
        <article>
          <h3>Semantics</h3>
          <p><strong>Use</strong> section for named regions, <em>not</em> div soup.</p>
        </article>
      </section>

      <section aria-labelledby="forms">
        <h2 id="forms">Forms</h2>
        <label for="email">Email</label>
        <input id="email" name="email" type="email" required />
        <img src="/kakade.png" alt="Kakade" width="80" height="80" />
      </section>
    </main>
  </body>
</html>
```

Output:

```text
Valid standards-mode page: semantic sections, labeled input,
alt text, and boolean required.
```

---

# Quick Revision

| Trap | Correct answer |
| ---- | -------------- |
| **div vs section vs article** | Generic vs named region vs standalone content |
| **b vs strong** | Style vs importance |
| **i vs em** | Style vs stress emphasis |
| **id vs class** | Unique vs reusable |
| **alt vs title** | Accessible name vs advisory tooltip |
| **GET vs POST** | Retrieval in URL vs change in body |
| **label for** | Must match the input `id` |
| **Void elements** | No children, no closing tag |
| **Block vs inline** | Default display, changeable with CSS |
| **Quirks mode** | Missing doctype → legacy rendering |
| **Table vs div layout** | Data vs CSS layout |
| **Boolean attributes** | True by presence |

## The Mental Model

```text
trap pair
  │
  ├── same look? → ask about meaning
  ├── same element? → ask about uniqueness/behavior
  ├── same attribute? → ask about accessible name vs hint
  └── same request? → ask about URL vs body
        │
        └── answer with "why it exists", not "what it looks like"
```

### The 3 things to remember

**1. Meaning over looks:**

```html
<article><h2>Kakade's Post</h2></article>
```

**2. Unique id, reusable class:**

```html
<h1 id="title" class="heading">Kakade</h1>
```

**3. Boolean is presence:**

```html
<input required />
```

> **Interview traps are pairs — answer with the meaning or behavior that separates them, never with appearance.**
