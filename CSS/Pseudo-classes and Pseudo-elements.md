# Pseudo-classes and Pseudo-elements

Pseudo-classes select elements by state or position (`:hover`, `:nth-child`). Pseudo-elements select a generated part (`::before`, `::selection`). They let CSS react to interaction and structure without extra classes or markup.

The core idea:

```text
selector:pseudo-class  → element in a state
selector::pseudo-element → generated sub-part
```

Single colon = state; double colon = generated part (legacy exceptions aside).

---

# 1. :hover, :focus, :active and :focus-visible

Interactive states: `:hover` (pointer over), `:focus` (focused), `:active` (being pressed), `:focus-visible` (focus should be visible, usually keyboard).

```css
.button:hover {
  background: #334155;
}

.button:active {
  transform: translateY(1px);
}

.button:focus-visible {
  outline: 2px solid #0f172a;
  outline-offset: 2px;
}
```

Output:

```text
Hover darkens; press nudges down; keyboard focus shows a ring.
```

Interview trap: never remove focus outlines without a visible replacement. `:focus-visible` avoids showing rings on mouse clicks while keeping keyboard access.

---

# 2. :nth-child, :nth-of-type, :first-child, :last-child

Structural pseudo-classes match position among siblings. `nth-child` counts all siblings; `nth-of-type` counts same-type siblings.

```css
li:nth-child(2n) { background: #f8fafc; }
li:nth-of-type(3) { font-weight: 600; }
p:first-child { margin-top: 0; }
p:last-child { margin-bottom: 0; }
```

Output:

```text
Even list items shaded; third li bold;
first/last paragraphs lose outer margins.
```

Formulas: `2n` even, `2n+1` odd, `3n` every third, `-n+3` first three, `n+4` from the fourth onward.

---

# 3. :checked, :disabled, :required and :invalid

Form-state pseudo-classes style controls based on their state or validity.

```css
input:checked + label {
  font-weight: 600;
}

input:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

input:required {
  border-left: 3px solid #0f172a;
}

input:invalid:not(:placeholder-shown) {
  border-color: #b91c1c;
}
```

Output:

```text
Checked option label bold; disabled input muted;
required field marked; invalid filled field red.
```

Interview trap: `:invalid` matches empty required fields on load. Combine with `:not(:placeholder-shown)` or `:user-invalid` to avoid errors before interaction.

---

# 4. ::before and ::after with content

Pseudo-elements create generated boxes before/after an element's content. They require `content` and are not in the DOM.

```css
.card::before {
  content: "";
  display: block;
  height: 4px;
  background: #0f172a;
  border-radius: 999px;
}

.external::after {
  content: " ↗";
  font-size: 0.8em;
}
```

Output:

```text
Card gets a 4px accent bar; external links get an arrow.
```

Interview trap: generated content is decorative only. Screen readers may ignore it, and it cannot be selected or copied reliably — never put essential information there.

---

# 5. ::placeholder, ::selection and ::marker

These style specific generated parts: placeholder text, selected text, and list markers.

```css
::placeholder {
  color: #64748b;
  opacity: 1;
}

::selection {
  background: #fde68a;
  color: #0f172a;
}

li::marker {
  color: #0f172a;
  font-weight: 600;
}
```

Output:

```text
Placeholder gray; selected text yellow; list bullets dark and bold.
```

Gotchas: placeholder contrast must meet 4.5:1 if it conveys information; `::marker` supports a limited set of properties.

---

# 6. CSS Counters

Counters generate automatic numbering without extra markup. `counter-reset` starts a counter, `counter-increment` advances it, and `content: counter()` displays it.

```css
.steps {
  counter-reset: step;
}

.steps li {
  counter-increment: step;
}

.steps li::before {
  content: counter(step) ". ";
  font-weight: 700;
  color: #0f172a;
}
```

Output:

```text
1. Install dependencies
2. Run the dev server
3. Open localhost
```

Counters can be nested and use styles like `decimal`, `lower-alpha`, `upper-roman`. They are presentation only — do not rely on them for content meaning.

---

# Real-World Uses

A form and card use state pseudo-classes, generated content, counters, and selection styling.

```css
.field:focus-within {
  outline: 2px solid #0f172a;
  outline-offset: 2px;
}

.field input:user-invalid {
  border-color: #b91c1c;
}

.card::before {
  content: "";
  display: block;
  height: 3px;
  background: #0f172a;
}

.faq {
  counter-reset: q;
}

.faq details {
  counter-increment: q;
}

.faq summary::before {
  content: counter(q) ". ";
  font-weight: 700;
}
```

Output:

```text
Focused field group outlined; invalid fields red after interaction;
card accent; numbered FAQ questions.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **:hover, :focus, :active** | Pointer, focus, pressed |
| **:focus-visible** | Keyboard-visible focus ring |
| **:nth-child/of-type** | Position among all/same-type siblings |
| **:first/last-child** | First/last sibling |
| **:checked, :disabled, :required** | Form states |
| **::before/::after** | Generated boxes, need `content` |
| **::placeholder/::selection** | Placeholder and selection styling |
| **::marker** | List markers |
| **Counters** | Automatic numbering |

## The Mental Model

```text
pseudo
  ├── state → :hover, :focus-visible, :checked, :disabled
  ├── structure → :nth-child, :first-child, :last-child
  ├── generated → ::before, ::after (+ content)
  ├── special parts → ::placeholder, ::selection, ::marker
  └── counters → reset + increment + counter()
```

### The 3 things to remember

**1. Visible focus:**

```css
.button:focus-visible {
  outline: 2px solid #0f172a;
  outline-offset: 2px;
}
```

**2. Generated content is decoration:**

```css
.external::after { content: " ↗"; }
```

**3. Counters need reset + increment:**

```css
.steps { counter-reset: step; }
.steps li { counter-increment: step; }
.steps li::before { content: counter(step) ". "; }
```

> **Pseudo-classes react to state and position; pseudo-elements generate decoration — keep real meaning in the DOM.**
