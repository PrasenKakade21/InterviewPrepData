# Selectors

Selectors are how CSS finds elements. Everything else — layout, color, animation — is downstream of matching the right nodes. Interviewers test selector specificity, combinators, and the difference between `:nth-child` and `:nth-of-type`.

The core idea:

```text
selector
    ↓
browser matches DOM nodes
    ↓
declarations apply
    ↓
cascade resolves conflicts
```

A selector that matches too much causes style leaks; one that matches too little causes "CSS not working".

---

# 1. Type, Class, ID and Universal Selectors

`type` matches an element name, `.class` matches a class, `#id` matches an id, and `*` matches every element. Class selectors are the workhorse because they are reusable.

```css
p { color: #475569; }
.card { padding: 1rem; }
#hero { min-height: 60vh; }
* { box-sizing: border-box; }
```

```html
<p class="card" id="hero">Kakade</p>
```

Output:

```text
Paragraph: muted text
.card: padding
#hero: min-height
*: border-box on everything
```

Specificity values:

| Selector | Specificity (a,b,c) |
| -------- | ------------------- |
| `*` | 0,0,0 |
| `p` | 0,0,1 |
| `.card` | 0,1,0 |
| `#hero` | 1,0,0 |

Interview trap: ids are hard to override and break reusability; prefer classes except for anchors and truly unique regions.

---

# 2. Attribute Selectors

Attribute selectors match elements by attribute presence or value. They are useful for links, inputs, and `data-*` states.

```css
a[target="_blank"] { color: #b91c1c; }
input[type="email"] { border-color: #0f172a; }
[data-state="open"] { display: block; }
a[href^="https://"] { text-decoration: underline; }
a[href$=".pdf"]::after { content: " (PDF)"; }
[class~="card"] { padding: 1rem; }
```

Output:

```text
External links red, email inputs dark-bordered,
open-state elements visible, PDF links annotated.
```

| Operator | Meaning |
| -------- | ------- |
| `[attr]` | Has attribute |
| `[attr="v"]` | Exact value |
| `[attr~="v"]` | Word in space-separated list |
| `[attr|="v"]` | Value or value-prefix with `-` |
| `[attr^="v"]` | Starts with |
| `[attr$="v"]` | Ends with |
| `[attr*="v"]` | Contains |

Attribute selectors have class-level specificity (0,1,0).

---

# 3. Combinators

Combinators express relationships: descendant (space), child (`>`), adjacent sibling (`+`), general sibling (`~`).

```css
.card p { color: #475569; }        /* any descendant */
.card > p { margin-top: 0; }       /* direct child */
h2 + p { margin-top: 0.25rem; }    /* next sibling */
h2 ~ p { color: #334155; }         /* all later siblings */
```

```html
<div class="card">
  <h2>Kakade</h2>
  <p>Direct child</p>
  <section><p>Nested descendant</p></section>
</div>
```

Output:

```text
Both paragraphs muted; only the direct child loses top margin;
the first paragraph after h2 is tightened.
```

Think:

```text
A B   → B anywhere inside A
A > B → B directly inside A
A + B → B immediately after A
A ~ B → every B after A (same parent)
```

Interview trap: `>` is stricter and faster to reason about; deep descendant selectors create accidental matches.

---

# 4. Selector Grouping and Specificity Intro

Group selectors with commas to share declarations. Each selector in the list is evaluated independently, and the most specific matching selector wins.

```css
h1,
h2,
.card__title {
  font-family: Inter, sans-serif;
  line-height: 1.2;
}

.card .title { color: #0f172a; }  /* 0,2,0 */
.title { color: #475569; }        /* 0,1,0 */
```

Output:

```text
All headings share the font stack.
.card .title wins over .title for matching elements.
```

Specificity is calculated as `(ids, classes/attributes/pseudo-classes, elements/pseudo-elements)`. Inline styles and `!important` sit outside this tuple. Details are in the cascade chapter.

---

# 5. Pseudo-classes and Pseudo-elements

