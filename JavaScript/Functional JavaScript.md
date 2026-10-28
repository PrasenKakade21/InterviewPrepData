# Functional JavaScript

Functional JavaScript is a **style**, not a library: build small functions that return values instead of changing shared state, treat data as immutable, and combine them into pipelines. The chain is **pure function → immutable input → transform → compose or curry → reusable pipeline**. Interviewers test the vocabulary — pure, side effect, higher-order, composition, currying, partial application — and whether you know the cost.

```text
input
  ↓
pure function   same input → same output
  ↓
immutable data  copy, then change
  ↓
map / filter / reduce
  ↓
compose / pipe / curry
  ↓
output + side effects at the edge
```

---

# 1. Pure Functions and Effects

### Pure functions

A function is **pure** when it returns the same output for the same input and has no observable side effects. It does not read or write external state, log, fetch, or touch the DOM.

```javascript
const add = (a, b) => a + b;

let total = 0;
const addToTotal = (n) => (total += n);

console.log(add(2, 3), add(2, 3));
console.log(addToTotal(5), addToTotal(5));
```

Output:

```text
5 5
5 10
```

`add` is pure. `addToTotal` depends on and mutates `total`, so the same argument produces different results. `Math.random()`, `Date.now()`, `console.log`, `fetch` and DOM writes are all impure.

### Side effects

Side effects are sometimes required — someone has to call the API and update the screen. The functional approach is to keep the **core** pure and push effects to the **edges** of the program.

```javascript
const applyDiscount = (price, percent) => price - (price * percent) / 100;
const prices = [100, 250, 400];

const discounted = prices.map((p) => applyDiscount(p, 10));
console.log(discounted);
```

Output:

```text
[ 90, 225, 360 ]
```

Think:

```text
input ──→ pure core ──→ result
                            │
                    effect at the edge
                    log / render / save
```

This separation makes the core trivial to test: no network, no clock, no DOM. “Referential transparency” is the formal name — you can replace a call with its result and nothing changes.

---

# 2. Immutability

### Immutability

Immutability means you create a **new value** instead of modifying the old one. Arrays and objects are references, so `sort`, `push`, `splice` and direct assignment mutate the shared original.

```javascript
const original = [3, 1, 2];
const sorted = [...original].sort((a, b) => a - b);
console.log(sorted, original);

const user = { name: "Kakade", roles: ["admin"] };
const updated = { ...user, roles: [...user.roles, "editor"] };
console.log(updated.roles, user.roles);
```

Output:

```text
[ 1, 2, 3 ] [ 3, 1, 2 ]
[ 'admin', 'editor' ] [ 'admin' ]
```

Bad: `arr.sort()` or `user.roles.push("editor")` — both edit the input. Good: copy first, then change. ES2023 adds non-mutating `toSorted`, `toReversed`, `toSpliced` and `with`, which read better than `[...arr]`.

```javascript
const config = Object.freeze({ theme: "dark", flags: { beta: false } });
config.theme = "light";
config.flags.beta = true;
console.log(config.theme, config.flags.beta);
```

Output:

```text
dark true
```

Notice: `Object.freeze` is **shallow** and, outside strict mode, failed writes are silent. The nested `flags` object is not frozen. Immutability is a discipline, not a guarantee.

---

# 3. The Core Trio

### map, filter and reduce

`map` transforms every element, `filter` keeps the ones that pass, and `reduce` folds the whole array into one value. All three return new values and leave the input alone.

```javascript
const users = [
  { name: "Kakade", age: 30, role: "admin" },
  { name: "Asha", age: 24, role: "editor" },
  { name: "Rohan", age: 35, role: "editor" },
];

console.log(users.map((u) => u.name));
console.log(users.filter((u) => u.age >= 30).map((u) => u.name));
console.log([10, 20, 30].reduce((acc, n) => acc + n, 0));
```

Output:

```text
[ 'Kakade', 'Asha', 'Rohan' ]
[ 'Kakade', 'Rohan' ]
60
```

`reduce` is the general one: grouping, counting, flattening and building objects all use it.

```javascript
const byRole = users.reduce((acc, user) => {
  (acc[user.role] ??= []).push(user.name);
  return acc;
}, {});

console.log(byRole);
```

Output:

