# Accessibility in HTML

Accessibility is not a later phase — it is a property of correct HTML. Semantic elements, labels, alt text, and heading order give assistive technology everything it needs. ARIA is a patch for gaps, and a bad one if you use it to replace native semantics.

The core idea:

```text
correct HTML
    ↓
names, roles, states exposed
    ↓
keyboard + screen reader can use it
```

The first rule of ARIA: don't use ARIA if a native element already does the job.

---

# 1. Semantic First and Heading Order

Native elements expose roles automatically: `button` is a button, `nav` is navigation, `h1` is a heading. The heading outline lets screen-reader users jump between sections.

```html
<main>
  <h1>Kakade's HTML Notes</h1>

  <section aria-labelledby="forms-heading">
    <h2 id="forms-heading">Forms</h2>
    <p>Labels, validation, and submission.</p>
  </section>

  <section aria-labelledby="a11y-heading">
    <h2 id="a11y-heading">Accessibility</h2>
    <p>Semantics before ARIA.</p>
  </section>
</main>
```

Output:

```text
Screen reader heading list:
  H1 Kakade's HTML Notes
    H2 Forms
    H2 Accessibility
```

Interview trap: a `div` styled as a button is invisible to assistive tech unless you add `role="button"`, `tabindex="0"`, and keyboard handlers — at which point you should have used `<button>`.

---

# 2. alt Text Rules and Labels

`alt` gives images an accessible name; `label` does the same for form controls. Both must describe purpose, and both are required unless the content is genuinely decorative.

```html
<img src="/img/kakade.png" alt="Kakade presenting HTML notes" width="320" height="200" />
<img src="/img/wave.svg" alt="" width="80" height="24" />

<label for="email">Email address</label>
<input id="email" name="email" type="email" autocomplete="email" />
```

Output:

```text
Image: "Kakade presenting HTML notes, image"
Decorative wave: skipped
Email: "Email address, edit text"
```

Alt decision table:

| Content | Alt |
| ------- | --- |
| Informative | Describe what it conveys |
| Decorative | `alt=""` |
| Functional (link/button) | Describe the action |
| Text in an image | Repeat the text |

---

# 3. Landmarks and Skip Links

Landmarks are the page's map: banner, navigation, main, complementary, contentinfo, search. A skip link lets keyboard users bypass repeated navigation and jump straight to `main`.

```html
<a href="#main" class="skip-link">Skip to content</a>

<header>...</header>
<nav aria-label="Primary">...</nav>
<main id="main">...</main>
<footer>...</footer>
```

Output:

```text
First Tab: "Skip to content" link
Landmark shortcuts: banner, navigation, main, contentinfo
```

CSS keeps the skip link visually hidden until focused:

```css
.skip-link {
  position: absolute;
  left: -9999px;
}
.skip-link:focus {
  left: 1rem;
  top: 1rem;
  background: #fff;
  padding: 0.5rem 1rem;
}
```

---

# 4. Table Scope and caption

Tables need a `caption` and header cells with `scope` so screen readers can announce a cell with its row and column headers.

```html
<table>
  <caption>Kakade's study hours</caption>
  <thead>
    <tr>
      <th scope="col">Month</th>
      <th scope="col">Hours</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">January</th>
      <td>42</td>
    </tr>
  </tbody>
</table>
```

Output:

```text
Screen reader on "42": "January, Hours, 42"
```

If a table is only for layout, it is the wrong element. If it is a real table, never drop the headers to make it look cleaner.

---

# 5. ARIA Roles and aria-label, aria-labelledby, aria-describedby

ARIA adds roles, names, and descriptions when native HTML cannot. `aria-label` supplies a name directly; `aria-labelledby` references visible text; `aria-describedby` adds a description.

```html
<button aria-label="Close settings">
  <svg aria-hidden="true">...</svg>
</button>

<section aria-labelledby="billing-heading">
  <h2 id="billing-heading">Billing</h2>
</section>

<input
  id="password"
  type="password"
  aria-describedby="password-help"
/>
<p id="password-help">Use at least 8 characters.</p>
```

Output:

