# Promises — Revision Notes (Beginner)

> One-line mental model: a promise is a one-shot async value — pending, then fulfilled or rejected, consumed with `then` / `catch` / `finally`.

## Core recall

- **Callback problems**: nesting, repeated error checks at every level, no returned value, inversion of control.
- **Promise states**: pending → fulfilled (value) or rejected (reason). Settles once; later calls are ignored. Executor runs synchronously, handlers run in microtasks.
- **then / catch / finally**: `catch(fn)` = `then(undefined, fn)` and recovers the chain; `finally` runs on both paths and passes the result through.
- **Chaining**: every `then` returns a new promise. Return a value to transform; return nothing → `undefined`; throw → rejection.
- **Returning promises**: a returned promise is flattened/adopted. `async` functions always return a promise.
- **Promise.all**: waits for all, input order, **fail-fast** on the first rejection. `all([])` → `[]`.
- **Promise.allSettled**: waits for all, never rejects per item; `{ status, value }` or `{ status, reason }`.
- **Promise.race**: first **settled** wins, fulfilment or rejection; `race([])` stays pending forever.
- **Promise.any**: first **fulfilled** wins; all reject → `AggregateError` with `.errors`; `any([])` rejects immediately.
- **Error propagation**: throw rejects; skipped `then`s until `catch`; catch can recover or re-throw. Unhandled rejections crash Node / fire `unhandledrejection`.
- **Creating promises**: `new Promise((resolve, reject) => ...)` to wrap callback APIs; `Promise.resolve` for values; never use an `async` executor.
- **Common traps**: missing `return`, `forEach` doesn't await, `all` vs `allSettled`, `race` vs `any`.

## Must-remember code

```javascript
// chain, recover, clean up
fetch("/api/user")
  .then((res) => res.json())
  .then((user) => console.log(user.name))
  .catch((err) => console.error(err.message))
  .finally(() => hideSpinner());
```

```javascript
// parallel work, all required
const [user, orders] = await Promise.all([getUser(), getOrders()]);
```

```javascript
// partial success
const results = await Promise.allSettled(tasks);
const failed = results.filter((r) => r.status === "rejected");
```

```javascript
// timeout
const timeout = new Promise((_, reject) =>
  setTimeout(() => reject(new Error("Timeout")), 5000)
);
const data = await Promise.race([fetchData(), timeout]);
```

## Interview traps

- Forgetting `return` in a `then` handler passes `undefined` to the next step.
- `forEach(async ...)` does not await; use `for...of` or `Promise.all(map(...))`.
- `Promise.all` rejects on the first failure and discards the rest.
- `Promise.allSettled` never rejects for item failures — check each `status`.
- `Promise.race` rejects if the fastest input rejects; `Promise.any` ignores rejections.
- `Promise.race([])` stays pending forever; `Promise.any([])` rejects with `AggregateError`.
- `new Promise(async (resolve) => ...)` is an anti-pattern — errors are not wired to `reject`.
- `resolve` twice is ignored; the first settlement wins.
- A `.catch` recovers the chain: later `.then`s run with the returned value.
- An unhandled rejection is not the same as a synchronous `throw`; it is reported asynchronously.

## 3 things to remember

1. **Promises settle once.**
   ```javascript
   const p = new Promise((resolve) => {
     resolve("first");
     resolve("second"); // ignored
   });
   ```

2. **Return to keep the chain.**
   ```javascript
   Promise.resolve(1)
     .then((n) => n + 1)   // return!
     .then(console.log);   // 2
   ```

3. **Pick the right combinator.**
   ```javascript
   Promise.all([a, b]);         // all or fail
   Promise.allSettled([a, b]);  // always fulfil
   Promise.race([a, timeout]);  // first settled
   Promise.any([a, b]);         // first success
   ```

> **A promise is a one-shot async value: chain with `then`, recover with `catch`, clean up with `finally`, and choose the combinator by failure policy.**
