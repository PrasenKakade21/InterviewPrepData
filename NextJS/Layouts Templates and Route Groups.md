# Layouts Templates and Route Groups

Shared UI clicks as: `layout persists` → `template resets` → `groups organize`. Layouts keep Kakade's nav mounted across pages; groups shape URLs without showing in them.

The core idea:

```text
app/layout.tsx → every page (nav, fonts)
    ↓
app/blog/layout.tsx → blog subtree only
    ↓
(m9469) → hidden grouping, no URL effect
```

Reach for nested layouts before context providers — persistence is free.

---

# 1. Root Layout, Nested Layouts, and Colocated Files

`Root layout` (required `app/layout.tsx` with `<html>`/`<body>`) wraps everything — fonts, metadata, nav. `Nested layouts` apply to their subtree and preserve state across sibling pages. `Colocated files` (components, styles next to pages) stay private unless reserved-named.

```jsx
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <nav>Nav for Kakade</nav>
        {children}
      </body>
    </html>
  );
}
```

Output:

```text
Every page shows Nav for Kakade (mounted once).
```

Think:

```text
/ → RootLayout
/blog/first → RootLayout + blog/layout + page
    └── nav state survives page switches
```

Layouts must render `{children}` — forgetting it blanks every child page silently.

---

# 2. Template vs Layout and Route Groups

`Template vs layout`: layouts preserve state (nav, sidebars); templates (`template.tsx`) remount on every navigation — for enter animations or per-page resets. `Route groups` (`(name)` folders) organize without URL segments — `(marketing)` vs `(app)` with different layouts.

```text
app/(marketing)/about/page.tsx → /about (group hidden)
app/(app)/dash/page.tsx → /dash (own layout)
```

```jsx
export default function Template({ children }) {
  return <div className="enter">{children} for Kakade</div>;
}
```

Output:

```text
Remounts each visit (animation replays for Kakade).
```

| File | State across nav |
| ---- | ---------------- |
| **`layout.tsx`** | **Preserved** |
| **`template.tsx`** | **Reset (remount)** |

Default to layouts; use templates only when reset-on-navigate is the point.

---

# Real-World Uses

Kakade's site uses a root layout (nav + fonts), an `(app)` group layout (sidebar dashboard), and a `(marketing)` group (wide landing) — one codebase, two shells.

```jsx
export default function BlogLayout({ children }) {
  return <aside>{children} — Kakade</aside>;
}
```

Output:

```text
Blog pages share Kakade sidebar, state intact.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Root layout** | Required, wraps all |
| **Nested layouts** | Subtree shells, state kept |
| **Template vs layout** | Reset vs preserve |
| **Route groups** | `(x)` organizes, hides segment |
| **Colocated files** | Private unless reserved |

## The Mental Model

```text
Shell for Kakade
  │
  ├── root layout → all pages
  │
  ├── nested / group layouts → subtrees
  │
  └── need reset? → template.tsx
```

### The 3 things to remember

**1. Root wraps all:**

```jsx
<html lang="en">
  <body>{children}</body>
</html>
```

**2. Groups hide:**

```text
(m9469)/about → /about
```

**3. Preserve default:**

```jsx
// layout.tsx keeps state; template.tsx resets it
```

> **Layouts persist shells, groups organize invisibly, and templates are the rare reset button.**