```text
Button: "Close settings, button" (icon ignored)
Section: named "Billing" by its heading
Password: "Password, edit text, Use at least 8 characters"
```

| Attribute | Purpose | Wins over |
| --------- | ------- | --------- |
| `aria-label` | Direct name | Visible text |
| `aria-labelledby` | Name from element id(s) | `aria-label` |
| `aria-describedby` | Extra description | — |

Never put both `aria-label` and visible text that disagree — the visual label and spoken name must match.

---

# 6. aria-hidden, aria-live, Focus Order and tabindex

`aria-hidden="true"` removes an element from the accessibility tree but **not** from keyboard order. `aria-live` announces dynamic updates. Focus order should follow visual order, which usually means avoiding positive `tabindex`.

```html
<div aria-live="polite" id="status"></div>

<span class="icon" aria-hidden="true">★</span>

<button tabindex="0">Save</button>
```

Output:

```text
Status updates are announced politely when text changes.
Decorative star is skipped by screen readers.
Save is in natural tab order.
```

Interview trap: `aria-hidden="true"` on a focusable element creates a "ghost" control — screen readers skip it but keyboard users can still land on it. Add `tabindex="-1"` or remove it from the DOM.

---

# 7. Contrast and Zoom

Text contrast should meet WCAG AA: 4.5:1 for normal text, 3:1 for large text. Never block zoom, and make sure the layout survives 200% text scaling.

```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

Output:

```text
Users can pinch-zoom and enlarge text.
Layout reflows instead of clipping content.
```

| Requirement | Minimum |
| ----------- | ------- |
| Normal text contrast | 4.5:1 |
| Large text contrast | 3:1 |
| Focus indicator | Visible, 3:1 against adjacent colors |
| Zoom | 200% without loss |

Interview trap: placeholder text often fails contrast and is not a label. Use a real label plus helper text with sufficient contrast.

---

# Real-World Uses

An accessible signup form combines labels, fieldset grouping, described-by errors, a live region, and a skip link.

```html
<a href="#main" class="skip-link">Skip to content</a>

<main id="main">
  <form action="/signup" method="post">
    <fieldset>
      <legend>Create Kakade's account</legend>

      <label for="email">Email</label>
      <input
        id="email"
        name="email"
        type="email"
        autocomplete="email"
        aria-describedby="email-error"
        required
      />
      <p id="email-error" role="alert">Enter a valid email address.</p>

      <button type="submit">Sign up</button>
    </fieldset>
  </form>

  <div aria-live="polite" id="status"></div>
</main>
```

Output:

```text
Screen reader: fieldset "Create Kakade's account",
  "Email, edit text, Enter a valid email address"
Status updates announced without moving focus.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Semantic first** | Native elements expose roles for free |
| **alt text** | Purpose; `alt=""` when decorative |
| **Labels and inputs** | `for` matches `id`; placeholder is not a label |
| **Heading order** | Logical outline, no skipped levels |
| **Landmarks and skip links** | Page map + bypass repeated nav |
| **Table scope and caption** | Headers announced with each cell |
| **ARIA roles** | Patch gaps, never replace native semantics |
| **aria-label/labelledby/describedby** | Direct name, referenced name, description |
| **aria-hidden and aria-live** | Hide from AT; announce updates |
| **Focus order and tabindex** | Visual order; avoid positive values |
| **Contrast and zoom** | 4.5:1 text, 200% zoom, no blocked scaling |

## The Mental Model

```text
accessible HTML
  │
  ├── structure → semantic elements + heading order
  ├── names → alt, label, aria-label/labelledby
  ├── context → landmarks, captions, descriptions
  ├── state → aria-live, aria-expanded, role
  └── interaction → focus order, keyboard, visible focus
        │
        └── native first, ARIA only for gaps
```

### The 3 things to remember

**1. Native beats ARIA:**

```html
<button>Save</button>
<!-- not <div role="button"> -->
```

**2. Every control needs a name:**

```html
<label for="email">Email</label>
<input id="email" type="email" />
```

**3. Never trap or hide focus:**

```html
<span aria-hidden="true">★</span>
```

> **Accessibility is correct HTML first — names, roles, and focus come free from native elements, and ARIA is only a patch.**
