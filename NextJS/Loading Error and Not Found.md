# Loading Error and Not Found

Route states click as: `loading streams` → `error catches` → `not-found designs 404`. File names do the wiring — Kakade just designs each state.

The core idea:

```text
app/blog/loading.tsx → instant skeleton
    ↓
app/blog/error.tsx → crash fallback + reset
    ↓
app/blog/not-found.tsx → missing with charm
```

Every data route deserves all three; empty states are features, not afterthoughts.

---

# 1. loading.tsx and Suspense with Routes

`loading.tsx` renders instantly while the page streams in — per-folder skeletons with zero config. `Suspense with routes`: wrap slow islands so fast content shows first.

```jsx
export default function Loading() {
  return <p>Loading Kakade…</p>;
}
```

Output:

```text
Skeleton paints immediately, page swaps in when ready.
```

Selective suspense:

```jsx
import { Suspense } from "react";

export default function Page() {
  return (
    <div>
      <p>Hi Kakade</p>
      <Suspense fallback={<p>Comments loading…</p>}>
        <SlowComments />
      </Suspense>
    </div>
  );
}
```

Output:

```text
Hi Kakade now, comments when done.
```

Keep skeletons shaped like content (bars, not spinners) — layout shift kills perceived speed.

---

# 2. error.tsx Boundaries and global-error Basics

`error.tsx boundaries` catch crashes in their folder — client components receiving `{ error, reset }`. `global-error basics` (`app/global-error.tsx` with own `<html>`) catches root layout failures as last resort.

```jsx
"use client";

export default function Error({ error, reset }) {
  return (
    <div>
      <p>Broke for Kakade: {error.message}</p>
      <button onClick={reset}>Retry</button>
    </div>
  );
}
```

Output:

```text
Crash → message + Retry (reset re-renders the folder).
```

Rules: boundaries need `"use client"`, log `error` to your tracker, and `reset()` only retries render — fix stale data with revalidation, not resets.

---

# 3. not-found.tsx

`not-found.tsx` renders on `notFound()` calls or unmatched dynamic ids — designed 404s per section beat generic ones.

```jsx
import { notFound } from "next/navigation";

export default async function User({ params }) {
  const { id } = await params;
  if (id !== "kakade") notFound();
  return <p>User Kakade</p>;
}
```

```jsx
export default function NotFound() {
  return <p>No Kakade here — try home.</p>;
}
```

Output:

```text
/users/xyz → No Kakade here (404 status, custom UI).
```

404 for missing resources, error.tsx for broken code — never mix them.

---

# Real-World Uses

Kakade's blog shows skeletons per section, isolates chart crashes with folder boundaries, and gives unknown slugs a helpful 404 with search.

```jsx
export default function Loading() {
  return <p>Shelf for Kakade…</p>;
}
```

Output:

```text
Shelf for Kakade… then content streams in.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **loading.tsx** | Instant per-folder skeleton |
| **error.tsx boundaries** | Client, `{ error, reset }` |
| **not-found.tsx** | Designed 404 via `notFound()` |
| **global-error basics** | Last-resort root catcher |
| **Suspense with routes** | Stream slow islands later |

## The Mental Model

```text
Route for Kakade
  │
  ├── loading → skeleton now
  │
  ├── crashes? → error.tsx + reset
  │
  └── missing? → notFound() → not-found.tsx
```

### The 3 things to remember

**1. Skeleton file:**

```jsx
export default function Loading() {
  return <p>Loading Kakade…</p>;
}
```

**2. Retry boundary:**

```jsx
<button onClick={reset}>Retry</button>
```

**3. Designed 404:**

```jsx
if (id !== "kakade") notFound();
```

> **Skeleton instantly, catch crashes locally with retry, and design every 404.**
