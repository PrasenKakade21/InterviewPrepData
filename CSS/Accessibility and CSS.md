# Accessibility and CSS

CSS can make an interface usable or unusable. Focus visibility, contrast, motion, hiding, reflow, and target size are all CSS concerns. Accessibility is not a separate stylesheet; it is a set of defaults you keep.

The core idea:

```text
accessible CSS
    ├── focus → visible, high-contrast ring
    ├── contrast → 4.5:1 text / 3:1 large & UI
    ├── motion → respect reduced motion
    ├── hiding → correct technique for intent
    └── layout → reflow, zoom, target size, RTL
```

---

# 1. Visible Focus Styles

Keyboard users need to see where they are. Never remove outlines without a replacement; use `:focus-visible` for a clear ring.

```css
:where(a, button, input, select, textarea, [tabindex]):focus-visible {
  outline: 3px solid #0f172a;
  outline-offset: 2px;
  border-radius: 4px;
}
```

Output:

```text
Every interactive element shows a dark 3px ring when focused via keyboard.
```

Gotchas: `outline: none` is the most common accessibility failure. If a custom ring is clipped by `overflow: hidden`, use `outline-offset` or adjust the clipping container.

---

# 2. Color Contrast

Text needs at least 4.5:1 contrast (3:1 for large text ≥24px or ≥18.66px bold). UI component boundaries and focus indicators need 3:1.

```css
:root {
  --color-text: #1e293b;      /* on white: strong contrast */
  --color-muted: #475569;     /* still AA for normal text */
}

.muted {
  color: var(--color-muted);
  background: #ffffff;
}
```

Output:

```text
Body and muted text both meet AA on white.
```

Gotchas: placeholder text, disabled controls, and text over images are frequent failures. Disabled controls are exempt from WCAG contrast, but should still be legible.

---

# 3. Reduced Motion and Color Scheme

Respect user preferences for motion and theme. These are CSS media features backed by OS settings.

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0f172a;
    --color-text: #e2e8f0;
  }
}
```

Output:

```text
Motion is effectively instant; dark tokens apply when the OS prefers dark.
```

Pair with `color-scheme: light dark` so native controls and scrollbars match.

---

# 4. Hiding Content Correctly

Pick the technique by intent: remove from everyone, hide visually but keep for screen readers, or hide from screen readers only.

```css
/* Available to screen readers, invisible visually */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
  border: 0;
}

/* Skip link visible on focus */
.skip-link:focus-visible {
  position: static;
  width: auto;
  height: auto;
  clip: auto;
  padding: 0.75rem 1rem;
  background: #0f172a;
  color: #fff;
}
```

Output:

```text
Screen-reader-only text stays available; skip link appears on keyboard focus.
```

Interview trap: `display:none` removes content from assistive tech. Use it when content is truly irrelevant, not for "visually hidden" labels.

---

# 5. RTL, Zoom, Reflow and Target Size

Logical properties support right-to-left languages. Layouts must reflow at 320px and 400% zoom without two-dimensional scrolling. Touch targets should be at least 24×24 CSS px (44×44 recommended).

```css
.card {
  margin-inline-start: 1rem;
  padding-block: 1rem;
  border-inline-start: 4px solid #0f172a;
}

.icon-button {
  min-width: 44px;
  min-height: 44px;
  display: inline-grid;
  place-items: center;
}
```

Output:

```text
Spacing flips correctly in RTL; buttons meet touch target guidance.
```

Reflow checklist: no horizontal scroll at 320px, text scales to 200%, no fixed heights that clip zoomed text, and content order stays logical.

---

# 6. Forced Colors Mode

Windows High Contrast / forced-colors mode replaces author colors with system colors. Use system color keywords and avoid background-image-only icons.

```css
@media (forced-colors: active) {
  .card {
    border: 1px solid CanvasText;
  }

  .icon {
    forced-color-adjust: none; /* only when necessary */
  }
}
```

Output:

```text
Borders remain visible in forced-colors mode; icons keep needed contrast.
```

Interview trap: `forced-color-adjust: none` opts out of the user's palette; use sparingly and verify contrast manually.

---

# Real-World Uses

A form uses focus rings, AA contrast, reduced motion, sr-only hints, and 44px targets.

```css
.field input {
  min-height: 44px;
  color: var(--color-text);
  border: 1px solid #94a3b8;
}

.field input:focus-visible {
  outline: 3px solid #0f172a;
  outline-offset: 2px;
}

.hint {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
}

@media (prefers-reduced-motion: reduce) {
  .field { transition: none; }
}
```

Output:

```text
Accessible field with visible focus, adequate target size,
and hidden hint text available to assistive tech.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Focus styles** | Never remove without a visible replacement |
| **Contrast** | 4.5:1 text, 3:1 large/UI |
| **Reduced motion** | Disable decorative animation |
| **Color scheme** | Follow OS preference + override |
| **Hiding content** | sr-only vs display:none by intent |
| **RTL/logical** | inline/block properties |
| **Zoom/reflow** | 320px, 400%, no 2D scroll |
| **Target size** | 24px min, 44px recommended |
| **Forced colors** | System colors, avoid opt-out |

## The Mental Model

```text
a11y CSS
  ├── perceivable → contrast, focus visibility
  ├── operable → target size, keyboard focus, no motion traps
  ├── understandable → logical order, RTL
  └── robust → zoom/reflow, forced colors, reduced motion
```

### The 3 things to remember

**1. Focus ring always:**

```css
:focus-visible {
  outline: 3px solid #0f172a;
  outline-offset: 2px;
}
```

**2. sr-only, not display:none:**

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
}
```

**3. Respect preferences:**

```css
@media (prefers-reduced-motion: reduce) { ... }
```

> **Accessible CSS is defaults you keep — visible focus, real contrast, honest hiding, reflow, and respect for user preferences.**
