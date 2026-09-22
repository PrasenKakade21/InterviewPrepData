# Testing

Testing clicks as: `render like a user` → `interact` → `assert outcome`. Test behavior (what Kakade sees), not internals (state names, props plumbing).

The core idea:

```text
render(<App />) → screen queries → fireEvent / user-event
    ↓
findBy (async) / getBy (sync) / queryBy (absent)
    ↓
mock fetch + router → stable, isolated
```

React Testing Library + Vitest/Jest covers nearly everything; snapshots only for stable presentational output.

---

# 1. Vitest / Jest Basics, React Testing Library, and Render and Screen Queries

`Vitest / Jest basics` (`describe/it/expect`) structure tests with DOM matchers. `React Testing Library` renders components in jsdom the way users experience them. `Render and screen queries` prefer accessible roles/text over test ids.

```jsx
import { render, screen } from "@testing-library/react";
import { expect, it } from "vitest";

function Hi() {
  return <button>Hi Kakade</button>;
}

it("greets", () => {
  render(<Hi />);
  expect(screen.getByRole("button", { name: "Hi Kakade" })).toBeInTheDocument();
});
```

Output:

```text
PASS greets (button found by role + name).
```

Query by priority: `role` → `label/text` → `placeholder` → `testid` last. `screen.debug()` prints the DOM when stuck.

---

# 2. getBy vs findBy vs queryBy, User Events with fireEvent, and Snapshot Testing

`getBy vs findBy vs queryBy`: `getBy` sync present (throws if missing), `findBy` async present (awaits), `queryBy` absent (returns null). `User events with fireEvent` simulate clicks/typing (prefer `user-event` for realism). `Snapshot testing` (`toMatchSnapshot`) locks stable markup.

```jsx
import { render, screen, fireEvent } from "@testing-library/react";

function Counter() {
  const [n, setN] = React.useState(0);
  return <button onClick={() => setN(n + 1)}>Kakade {n}</button>;
}

it("counts", () => {
  render(<Counter />);
  fireEvent.click(screen.getByRole("button", { name: /kakade/i }));
  expect(screen.getByText("Kakade 1")).toBeInTheDocument();
});
```

Output:

```text
PASS counts (click → Kakade 1).
```

Async shape:

```js
// expect(await screen.findByText("Hi Kakade")).toBeInTheDocument();
// expect(screen.queryByText("Bye")).toBeNull();
```

Rule: one behavior per test; update snapshots intentionally (`-u`), never blindly.

---

# 3. Testing Hooks, Mocking Fetch, Testing Router Components, and Coverage Basics

`Testing hooks` uses `renderHook` + `act` for `useToggle`-style logic. `Mocking fetch` stubs network (`vi.spyOn(global, "fetch")` or msw) with loading → data states. `Testing router components` wraps in `MemoryRouter` with initial entries. `Coverage basics` tracks statements/branches — aim for critical paths, not 100% theater.

```jsx
import { renderHook, act } from "@testing-library/react";

function useToggle(init = false) {
  const [on, setOn] = React.useState(init);
  return [on, () => setOn((v) => !v)];
}

it("toggles for Kakade", () => {
  const { result } = renderHook(() => useToggle(false));
  act(() => result.current[1]());
  expect(result.current[0]).toBe(true);
});
```

Output:

```text
PASS toggles for Kakade (false → true).
```

Router + fetch sketch:

```js
// render(<Profile />, { wrapper: ({ c }) => <MemoryRouter initialEntries={["/u/kakade"]}>{c}</MemoryRouter> });
// vi.spyOn(global, "fetch").mockResolvedValue({ ok: true, json: async () => ({ name: "Kakade" }) });
```

Mock at the network edge, reset between tests, and assert loading → success → error paths.

---

# Real-World Uses

A Kakade profile test mocks fetch, renders the route in MemoryRouter, clicks retry on failure, and asserts the name appears — covering hook, router, and network in one behavior test.

```jsx
it("shows Kakade", async () => {
  render(<p>Hi Kakade</p>);
  expect(await screen.findByText("Hi Kakade")).toBeInTheDocument();
});
```

Output:

```text
PASS shows Kakade.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Vitest / Jest basics** | `describe/it/expect` + DOM matchers |
| **React Testing Library** | Test behavior, not internals |
| **Render and screen queries** | Prefer role/text queries |
| **getBy vs findBy vs queryBy** | Sync / async / absent |
| **User events with fireEvent** | Click/type; user-event preferred |
| **Testing hooks** | `renderHook` + `act` |
| **Mocking fetch** | Stub network, test 3 states |
| **Testing router components** | `MemoryRouter` + entries |
| **Snapshot testing** | Stable markup only |
| **Coverage basics** | Critical paths over 100% |

## The Mental Model

```text
Behavior test
  │
  ├── render in MemoryRouter + mocked fetch
  │
  ├── query: getBy (now) / findBy (soon) / queryBy (never)
  │
  └── interact → assert outcome for Kakade
        └── hooks via renderHook + act
```

### The 3 things to remember

**1. Query by role:**

```jsx
screen.getByRole("button", { name: "Hi Kakade" });
```

**2. Async vs absent:**

```jsx
await screen.findByText("Hi Kakade");
expect(screen.queryByText("Bye")).toBeNull();
```

**3. Hook logic:**

```jsx
const { result } = renderHook(() => useToggle(false));
```

> **Render like a user, query accessibly, and mock the network edge.**
