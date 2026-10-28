# Projects

The fastest way to make JavaScript stick is to build small things end to end. The chain is **idea → state model → render → events → persist or test**: each project below is deliberately tiny, uses plain DOM APIs, and teaches one interview-relevant idea you can explain in a minute.

```text
Idea
  ↓
State model        arrays / objects / Map
  ↓
Render             textContent + createElement
  ↓
Events             submit / click / drop / observer
  ↓
Persist or test    localStorage / fetch / Vitest
```

---

# 1. Todo App

An array is the source of truth; the DOM is a projection. Re-render from data so add, toggle and delete cannot drift apart.

```javascript
const KEY = "todos:kakade";
let todos = JSON.parse(localStorage.getItem(KEY) ?? "[]");
const list = document.querySelector("#list");
const save = () => localStorage.setItem(KEY, JSON.stringify(todos));

function render() {
  list.replaceChildren(...todos.map((todo) => {
    const li = document.createElement("li");
    li.textContent = todo.text;          // never innerHTML
    li.dataset.id = todo.id;
    return li;
  }));
}

document.querySelector("#form").addEventListener("submit", (event) => {
  event.preventDefault();
  const text = new FormData(event.currentTarget).get("text").trim();
  if (!text) return;
  todos = [...todos, { id: crypto.randomUUID(), text, done: false }];
  save();
  event.currentTarget.reset();
  render();
});

list.addEventListener("click", (event) => {
  const id = event.target.closest("li")?.dataset.id;
  todos = todos.map((t) => (t.id === id ? { ...t, done: !t.done } : t));
  save();
  render();
});
```

Output:

```text
Type "Revise closures" → a new <li> appears.
Refresh → the item persists (localStorage). Click it → state toggles.
```

Teaches state → render, `FormData`, persistence and safe text. Trap: storing DOM nodes, or `innerHTML` on todo text.

---

# 2. Weather Dashboard

A dashboard is mostly **async state**: loading, success, error, stale. `AbortController` stops a slow earlier request from overwriting a newer one.

```javascript
let controller;

async function loadWeather(city) {
  controller?.abort();
  controller = new AbortController();

  try {
    const url = `https://api.open-meteo.com/v1/forecast?latitude=${city.lat}&longitude=${city.lon}&current=temperature_2m`;
    const response = await fetch(url, { signal: controller.signal });
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    const data = await response.json();
    out.textContent = `${city.name}: ${data.current.temperature_2m}°C`;
  } catch (error) {
    if (error.name !== "AbortError") out.textContent = "Could not load weather";
  }
}
```

Output:

```text
Pune: 31.2°C
(selecting another city aborts the first request — no stale overwrite)
```

Teaches `response.ok`, `try/catch` and abort-on-supersede. Trap: `fetch` resolves on 404; always check `ok`.

---

# 3. Quiz App

A quiz is a small state machine: `index`, `score`, `locked`. Everything renders from those three values, so the UI cannot desync.

```javascript
const questions = [
  { q: "What does === compare?", options: ["Value only", "Type and value"], answer: 1 },
  { q: "Which is block-scoped?", options: ["var", "let"], answer: 1 },
];
let index = 0;
let score = 0;
let locked = false;

function render() {
  const item = questions[index];
  question.textContent = `${index + 1}. ${item.q}`;
  options.replaceChildren(...item.options.map((label, i) => {
    const button = document.createElement("button");
    button.textContent = label;
    button.disabled = locked;
    button.onclick = () => choose(i);
    return button;
  }));
  progress.textContent = `Score ${score}/${questions.length}`;
}

function choose(i) {
  if (locked) return;
  locked = true;
  if (i === questions[index].answer) score += 1;
  render();
  setTimeout(() => {
    index += 1;
    locked = false;
    if (index < questions.length) render();
    else question.textContent = `Done — ${score}/${questions.length}`;
  }, 700);
}
```

Output:

```text
1. What does === compare?  [Value only] [Type and value]
After a click: Score 1/2, buttons disabled, next question after 700ms.
```

Teaches state machines, disabled transitions and derived UI. Trap: hand-patching the DOM and forgetting to reset `locked`.

---

# 4. Expense Tracker

Money is where floating point bites. Store **integer paise**, sum with `reduce`, and format only at the edge.

```javascript
const expenses = [
  { category: "books", amount: 49900 },
  { category: "travel", amount: 120000 },
  { category: "books", amount: 25000 },
];

const total = (items) => items.reduce((sum, item) => sum + item.amount, 0);
const byCategory = (items) => items.reduce((acc, item) => {
  acc[item.category] = (acc[item.category] ?? 0) + item.amount;
  return acc;
}, {});

