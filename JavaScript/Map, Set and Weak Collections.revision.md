# Map, Set and Weak Collections — Revision Notes (Beginner)

> One-line mental model: `Map` stores any key, `Set` stores unique values, and weak versions hold object keys without keeping them alive.

## Core recall

- **Map vs object**: object keys are strings/symbols; `Map` keys can be any value. `Map` has `size`, is iterable, has no prototype keys, and keeps strict insertion order.
- **Map methods**: `set` returns the map (chainable), `get` returns the value or `undefined`, `has` returns a boolean, `delete` returns a boolean, `clear()` empties it. `size` is a property, not a method.
- **Set methods**: `add` returns the set, `has`, `delete`, `size`. Duplicate `add` is silently ignored.
- **WeakMap**: object keys only, weak references, no `size` / `clear` / iteration. Used for metadata and private state.
- **WeakSet**: object membership only, no size, no iteration. Used for "already seen" marks.
- **WeakRef**: `ref.deref()` returns the object or `undefined`; collection timing is non-deterministic. `FinalizationRegistry` callbacks are not guaranteed.
- **Iteration**: both `Map` and `Set` iterate in insertion order. `Map` yields `[key, value]`; `map.forEach((value, key) => ...)` is **value first**.
- **Key equality**: SameValueZero — `NaN` equals `NaN`, `+0` equals `-0`, objects compare by reference.
- **Use cases**: `Map` for counters, caches, grouping; `Set` for dedupe and membership; weak collections for per-object data.
- **Memory**: `Map`/`Set` hold strong references; weak collections do not. `JSON.stringify(new Map())` is `"{}"`.

## Must-remember code

```javascript
// dedupe primitives, order preserved
const unique = [...new Set(items)];

// dedupe objects by a field
const byId = new Map(users.map((u) => [u.id, u]));
const uniqueUsers = [...byId.values()];
```

```javascript
// count with a Map
const counts = new Map();
for (const word of words) {
  counts.set(word, (counts.get(word) ?? 0) + 1);
}
console.log([...counts]);
```

```javascript
// per-object metadata that does not leak
const meta = new WeakMap();
meta.set(user, { visits: 1 });
console.log(meta.get(user));
```

```javascript
// Map <-> object
const map = new Map(Object.entries({ a: 1, b: 2 }));
const obj = Object.fromEntries(map);
```

## Interview traps

- `map[key]` does **not** read entries; it is a plain property. Use `map.get(key)`.
- `map.length` is `undefined`; use `map.size`.
- `map.set({ id: 1 }, "x")` then `map.get({ id: 1 })` is `undefined` — different reference.
- `{ id: 1 }` twice in a `Set` gives `size` 2, not 1.
- `new Set([1, "1"])` has size 2 — types differ.
- `map.forEach((key, value) => ...)` is wrong; the real order is `(value, key)`.
- WeakMap keys must be objects; `weakMap.set("a", 1)` throws `TypeError`.
- `[...weakMap]`, `weakMap.size`, `weakMap.clear` do not exist.
- `JSON.stringify(map)` silently gives `{}` — convert first.
- A `WeakMap` prevents **itself** from leaking, not objects your app still references.

## 3 things to remember

1. **Map is for dynamic keys.**
   ```javascript
   const m = new Map();
   m.set(kakade, "admin").set(asha, "editor");
   m.get(kakade); // "admin"
   ```

2. **Set is dedupe plus membership.**
   ```javascript
   [...new Set([1, 1, 2, NaN, NaN])]; // [ 1, 2, NaN ]
   new Set([1, 2]).has(2);            // true
   ```

3. **Weak collections are not enumerable.**
   ```javascript
   const wm = new WeakMap();
   wm.set({}, 1);
   wm.size;            // undefined
   [...wm];            // TypeError
   ```

> **Map for keyed lookup, Set for uniqueness, WeakMap/WeakSet for metadata that must not keep objects alive.**
