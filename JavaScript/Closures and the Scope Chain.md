# Closures and the Scope Chain

A closure is what happens when a function **remembers the variables where it was defined**, even after that outer function has returned. It falls out of two simpler ideas: JavaScript uses **lexical scope** (position in the source decides what you can see) and functions are values that can outlive their creation context. The chain is **lexical scope → nested function → closure → private state**, and the interview favourites are the loop bug, counters and memoization.

```text
Where a function is WRITTEN
        ↓
decides what it can SEE   (lexical scope)
        ↓
inner function captures outer variables
        ↓
outer returns, variables survive  (closure)
        ↓
private state · counters · memoization
```

---

# 1. Scope Foundations

### Lexical scope

Scope is decided by **where code is written**, not where it is called. A function can see its own variables, then its enclosing function's, then the module/global scope — that lookup path is the **scope chain**.

```javascript
const app = "InterviewPrep";

function outer() {
  const topic = "Closures";

  function inner() {
    console.log(app, topic);
  }

  inner();
}

outer();
```

Output:

```text
InterviewPrep Closures
```

`inner` finds `topic` one level up and `app` two levels up. If a name is missing at every level you get `ReferenceError`, not `undefined` — `undefined` only appears when a declared variable has no value.

### Nested functions

A nested function is defined inside another function. It can read and write the outer variables, but the outer function **cannot** see the inner ones.

```javascript
function outer() {
  const secret = "Kakade";

  function inner() {
    return secret.toUpperCase();
  }

  return inner();
}

console.log(outer());
console.log(typeof secret);
```

Output:

```text
KAKADE
undefined
```

`secret` lives only for the duration of that call. Each call of `outer` creates a **new** environment, which is why two calls never share state.

---

# 2. Closures

### Closure definition

A closure is a function **plus the lexical environment it captured**. The function keeps that environment alive as long as the function itself is reachable.

```javascript
function makeGreeter(greeting) {
  return (name) => `${greeting}, ${name}!`;
}

const hi = makeGreeter("Hi");
const namaste = makeGreeter("Namaste");

console.log(hi("Kakade"));
console.log(namaste("Asha"));
```

Output:

```text
Hi, Kakade!
Namaste, Asha!
```

`makeGreeter` has already returned when `hi("Kakade")` runs, yet `greeting` is still there. Each call to `makeGreeter` produced a **separate** environment, so `hi` and `namaste` do not interfere.

Think:

```text
makeGreeter("Hi")
      │
      ├── environment { greeting: "Hi" }
      │
      └── returns (name) => ...   ← closure keeps the environment
                    │
                    └── later called: still sees greeting = "Hi"
```

### Counter example

The classic closure demo: state lives between calls without any global variable.

```javascript
function createCounter() {
  let count = 0;
  return () => ++count;
}

const next = createCounter();
console.log(next(), next(), next());

const other = createCounter();
console.log(other());
```

Output:

```text
1 2 3
1
```

`count` is not reset between calls because it belongs to the **environment**, not to the inner function. `other` has its own environment and starts from `0`. This is the difference between a closure and a module-level variable: state is per-instance.

---

# 3. Private State

### Private state

Variables captured by a closure are unreachable from outside except through the functions you expose. That is real privacy, not a naming convention.

```javascript
function createAccount(balance) {
  return {
    deposit: (amount) => (balance += amount),
    getBalance: () => balance,
  };
}

const account = createAccount(100);
account.deposit(50);

console.log(account.getBalance(), account.balance);
```

Output:

```text
150 undefined
```

There is no `balance` property on the returned object — only the two functions can touch it. A `_balance` naming convention or an object property would be editable from outside; a closure variable is not. Modern classes achieve the same with `#balance` private fields.

### Module pattern

The module pattern wraps state in an IIFE and returns only the public API. It was the standard before ES modules and still shows up in older codebases.

```javascript
const cart = (() => {
  const items = [];
  return {
    add: (item) => items.push(item),
    list: () => [...items],
  };
})();

cart.add("tea");
cart.add("coffee");
console.log(cart.list(), cart.items);
```

Output:

```text
[ 'tea', 'coffee' ] undefined
```

`items` is captured by both methods, so they share it, while `cart.items` is `undefined` — the array itself is hidden. `list` returns a **copy**, so callers cannot mutate the internal array through the returned value.

---

# 4. Closures in Practice

### Loop closure bug

With `var`, the loop has **one** binding shared by every callback. With `let`, each iteration gets a **fresh** binding.

```javascript
for (var i = 0; i < 3; i++) setTimeout(() => console.log("var", i), 0);
for (let j = 0; j < 3; j++) setTimeout(() => console.log("let", j), 0);
```

Output:

```text
var 3
var 3
var 3
let 0
let 1
let 2
```

All three `var` callbacks close over the same `i`, which is `3` by the time the timers run. `let` creates a new `j` per iteration, so each callback captures its own value.

Bad — the array of handlers all read the same `i`:

```javascript
const bad = [];
for (var i = 0; i < 3; i++) bad.push(() => i);
console.log(bad.map((fn) => fn()));
```

