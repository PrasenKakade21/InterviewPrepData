# Dynamic Routes Params and Search Params

Dynamic URLs click as: `[id] captures` → `params prop reads` → `searchParams filters`. One file serves every Kakade profile — params pick which.

The core idea:

```text
app/users/[id]/page.tsx → /users/kakade
    ↓
{ params: { id: "kakade" } } in page
    ↓
?tab=posts → searchParams (no extra file)
```

Validate params (they are strings, possibly hostile) before DB queries.

---

# 1. Dynamic Segments, Params in Pages, and Catch-all Routes

`Dynamic segments` (`[id]` folders) match any value at that position. `Params in pages`: server pages receive `{ params }` (await in newer Next). `Catch-all routes` (`[...slug]`) swallow multi-segment paths like docs trees.

```text
app/users/[id]/page.tsx → /users/kakade (id = kakade)
app/docs/[...slug]/page.tsx → /docs/a/b (slug = [a, b])
```

```jsx
export default async function User({ params }) {
  const { id } = await params;
  return <p>User {id || "Kakade"}</p>;
}
```

Output:

```text
/users/kakade renders User kakade.
```

Missing id? Pair with `not-found.tsx` — unknown Kakade ids get a designed 404, not a crash.

---

# 2. generateStaticParams and Search Params

`generateStaticParams` prebuilds known dynamic pages at build time (SSG for dynamic routes). `Search params` (`?q=…`) arrive via the `searchParams` prop — filters and pagination without new files.

```jsx
export async function generateStaticParams() {
  return [{ id: "kakade" }, { id: "prasen" }];
}

export default async function User({ params }) {
  const { id } = await params;
  return <p>Prebuilt {id}</p>;
}
```

Output:

```text
/kakade and /prasen prerendered at build.
```

Search shape:

```jsx
export default async function List({ searchParams }) {
  const q = (await searchParams).q || "Kakade";
  return <p>Search {q}</p>;
}
```

Output:

```text
/list?q=Kakade renders Search Kakade.
```

Static params for popular pages + dynamic fallback for the long tail; searchParams for anything filterable.

---

# Real-World Uses

Kakade's blog prebuilds top posts with `generateStaticParams`, renders the rest on demand, and paginates with `?page=2` — one file, infinite URLs.

```jsx
export default async function Post({ params }) {
  const { id } = await params;
  return <p>Post {id} by Kakade</p>;
}
```

Output:

```text
Post hello by Kakade (/blog/hello).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Dynamic segments** | `[id]` matches any value |
| **Params in pages** | `{ params }` prop (await) |
| **Catch-all routes** | `[...slug]` multi-segment |
| **generateStaticParams** | Prebuild known pages |
| **Search params** | `?q` filters, no new file |

## The Mental Model

```text
Dynamic URL for Kakade
  │
  ├── [id] → params.id → fetch one
  │
  ├── [...slug] → docs trees
  │
  └── ?q= → searchParams → filter
        └── known ids → generateStaticParams
```

### The 3 things to remember

**1. Bracket = variable:**

```text
app/users/[id]/page.tsx → /users/kakade
```

**2. Read params:**

```jsx
const { id } = await params;
```

**3. Prebuild + filter:**

```jsx
export async function generateStaticParams() {
  return [{ id: "kakade" }];
}
```

> **Brackets capture, params select, searchParams filter, and static params prebuild the popular.**
