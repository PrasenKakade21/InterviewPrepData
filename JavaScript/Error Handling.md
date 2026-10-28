# Error Handling

Errors are how JavaScript says "this operation cannot continue". Handling them well means **throwing meaningful objects → catching only what you can fix → cleaning up in `finally` → letting the rest bubble up**. The chain is **detect → throw or reject → catch → recover, wrap, or re-throw**.

```text
Something fails
      ↓
throw new Error("message")        or  Promise.reject(...)
      ↓
try { ... } catch (err) { ... }   or  .catch(...)
      ↓
finally { cleanup }
      ↓
recover | wrap with cause | re-throw
```

---

# 1. try, catch and finally

### try, catch and finally

`try` contains the risky work. If anything throws, control jumps to `catch`, which receives the thrown value. `finally` runs **no matter what** — on success, on throw, and even when `try` returns.

```javascript
function parseAge(value) {
  try {
    const age = Number(value);
    if (Number.isNaN(age)) throw new TypeError("Age must be a number");
    return age;
  } catch (err) {
    console.log("caught:", err.message);
    return null;
  } finally {
    console.log("finally runs");
  }
}

console.log(parseAge("21"));
console.log(parseAge("abc"));
```

Output:

```text
finally runs
21
caught: Age must be a number
finally runs
null
```

Notice the order: `finally` runs **before** the caller receives the return value, so it prints before `21` and before `null`. Do not return from `finally` — it silently overrides the `try`/`catch` result. Keep it for cleanup (closing files, clearing timers, hiding spinners), not for control flow.

### throw

`throw` stops normal execution and sends a value to the nearest `catch`. Throw **`Error` objects**, not strings or plain objects, so you get a message, a name, and a stack trace.

```javascript
function withdraw(balance, amount) {
  if (amount <= 0) throw new RangeError("Amount must be positive");
  if (amount > balance) throw new Error("Insufficient funds");
  return balance - amount;
}

try {
  console.log(withdraw(500, 200));
  console.log(withdraw(500, 900));
  console.log("never reached");
} catch (err) {
  console.log(`${err.name}: ${err.message}`);
}
```

Output:

```text
300
Error: Insufficient funds
```

The second call throws, so `"never reached"` is skipped. Bad: `throw "Insufficient funds"` — the catcher gets a string with no `name`, `message`, or `stack`. Good: `throw new Error("Insufficient funds")`.

A `throw` inside a `catch` replaces the original error. If the original matters, attach it with `cause` (covered below).

---

# 2. Error Types and Custom Errors

### Error types

All built-in errors extend `Error`, so `instanceof Error` is a reliable catch-all. The subclass tells you **what kind of failure** happened.

| Type | Typical cause |
| --- | --- |
| `SyntaxError` | Invalid code or JSON (`JSON.parse("{bad")`) |
| `TypeError` | Wrong type or missing property (`null.name`) |
| `ReferenceError` | Unknown variable (`missingFunction()`) |
| `RangeError` | Value outside the allowed range (`new Array(-1)`) |
| `URIError` | Bad URI encoding (`decodeURIComponent("%")`) |
| `AggregateError` | Several errors together, e.g. `Promise.any` |

```javascript
const cases = [
  () => JSON.parse("{bad"),
  () => null.name,
  () => missingFunction(),
  () => new Array(-1),
];

for (const fn of cases) {
  try {
    fn();
  } catch (err) {
    console.log(err.name, err instanceof Error);
  }
}
```

Output:

```text
SyntaxError true
TypeError true
ReferenceError true
RangeError true
```

`AggregateError` carries an `errors` array and is what `Promise.any` rejects with when every promise fails. Do not test error types with `err.name === "TypeError"` if you can use `instanceof TypeError`; names are strings you can accidentally copy, and subclasses inherit the wrong name unless you set it.

### Custom errors

A custom class gives callers something specific to catch, plus extra fields such as which input failed. Extend `Error`, call `super(message)`, set `this.name`, and add your own properties.

