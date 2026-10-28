# Iterators and Generators

An **iterable** is something you can loop over; an **iterator** is the cursor that produces values one at a time; a **generator** is a function that writes iterators for you. The chain is: **iterable → `Symbol.iterator` → iterator → `next()` → `{ value, done }`**. Once you see that chain, `for...of`, spread, and destructuring all stop being separate features — they all consume the same protocol.

```text
for...of / spread / [...x]
        ↓
x[Symbol.iterator]()
        ↓
iterator.next() → { value, done: false }
        ↓
repeat until { value: undefined, done: true }
```

---

# 1. The Two Protocols

The iterator protocol describes **how values come out**. The iterable protocol describes **how to get an iterator**.

### Iterator protocol

An iterator is any object with a `next()` method returning `{ value, done }`. When `done` is `true`, iteration is over.

```javascript
const iterator = {
  current: 0,
  next() {
    if (this.current < 3) {
      return { value: this.current++, done: false };
    }
    return { value: undefined, done: true };
  },
};

console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
```

Output:

```text
{ value: 0, done: false }
{ value: 1, done: false }
{ value: 2, done: false }
{ value: undefined, done: true }
```

`done: true` is final. Notice that well-behaved iterators keep returning `{ value: undefined, done: true }` after that.

### Iterable protocol

An iterable is any object with a `[Symbol.iterator]()` method that **returns an iterator**.

```javascript
const iterable = {
  [Symbol.iterator]() {
    let n = 0;
    return {
      next: () => (n < 2 ? { value: n++, done: false } : { value: undefined, done: true }),
    };
  },
};

const [first, second] = iterable;

console.log(first, second);
console.log([...iterable]);
```

Output:

```text
0 1
[ 0, 1 ]
```

`Symbol.iterator` is a **well-known symbol** — a unique key that language features look for. Arrays, strings, `Map`, `Set`, `arguments`, and `NodeList` all implement it.

---

# 2. `for...of`

`for...of` calls `[Symbol.iterator]()` once, then calls `next()` repeatedly until `done` is true.

### What it actually does

```javascript
const kakade = ['K', 'a', 'k'];

for (const letter of kakade) {
  console.log(letter);
}

const manual = kakade[Symbol.iterator]();
console.log(manual.next());
```

Output:

```text
K
a
k
{ value: 'K', done: false }
```

The loop is just the manual version with nicer syntax.

### What is not iterable

Plain objects have no `Symbol.iterator`, so `for...of` cannot walk them. Use `Object.keys`, `Object.values`, `Object.entries`, or `for...in` (which walks keys, including inherited ones).

Bad — looping the object directly:

```javascript
const user = { name: 'Kakade', role: 'admin' };

try {
  for (const [key, value] of user) console.log(key, value);
} catch (error) {
  console.log(error.constructor.name);
}
```

Output:

```text
TypeError
```

Good — iterate the entries:

```javascript
for (const [key, value] of Object.entries({ name: 'Kakade', role: 'admin' })) {
  console.log(key, value);
}
```

Output:

```text
name Kakade
role admin
```

> **Trap:** `for...of` reads **values**; `for...in` reads **keys**, and `for...in` includes inherited enumerable properties. Never use `for...in` on arrays.

| | `for...of` | `for...in` |
| --- | --- | --- |
| Gives | values | keys as strings |
| Uses | `Symbol.iterator` | property enumeration |
| Inherited | no | yes |
| Arrays | good | avoid |

---

# 3. Custom Iterators

You can make any object iterable by implementing `[Symbol.iterator]`. This is useful for domain objects that should behave like collections.

### A range object

```javascript
const range = {
  from: 1,
  to: 4,
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;

    return {
      next() {
        return current <= last
          ? { value: current++, done: false }
          : { value: undefined, done: true };
      },
    };
  },
};

console.log([...range]);
console.log(Math.max(...range));
```

Output:

```text
[ 1, 2, 3, 4 ]
4
```

### Each loop gets a fresh iterator

```javascript
for (const n of range) process.stdout.write(`${n} `);
process.stdout.write('\n');
for (const n of range) process.stdout.write(`${n} `);
process.stdout.write('\n');
```

Output:

