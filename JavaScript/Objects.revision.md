# Objects — Revision Notes (Intermediate)

> One-line mental model: Objects are reference values — `=` aliases, spread shallow-copies, `structuredClone` deep-copies, `freeze` protects one level.

## Core recall

- **Literal**: `{ name: "Kakade", "home city": "Pune" }`; keys are strings/symbols. `Object.create(null)` = no prototype (no `hasOwnProperty`).
- **Access**: `obj.key` vs `obj["key"]`; missing → `undefined`. Guard nested reads with `obj?.a?.b`. `"key" in obj` walks the prototype chain.
- **Computed keys**: `{ [expr]: value }` evaluates the key; without brackets it is a literal name.
- **Shorthand**: `{ name, role }` = `{ name: name, role: role }`.
- **Methods**: `greet() {}` gets `this` from the call site; extracting the function loses `this`; arrow methods don't bind their own `this`.
- **delete**: removes an **own** property, returns `true` even if missing, never removes inherited ones, leaves holes in arrays.
- **keys/values/entries**: own **enumerable string** keys only; integer-like keys sort first ascending, then insertion order; symbols/non-enumerable skipped.
- **Object.assign(target, ...src)**: **mutates and returns target**, later sources win, shallow. Use `Object.assign({}, a, b)`.
- **Spread**: `{ ...src }` → new object, own enumerable only, shallow, later keys win.
- **Shallow vs deep**: spread/assign copy one level; `structuredClone` handles nested, `Date`, `Map`, `Set`, cycles; throws `DataCloneError` on functions.
- **JSON clone limits**: `Date` → ISO string, `undefined`/function/symbol dropped, `NaN`/`Infinity` → `null`, `Map`/`Set` → `{}`, `BigInt`/cycles throw.
- **freeze/seal**: freeze = no add/edit/delete; seal = no add/delete but values writable. Both **shallow**. Sloppy mode fails silently; strict/modules throw.
- **Descriptors**: `value`, `writable`, `enumerable`, `configurable`. `defineProperty` defaults omitted flags to **false**; literal defaults are all `true`.
- **Own vs inherited**: `in` and `for...in` walk the chain; `Object.keys` / `Object.hasOwn` do not.

## Must-remember code

```javascript
// immutable top-level update
const next = { ...state, user: { ...state.user, name: "Kakade" } };
```

```javascript
// safe merge (does not mutate defaults)
const settings = Object.assign({}, defaults, overrides);
```

```javascript
// real deep copy
const clone = structuredClone(original);
```

```javascript
// own-key iteration
Object.entries(obj).forEach(([key, value]) => console.log(key, value));
```

## Interview traps

- `Object.assign(defaults, x)` mutates `defaults` and returns it (`settings === defaults`).
- Spread is shallow: `{ ...obj }.nested.x = 1` edits the original.
- `JSON.parse(JSON.stringify(obj))` turns `Date` into a string and drops `undefined`.
- `Object.freeze` doesn't protect nested objects; writes fail silently outside strict mode.
- `for...in` prints inherited keys; use `Object.keys` or `Object.hasOwn`.
- `delete obj.x` on an inherited key returns `true` but changes nothing.
- `.hasOwnProperty` breaks on `Object.create(null)`; prefer `Object.hasOwn`.
- `defineProperty` makes properties non-enumerable by default → invisible to `keys`, spread and `JSON.stringify`.
- Key order is integer keys first, not pure insertion order.

## 3 things to remember

1. **`=` is not a copy.**
   ```javascript
   const a = { n: 1 };
   const b = a;        // same object
   const c = { ...a }; // new object, shallow
   ```

2. **Shallow vs deep.**
   ```javascript
   const shallow = { ...obj };          // nested values shared
   const deep = structuredClone(obj);   // fully independent
   ```

3. **Freeze is one level.**
   ```javascript
   const cfg = Object.freeze({ nested: { b: 1 } });
   cfg.nested.b = 2; // still allowed
   ```