```javascript
class ValidationError extends Error {
  constructor(field, message) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

const err = new ValidationError("email", "Email is required");
console.log(err.name, err.field, err.message);
console.log(err instanceof ValidationError, err instanceof Error);
```

Output:

```text
ValidationError email Email is required
true true
```

Bad: `throw { message: "Invalid email" }` — no stack, no `instanceof`. Good: `throw new ValidationError("email", "Invalid email")`. A caller can then distinguish it from any other object. If you also want to catch it by class across packages, export the class and check `instanceof`, not the name.

### Error cause

`new Error(message, { cause })` keeps the original failure while presenting a clearer message. The cause is available as `err.cause` and is printed by Node's console output.

```javascript
function loadConfig() {
  try {
    JSON.parse("{bad");
  } catch (err) {
    throw new Error("Config is invalid", { cause: err });
  }
}

try {
  loadConfig();
} catch (e) {
  console.log(e.message);
  console.log(e.cause.name);
}
```

Output:

```text
Config is invalid
SyntaxError
```

This is the right pattern for layers: the low-level layer explains **what** broke (`SyntaxError`), the high-level layer explains **what it was doing** (`Config is invalid`). Without `cause`, the original stack is lost. `cause` is ES2022 and supported in all modern Node and browsers.

---

# 3. Async Errors

### Async errors

Inside an `async` function, a thrown error becomes a **rejected promise**. A `try/catch` around `await` catches both synchronous throws and rejections from the awaited promise.

```javascript
const getUser = async (id) => {
  if (id !== 1) throw new Error("User not found");
  return { id, name: "Kakade" };
};

const main = async () => {
  try {
    console.log(await getUser(1));
    console.log(await getUser(2));
  } catch (err) {
    console.log("caught:", err.message);
  }
};

main();
```

Output:

```text
{ id: 1, name: 'Kakade' }
caught: User not found
```

The critical rule: **you must `await`** the promise for `try/catch` to see its rejection. `const p = getUser(2);` without `await` returns a pending promise, the `try` block finishes successfully, and the rejection surfaces later as unhandled. `return await` inside `try` is also correct when you want the local `catch` to run.

### Promise rejection

A promise that rejects with no handler triggers an **unhandled rejection**. Attach `.catch()` at the end of a chain, or handle it with `try/catch` around `await`.

```javascript
Promise.reject(new Error("bad")).catch((e) => console.log("handled:", e.message));

process.on("unhandledRejection", (reason) => {
  console.log("unhandled:", reason.message);
});

Promise.reject(new Error("nobody caught me"));
```

Output:

```text
handled: bad
unhandled: nobody caught me
```

Notice: a `.catch()` handles only that link; a later `.then()` that throws creates a new rejected promise. Always end a chain with a handler.

### Global error handlers

Some errors escape every local `try`. Browsers expose `window.onerror` / the `error` event for uncaught exceptions, and `unhandledrejection` for promises. Node exposes `process.on("uncaughtException")` and `process.on("unhandledRejection")`.

```html
<script>
  window.addEventListener("error", (event) => {
    console.log("Script error:", event.message);
  });

  window.addEventListener("unhandledrejection", (event) => {
    console.log("Promise error:", event.reason.message);
  });

  setTimeout(() => {
    throw new Error("boom");
  });

  Promise.reject(new Error("nope"));
</script>
```

Output:

```text
Script error: Uncaught Error: boom
Promise error: nope
```

Global handlers are a **last resort for logging**, not a recovery strategy. After an uncaught exception the app may be inconsistent, so show a friendly message and reload or reset rather than continuing silently.

---

# 4. Error Discipline

### Re-throwing

Catch an error only if you can **do something useful**: retry, fall back, translate it, or add context. Otherwise let it bubble, or re-throw it wrapped with `cause`.

```javascript
function readConfig(raw) {
  try {
    return JSON.parse(raw);
  } catch (err) {
    throw new Error("readConfig: invalid JSON", { cause: err });
  }
}

try {
  readConfig("{oops");
} catch (e) {
  console.log(e.message, "|", e.cause.name);
}
```

Output:

```text
readConfig: invalid JSON | SyntaxError
```

