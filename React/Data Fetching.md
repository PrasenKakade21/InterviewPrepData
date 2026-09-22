# Data Fetching

Data fetching clicks as: `event or effect starts request` → `loading shows` → `data or error renders`. The fetch itself is easy; managing timing, cancellation, and stale responses is the real topic.

The core idea:

```text
Trigger (mount / click / page change)
    ↓
loading state → fetch → data / error
    ↓
cleanup cancels outdated work
```

Prefer fetching in events for user actions, effects for mount-driven loads, and a library for cache + dedupe.

---

# 1. fetch in React, axios Basics, and Fetching in Events vs Effects

`fetch in React` uses the built-in promise API with `useEffect` for load-on-mount. `axios basics` adds a client with base URL, JSON by default, and interceptors. `Fetching in events vs effects`: buttons fetch in handlers; initial data loads in effects.

```jsx
import { useEffect, useState } from "react";

function User() {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/users/1")
      .then((r) => r.json())
      .then(setUser);
  }, []);
  return <p>{user ? user.name : "…"} friend of Kakade</p>;
}
```

Output:

```text
… then Leanne Graham friend of Kakade after load.
```

Event-driven version (axios shape):

```js
// axios.get("/api/users/kakade") → response.data (JSON parsed).
// fetch("/api/users/kakade").then(r => r.json()) → same, manual parse.
```

```jsx
import { useState } from "react";

function Searcher() {
  const [q, setQ] = useState("Kakade");
  function search() {
    fetch(`/api/users?name=${q}`).then((r) => r.json());
  }
  return <button onClick={search}>Search {q}</button>;
}
```

Output:

```text
Click Search Kakade → request fires once (event, not effect).
```

Rule: effects for "load when shown", events for "load when clicked". Never fetch during render.

---

# 2. Loading States, Error States, and Pagination Pattern

`Loading states` (`idle → loading → done`) disable buttons and show skeletons. `Error states` store the message and offer retry. `Pagination pattern` tracks `page`, appends or replaces items, and disables next at the end.

```jsx
import { useEffect, useState } from "react";

function Users() {
  const [users, setUsers] = useState([]);
  const [status, setStatus] = useState("idle");
  const [error, setError] = useState(null);

  useEffect(() => {
    setStatus("loading");
    fetch("https://jsonplaceholder.typicode.com/users")
      .then((r) => {
        if (!r.ok) throw new Error("Bad response");
        return r.json();
      })
      .then((d) => {
        setUsers(d);
        setStatus("done");
      })
      .catch((e) => {
        setError(e.message);
        setStatus("error");
      });
  }, []);

  if (status === "loading") return <p>Loading Kakade…</p>;
  if (status === "error") return <p>Failed: {error}</p>;
  return <p>{users.length} users found</p>;
}
```

Output:

```text
Loading Kakade… → 10 users found (or Failed: … on error).
```

Paginated shape:

```jsx
import { useState } from "react";

function Paged() {
  const [page, setPage] = useState(1);
  return <button onClick={() => setPage(page + 1)}>Page {page} for Kakade</button>;
}
```

Output:

```text
Page 1 for Kakade → Page 2 (each page triggers a new fetch).
```

Convention: keep `{ data, status, error }` together; cursor pagination (`?after=id`) beats offset for live lists.

---

# 3. AbortController, Race Conditions, and Caching Basics

`AbortController` cancels in-flight requests on unmount or new searches. `Race conditions` happen when a slow older response overwrites a fresh one — abort or ignore stale replies. `Caching basics` reuses prior results by key instead of refetching.

```jsx
import { useEffect, useState } from "react";

function Search({ query }) {
  const [result, setResult] = useState(null);
  useEffect(() => {
    const ctrl = new AbortController();
    fetch(`/api/users?name=${query}`, { signal: ctrl.signal })
      .then((r) => r.json())
      .then((d) => setResult(d))
      .catch((e) => {
        if (e.name !== "AbortError") throw e;
      });
    return () => ctrl.abort();
  }, [query]);
  return <p>Result for {query || "Kakade"}</p>;
}
```

Output:

```text
Typing fast: old requests abort, only latest query renders.
```

Think:

```text
query=k → request 1
query=ka → abort 1, request 2
    ↓
only response 2 updates state
```

Cache sketch:

```js
// const cache = new Map();
// key = `/users?page=${page}`; if (cache.has(key)) return cache.get(key).
```

Interview trap: missing cleanup causes "setState on unmounted component" warnings and stale UI.

---

# 4. TanStack Query Basics

`TanStack Query basics`: `useQuery({ queryKey, queryFn })` gives `{ data, isLoading, error }` plus caching, retries, and refetch — replacing most hand-rolled fetch effects.

```jsx
import { useQuery } from "@tanstack/react-query";

function Profile() {
  const { data, isLoading, error } = useQuery({
    queryKey: ["user", "kakade"],
    queryFn: () =>
      fetch("https://jsonplaceholder.typicode.com/users/1").then((r) => r.json()),
  });
  if (isLoading) return <p>Loading Kakade…</p>;
  if (error) return <p>Failed</p>;
  return <p>{data.name}</p>;
}
```

Output:

```text
Loading Kakade… → Leanne Graham (cached on revisit).
```

Think:

```text
queryKey ["user", "kakade"] → cache slot
    ├── fresh? → return cached
    └── stale? → background refetch
```

Use Query for server state (cacheable, shared, refetchable); keep `useState` for client state (toggles, forms).

---

# Real-World Uses

A directory page combines all of it: event-driven search with abort, `status/error` UI, page buttons appending results, and TanStack Query caching Kakade's profile so back-navigation is instant.

```jsx
import { useState } from "react";

function Directory() {
  const [page, setPage] = useState(1);
  return <button onClick={() => setPage(page + 1)}>More for Kakade: {page}</button>;
}
```

Output:

```text
More for Kakade: 1 → 2 (next page loads).
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **fetch in React** | Effect + `.json()` for mount loads |
| **axios basics** | Client, parsed data, interceptors |
| **Loading states** | `idle → loading → done` + skeleton |
| **Error states** | Store message + retry button |
| **AbortController** | Cancel stale/unmounted requests |
| **Race conditions** | Slow reply overwrites fresh; abort or guard |
| **Fetching in events vs effects** | Clicks in handlers, initial in effects |
| **Caching basics** | Reuse by key (`queryKey` / Map) |
| **TanStack Query basics** | `useQuery` cache + retry + refetch |
| **Pagination pattern** | `page` state + append, disable at end |

## The Mental Model

```text
Trigger (mount / click / page)
  │
  ├── loading + error states → skeleton / retry
  ├── AbortController → cancel stale
  │       │
  │       └── latest response only → setState
  │
  └── TanStack Query → queryKey cache + refetch
```

### The 3 things to remember

**1. Mount load with states:**

```jsx
const [status, setStatus] = useState("idle");
```

**2. Cancel stale work:**

```jsx
const ctrl = new AbortController();
return () => ctrl.abort();
```

**3. Cache server state:**

```jsx
useQuery({ queryKey: ["user", "kakade"], queryFn: fetchUser });
```

> **Show loading, handle errors, cancel stale requests, and cache what the server owns.**
