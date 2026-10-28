# Promises

A promise is a **placeholder for a value that arrives later**. The chain is **callback → promise state → `.then` chain → combinators (`all`, `allSettled`, `race`, `any`)**. Promises turn async results into first-class values you can chain, combine, and `await`.

```text
Async work
    ↓
callback                 result passed to a function
    ↓
Promise                  pending → fulfilled | rejected
    ↓
.then / .catch / .finally
    ↓
combinators              all / allSettled / race / any
```

---

# 1. Why Promises

### Callback problems

With callbacks, the result is pushed into a function instead of returned. Each extra step adds a nesting level, every level needs its own error branch, and the caller has no value to store or combine. This is "callback hell" and **inversion of control**: you hand your continuation to code you do not own.

```javascript
function getUser(id, callback) {
  setTimeout(() => callback(null, { id, name: "Kakade" }), 10);
}

function getOrders(user, callback) {
  setTimeout(() => callback(null, ["order-1"]), 10);
}

getUser(1, (err, user) => {
  if (err) return console.log(err);
  getOrders(user, (err, orders) => {
    if (err) return console.log(err);
    console.log(user.name, orders);
  });
});
```

Output:

```text
Kakade [ 'order-1' ]
```

Every step repeats the `if (err)` check, and returning from the outer function does not cancel the work. A promise version returns the user, then the orders, with one `.catch` at the end.

### Promise states

A promise has three states: **pending**, **fulfilled** with a value, or **rejected** with a reason. Once settled, it can never change again — extra `resolve` or `reject` calls are ignored. The executor function runs **synchronously**, but `.then` callbacks always run later, in a microtask.

```javascript
const p = new Promise((resolve) => {
  console.log("executor runs");
  resolve("done");
});

p.then((value) => console.log("then:", value));
console.log("after then");
```

Output:

```text
executor runs
after then
then: done
```

Think:

```text
pending → resolve(value) → fulfilled
        → reject(error)  → rejected
              ↓
        settled forever
```

`new Promise` runs its executor immediately, which is why `executor runs` appears before `after then`. If the executor throws, the promise rejects automatically. A promise that never settles stays pending — the classic "hung request" bug.

---

# 2. Consuming Promises

### then, catch and finally

`then(onFulfilled, onRejected)` registers handlers and returns a **new** promise. `catch(fn)` is shorthand for `then(undefined, fn)`. `finally(fn)` runs on either outcome, receives no value, and passes the original result through.

```javascript
Promise.resolve("data")
  .finally(() => console.log("cleanup"))
  .then((value) => console.log("value:", value));
```

Output:

```text
cleanup
value: data
```

```javascript
Promise.reject(new Error("bad"))
  .catch((err) => console.log("caught:", err.message))
  .finally(() => console.log("done"));
```

Output:

```text
caught: bad
done
```

Use `finally` for cleanup (hiding a spinner, clearing a timer) because it does not change the value or the rejection. `catch` **recovers** the chain: whatever it returns becomes a fulfilled value. If it throws or returns a rejected promise, the chain stays rejected.

### Chaining

Each `.then` returns a new promise, so you can build a pipeline. The value returned by a handler becomes the next handler's input. If a handler returns nothing, the next step receives `undefined`.

```javascript
Promise.resolve(2)
  .then((n) => n * 3)
  .then((n) => {
    console.log("value:", n);
    return undefined;
  })
  .then((v) => console.log("next:", v));

Promise.reject(new Error("fail"))
  .then(() => console.log("skipped"))
  .catch((e) => console.log("caught:", e.message));
```

Output:

```text
value: 6
caught: fail
next: undefined
```

Notice: a rejection **skips every `then` without a rejection handler** until a `catch` is found. This is why one `.catch` at the end can replace error checks at every step. Forgetting `return` inside a handler is the most common chaining bug — the next step silently receives `undefined`.

### Returning promises

If a handler returns a promise, the chain **flattens** it: the next `.then` waits for that promise and receives its value, not the promise itself. The same happens for any "thenable" object with a `then` method.