A bare `catch {}` is fine for genuinely optional work (parsing an optional cookie), but only deliberately. If you catch and do not re-throw, return a clear fallback value so the caller is not left with `undefined`. Logging an error and continuing as if nothing happened is the most common form of swallowing — either recover explicitly or re-throw.

### Validation vs exceptions

Use **validation** for expected, user-driven problems: empty form fields, wrong file type, a missing query parameter. Return a boolean or a result object. Use **exceptions** for programmer errors and broken invariants: a required dependency is missing, a database is down, a switch hit an impossible state.

```javascript
function isEmail(value) {
  return typeof value === "string" && value.includes("@");
}

console.log(isEmail("kakade@example.com"), isEmail("kakade"));

function saveEmail(value) {
  if (!isEmail(value)) throw new TypeError("Invalid email");
  return "saved";
}

try {
  saveEmail("kakade");
} catch (e) {
  console.log(e.name, e.message);
}
```

Output:

```text
true false
TypeError Invalid email
```

`isEmail` is a cheap predicate and should not throw for normal invalid input. `saveEmail` enforces a precondition and throws because continuing would store bad data. Throwing for every typo makes normal control flow expensive and hard to read; returning `null` for every system failure hides bugs.

### Reading stack traces

A stack trace reads **top-down**: the first frame is where the error was created, the next is its caller, and so on. In Node or Chrome the frames include `file:line:column`.

```javascript
function inner() {
  throw new Error("where am I");
}
function outer() {
  inner();
}

try {
  outer();
} catch (err) {
  const names = [...err.stack.matchAll(/at (\w+)/g)].map((m) => m[1]);
  console.log(names.slice(0, 2));
}
```

Output:

```text
[ 'inner', 'outer' ]
```

Read it as: the error started in `inner`, called from `outer`. When debugging, find the **first frame in your own code**, not the framework frame above it. Async stacks can show `await` boundaries as separate frames, and source maps map bundled code back to the original file.

---

# Real-World Uses

- **API layers** — wrap low-level failures in a domain error with `cause`, then map to HTTP status codes at the edge.
- **Forms** — validate with predicates or a schema and return field errors; throw only for broken assumptions.
- **Monitoring** — global handlers report uncaught errors and unhandled rejections to a logging service.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **try, catch and finally** | `finally` always runs, even on `return`; never return from it |
| **throw** | Throw `Error` objects, not strings or plain objects |
| **Error types** | `SyntaxError`, `TypeError`, `ReferenceError`, `RangeError`, `AggregateError` |
| **Custom errors** | `extends Error`, call `super`, set `name`, add fields |
| **Error cause** | `new Error(msg, { cause })` preserves the original error |
| **Async errors** | Thrown errors become rejections; `await` inside `try` catches them |
| **Promise rejection** | End chains with `.catch`; unhandled rejections crash/report |
| **Global error handlers** | Last-resort logging: `error` / `unhandledrejection`, `process.on` |
| **Re-throwing** | Catch only what you can handle; otherwise re-throw with `cause` |
| **Validation vs exceptions** | Expected bad input → return a result; broken invariant → throw |
| **Reading stack traces** | Top frame is the throw site; find the first frame in your code |

## The Mental Model

```text
                     ERROR
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       detect        throw        reject
     validation    new Error     async / Promise
          │            │            │
          ↓            └─────┬──────┘
      return result         ↓
                       try / catch
                            │
                 ┌──────────┼──────────┐
                 ↓          ↓          ↓
             recover     wrap       re-throw
                          │
                       finally (cleanup)
```

### The 3 things to remember

**1. Throw real errors.**

```javascript
throw new Error("Insufficient funds");
// not: throw "Insufficient funds";
```

**2. `finally` always runs — and can override a return.**

```javascript
function f() {
  try { return "try"; } finally { console.log("cleanup"); }
}
```

**3. Await before you catch.**

```javascript
try {
  await doWork();   // rejection is caught here
} catch (err) {
  console.error(err.message);
}
```

> **Throw `Error` objects, catch only what you can handle, preserve the cause, and always clean up in `finally`.**
