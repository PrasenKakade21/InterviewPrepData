# Server Components

Server Components click as: `async on server` → `HTML to browser` → `zero JS shipped`. Default every component server; go client only with a reason.

The core idea:

```text
async function Page() → fetch + render on server
    ↓
HTML streams to Kakade
    ↓
no hooks, no events, no bundle cost
```

If it does not click, type, or animate — it stays server.

---

# 1. Default Server Rendering and Async Components

`Default server rendering`: every App Router component is server unless marked `"use client"`. `Async components`: pages can `await` data directly — no `useEffect` fetch chains.

```jsx
async function Users() {
  const users = await fetch("https://api.kakade.dev/users").then((r) => r.json());
  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

Output:

```text
Rendered list for Kakade (data resolved before HTML).
```

This replaces loading-spinners-first SPAs — content arrives painted, not pending.

---

# 2. No Hooks or Browser APIs and Composition Patterns

`No hooks or browser APIs`: no `useState`, `useEffect`, `window`, or event handlers — compile errors if tried. `Composition patterns`: pass server-rendered children into client shells to keep interactivity islands small.

```jsx
async function Bio() {
  return <p>Bio of Kakade (server)</p>;
}

("use client");
```

Bad (server file):

```jsx
// "useState in a server component" → error for Kakade.
// Move state into a "use client" child instead.
```

Good composition:

```jsx
// <ClientTabs>
//   <Bio /> ← server child, zero JS
// </ClientTabs>
```

```text
Tabs interactive, bio static — Kakade ships JS only for tabs.
```

Push client boundaries down to leaves — a client page root ships JS for everything inside.

---

# 3. Benefits for Kakade Apps

`Benefits`: secrets stay server (DB URLs, tokens), bundles shrink (heavy libs never download), SEO gets full HTML, and data waterfalls collapse into parallel awaits.

```jsx
async function Page() {
  const user = await fetch("https://api.kakade.dev/me", {
    headers: { Authorization: process.env.API_KEY },
  }).then((r) => r.json());
  return <p>Hi {user.name || "Kakade"}</p>;
}
```

Output:

```text
Hi Kakade (API key never left the server).
```

Interview line: Server Components move data-to-code instead of code-to-data — fetch beside the DB, send HTML over the wire.

---

# Real-World Uses

Kakade's dashboard fetches user + stats in parallel server components, keeps keys server-side, and wraps only the filter bar in a client island.

```jsx
async function Stats() {
  return <p>Stats for Kakade</p>;
}
```

Output:

```text
Stats for Kakade (HTML, no JS).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Default server rendering** | Server unless `"use client"` |
| **Async components** | `await` data directly |
| **No hooks or browser APIs** | State/events go to children |
| **Composition patterns** | Server children in client shells |
| **Benefits for Kakade apps** | Secrets, bundles, SEO, waterfalls |

## The Mental Model

```text
Component for Kakade
  │
  ├── interactive? → "use client" leaf
  │
  └── static/data? → async server
        └── compose: client shell + server children
```

### The 3 things to remember

**1. Async by default:**

```jsx
async function Users() {
  const users = await fetch(url).then((r) => r.json());
}
```

**2. Push boundaries down:**

```jsx
// <ClientTabs><Bio /></ClientTabs>
```

**3. Secrets stay server:**

```jsx
headers: { Authorization: process.env.API_KEY },
```

> **Server by default, client by reason, and compose so islands stay small.**