```text
{ admin: [ 'Kakade' ], editor: [ 'Asha', 'Rohan' ] }
```

Bad: `[].reduce((a, b) => a + b)` with no initial value → `TypeError: Reduce of empty array with no initial value`. `acc[key] ??= []` handles the first occurrence cleanly. Mutating the **local accumulator** inside `reduce` is normal; mutating the input array is not.

---

# 4. Higher-Order Functions

### Higher-order functions

A higher-order function either **takes** a function, **returns** a function, or both. `map`, `filter`, `setTimeout` and `addEventListener` are built-in examples.

```javascript
const twice = (fn) => (x) => fn(fn(x));
const inc = (n) => n + 1;
console.log(twice(inc)(5));

const withLogging = (fn) => (...args) => {
  console.log("call", args);
  return fn(...args);
};
const multiply = withLogging((a, b) => a * b);
console.log(multiply(3, 4));
```

Output:

```text
7
call [ 3, 4 ]
12
```

`twice` returns a new function; `withLogging` wraps one. This is how decorators, middleware and memoization are built: behaviour is added by wrapping, not by editing the original.

| Built-in | Takes | Returns |
| --- | --- | --- |
| `map(fn)` | transform | new array |
| `filter(fn)` | predicate | new array |
| `reduce(fn, init)` | reducer | one value |
| `setTimeout(fn, ms)` | callback | timer id |
| `addEventListener(fn)` | handler | nothing useful |

---

# 5. Composition and Pipelines

### Composition

Composition combines small functions into one. `compose` applies **right to left** (mathematical order); `pipe` applies **left to right** (reading order). Both are just `reduce` over an array of functions.

```javascript
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);

const trim = (s) => s.trim();
const lower = (s) => s.toLowerCase();
const slug = (s) => s.replace(/\s+/g, "-");

console.log(compose(slug, lower, trim)("  Hello World  "));
console.log(pipe(trim, lower, slug)("  Hello World  "));
```

Output:

```text
hello-world
hello-world
```

Think:

```text
compose(f, g, h)(x) = f(g(h(x)))   ← right to left
pipe(h, g, f)(x)    = f(g(h(x)))   ← left to right
```

### Pipelines

A pipeline is a composition applied to real data with several steps. It reads like a recipe because each stage is a named, testable function.

```javascript
const pipeline = (...fns) => (input) => fns.reduce((acc, fn) => fn(acc), input);

const result = pipeline(
  (xs) => xs.filter((n) => n % 2 === 0),
  (xs) => xs.map((n) => n * 10),
  (xs) => xs.reduce((a, b) => a + b, 0)
)([1, 2, 3, 4]);

console.log(result);
```

Output:

```text
60
```

Bad: one 30-line loop that filters, maps, sums and logs. Good: three pure stages plus a final effect. Composition is associative — `compose(f, compose(g, h))` equals `compose(compose(f, g), h)` — so you can regroup stages without changing the result.

---

# 6. Currying and Partial Application

### Currying

Currying turns a function of several arguments into a chain of **unary** functions: `f(a, b, c)` becomes `f(a)(b)(c)`. Each call returns a function until all arguments are supplied.

```javascript
const volume = (l) => (w) => (h) => l * w * h;
console.log(volume(2)(3)(4));

function curry(fn) {
  return function curried(...args) {
    return args.length >= fn.length
      ? fn(...args)
      : (...next) => curried(...args, ...next);
  };
}

const format = curry((greeting, name, mark) => `${greeting}, ${name}${mark}`);
console.log(format("Hi")("Kakade")("!"));
```

Output:

```text
24
Hi, Kakade!
```

The generic version compares `args.length` with `fn.length` (the declared parameter count). That fails for default or rest parameters, because `fn.length` stops counting there.

### Partial application

Partial application fixes **some** arguments now and supplies the rest later. Unlike currying, it does not require one argument per call.

```javascript
const greet = (greeting, name, punctuation) => `${greeting}, ${name}${punctuation}`;

const hi = greet.bind(null, "Hi");
console.log(hi("Kakade", "!"));

const double = (factor) => (n) => n * factor;
console.log([1, 2, 3].map(double(2)));
```

Output:

```text
Hi, Kakade!
[ 2, 4, 6 ]
```

