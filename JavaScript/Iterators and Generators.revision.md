# Iterators and Generators — Revision Notes (Intermediate)
> One-line mental model: an iterable returns an iterator, an iterator returns `{ value, done }`, and a generator is the lazy function that produces both.

## Core recall
- **Iterable protocol:** `obj[Symbol.iterator]()` returns an iterator. Arrays, strings, `Map`, `Set`, `arguments`, `NodeList`, generators implement it.
- **Iterator protocol:** `next()` returns `{ value, done }`. `done: true` is final; later calls keep returning done.
- **`for...of`:** calls `Symbol.iterator` **once**, then `next()` until done. Reads **values**; `for...in` reads keys (and inherited ones).
- **Plain objects are not iterable** — use `Object.entries/keys/values` or add `[Symbol.iterator]`.
- **Custom iterators:** keep cursor state in the iterator closure; return a **new** iterator per `[Symbol.iterator]()` call so loops restart.
- **Generators:** `function*`; calling returns a generator object without running the body. Body starts on the first `next()`.
- **`yield`:** pauses and emits. `next(value)` resumes and the value becomes the result of the paused `yield`. The first `next(arg)` argument is ignored.
- **`return` in a generator:** produces `{ value, done: true }`; `for...of` does not expose it.
- **`yield*`:** delegates to any iterable/generator; evaluates to the delegate's return value.
- **Single-use:** a generator is both iterable and iterator (`gen[Symbol.iterator]() === gen`), so it cannot be looped twice.
- **Lazy:** work happens only on demand; `break` calls `return()` and runs `finally`.
- **Infinite sequences:** safe with `take`/`break`; never `[...infinite]`.
- **Async generators:** `async function*`, consumed with `for await...of`; `next()` returns a promise of `{ value, done }`.

## Must-remember code
- ```javascript
  const range = {
    from: 1,
    to: 3,
    [Symbol.iterator]() {
      let n = this.from;
      return { next: () => (n <= this.to ? { value: n++, done: false } : { value: undefined, done: true }) };
    },
  };
  console.log([...range]); // [ 1, 2, 3 ]
  ```
- ```javascript
  function* greet() {
    const name = yield 'Who are you?';
    yield `Hello ${name}`;
  }
  const g = greet();
  console.log(g.next().value);        // Who are you?
  console.log(g.next('Kakade').value); // Hello Kakade
  ```
- ```javascript
  function* inner() { yield 'a'; yield 'b'; }
  function* outer() { yield 0; yield* inner(); yield 'c'; }
  console.log([...outer()]); // [ 0, 'a', 'b', 'c' ]
  ```
- ```javascript
  async function* ticker() { yield 1; yield 2; }
  for await (const n of ticker()) console.log(n); // 1, 2
  ```

## Interview traps
- Spreading a generator twice: second result is `[]` — generators are single-use.
- Spreading an infinite generator hangs the process; bound it with `take`/`break`.
- `yield` inside `forEach`/`map` callback → `SyntaxError`; use `for...of`.
- Returning `this` from `[Symbol.iterator]` with shared state makes the second loop empty.
- `for...of` on a plain object → `TypeError`; `for...in` over an array gives string indices.
- `done: true` values are not emitted by `for...of`, even if `value` is set.
- `next(arg)` on the first call is ignored.
- Async iterators need `for await...of`, not `for...of`.

## 3 things to remember
1. **Two methods:** `[Symbol.iterator]()` → iterator; `next()` → `{ value, done }`.
2. **Generators pause:** `yield` emits, `next(v)` resumes with `v`.
3. **Pull, don't push:** consume lazily and never exhaust an infinite iterator.