```text
1 2 3 4 
1 2 3 4 
```

`[Symbol.iterator]()` creates a **new** iterator each time, so the second loop starts from the beginning. If `[Symbol.iterator]()` returned `this` and stored state on the object, the second loop would be empty — a classic bug.

---

# 4. Generator Functions

A generator function is declared with `function*`. Calling it does **not** run the body; it returns a generator object.

### `function*` returns a generator

```javascript
function* counter() {
  console.log('body started');
  yield 1;
  yield 2;
  return 3;
}

const gen = counter();
console.log(typeof gen.next);
console.log(gen.next());
```

Output:

```text
function
body started
{ value: 1, done: false }
```

The body runs lazily, on the first `next()`. The `return 3` becomes `{ value: 3, done: true }` and is **not** produced by `for...of`.

### Generators are iterable iterators

```javascript
function* pair() {
  yield 'Kakade';
  yield 'Asha';
}

const gen = pair();

console.log(gen[Symbol.iterator]() === gen);
console.log([...gen]);
console.log([...gen]);
```

Output:

```text
true
[ 'Kakade', 'Asha' ]
[]
```

A generator is **both** an iterator (`next`) and an iterable (`[Symbol.iterator]` returning itself). It is also **single-use**: once exhausted, it stays exhausted.

> **Notice:** because it returns itself, `gen[Symbol.iterator]() === gen` — which is exactly why it cannot be looped twice.

---

# 5. `yield`

`yield` pauses the generator and emits a value. Execution resumes at the next `next()` call, right after the `yield`.

### Pausing and resuming

```javascript
function* steps() {
  console.log('step 1');
  yield 'one';
  console.log('step 2');
  yield 'two';
  console.log('done');
}

const gen = steps();
console.log(gen.next().value);
console.log(gen.next().value);
console.log(gen.next().done);
```

Output:

```text
step 1
one
step 2
two
done
true
```

### Sending values in

The argument to `next(value)` becomes the **result of the paused `yield`**.

```javascript
function* greet() {
  const name = yield 'Who are you?';
  yield `Hello ${name}`;
}

const gen = greet();

console.log(gen.next().value);
console.log(gen.next('Kakade').value);
```

Output:

```text
Who are you?
Hello Kakade
```

The first `next()` starts the body and stops at the first `yield`. The second `next('Kakade')` resumes with `name === 'Kakade'`. Sending a value on the **first** call is meaningless — there is no paused `yield` yet.

> **Trap:** `yield` is an expression, so `const x = yield` is valid, but `yield` inside a normal callback (`array.map(function () { yield 1; })`) is a syntax error — only the generator body can yield.

---

# 6. `yield*`

`yield*` **delegates** to another iterable: it yields every value from it, one by one, and evaluates to that iterable's return value.

### Delegating to another iterable

```javascript
function* letters() {
  yield 'a';
  yield 'b';
  return 'done';
}

function* combined() {
  yield 0;
  const result = yield* letters();
  yield result;
  yield* ['c', 'd'];
}

console.log([...combined()]);
```

Output:

```text
[ 0, 'a', 'b', 'done', 'c', 'd' ]
```

`yield*` is not a loop over an array only — it works with **any iterable**, including another generator, a string, a `Set`, or a custom iterator.

### Flattening nested generators

```javascript
function* inner(start) {
  yield start;
  yield start + 1;
}

function* outer() {
  yield* inner(1);
  yield* inner(10);
}

console.log([...outer()]);
```

Output:

```text
[ 1, 2, 10, 11 ]
```

This is the idiomatic way to compose generator pipelines without manually forwarding `next()`.

---

# 7. Lazy Evaluation

Generators compute values **on demand**. Nothing runs until the consumer asks, and nothing after the consumer stops.

### Work on demand

```javascript
function* numbers() {
  console.log('start');
  yield 1;
  console.log('continue');
  yield 2;
  console.log('end');
}

const gen = numbers();
console.log('before');
console.log(gen.next().value);
console.log('after first');
```

Output:

```text
before
start
1
after first
```

The `continue` and `end` lines never run because we stopped consuming. That is laziness: unused work is never performed.

### Infinite sequences