Pseudo-classes select state or position (`:hover`, `:first-child`). Pseudo-elements select a generated sub-part (`::before`, `::selection`) and use double colons.

```css
a:hover { color: #0f172a; }
li:nth-child(2n) { background: #f8fafc; }
button:focus-visible { outline: 2px solid #0f172a; }

.card::before {
  content: "";
  display: block;
  height: 4px;
  background: #0f172a;
}

::selection { background: #fde68a; }
```

Output:

```text
Hover darkens links; even rows shaded; keyboard focus ring;
a 4px bar appears above each card; selected text highlighted.
```

Interview trap: `::before`/`::after` require `content` (even `content: ""`), are not in the DOM, and are ignored by some screen readers — never put essential text there.

---

# 6. :is(), :where() and :not()

`:is()` matches any selector in its list and takes the highest specificity among them. `:where()` does the same but contributes **zero** specificity. `:not()` excludes matches.

```css
:is(h1, h2, h3) { line-height: 1.2; }        /* specificity of element */
:where(.card, .panel) { padding: 1rem; }     /* zero specificity */
.card :not(.card__title) { color: #475569; }
```

Output:

```text
Headings share line-height; card/panel padding is easy to override;
all card text except the title is muted.
```

Think:

```text
:is()    → flexible matching, normal specificity
:where() → flexible matching, zero specificity (reset-friendly)
:not()   → exclusion
```

Interview trap: `:where()` is the tool for low-specificity defaults and resets; `:is()` can accidentally raise specificity.

---

# 7. Selector Performance

Browsers match selectors right-to-left and optimize heavily. The real cost is over-matching and complexity, not micro-optimizing `*`.

```css
/* Hard to reason about */
.page .content .card .body p span.highlight { color: #b91c1c; }

/* Clear and scoped */
.card__highlight { color: #b91c1c; }
```

Output:

```text
Both work; the class version is faster to read, safer to override,
and survives markup changes.
```

Guidelines:

- Prefer classes over deep descendant chains.
- Avoid ids for styling.
- Keep specificity flat.
- Use `:where()` for resets.

---

# Real-World Uses

A card component uses a class, attribute state, pseudo-element accent, and a hover transition.

```css
.card {
  padding: 1.25rem;
  border: 1px solid #e2e8f0;
  border-radius: 0.75rem;
  transition: border-color 150ms ease;
}

.card[data-state="active"] {
  border-color: #0f172a;
}

.card:hover {
  border-color: #94a3b8;
}

.card::before {
  content: "";
  display: block;
  height: 3px;
  background: #0f172a;
  border-radius: 999px;
}

.card :where(h2, p) {
  margin: 0;
}
```

Output:

```text
Card with accent bar, active border state, hover feedback,
and zero-specificity heading/paragraph margins.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Type, class, id** | Element, reusable class, unique id |
| **Universal** | `*` matches everything, zero specificity |
| **Attribute selectors** | Match presence/value/prefix/suffix |
| **Descendant and child** | Space vs `>` |
| **Sibling combinators** | `+` next, `~` all later |
| **Grouping** | Comma shares declarations |
| **Pseudo-classes** | State/position, single colon |
| **Pseudo-elements** | Generated parts, double colon |
| **:is(), :where(), :not()** | Flexible matching, zero specificity, exclusion |
| **Performance** | Flat classes beat deep chains |

## The Mental Model

```text
selector
  │
  ├── simple → type, .class, #id, *, [attr]
  ├── combine → space, >, +, ~
  ├── state → :hover, :focus-visible, :nth-child
  ├── generate → ::before, ::after, ::selection
  └── logic → :is(), :where(), :not()
        │
        └── specificity = (ids, classes, elements)
```

### The 3 things to remember

**1. Classes are the default:**

```css
.card { padding: 1rem; }
```

**2. Know the combinators:**

```css
.card > p { margin-top: 0; }
h2 + p { margin-top: 0.25rem; }
```

**3. Low-specificity defaults:**

```css
:where(h2, p) { margin: 0; }
```

> **Selectors are the matching layer — prefer flat classes, know the combinators, and use :where() when you want easy overrides.**
