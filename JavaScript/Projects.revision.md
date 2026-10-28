# Projects — Revision Notes (Beginner)

> One-line mental model: state is the source of truth, the DOM is a projection, and every project is event → update state → render → test.

## Core recall

- **Todo app**: array of `{ id, text, done }`; `render()` rebuilds `<li>` with `textContent`; `FormData` for input; persist JSON to localStorage; click delegation toggles by `data-id`.
- **Weather dashboard**: states idle/loading/success/error; check `response.ok`; abort the previous request with `AbortController`; ignore `AbortError`; show a real error message.
- **Quiz app**: state machine of `index`, `score`, `locked`; disable buttons after a choice; reset the lock before the next question; derive progress from state.
- **Expense tracker**: store integer paise; `reduce` with an initial `0`; group with `reduce` into an object; format only for display with `Intl.NumberFormat`.
- **Kanban board**: columns are arrays; `dragstart` stores the id, `drop` reads the target column; `dragover` must `preventDefault()`; re-render from arrays.
- **Chat UI**: WebSocket `message` → parse JSON → create `<li>` → `textContent` → append → scroll; optimistic send adds `.pending`; reconnect with backoff.
- **Infinite scroll gallery**: `IntersectionObserver` on a sentinel; `rootMargin` preloads; `loading` guard before `await`; `seen` Set dedupes; disconnect when done.
- **Form validation library**: validators are `(value) => errorString`; compose per field; return an object of errors; pure and reusable on the server.
- **Mini router**: route table with patterns; split and compare segments; collect `:params` with `decodeURIComponent`; `hashchange` listener; `null` → 404.
- **Testing project**: test pure logic first, then DOM wiring, then mock `fetch` at the boundary; edge cases are empty, error and duplicate paths.

## Must-remember code

```javascript
// state → render → persist
todos = [...todos, { id: crypto.randomUUID(), text, done: false }];
localStorage.setItem(KEY, JSON.stringify(todos));
render();
```

```javascript
// safe rendering, no HTML parsing
const li = document.createElement("li");
li.textContent = `${user}: ${text}`;
list.append(li);
```

```javascript
// async request with abort + ok check
controller?.abort();
controller = new AbortController();
const response = await fetch(url, { signal: controller.signal });
if (!response.ok) throw new Error(`HTTP ${response.status}`);
```

```javascript
// money in integer paise
const total = expenses.reduce((sum, item) => sum + item.amount, 0);
new Intl.NumberFormat("en-IN", { style: "currency", currency: "INR" }).format(total / 100);
```

## Interview traps

- Rendering user text with `innerHTML` turns a todo or chat message into stored XSS; use `textContent`.
- `fetch` resolves on 404/500; without `response.ok` the error path never runs.
- A slow earlier request can overwrite a newer one; abort it or tag responses with an id.
- `setTimeout` and observers can fire while a request is in flight; guard with a synchronous flag.
- Summing rupee floats (`0.1 + 0.2`) gives `0.30000000000000004`; store paise.
- `reduce` without an initial value throws on an empty array.
- Missing `dragover` preventDefault means the `drop` event never fires.
- `IntersectionObserver` fires immediately on `observe()` and can repeat while visible.
- Recreating an observer or listener on every render leaks handlers unless you disconnect the old one.
- Client-side validation is UX; the server must validate and authorize again.

## 3 things to remember

1. **Event → update state → render; never read state back from the DOM.**
   ```javascript
   todos = [...todos, newTodo];
   render();
   ```

2. **Create nodes and set `textContent`; never concatenate data into HTML.**
   ```javascript
   li.textContent = message.text;
   list.append(li);
   ```

3. **Test logic first, wiring second, boundaries last.**
   ```javascript
   expect(remaining(toggleTodo(list, id))).toBe(1);
   ```