```javascript
Promise.resolve(1)
  .then((n) => Promise.resolve(n + 1))
  .then((n) => console.log(n));
```

Output:

```text
2
```

This is exactly what `async` / `await` does: an `async` function always returns a promise, and `return value` resolves it while `throw` rejects it. You rarely need `new Promise` when you already have a promise-returning function — just return it.

---

# 3. Combinators

### Promise.all

`Promise.all(iterable)` waits for **every** promise and resolves with an array of values **in input order**, regardless of completion order. If any promise rejects, the whole thing rejects immediately with that reason (**fail-fast**); the other results are ignored.

```javascript
const wait = (ms, value) =>
  new Promise((resolve) => setTimeout(() => resolve(value), ms));

Promise.all([wait(30, "Kakade"), wait(10, "Asha")])
  .then((names) => console.log(names));
```

Output:

```text
[ 'Kakade', 'Asha' ]
```

```javascript
Promise.all([
  new Promise(() => {}), // never settles
  Promise.reject(new Error("boom")),
]).catch((err) => console.log("rejected:", err.message));
```

Output:

```text
rejected: boom
```

`Promise.all([])` resolves to `[]` immediately. Non-promise values are treated as resolved. Use it when **all** results are required; use `allSettled` for partial success.

### Promise.allSettled

`Promise.allSettled` waits for every promise and **never rejects** for individual failures. It resolves with one object per input: `{ status: "fulfilled", value }` or `{ status: "rejected", reason }`.

```javascript
Promise.allSettled([
  Promise.resolve("ok"),
  Promise.reject(new Error("bad")),
]).then((results) => {
  console.log(results.map((r) => r.status));
  console.log(results[1].reason.message);
});
```

Output:

```text
[ 'fulfilled', 'rejected' ]
bad
```

Use it for batch jobs, uploads, or independent widgets where you can report "3 succeeded, 1 failed" instead of losing everything.

### Promise.race

`Promise.race` settles as soon as the **first** input settles — whether it fulfils **or rejects**. The remaining promises keep running but their results are ignored.

```javascript
const slow = new Promise((resolve) => setTimeout(() => resolve("slow"), 50));
const fast = new Promise((resolve) => setTimeout(() => resolve("fast"), 10));

Promise.race([slow, fast]).then((v) => console.log("winner:", v));
```

Output:

```text
winner: fast
```

`race` is the basis of timeouts: race the request against a promise that rejects after N milliseconds. A rejection in the array wins the race too — `Promise.race([slow, Promise.reject(err)])` rejects immediately. `Promise.race([])` stays **pending forever**, and a real bug if the array can be empty.

### Promise.any

`Promise.any` resolves with the **first fulfilled** value and ignores rejections. Only if **every** promise rejects does it reject, with an `AggregateError` whose `errors` array holds each reason. It mirrors `all`.

```javascript
Promise.any([
  Promise.reject(new Error("a")),
  new Promise((resolve) => setTimeout(() => resolve("winner"), 10)),
]).then((v) => console.log(v));
```

Output:

```text
winner
```

```javascript
Promise.any([
  Promise.reject(new Error("a")),
  Promise.reject(new Error("b")),
]).catch((e) => console.log(e.name, e.errors.map((x) => x.message)));
```

Output:

```text
AggregateError [ 'a', 'b' ]
```

Use `any` for redundant sources where the first success is enough. `Promise.any([])` rejects immediately with an empty `AggregateError`.

---

# 4. Errors and Creation

### Error propagation

A `throw` inside a `.then` (or `async`) handler rejects the returned promise. The rejection travels down the chain, skipping fulfilled handlers until a `catch`. A `catch` that returns a normal value **recovers** the chain; a `catch` that throws or returns a rejected promise keeps it rejected.

```javascript
Promise.resolve("start")
  .then(() => {
    throw new Error("step failed");
  })
  .then(() => console.log("skipped"))
  .catch((err) => {
    console.log("caught:", err.message);
    return "recovered";
  })
  .then((v) => console.log("after:", v));
```

Output:

```text
caught: step failed
after: recovered
```

