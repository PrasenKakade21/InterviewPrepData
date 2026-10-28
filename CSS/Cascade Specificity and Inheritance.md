# Cascade, Specificity and Inheritance

Three rules decide which CSS wins: where the style came from (origin/importance), how specific the selector is, and which rule comes last. Inheritance then passes some properties from parent to child. This chapter is the source of most "why isn't my CSS applying?" bugs.

The core idea:

```text
conflicting declarations
    ↓
1. origin + importance
    ↓
2. specificity
    ↓
3. source order
    ↓
one computed value
```

---

# 1. Cascade Origin and Importance

Styles can come from the browser, the user, or the author. Important declarations flip the normal order. Author styles normally beat user-agent styles, but user `!important` styles beat author `!important` ones for accessibility.

```css
/* Browser default */
h1 { font-size: 2em; }

/* Author */
h1 { font-size: 2.5rem; }

/* Author important */
h1 { font-size: 3rem !important; }
```

Output:

```text
2.5rem normally; 3rem because of !important.
```

Cascade order (simplified, low → high):

```text
user-agent normal
author normal
author !important
user !important
```

Interview trap: transitions and animations can outrank normal declarations during their run, which explains styles that "snap back" after an animation.

---

# 2. Specificity Calculation

Specificity is a three-part score: ids, classes/attributes/pseudo-classes, and elements/pseudo-elements. It is compared left to right, not added as a single number.

```css
/* 0,1,0 */
.card { color: #475569; }

/* 0,2,0 */
.card .title { color: #0f172a; }

/* 1,1,0 */
#hero .title { color: #b91c1c; }
```

Output:

```text
.title inside .card inside #hero → red (1,1,0 wins)
```

Examples:

| Selector | Score |
| -------- | ----- |
| `p` | 0,0,1 |
| `.card` | 0,1,0 |
| `.card .title` | 0,2,0 |
| `#hero .title` | 1,1,0 |
| `a:hover` | 0,1,1 |
| `:where(.card)` | 0,0,0 |

`!important` and inline styles are tracked outside the tuple — avoid mixing them into specificity math.

---

# 3. !important

`!important` overrides normal declarations regardless of specificity. It is a last resort for third-party overrides, not a styling strategy.

```css
.btn {
  background: #0f172a !important;
}

.btn--ghost {
  background: transparent;
}
```

```html
<button class="btn btn--ghost">Kakade</button>
```

Output:

```text
Button stays dark: .btn !important beats .btn--ghost.
```

Better:

```css
.btn--ghost {
  background: transparent;
}
```

Use `!important` only when you cannot control the competing rule (vendor CSS, email clients, utility escape hatches). If you need it to beat your own code, fix the specificity instead.

---

# 4. Inheritance and Global Keywords

Some properties inherit by default (color, font, line-height, visibility); others do not (margin, padding, border, width). Global keywords let you control inheritance explicitly.

```css
.card {
  color: #0f172a;
}

.card p {
  color: inherit;          /* take parent color */
  margin: initial;         /* property's initial value */
  border: unset;           /* inherited if inheritable, else initial */
}

.reset-all {
  all: revert;             /* roll back to previous origin */
}
```

Output:

```text
Paragraph uses the card color; margin returns to 0;
border cleared; .reset-all restores earlier cascade values.
```

| Keyword | Meaning |
| ------- | ------- |
| `inherit` | Take parent's computed value |
| `initial` | Property's spec initial value |
| `unset` | Inherit if inheritable, else initial |
| `revert` | Roll back to previous origin/layer |
| `revert-layer` | Roll back within the layer stack |

Interview trap: `all: unset` is a blunt reset — it also wipes inherited typography you probably wanted.

---

# 5. Cascade Layers

`@layer` groups styles into named layers; later layers win over earlier ones regardless of specificity inside them. This lets you keep resets and third-party CSS weak without `!important`.

```css
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; }
}

@layer base {
  .card { padding: 1rem; }
}

@layer utilities {
  .p-0 { padding: 0; }
}
```

Output:

```text
utilities beats components beats base beats reset,
even if a lower layer has a more specific selector.
```

Order matters: declare the layer order once, then fill layers. Unlayered styles normally beat layered normal styles, which surprises people.

```text
unlayered normal > utilities > components > base > reset
```

---

# 6. Shorthand vs Longhand and Debugging the Cascade

Shorthand properties set several longhands at once and can reset values you did not intend. DevTools shows which rule won and what was overridden.

```css
/* Sets all four sides + resets other background longhands */
background: #0f172a;

/* Explicit */
background-color: #0f172a;
background-image: none;
```

```css
/* margin shorthand resets all sides */
.card {
  margin: 0 auto;
}
```

Output:

```text
margin-top/bottom = 0, left/right = auto.
Any earlier margin-top is overwritten.
```

Debugging checklist:

- Inspect the element's **Styles** pane; struck-through rules lost.
- Check **Computed** for the final value.
- Look for `!important`, inline styles, and layer order.
- Test with the `:hov` state toggle.
- Search the codebase for the selector before adding more specificity.

---

# Real-World Uses

A design system uses layers for reset/base/components/utilities, keeps specificity flat, and avoids `!important`.

```css
@layer reset, base, components, utilities;

@layer base {
  :root {
    --brand: #0f172a;
    --text: #334155;
  }
  body {
    color: var(--text);
    font-family: Inter, sans-serif;
  }
}

@layer components {
  .card {
    padding: 1.25rem;
    border: 1px solid #e2e8f0;
  }
  .card__title {
    color: var(--brand);
  }
}

@layer utilities {
  .mt-0 { margin-top: 0; }
}
```

Output:

```text
Utilities always win, components compose safely,
and no !important is needed.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Cascade origin** | User-agent < author < author important < user important |
| **Importance** | `!important` flips normal precedence |
| **Specificity** | (ids, classes, elements), compared left to right |
| **!important** | Last resort, not a strategy |
| **Inheritance** | Some properties inherit by default |
| **Global keywords** | inherit, initial, unset, revert, revert-layer |
| **Cascade layers** | Later layers beat earlier ones |
| **Shorthand vs longhand** | Shorthand resets unspecified longhands |
| **Debugging** | Styles pane, Computed pane, state toggle |

## The Mental Model

```text
conflict
  │
  ├── 1. origin/importance
  │       user-agent → author → author !important → user !important
  ├── 2. layers (later wins)
  ├── 3. specificity (ids, classes, elements)
  └── 4. source order (later wins)
        │
        └── inheritance fills unset inherited properties
```

### The 3 things to remember

**1. Specificity is a tuple:**

```css
#hero .title { color: #b91c1c; } /* 1,1,0 */
```

**2. Avoid !important:**

```css
.card--active { color: #0f172a; }
```

**3. Layers replace specificity wars:**

```css
@layer reset, base, components, utilities;
```

> **The cascade resolves conflicts by origin, layers, specificity, and order — fix the cause instead of adding !important.**
