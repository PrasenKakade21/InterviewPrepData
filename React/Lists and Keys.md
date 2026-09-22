# Lists and Keys

Lists click as: `array` → `map() to JSX` → `key per item`. React uses keys to track which row is which across re-renders, so inserts and deletes stay correct.

The core idea:

```text
Data array
    ↓
map() to elements
    ↓
stable key per item
    ↓
React updates minimal rows
```

Any time you render repeated UI — users, todos, products — you map over data.

---

# 1. Rendering Lists with map(), Keys, and Why Keys Matter

`Rendering lists with map()` turns each data item into JSX. `Keys` are stable strings React uses as identity. `Why keys matter`: without them React reuses rows by position, so input focus, checkbox state, and animations attach to the wrong item after inserts.

```jsx
function UserList() {
  const users = ["Kakade", "Asha", "Rohan"];
  return users.map((name) => <li key={name}>{name}</li>);
}
```

Output:

```text
• Kakade
• Asha
• Rohan
```

Think:

```text
users array
  ├── key=Kakade → <li>Kakade</li>
  ├── key=Asha → <li>Asha</li>
  └── key=Rohan → <li>Rohan</li>
```

Rules: keys go on the element returned from `map()`, must be unique among siblings, and never need to be globally unique. Use ids from data, not random values per render.

---

# 2. Index as Key Pitfalls and Stable ids

`Index as key` breaks when the list reorders, filters, or inserts at the top — React thinks the item at position 0 is the same as before. `Stable ids` (`user.id`, `todo.id`, `crypto.randomUUID()` at creation) survive reordering.

```jsx
function StableList() {
  const users = [
    { id: "u1", name: "Kakade" },
    { id: "u2", name: "Asha" },
  ];
  return users.map((u) => <li key={u.id}>{u.name}</li>);
}
```

Output:

```text
• Kakade
• Asha (keys u1, u2 stay correct on reorder)
```

Bad (index key with reorderable input):

```jsx
// Bad: typing in row 1, then sorting, moves text to wrong row
// {users.map((u, i) => <input key={i} defaultValue={u.name} />)}
```

Good:

```jsx
function FixedList({ users }) {
  return users.map((u) => <li key={u.id}>{u.name} done</li>);
}
```

Output:

```text
Rows keep identity after sort or filter.
```

Index keys are only safe for static lists that never reorder, filter, or hold state. Interview one-liner: index is position, id is identity.

---

# 3. Filtering Lists, Sorting Lists, and Empty List Handling

`Filtering` with `filter()` + `map()` shows subsets. `Sorting` needs a copy (`[...arr].sort()`) since `sort()` mutates. `Empty list handling` shows guidance when nothing matches.

```jsx
function ActiveUsers({ users }) {
  const active = users.filter((u) => u.active);
  if (active.length === 0) {
    return <p>No active users. Invite Kakade!</p>;
  }
  return active.map((u) => <p key={u.id}>{u.name}</p>);
}
```

Output:

```text
Only active users render; empty message when none.
```

Sorting without mutation:

```jsx
function Sorted({ names }) {
  const ordered = [...names].sort();
  return ordered.map((n) => <p key={n}>{n} hi</p>);
}
```

Output:

```text
Names in A–Z order: Asha, Kakade, Rohan.
```

Notice: never `names.sort()` directly on props or state — it mutates the source. Combine: `users.filter(...).sort(...).map(...)` in one chain, computed during render.

---

# 4. Nested Lists

`Nested lists` are maps inside maps — groups containing items. Each level needs its own keys: one for the group, one for each child.

```jsx
function Teams() {
  const teams = [
    { id: "t1", name: "Frontend", members: ["Kakade", "Asha"] },
    { id: "t2", name: "Backend", members: ["Rohan"] },
  ];
  return teams.map((t) => (
    <div key={t.id}>
      <h3>{t.name}</h3>
      {t.members.map((m) => (
        <p key={m}>{m}</p>
      ))}
    </div>
  ));
}
```

Output:

```text
Frontend
  Kakade
  Asha
Backend
  Rohan
```

Think:

```text
teams (key=t.id)
  └── members (key=member id)
```

If member names can repeat across teams, use composite keys like `` `${t.id}-${m}` ``. Keep nesting shallow — extract a `TeamGroup` component when the inner map grows.

---

# Real-World Uses

A product grid filters by category, sorts by price, handles zero results, and nests tags per product — all with stable product ids as keys.

```jsx
function Products({ items }) {
  if (items.length === 0) {
    return <p>No products for Kakade yet.</p>;
  }
  return items.map((p) => <p key={p.id}>{p.title}</p>);
}
```

Output:

```text
One line per product, or the empty message.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Rendering lists with map()** | `{arr.map(item => <Row />)}` |
| **Keys** | Unique-per-sibling identity string |
| **Why keys matter** | Preserve focus, state, animation on change |
| **Index as key pitfalls** | Breaks on reorder / insert / filter |
| **Stable ids** | `id` from data, UUID at creation |
| **Filtering lists** | `filter(...).map(...)` |
| **Sorting lists** | `[...arr].sort()` copy first |
| **Empty list handling** | Early return with helpful message |
| **Nested lists** | Keys at every `map()` level |

## The Mental Model

```text
Source array
  │
  ├── filter → subset
  ├── sort copy → order
  │       │
  │       ↓
  └── map(item → <Row key={stableId} />)
            │
            └── empty? → helpful fallback
```

### The 3 things to remember

**1. Map with stable keys:**

```jsx
{users.map((u) => <li key={u.id}>{u.name}</li>)}
```

**2. Never mutate to sort:**

```jsx
const ordered = [...names].sort();
```

**3. Handle empty:**

```jsx
if (items.length === 0) {
  return <p>No items for Kakade</p>;
}
```

> **Map data to UI with a stable key per item — identity, not position.**