Output:

```text
[ 3, 3, 3 ]
```

Good — either use `let`, or create a scope per iteration with an IIFE:

```javascript
const good = [];
for (var k = 0; k < 3; k++) good.push(((n) => () => n)(k));
console.log(good.map((fn) => fn()));
```

Output:

```text
[ 0, 1, 2 ]
```

The same bug appears with `forEach` callbacks only if you capture a shared `var`; the array method itself already provides a per-item parameter.

### Event handlers

Each listener closes over the element it was created for, which is why you do not need to look the element up again inside the handler.

```html
<ul id="users">
  <li data-id="1">Kakade</li>
  <li data-id="2">Asha</li>
</ul>
<script>
  document.querySelectorAll("#users li").forEach((li) => {
    li.addEventListener("click", () => {
      console.log("clicked", li.dataset.id);
    });
  });
</script>
```

Output:

```text
clicked 1      ← after clicking Kakade
clicked 2      ← after clicking Asha
```

The arrow callback captures `li` from that iteration. Writing `var li` in an old-style `for` loop would give every handler the **last** element — the loop bug again. Remember to remove listeners (`removeEventListener`, `AbortSignal`) when the element or component goes away, or the closure keeps the element alive.

### Memoization

Memoization caches results in a closure. The cache is private and survives between calls, turning exponential recursion into linear work.

```javascript
function memoize(fn) {
  const cache = new Map();
  return (n) => {
    if (cache.has(n)) return cache.get(n);
    const result = fn(n);
    cache.set(n, result);
    return result;
  };
}

let calls = 0;
const fib = memoize((n) => {
  calls++;
  return n < 2 ? n : fib(n - 1) + fib(n - 2);
});

console.log(fib(10), calls);
```

Output:

```text
55 11
```

Without the cache, `fib(10)` would make 177 calls. Here each value `0..10` is computed once, so 11 calls. Note the recursion calls the **memoized** `fib`, not the raw function — that is what shares the cache. The `Map` lives in the closure and grows forever unless you add a size limit.

---

# 5. Memory Considerations

### Memory considerations

A closure keeps its captured environment alive as long as the function is reachable. That is the feature — and the leak.

```javascript
function attach() {
  const bigData = new Array(100000).fill("x");
  return () => bigData.length;
}

const reader = attach();
console.log(reader());
```

Output:

```text
100000
```

`bigData` cannot be garbage-collected while `reader` exists. V8 only retains variables the closure actually **references**, but one returned function is enough to keep a large object alive.

Common leaks:

| Cause | Fix |
| --- | --- |
| `setInterval` never cleared | `clearInterval(id)` |
| Listener never removed | `removeEventListener` / `AbortSignal` |
| Cache grows without bound | cap size, or use `WeakMap` |
| Closure captures a huge object for one field | capture the field, not the object |

`WeakMap`/`WeakSet` hold keys weakly, so entries disappear when the key object is collected — the right tool for attaching private data to objects you do not own.

---

# Real-World Uses

- **Counters and IDs** — per-instance state without globals.
- **Debounce/throttle** — the timer id and last-call time live in a closure.
- **Memoization** — caches for expensive pure functions.
- **Module pattern** — private state plus a small public API.
- **Event handlers** — each listener remembers its element and data.
- **React hooks** — every render closes over that render's props and state.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Lexical scope** | Scope is fixed by where code is written, not where it is called |
| **Nested functions** | Inner sees outer; outer never sees inner |
| **Closure definition** | Function + the environment it captured, alive after the outer returns |
| **Counter example** | State lives in the environment; each factory call gets its own |
| **Private state** | Captured variables are unreachable from outside unless exposed |
| **Loop closure bug** | `var` = one shared binding; `let` = one per iteration |
| **Event handlers** | Each listener captures its own element; clean up to avoid leaks |
| **Memoization** | Private `Map` cache in a closure; recurse through the memoized fn |
| **Module pattern** | IIFE + returned API; internals stay hidden |
| **Memory** | Closures keep captured values alive; unbounded caches and timers leak |

## The Mental Model

```text
        SOURCE CODE
             │
      lexical scope (where written)
             │
      ┌──────┴──────┐
      ↓             ↓
  outer vars    inner function
                    │
                    ↓
              captures environment
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     counter    private      memo cache
     state      variables    (Map)
        │           │           │
        └───────────┴───────────┘
                    ↓
            lives while reachable
                    ↓
             leaks if never freed
```

### The 3 things to remember

**1. Scope is lexical.**

```javascript
const x = "outer";
function f() { return x; }   // looks up where f was WRITTEN
f();                         // "outer"
```

**2. `var` shares one binding in loops.**

```javascript
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 3 3 3
for (let j = 0; j < 3; j++) setTimeout(() => console.log(j)); // 0 1 2
```

**3. A closure is private state.**

```javascript
const counter = (() => { let n = 0; return () => ++n; })();
counter(); // 1
```

> **A closure is a function that carries its birth environment with it — that is how private state, counters, memoization and per-element handlers work, and why long-lived closures can leak memory.**