```javascript
function* naturals() {
  let n = 1;
  while (true) {
    yield n++;
  }
}

function take(iterable, count) {
  const result = [];
  const iterator = iterable[Symbol.iterator]();

  while (result.length < count) {
    const { value, done } = iterator.next();
    if (done) break;
    result.push(value);
  }

  return result;
}

console.log(take(naturals(), 5));
```

Output:

```text
[ 1, 2, 3, 4, 5 ]
```

The generator is infinite, but `take` calls `next()` exactly five times and then stops. Nothing after the fifth `yield` runs. If you do `break` out of a `for...of`, the iterator's `return()` is called, which runs any `finally` block in the generator.

> **Trap:** spreading an infinite generator (`[...naturals()]`) never finishes. Always bound consumption with `take`, `break`, or a condition.

---

# 8. Async Generators

An **async generator** (`async function*`) yields promises and is consumed with `for await...of`. It is the iterator protocol for streams and paginated APIs.

### `async function*` and `for await`

```javascript
async function* ticker() {
  yield 1;
  yield 2;
}

async function main() {
  const seen = [];
  for await (const value of ticker()) {
    seen.push(value);
  }
  console.log(seen);
}

main();
```

Output:

```text
[ 1, 2 ]
```

Async iterators return a promise from `next()` that resolves to `{ value, done }`. `for await...of` awaits each step, so you can `await` inside the generator between yields:

```javascript
async function* pages() {
  for (const id of [1, 2]) {
    const page = await Promise.resolve(`page ${id}`);
    yield page;
  }
}
```

Use async generators for paginated fetch loops, file/stream readers, and websocket message queues. Do not use them when a simple `Promise.all` over a fixed list is enough.

---

# Real-World Uses

- **Pagination**: `for await (const page of fetchPages(url))` instead of manual cursor handling.
- **Streams**: Node.js readable streams are async iterables.
- **IDs and sequences**: `idGenerator()` that never repeats.
- **Tree traversal**: `yield*` recursing into children produces a flat walk.
- **Custom collections**: make a `Playlist` iterable so `[...playlist]` and `for...of` work.

```javascript
function* walk(node) {
  yield node.name;
  for (const child of node.children ?? []) {
    yield* walk(child);
  }
}

const tree = { name: 'root', children: [{ name: 'Rohan' }, { name: 'b', children: [{ name: 'b1' }] }] };
console.log([...walk(tree)]);
```

Output:

```text
[ 'root', 'Rohan', 'b', 'b1' ]
```

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| Iterable protocol | `obj[Symbol.iterator]()` returns an iterator |
| Iterator protocol | `next()` returns `{ value, done }`; `done: true` is final |
| `for...of` | Calls `Symbol.iterator` once, then `next()` until done |
| Custom iterators | Implement `[Symbol.iterator]`; return a fresh iterator per loop |
| Generator functions | `function*` — calling returns a generator, body runs lazily |
| `yield` | Pauses and emits; `next(value)` sends a value back in |
| `yield*` | Delegates to any iterable, including another generator |
| Generators as iterators | Generator is both iterable and iterator; single-use |
| Lazy evaluation | Work happens only when consumed |
| Infinite sequences | Safe with `take`/`break`; never spread them |
| Async generators | `async function*` + `for await...of`; `next()` returns a promise |

## The Mental Model

```text
                 iterable
                    │
          [Symbol.iterator]()
                    │
                    ↓
                 iterator
                    │  next()
                    ↓
        { value, done: false } ──► repeat
                    │
              done: true
                    │
                    ↓
                  stop

   generator = function* ──► writes that iterator for you
```

### The 3 things to remember

**1. The protocol is two methods:**

```javascript
const it = [1, 2][Symbol.iterator]();
console.log(it.next()); // { value: 1, done: false }
```

**2. Generators are lazy and single-use:**

```javascript
function* g() { yield 1; }
const gen = g();
console.log([...gen], [...gen]); // [ 1 ] []
```

**3. `yield*` delegates; `yield` emits:**

```javascript
function* g() { yield 1; yield* [2, 3]; }
console.log([...g()]); // [ 1, 2, 3 ]
```

> **An iterable hands you an iterator, an iterator hands you `{ value, done }`, and a generator is the lazy function that produces both — pull only what you need.**
