# List Tags

Lists group related items. `ul` is unordered, `ol` is ordered, `dl` is a description list. Only `li` may be a direct child of `ul`/`ol`; nested lists live inside an `li`.

The core idea:

```text
list choice
    ├── order irrelevant → ul → li
    ├── order matters    → ol → li (start/reversed/type)
    └── term + meaning   → dl → dt + dd
```

---

# `<ul>`

An unordered list. Use it when sequence carries no meaning: features, navigation, tags.

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

Gotchas: only `li` children (script/template allowed); never put `div` directly inside.

---

# `<ol>`

An ordered list. Use it when position matters: steps, rankings, recipes.

| Attribute | Purpose | Example |
| --------- | ------- | ------- |
| `start` | First number | `start="4"` |
| `reversed` | Count down | `reversed` |
| `type` | Marker style | `type="A"` (`1`, `A`, `a`, `I`, `i`) |

```html
<ol start="4">
  <li>Build the layout</li>
  <li>Add styles</li>
</ol>

<ol reversed>
  <li>Ship it</li>
  <li>Test it</li>
</ol>
```

Output:

```text
4. Build the layout
5. Add styles

2. Ship it
1. Test it
```

Gotchas: `type` changes the numbering, not the visual style; for custom counters use CSS.

---

# `<li>`

A list item. Direct child of `ul`/`ol` (and `menu`). Its `value` attribute sets the number in an `ol`.

```html
<ol>
  <li value="10">Ten</li>
  <li>Eleven</li>
</ol>
```

Output:

```text
10. Ten
11. Eleven
```

Gotchas: block content goes inside the `li`, not beside it; nested lists also go inside the `li`.

---

# `<dl>`, `<dt>`, `<dd>`

Description list: `dl` wraps term/description pairs. `dt` is the term, `dd` the description. One term can have multiple descriptions.

```html
<dl>
  <dt>HTML</dt>
  <dd>Markup language for structure.</dd>

  <dt>CSS</dt>
  <dd>Language for presentation.</dd>
</dl>
```

Output:

```text
HTML
    Markup language for structure.
CSS
    Language for presentation.
```

Gotchas: `dl` is not a layout table; order is dt → dd (or dt → multiple dd). Modern HTML also allows `div` wrappers inside `dl` for grouping.

---

# `<menu>`

A semantic list of commands. Behaves like `ul` in browsers.

```html
<menu>
  <li><button>Save</button></li>
  <li><button>Cancel</button></li>
</menu>
```

Output:

```text
• Save
• Cancel
```

Gotchas: rarely needed; use `ul` for navigation and `button` groups for commands unless the semantics truly fit.

---

# Nesting Lists

Nested lists go inside an `li`. This is the structure behind menus and outlines.

```html
<ul>
  <li>
    Frontend
    <ul>
      <li>HTML</li>
      <li>CSS</li>
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
• Backend
```

Gotchas: nesting directly under the outer `ul` is invalid and reparsed unpredictably.

---

# List Accessibility

Real list markup is announced as "list, N items", which helps screen-reader navigation.

```html
<nav aria-label="Primary">
  <ul class="nav-list">
    <li><a href="/">Home</a></li>
    <li><a href="/notes">Notes</a></li>
  </ul>
</nav>
```

Output:

```text
Screen reader: "Primary navigation, list, 2 items"
```

Gotchas: `list-style: none` can drop list semantics in Safari/VoiceOver — add `role="list"` to restore them.

---

# Quick Reference

| Tag | Purpose |
| --- | ------- |
| `ul` | Unordered list |
| `ol` | Ordered list (`start`, `reversed`, `type`) |
| `li` | List item (only child of `ul`/`ol`) |
| `dl` | Description list |
| `dt` | Term |
| `dd` | Description |
| `menu` | List of commands |

> **Lists are structure: `ul` for sets, `ol` for sequences, `dl` for pairs, and nesting always inside an `li`.**