| | Currying | Partial application |
| --- | --- | --- |
| Shape | One argument per call | Any number fixed now |
| Result | Chain of unary functions | Function with fewer parameters |
| Typical use | Reusable config | Pre-fill `greeting` or `factor` |
| Example | `volume(2)(3)(4)` | `greet.bind(null, "Hi")` |

`bind` also fixes `this`; with `null` you are using only its argument-fixing behaviour. A closure (`(name) => greet("Hi", name, "!")`) is often clearer than `bind` in modern code.

---

# 7. Style and Cost

### Declarative vs imperative

Imperative code says **how**: create an array, loop, push. Declarative code says **what**: keep evens, square them. Functional JavaScript prefers declarative transforms because the intent is visible and the loop bookkeeping is gone.

```javascript
const numbers = [1, 2, 3, 4, 5, 6];

const evenSquares = numbers
  .filter((n) => n % 2 === 0)
  .map((n) => n ** 2);

console.log(evenSquares);
```

Output:

```text
[ 4, 16, 36 ]
```

Bad:

```javascript
const result = [];
for (let i = 0; i < numbers.length; i++) {
  if (numbers[i] % 2 === 0) result.push(numbers[i] ** 2);
}
```

Good: the chain states the transformation and does not mutate `numbers`. The loop is not “wrong” — it is harder to reuse and easier to get an off-by-one wrong.

### Performance tradeoffs

Each chained method walks the array and allocates a new one. A single `reduce` can filter and map in one pass, which matters for very large arrays or hot paths.

```javascript
const result = numbers.reduce((acc, n) => {
  if (n % 2 === 0) acc.push(n ** 2);
  return acc;
}, []);

console.log(result);
```

Output:

```text
[ 4, 16, 36 ]
```

`filter().map()` is two passes and two arrays; the `reduce` is one pass and one array. For a few thousand items the difference is noise, so prefer the clearer chain. Optimize only after measuring with `performance.now()` or the profiler, and keep the pure stages so the fast version stays testable.

---

# Real-World Uses

- **Data dashboards** — filter rows, map to view models, reduce to totals.
- **State reducers** — pure `(state, action) => newState` functions.
- **Validation pipelines** — trim, normalize, then run small predicate functions.
- **API response shaping** — `compose` a serializer from reusable steps.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Pure functions** | Same input → same output; no external reads or writes |
| **Immutability** | Copy before changing; spread/`toSorted`; `freeze` is shallow |
| **map, filter and reduce** | Transform / keep / fold; all leave the input alone |
| **Composition** | `compose` right-to-left, `pipe` left-to-right, both `reduce` |
| **Currying** | `f(a)(b)(c)`; one argument per call; generic version uses `fn.length` |
| **Partial application** | Fix some args now (`bind` or closure), supply the rest later |
| **Higher-order functions** | Take and/or return functions; `map`, `filter`, wrappers |
| **Declarative vs imperative** | Say what, not how; chain transforms instead of manual loops |
| **Side effects** | Keep them at the edges; pure core is easy to test |
| **Pipelines** | Named stages composed into a reusable data flow |
| **Performance tradeoffs** | Chains allocate intermediate arrays; `reduce` is one pass |

## The Mental Model

```text
                     INPUT
                       │
                       ↓
                 pure function
             same in → same out
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      map           filter         reduce
   transform         keep           fold
        └──────────────┼──────────────┘
                       ↓
                 immutable data
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     compose          pipe          curry
   right-to-left   left-to-right   unary chain
        └──────────────┼──────────────┘
                       ↓
                 OUTPUT + effects
```

### The 3 things to remember

**1. Pure means predictable.**

```javascript
const add = (a, b) => a + b; // pure
let total = 0;
const addToTotal = (n) => (total += n); // impure
```

**2. Copy before you change.**

```javascript
const next = [...items].sort((a, b) => a - b);
const user2 = { ...user, roles: [...user.roles, "editor"] };
```

**3. Compose small functions.**

```javascript
const slugify = pipe(trim, lower, (s) => s.replace(/\s+/g, "-"));
slugify("  Hello World  "); // hello-world
```

> **Functional JavaScript is small pure functions over immutable data, composed into pipelines — with side effects pushed to the edges and performance measured, not assumed.**