To add context without losing the original, throw a new error with `{ cause: err }`. A rejected promise with no handler triggers an **unhandled rejection**; Node terminates by default, browsers fire `unhandledrejection`. Always end a chain with a handler or `await` it in `try/catch`.

### Creating promises

Use `new Promise((resolve, reject) => { ... })` only to wrap a callback-style or event-based API. The executor runs synchronously; call `resolve(value)` once on success and `reject(error)` once on failure. For an existing value, use `Promise.resolve(value)`; for async logic, an `async` function is simpler.

```javascript
const loadUser = () =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({ id: 1, name: "Kakade" });
    }, 10);
  });

loadUser().then((user) => console.log(user.name));
```

Output:

```text
Kakade
```

Bad: an async executor loses errors.

```javascript
new Promise(async (resolve) => {
  const user = await loadUser(); // a rejection here is not wired to reject
  resolve(user);
});
```

Good: return the promise, or use an async function.

```javascript
async function getUser() {
  return loadUser();
}
```

`resolve` with a promise adopts its state, so `resolve(loadUser())` is valid. But an `async` executor is an anti-pattern because its returned promise is ignored.

### Common traps

**1. Forgetting to return.** A handler without `return` passes `undefined` down the chain:

```javascript
Promise.resolve(1)
  .then((n) => { n + 1; })
  .then((n) => console.log("got:", n));
```

Output:

```text
got: undefined
```

**2. `forEach` does not await.** Async callbacks are started, but `forEach` returns immediately and ignores their promises:

```javascript
[1, 2, 3].forEach(async (n) => {
  await Promise.resolve();
  console.log("item", n);
});
console.log("after forEach");
```

Output:

```text
after forEach
item 1
item 2
item 3
```

Use `for...of` with `await`, or `await Promise.all(items.map(async ...))` when the work can run in parallel.

---

# Real-World Uses

- **HTTP requests** — `fetch()` returns a promise; chains handle parsing and errors.
- **Parallel loading** — `Promise.all` for independent API calls that are all required.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Callback problems** | Nesting, repeated error checks, no return value, inverted control |
| **Promise states** | pending → fulfilled / rejected; settled once, forever |
| **then, catch, finally** | `catch` recovers; `finally` passes the result through |
| **Chaining** | Each `then` returns a new promise; no `return` → `undefined` |
| **Returning promises** | Returned promises flatten; thenables are adopted |
| **Promise.all** | Waits for all, input order, rejects on first failure |
| **Promise.allSettled** | Waits for all, never rejects, gives `status` per item |
| **Promise.race** | First **settled** wins — fulfilment or rejection |
| **Promise.any** | First **fulfilled** wins; all fail → `AggregateError` |
| **Error propagation** | Throw rejects; skips `then`s; `catch` recovers or re-throws |
| **Creating promises** | `new Promise` for callback APIs; `Promise.resolve` for values |
| **Common traps** | Missing `return`, `forEach` without await, unhandled rejections |

## The Mental Model

```text
                     PROMISE
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       pending       fulfilled      rejected
          │             │             │
          └─────────────┼─────────────┘
                        ↓
                 .then / .catch / .finally
                        │
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
      all           allSettled        race / any
   all or fail     never rejects    first settled / first ok
```

### The 3 things to remember

**1. A promise is settled once.**

```javascript
const p = new Promise((resolve) => {
  resolve("first");
  resolve("second"); // ignored
});
p.then(console.log); // first
```

**2. Returning a promise flattens the chain.**

```javascript
Promise.resolve(1)
  .then((n) => Promise.resolve(n + 1))
  .then(console.log); // 2
```

**3. `all` fails fast; `allSettled` does not.**

```javascript
const ok = Promise.resolve(1);
const bad = Promise.reject(new Error("x"));

Promise.all([ok, bad]).catch(() => {}); // rejects
Promise.allSettled([ok, bad]);         // fulfils
```

> **A promise is a one-shot async value: chain with `then`, recover with `catch`, clean up with `finally`, and pick the combinator that matches the failure policy.**
