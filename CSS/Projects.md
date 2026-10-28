# Projects

Projects turn CSS knowledge into muscle memory. Each project below targets a specific skill cluster: layout, components, theming, motion, and responsive design. Build them without frameworks first, then compare with Tailwind or a design system.

The core idea:

```text
project
    ├── layout → page shell + grid
    ├── component → card/button/nav
    ├── state → hover/focus/open
    ├── theme → tokens + dark mode
    └── responsive → mobile-first + container queries
```

---

# 1. Responsive Landing Page

**Skills:** grid shell, hero, clamp typography, mobile-first nav.

```html
<header class="site-header">
  <a class="brand" href="/">Kakade</a>
  <nav class="nav" aria-label="Main">
    <a href="#work">Work</a>
    <a href="#about">About</a>
    <a class="btn" href="#contact">Contact</a>
  </nav>
</header>

<section class="hero">
  <h1>Frontend systems that scale.</h1>
  <p>Design, build, and ship accessible interfaces.</p>
  <a class="btn" href="#work">See work</a>
</section>
```

```css
*,
*::before,
*::after { box-sizing: border-box; }

body {
  margin: 0;
  font-family: Inter, "Segoe UI", system-ui, sans-serif;
  line-height: 1.6;
  color: #0f172a;
}

.site-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 1rem clamp(1rem, 4vw, 3rem);
}

.nav {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.hero {
  min-height: 60dvh;
  display: grid;
  place-content: center;
  gap: 1rem;
  padding: clamp(2rem, 8vw, 6rem);
  text-align: center;
}

.hero h1 {
  margin: 0;
  font-size: clamp(2rem, 6vw + 1rem, 4.5rem);
  line-height: 1.05;
  text-wrap: balance;
}

.btn {
  display: inline-block;
  padding: 0.75rem 1.25rem;
  border-radius: 999px;
  background: #0f172a;
  color: #fff;
  text-decoration: none;
}
```

Output:

```text
Responsive header, fluid hero, readable type,
and a clear primary action.
```

---

# 2. Dashboard Layout

**Skills:** named grid areas, fixed sidebar, responsive widget grid.

```css
.dashboard {
  display: grid;
  grid-template-columns: 220px 1fr;
  grid-template-rows: auto 1fr;
  grid-template-areas:
    "sidebar header"
    "sidebar main";
  min-height: 100dvh;
  gap: 1rem;
}

.dashboard__header { grid-area: header; }
.dashboard__sidebar { grid-area: sidebar; }
.dashboard__main {
  grid-area: main;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
  align-content: start;
}

@media (max-width: 720px) {
  .dashboard {
    grid-template-columns: 1fr;
    grid-template-areas:
      "header"
      "main";
  }
  .dashboard__sidebar { display: none; }
}
```

Output:

```text
Desktop: sidebar + header + widget grid
Mobile: header + stacked widgets, sidebar hidden
```

---

# 3. Pricing Cards

**Skills:** flex/grid, equal heights, featured state, tokens.

```css
.pricing {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.5rem;
  align-items: stretch;
}

.plan {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  padding: 1.5rem;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
}

.plan--featured {
  border-color: #0f172a;
  box-shadow: 0 12px 32px rgb(15 23 42 / 0.12);
}

.plan__cta {
  margin-top: auto;
}
```

Output:

```text
Equal-height plans; CTA pinned to the bottom;
featured plan visually elevated.
```

---

# 4. Animated Modal

**Skills:** dialog, backdrop, transitions, focus.

```css
dialog {
  border: 0;
  border-radius: 16px;
  padding: 1.5rem;
  width: min(90vw, 440px);
}

dialog::backdrop {
  background: rgb(15 23 42 / 0.55);
}

dialog[open] {
  animation: modal-in 180ms ease-out;
}

@keyframes modal-in {
  from { opacity: 0; transform: translateY(8px) scale(0.98); }
  to   { opacity: 1; transform: translateY(0) scale(1); }
}

@media (prefers-reduced-motion: reduce) {
  dialog[open] { animation: none; }
}
```

Output:

```text
Native modal with backdrop and a short entrance;
no motion when reduced motion is requested.
```

---

# 5. Dark and Light Theme