const inr = new Intl.NumberFormat("en-IN", { style: "currency", currency: "INR" });

console.log(inr.format(total(expenses) / 100));
console.log(byCategory(expenses));
```

Output:

```text
₹1,949.00
{ books: 74900, travel: 120000 }
```

Teaches `reduce` with an initial value, `Intl` and integer money. Trap: summing floats or rounding each line instead of the total.

---

# 5. Kanban Board

Drag-and-drop looks visual, but the model is three arrays. On `drop`, move an id between arrays and re-render; never move DOM nodes as the source of truth.

```javascript
const board = {
  todo: [{ id: "t1", text: "Write tests" }],
  doing: [{ id: "t2", text: "Ship API" }],
  done: [{ id: "t3", text: "Setup CI" }],
};

function render() {
  for (const [column, cards] of Object.entries(board)) {
    const list = document.querySelector(`[data-column="${column}"]`);
    list.replaceChildren(...cards.map((card) => {
      const li = document.createElement("li");
      li.textContent = card.text;
      li.draggable = true;
      li.dataset.id = card.id;
      return li;
    }));
  }
}

document.addEventListener("dragstart", (event) => {
  event.dataTransfer.setData("text/plain", event.target.dataset.id);
});
document.addEventListener("dragover", (event) => event.preventDefault());
document.addEventListener("drop", (event) => {
  const id = event.dataTransfer.getData("text/plain");
  const target = event.target.closest("[data-column]");
  if (!target) return;
  const from = Object.values(board).find((cards) => cards.some((c) => c.id === id));
  const card = from?.find((c) => c.id === id);
  if (!card) return;
  from.splice(from.indexOf(card), 1);
  board[target.dataset.column].push(card);
  render();
});
```

Output:

```text
Drag "Write tests" from Todo to Doing:
Todo loses the card, Doing gains it, both columns re-render from the arrays.
```

Teaches model-first UI, event delegation and `dataTransfer`. Trap: without `dragover` preventDefault, `drop` never fires.

---

# 6. Chat UI

A chat window is a WebSocket plus three details: optimistic rendering, safe text and scroll position. Treat the socket as a stateful resource.

```javascript
const socket = new WebSocket("wss://chat.example.com/room/kakade");
const list = document.querySelector("#messages");

socket.addEventListener("message", (event) => {
  const { user, text } = JSON.parse(event.data);
  const li = document.createElement("li");
  li.textContent = `${user}: ${text}`;   // no HTML parsing
  list.append(li);
  list.scrollTop = list.scrollHeight;
});

function send(text) {
  socket.send(JSON.stringify({ user: "Kakade", text, at: Date.now() }));
  const li = document.createElement("li");
  li.textContent = `Kakade: ${text}`;    // optimistic echo
  list.append(li);
}
```

Output:

```text
"Deploy is green" appears immediately.
The server echo reaches other clients; incoming text renders literally.
```

Teaches WebSocket lifecycle, optimistic UI and safe rendering. Trap: `innerHTML` for messages, or reconnecting without backoff.

---

# 7. Infinite Scroll Gallery

Infinite scroll is pagination triggered by a sentinel. `IntersectionObserver` avoids scroll handlers; a `loading` flag and a `seen` set prevent duplicate pages.

```javascript
let page = 1;
let loading = false;
const seen = new Set();
const grid = document.querySelector("#grid");

const observer = new IntersectionObserver(async ([entry]) => {
  if (!entry.isIntersecting || loading) return;
  loading = true;
  try {
    const response = await fetch(`/api/photos?page=${page}&limit=12`);
    const photos = await response.json();
    for (const photo of photos) {
      if (seen.has(photo.id)) continue;
      seen.add(photo.id);
      const img = new Image();
      img.src = photo.url;
      img.loading = "lazy";
      grid.append(img);
    }
    page += 1;
  } finally {
    loading = false;
  }
}, { rootMargin: "300px" });

observer.observe(document.querySelector("#sentinel"));
```

Output:

```text
Page 1 loads when the sentinel is within 300px of the viewport.
Scrolling appends page 2, then 3; duplicate ids are skipped.
```

Teaches `IntersectionObserver`, pagination and idempotent rendering. Trap: the observer fires repeatedly — the `loading` guard is essential.

---

# 8. Form Validation Library

A validation library should be **pure functions** returning error strings, with no DOM. That makes it easy to unit test and reuse on the server.

```javascript
const required = (message) => (value) =>
  value.trim() ? "" : message;

