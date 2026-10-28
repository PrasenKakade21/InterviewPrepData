# Responsive Design

Responsive design makes one codebase work across phones, tablets, and desktops. CSS gives you media queries, fluid units, container queries, and responsive images. Interviewers test mobile-first thinking and whether you reach for `clamp()`/container queries instead of a dozen fixed breakpoints.

The core idea:

```text
layout
    ├── viewport meta → correct scaling
    ├── fluid base → %/fr/clamp
    ├── media queries → layout changes
    ├── container queries → component-level changes
    └── responsive media → srcset/sizes
```

---

# 1. Viewport Meta and Media Queries

The viewport meta tag makes mobile use the device width. Media queries apply CSS conditionally based on viewport features.

```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

```css
@media (min-width: 768px) {
  .layout {
    grid-template-columns: 240px 1fr;
  }
}

@media (max-width: 767px) {
  .nav {
    display: none;
  }
}
```

Output:

```text
Phones: single column, nav hidden
768px+: sidebar + content layout
```

Interview trap: `max-width` and `min-width` media queries overlap at the boundary if you use the same value. With mobile-first, use `min-width` and one breakpoint per range.

---

# 2. Mobile-First and Breakpoints

Mobile-first means writing the base styles for small screens, then adding enhancements with `min-width`. Base CSS stays simple and avoids overrides.

```css
/* Base: mobile */
.cards {
  display: grid;
  gap: 1rem;
  grid-template-columns: 1fr;
}

/* Tablet and up */
@media (min-width: 640px) {
  .cards {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .cards {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

Output:

```text
1 column → 2 columns at 640px → 3 columns at 1024px
```

Breakpoints should follow content, not specific devices. Common ranges: 640, 768, 1024, 1280.

---

# 3. Container Queries

Container queries let a component respond to its container's size, not the viewport. This makes reusable cards truly responsive.

```css
.card-wrap {
  container-type: inline-size;
  container-name: card;
}

.card {
  display: grid;
  gap: 1rem;
}

@container card (min-width: 480px) {
  .card {
    grid-template-columns: 160px 1fr;
    align-items: center;
  }
}
```

Output:

```text
Narrow container: stacked card
480px+ container: image beside content
```

Interview trap: container queries need a containment context (`container-type`); they do not respond to the viewport. Use media queries for page-level layout and container queries for components.

---

# 4. Fluid Typography, Spacing and Responsive Images

`clamp()` removes the need for font-size media queries. Responsive images deliver the right file per viewport/DPR.

```css
.hero__title {
  font-size: clamp(2rem, 5vw + 1rem, 4.5rem);
  line-height: 1.1;
}

.section {
  padding: clamp(1.5rem, 4vw, 4rem);
}
```

```html
<img
  src="/img/hero-800.jpg"
  srcset="/img/hero-400.jpg 400w, /img/hero-800.jpg 800w, /img/hero-1600.jpg 1600w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="Kakade's desk"
  width="800"
  height="450"
/>
```

Output:

```text
Type and spacing scale smoothly; the browser downloads the smallest
sufficient image and reserves the right aspect ratio.
```

---

# 5. Print Styles and Testing

Print styles adjust content for paper. Test responsiveness in DevTools device mode, by resizing, and with real devices when possible.

```css
@media print {
  .site-header,
  .site-footer,
  .no-print {
    display: none;
  }

  body {
    color: #000;
    background: #fff;
  }

  a[href]::after {
    content: " (" attr(href) ")";
  }
}
```

Output:

```text
Printed page hides navigation/footer and shows link URLs after links.
```

Testing checklist:

- 320px, 375px, 768px, 1024px, 1440px.
- 200% zoom and text-only zoom.
- Long words, long titles, missing images.
- Landscape phone orientation.
- Both `prefers-reduced-motion` and `prefers-color-scheme`.

---

# Real-World Uses

A responsive article page uses mobile-first layout, a container query for cards, fluid type, and print styles.

```css
.page {
  display: grid;
  gap: clamp(1rem, 3vw, 2rem);
  padding: clamp(1rem, 4vw, 3rem);
}

@media (min-width: 900px) {
  .page {
    grid-template-columns: 1fr 280px;
    align-items: start;
  }
}

.aside {
  container-type: inline-size;
}

@container (min-width: 260px) {
  .aside__card {
    padding: 1.25rem;
  }
}

@media print {
  .aside { display: none; }
}
```

Output:

```text
Single column on mobile, article + aside on desktop,
card padding adapts to its container, aside removed for print.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Viewport meta** | `width=device-width, initial-scale=1` |
| **Media queries** | Conditional CSS by viewport features |
| **Mobile-first** | Base small, enhance with `min-width` |
| **Breakpoints** | Follow content, not devices |
| **Container queries** | Respond to container size |
| **Fluid typography** | `clamp()` instead of many size queries |
| **Responsive images** | `srcset` + `sizes` |
| **Print styles** | Hide chrome, show link URLs |
| **Testing** | Resize, zoom, real devices, edge content |

## The Mental Model

```text
responsive layers
  ├── viewport → correct mobile scaling
  ├── fluid base → %, fr, clamp
  ├── viewport queries → page layout
  ├── container queries → component layout
  ├── responsive media → srcset/sizes
  └── print → paper-specific rules
```

### The 3 things to remember

**1. Mobile-first min-width:**

```css
@media (min-width: 768px) { ... }
```

**2. Fluid type:**

```css
font-size: clamp(2rem, 5vw + 1rem, 4.5rem);
```

**3. Components use containers:**

```css
.wrap { container-type: inline-size; }
@container (min-width: 480px) { ... }
```

> **Responsive design is layers — correct viewport, fluid base, viewport queries for pages, container queries for components.**