**Skills:** tokens, `data-theme`, OS preference, persistence hook.

```css
:root {
  --bg: #ffffff;
  --surface: #f8fafc;
  --text: #0f172a;
  --brand: #0f172a;
}

[data-theme="dark"] {
  --bg: #0f172a;
  --surface: #1e293b;
  --text: #e2e8f0;
  --brand: #93c5fd;
}

body {
  background: var(--bg);
  color: var(--text);
}

.card {
  background: var(--surface);
  border: 1px solid color-mix(in oklab, var(--text) 12%, transparent);
}
```

Output:

```text
Theme switch only changes tokens; components stay untouched.
```

---

# 6. Responsive Navbar

**Skills:** flex, wrapping, mobile menu pattern, focus states.

```css
.navbar {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 1rem;
}

.navbar__links {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  list-style: none;
  margin: 0;
  padding: 0;
}

@media (max-width: 600px) {
  .navbar__links {
    flex-direction: column;
    width: 100%;
  }
}
```

Output:

```text
Desktop: inline links
Mobile: stacked links below the brand
```

---

# 7. CSS Illustration

**Skills:** pseudo-elements, gradients, transforms, shapes.

```css
.planet {
  width: 160px;
  aspect-ratio: 1;
  border-radius: 50%;
  background: radial-gradient(circle at 30% 30%, #93c5fd, #0f172a 70%);
  position: relative;
}

.planet::after {
  content: "";
  position: absolute;
  inset: 55% -30% auto;
  height: 24px;
  border-radius: 50%;
  border: 6px solid #e2e8f0;
  transform: rotate(-18deg);
}
```

Output:

```text
A gradient planet with a tilted ring, no images.
```

---

# 8. Design System Page

**Skills:** tokens, typography scale, buttons, forms, states, documentation.

```css
:root {
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-4: 1rem;
  --space-8: 2rem;
  --radius-md: 12px;
  --color-brand: #0f172a;
  --color-text: #0f172a;
  --color-muted: #475569;
}

.btn {
  padding: var(--space-2) var(--space-4);
  border-radius: var(--radius-md);
  border: 1px solid var(--color-brand);
  background: var(--color-brand);
  color: #fff;
}

.btn:focus-visible {
  outline: 3px solid var(--color-brand);
  outline-offset: 2px;
}

.swatch {
  height: 64px;
  border-radius: var(--radius-md);
}
```

Output:

```text
A living reference page with tokens, components, and states.
```

---

# Real-World Uses

Build order that teaches the most:

```text
1. Landing page      → layout + responsive + type
2. Pricing cards     → flex + equal heights + states
3. Dashboard         → grid areas + widgets
4. Modal             → dialog + motion + focus
5. Theme             → tokens + dark mode
6. Navbar            → responsive patterns
7. Illustration      → pseudo-elements + gradients
8. Design system     → documentation + consistency
```

Output:

```text
Each project adds one skill cluster; together they cover
the CSS used in real product work.
```

---

# Quick Revision

| Project | Core skills |
| ------- | ----------- |
| **Landing page** | Grid, clamp, mobile-first |
| **Dashboard** | Named grid areas, auto-fit |
| **Pricing cards** | Flex, equal heights, featured state |
| **Animated modal** | dialog, backdrop, keyframes |
| **Theme** | Tokens, data-theme, dark mode |
| **Navbar** | Flex wrap, responsive menu |
| **Illustration** | Pseudo-elements, gradients |
| **Design system** | Tokens, components, docs |

## The Mental Model

```text
project practice
  ├── layout → landing, dashboard
  ├── components → cards, navbar, buttons
  ├── state/motion → modal, hover/focus
  ├── theming → tokens + dark mode
  └── system → design system page
```

### The 3 things to remember

**1. Start with layout:**

```css
.shell {
  display: grid;
  grid-template-columns: 240px 1fr;
}
```

**2. Tokenize early:**

```css
:root { --space-4: 1rem; --radius-md: 12px; }
```

**3. Ship accessible states:**

```css
.btn:focus-visible {
  outline: 3px solid #0f172a;
  outline-offset: 2px;
}
```

> **Build small projects that each add one skill — layout, components, motion, theming — and the system emerges.**