const email = (message) => (value) =>
  /^[^@\s]+@[^@\s]+\.[^@\s]+$/.test(value) ? "" : message;

const validate = (rules) => (values) =>
  Object.fromEntries(
    Object.entries(rules).flatMap(([field, checks]) => {
      const error = checks.map((check) => check(values[field] ?? "")).find(Boolean);
      return error ? [[field, error]] : [];
    })
  );

const checkSignup = validate({
  name: [required("Name is required")],
  email: [required("Email is required"), email("Enter a valid email")],
});

console.log(checkSignup({ name: "Kakade", email: "kakade@" }));
```

Output:

```text
{ email: 'Enter a valid email' }
```

Teaches higher-order functions, composition and error objects. Trap: client validation is UX, not security — validate on the server too.

---

# 9. Mini Router

A hash router maps `location.hash` to a render function: route table, parameter extraction, a 404 branch and a `hashchange` listener.

```javascript
const routes = [
  { pattern: /^\/users\/(\d+)$/, render: (id) => `User #${id} — Rohan` },
  { pattern: /^\/about$/, render: () => "About Kakade" },
  { pattern: /^\/$/, render: () => "Home" },
];

function router() {
  const path = decodeURIComponent(location.hash.slice(1)) || "/";
  const match = routes
    .map((route) => ({ route, result: path.match(route.pattern) }))
    .find(({ result }) => result);

  document.querySelector("#app").textContent = match
    ? match.route.render(...match.result.slice(1))
    : "404 — not found";
}

window.addEventListener("hashchange", router);
router();
```

Output:

```text
#/            → Home
#/about       → About Kakade
#/users/42    → User #42 — Rohan
#/nope        → 404 — not found
```

Teaches routing as data, params and client-side navigation. Trap: the History API needs a server rewrite; hash routing does not.

---

# 10. Testing Project

Test the Todo **logic** separately from the DOM. Pure functions are trivial to test; a DOM test covers wiring; a mocked `fetch` covers the network edge.

```javascript
// todos.js
export const addTodo = (todos, text) => [
  ...todos,
  { id: crypto.randomUUID(), text, done: false },
];
export const toggleTodo = (todos, id) =>
  todos.map((todo) => (todo.id === id ? { ...todo, done: !todo.done } : todo));
export const remaining = (todos) => todos.filter((todo) => !todo.done).length;

// todos.test.js
it("adds without mutating the input", () => {
  const before = [];
  const after = addTodo(before, "Revise closures");
  expect(before).toHaveLength(0);
  expect(after[0].text).toBe("Revise closures");
  expect(remaining(toggleTodo(after, after[0].id))).toBe(0);
});
```

Output:

```text
 ✓ todos.test.js (1 test)
   ✓ adds without mutating the input
```

Teaches pure logic first, DOM second, mocks only at boundaries. Trap: mocking the function under test, or asserting internals.

---

# Real-World Uses

- **Interviews** — describe a project as state → render → events → test.
- **Portfolio** — three polished small apps beat ten half-finished clones.

---

# Quick Revision

| Project | What it teaches |
| --- | --- |
| **Todo app** | Array state, `FormData`, storage, safe text |
| **Weather dashboard** | `fetch`, loading/error, `AbortController` |
| **Quiz app** | State machine, disabled transitions |
| **Expense tracker** | `reduce`, integer paise, `Intl` |
| **Kanban board** | Model-first drag/drop, delegation |
| **Chat UI** | WebSocket lifecycle, optimistic UI |
| **Infinite scroll gallery** | `IntersectionObserver`, pagination, dedupe |
| **Form validation library** | Pure validators, composition |
| **Mini router** | Route table, params, `hashchange` |
| **Testing project** | Pure tests, DOM test, mocks |

## The Mental Model

```text
        IDEA
          ↓
     STATE MODEL
   array · object · Map
          ↓
       RENDER
   textContent + DOM
          ↓
       EVENTS
  submit · click · observe
          ↓
    PERSIST ←→ TEST
          ↓
      EXPLAIN IT
```

### The 3 things to remember

**1. State is the source of truth; the DOM is a projection.**

```javascript
todos = [...todos, newTodo];
render(); // rebuild from data, do not patch five places by hand
```

**2. Never render user text with `innerHTML`.**

```javascript
li.textContent = todo.text; // safe
```

**3. Test the logic, then the wiring, then the network.**

```javascript
expect(remaining(toggleTodo(list, id))).toBe(1);
document.querySelector("#add").click();
vi.fn().mockResolvedValue({ ok: true });
```

> **Build small, model state explicitly, render with safe DOM APIs, handle async and errors, and test logic before pixels.**
