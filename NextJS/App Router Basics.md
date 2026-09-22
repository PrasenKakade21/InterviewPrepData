# App Router Basics

Routing clicks as: `folder = URL` → `page.tsx = view` → `Link navigates`. No router config file — Kakade's filesystem is the config.

The core idea:

```text
app/about/page.tsx → /about
    ↓
<Link href="/about"> → client-side nav
    ↓
useRouter for programmatic moves
```

Every folder needs a `page.tsx` to be visitable; components without one are just helpers.

---

# 1. Folders Define Routes and page.tsx Conventions

`Folders define routes`: nested folders nest URLs (`app/blog/first/page.tsx` → `/blog/first`). `page.tsx conventions`: default-exported component, can be async (server), receives `params`/`searchParams` props.

```text
app/page.tsx → /
app/about/page.tsx → /about
app/blog/first/page.tsx → /blog/first
```

```jsx
export default function About() {
  return <p>About Kakade</p>;
}
```

Output:

```text
/about renders About Kakade.
```

Only reserved names (`page`, `layout`, `loading`, `error`) create routes — other files in folders are invisible to the router.

---

# 2. Link and Navigation, useRouter Basics, and Active Link Patterns

`Link and navigation` (`<Link href>`) prefetches and navigates without full reloads. `useRouter basics` (`useRouter` from `next/navigation`, client only) pushes/refreshes programmatically. `Active link patterns` compare `usePathname()` to highlight Kakade's current page.

```jsx
import Link from "next/link";

export default function Nav() {
  return <Link href="/about">About Kakade</Link>;
}
```

Output:

```text
Click → /about, no full reload (prefetched).
```

Active + programmatic:

```jsx
"use client";
import { usePathname, useRouter } from "next/navigation";

export function Menu() {
  const path = usePathname();
  const router = useRouter();
  return <button onClick={() => router.push("/about")}>{path === "/about" ? "Here" : "Go"} Kakade</button>;
}
```

Output:

```text
On /about shows Here Kakade; elsewhere jumps there.
```

Use `<Link>` for visible links (SEO + prefetch); `router.push` only for post-action moves like after login.

---

# Real-World Uses

Kakade's portfolio nav uses Links with active highlighting via `usePathname`, and her login form pushes to the dashboard on success.

```jsx
import Link from "next/link";

export default function Home() {
  return <Link href="/blog">Blog of Kakade</Link>;
}
```

Output:

```text
Blog of Kakade → /blog prefetched on hover.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Folders define routes** | Filesystem = URL map |
| **page.tsx conventions** | Default export, async ok |
| **Link and navigation** | Prefetch, no reload |
| **useRouter basics** | Client-only, push/refresh |
| **Active link patterns** | `usePathname()` compare |

## The Mental Model

```text
URL for Kakade
  │
  ├── folder path → page.tsx renders
  │
  ├── <Link> → declarative nav
  │
  └── router.push → after actions
        └── usePathname → active state
```

### The 3 things to remember

**1. File = route:**

```text
app/about/page.tsx → /about
```

**2. Link first:**

```jsx
<Link href="/about">About Kakade</Link>
```

**3. Active check:**

```jsx
const path = usePathname();
```

> **Folders are routes, Links navigate free, and router.push is for post-action moves.**
