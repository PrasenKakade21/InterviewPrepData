# Lists

Lists are how HTML expresses a group of related items: unordered, ordered, or described. Interviewers look for correct nesting (`li` must be a direct child of `ul`/`ol`) and for `dl` used for name–value pairs rather than fake tables.

The core idea:

```text
list type
    ├── ul → bullets, order does not matter
    ├── ol → numbers, order matters
    └── dl → term/description pairs
```

Use a list whenever content is a set — navigation menus, steps, features, FAQs, metadata.

---

# 1. The ul Element and li

`ul` is an unordered list: the order carries no meaning, so bullets are appropriate. Every direct child should be an `li`; other elements belong *inside* the `li`.

```html
<ul>
  <li>HTML</li>
  <li>CSS</li>
  <li>JavaScript</li>
</ul>
```

Output:

```text
• HTML
• CSS
• JavaScript
```

Bad:

```html
<!-- Invalid: divs are not list items -->
<ul>
  <div>HTML</div>
  <div>CSS</div>
</ul>
```

Good:

```html
<ul>
  <li>HTML</li>
  <li>CSS</li>
</ul>
```

Screen readers announce "list, 3 items", which is why real list markup matters for navigation.

---

# 2. The ol Element and Attributes

`ol` is ordered: sequence matters. `start` changes the first number, `reversed` counts down, and `type` changes the marker (`1`, `A`, `a`, `I`, `i`).

```html
<ol start="4">
  <li>Build the layout</li>
  <li>Add styles</li>
  <li>Ship it</li>
</ol>

<ol reversed>
  <li>Ship it</li>
  <li>Test it</li>
  <li>Write it</li>
</ol>
```

Output:

```text
4. Build the layout
5. Add styles
6. Ship it

3. Ship it
2. Test it
1. Write it
```

Notice: `type` and `start` are presentational-ish but still semantic — they change the *numbering*, not just the look. For anything else, style with CSS counters.

---

# 3. Nested Lists

A nested list goes inside an `li`, never between `li` elements. This is how menus, outlines, and file trees are built.

```html
<ul>
  <li>
    Frontend
    <ul>
      <li>HTML</li>
      <li>CSS</li>
      <li>JavaScript</li>
    </ul>
  </li>
  <li>Backend</li>
</ul>
```

Output:

```text
• Frontend
    ◦ HTML
    ◦ CSS
    ◦ JavaScript
• Backend
```

Think:

```text
ul
 ├── li Frontend
 │     └── ul
 │          ├── li HTML
 │          └── li CSS
 └── li Backend
```

Interview trap: putting a `<ul>` directly inside `<ul>` (outside an `li`) is invalid and produces unpredictable nesting in the DOM.

---

# 4. dl, dt and dd

`dl` is a description list: `dt` is the term, `dd` is the description. It is the correct markup for glossaries, metadata, and FAQ-style pairs.

```html
<dl>
  <dt>HTML</dt>
  <dd>Markup language for document structure.</dd>

  <dt>CSS</dt>
  <dd>Language for presentation and layout.</dd>

  <dt>JavaScript</dt>
  <dd>Language for behavior and interactivity.</dd>
</dl>
```

Output:

```text
HTML
    Markup language for document structure.

CSS
    Language for presentation and layout.

JavaScript
    Language for behavior and interactivity.
```

A `dt` can have multiple `dd` values (synonyms, multiple definitions), and a `dd` can contain paragraphs, lists, and even images.

---

# 5. The menu Element, List Accessibility and Styling

`menu` is a semantic list of commands; in practice it behaves like `ul` and is rarely needed. For accessibility, keep real list structure and style the markers with CSS rather than stripping semantics.

```html
<nav aria-label="Primary">
  <ul class="nav-list">
    <li><a href="/">Home</a></li>
    <li><a href="/notes">Notes</a></li>
    <li><a href="/projects">Projects</a></li>
  </ul>
</nav>
```

Output:

```text
Home   Notes   Projects       ← styled horizontally with CSS

Screen reader: "Primary navigation, list, 3 items"
```

Bad:

```html
<!-- Removing list semantics to "fix" bullets -->
<ul style="list-style: none; padding: 0">
```

Good:

```css
.nav-list {
  list-style: none;
  padding: 0;
  display: flex;
  gap: 1rem;
}
```

`list-style: none` removes the visual bullet in some browsers' accessibility tree; adding `role="list"` preserves list semantics for VoiceOver users. The safest approach: keep the `ul` and style it.

---

# Real-World Uses

A documentation sidebar uses nested unordered lists for sections, an ordered list for steps, and a description list for API metadata.

```html
<nav aria-label="Docs">
  <ul>
    <li>
      HTML
      <ul>
        <li><a href="/html/text">Text</a></li>
        <li><a href="/html/forms">Forms</a></li>
      </ul>
    </li>
    <li><a href="/css">CSS</a></li>
  </ul>
</nav>

<ol>
  <li>Install dependencies</li>
  <li>Run the dev server</li>
</ol>

<dl>
  <dt>Status</dt>
  <dd>Stable</dd>
  <dt>Version</dt>
  <dd>1.4.0</dd>
</dl>
```

Output:

```text
• HTML
    ◦ Text
    ◦ Forms
• CSS

1. Install dependencies
2. Run the dev server

Status
    Stable
Version
    1.4.0
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **ul** | Unordered list of `li` children |
| **ol and attributes** | Ordered; `start`, `reversed`, `type` |
| **li** | Direct child of `ul`/`ol` only |
| **Nested lists** | Nest inside an `li`, never directly in the list |
| **dl, dt and dd** | Term/description pairs |
| **menu** | List of commands; behaves like `ul` |
| **List accessibility** | Real list markup announces item counts |
| **List styling** | CSS for layout; keep semantics |

## The Mental Model

```text
list choice
  │
  ├── order irrelevant → ul → li
  ├── order matters    → ol → li (start/reversed/type)
  └── term + meaning   → dl → dt + dd
        │
        └── nesting goes inside an li
```

### The 3 things to remember

**1. List items only:**

```html
<ul>
  <li>HTML</li>
  <li>CSS</li>
</ul>
```

**2. Nest inside the item:**

```html
<li>Frontend
  <ul><li>HTML</li></ul>
</li>
```

**3. Pairs use dl:**

```html
<dl>
  <dt>Status</dt>
  <dd>Stable</dd>
</dl>
```

> **Lists are structure, not bullets — `ul` for sets, `ol` for sequences, `dl` for term/description pairs, and nesting always inside an `li`.**
