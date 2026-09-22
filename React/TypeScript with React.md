# TypeScript with React

TypeScript clicks as: `props first` → `state + events inferred` → `generics for reuse`. Types document the contract Kakade's components promise — strict where it matters, inferred elsewhere.

The core idea:

```text
type Props = { … } → ({ … }: Props)
    ↓
useState<T> only when inference fails
    ↓
events / refs / children typed at boundary
```

Prefer `type` + unions for props; let handlers infer.

---

# 1. Typing Props, Optional Props, Typing State, and Typing Children

`Typing props` declares the component contract; `optional props` (`?` + defaults) keep call sites light. `Typing state` (`useState<User | null>`) pins nullable/async shapes. `Typing children` (`ReactNode`) allows text, elements, and fragments.

```tsx
import { useState, type ReactNode } from "react";

type CardProps = { name: string; vip?: boolean; children: ReactNode };

function Card({ name, vip = false, children }: CardProps) {
  const [likes, setLikes] = useState(0);
  return (
    <div onClick={() => setLikes(likes + 1)}>
      {name || "Kakade"} {vip ? "VIP" : ""} {likes} {children}
    </div>
  );
}
```

Output:

```text
Card for Kakade, likes count up on click.
```

Think:

```text
<Card name="Kakade">hi</Card> ✓
<Card /> ✗ (name required)
```

Use `ReactNode` for slots, `ComponentType` for component props — never `any` for children.

---

# 2. Typing Events, Typing Refs, and Union Props

`Typing events` (`ChangeEvent<HTMLInputElement>`, `FormEvent`) types targets without casts. `Typing refs` (`useRef<HTMLInputElement>(null)`) unlocks `focus()` safely. `Union props` model variants so impossible combos do not compile.

```tsx
import { useRef, useState, type ChangeEvent } from "react";

function Search() {
  const [q, setQ] = useState("Kakade");
  const ref = useRef<HTMLInputElement>(null);
  function onChange(e: ChangeEvent<HTMLInputElement>) {
    setQ(e.target.value);
  }
  return <input ref={ref} value={q} onChange={onChange} />;
}
```

Output:

```text
Typed input for Kakade with focusable ref.
```

Union shape:

```tsx
// type Badge = { kind: "vip"; since: number } | { kind: "guest" };
// kind === "vip" → since exists; "guest" → no since (narrowed).
```

Narrow unions with discriminants (`kind`) — optional-everything props hide bugs.

---

# 3. Generics in Components and Type-Safe Hooks

`Generics in components` (`List<T>`) reuse rendering across types. `Type-safe hooks` type return tuples/objects and guards so callers get inference for free.

```tsx
type ListProps<T> = { items: T[]; render: (item: T) => React.ReactNode };

function List<T>({ items, render }: ListProps<T>) {
  return <ul>{items.map((it, i) => <li key={i}>{render(it)}</li>)}</ul>;
}

function Users() {
  return <List items={["Kakade"]} render={(n) => <b>{n.toUpperCase()}</b>} />;
}
```

Output:

```text
KAKADE list item (T inferred as string).
```

Safe hook sketch:

```tsx
// function useToggle(init = false): [boolean, () => void]
// const [on, toggle] = useToggle() → typed tuple, no boolean[] widening.
```

Constrain generics (`T extends { id: string }`) when you need keys; export prop types for tests and stories.

---

# Real-World Uses

A Kakade user list types props as a discriminated union (admin vs guest), infers event handlers, and reuses a generic `List<T>` so users, orders, and tags share one tested component.

```tsx
function Hi({ name = "Kakade" }: { name?: string }) {
  return <p>Hi {name}</p>;
}
```

Output:

```text
Hi Kakade (optional prop with default).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Typing props** | `type Props` at boundary |
| **Optional props** | `?` + defaults |
| **Typing state** | `useState<T>` for nullable shapes |
| **Typing children** | `ReactNode` for slots |
| **Typing events** | `ChangeEvent/FormEvent<T>` |
| **Typing refs** | `useRef<Element>(null)` |
| **Union props** | Discriminant narrows variants |
| **Generics in components** | `List<T>` reuse |
| **Type-safe hooks** | Typed tuples, no `any` |

## The Mental Model

```text
Component boundary
  │
  ├── Props (required / optional / union) + children: ReactNode
  │
  ├── inside: inferred state + events + refs
  │
  └── reuse: generics List<T> + typed hooks
```

### The 3 things to remember

**1. Props contract:**

```tsx
type CardProps = { name: string; vip?: boolean; children: ReactNode };
```

**2. Events + refs:**

```tsx
function onChange(e: ChangeEvent<HTMLInputElement>) {}
const ref = useRef<HTMLInputElement>(null);
```

**3. Generic reuse:**

```tsx
function List<T>({ items, render }: ListProps<T>) {}
```

> **Type the boundaries, infer the insides, and use unions + generics for variants and reuse.**
