# Destructuring, Rest and Spread — Revision Notes (Intermediate)
> One-line mental model: destructuring unpacks by shape, rest collects the leftovers, spread expands into a new shallow container.

## Core recall
- **Array destructuring** binds by position; works on any iterable (strings, `Set`, `Map`). Empty slots skip positions.
- **Object destructuring** binds by key name; order is irrelevant. `{ name: userName }` renames, `{ name = 'Guest' }` defaults.
- **Defaults only for `undefined`** — missing key or explicit `undefined`. `null` skips the default.
- **Nested destructuring** dereferences each level; `{ user: { name } }` throws if `user` is missing.
- **Rest** is `...` on the left / in parameters: `[a, ...rest]`, `{ a, ...rest }`, `fn(a, ...rest)`. Must be last; creates a new array/object.
- **Object rest** copies **own enumerable** keys and drops the extracted ones — great for omitting secrets.
- **Spread** is `...` on the right / in literals / at call sites: `[...a]`, `{ ...a }`, `fn(...args)`.
- **Array spread** expands any iterable and creates a new shallow array. `Math.max(...arr)` works but has an argument limit.
- **Object spread** copies own enumerable props only; later keys win; prototype and accessors are not preserved (getter values are evaluated).
- **Function parameters**: `function f({ a = 1 } = {})` — the trailing `= {}` makes the whole argument optional.
- **Copying vs referencing**: new container, nested values shared. Use `structuredClone` for deep data clones.
- **Swap**: `[a, b] = [b, a]`. Assignment destructuring must be wrapped: `({ a } = obj);`.

## Must-remember code
- ```javascript
  const [first, second = 'Asha', ...others] = ['Kakade', undefined, 'Rohan'];
  console.log(first, second, others); // Kakade Asha [ 'Rohan' ]
  ```
- ```javascript
  const { name: userName, age = 30, address: { city = 'Pune' } = {} } = { name: 'Kakade' };
  console.log(userName, age, city); // Kakade 30 Pune
  ```
- ```javascript
  const user = { name: 'Kakade', password: 'secret', role: 'admin' };
  const { password, ...safeUser } = user;
  console.log(safeUser); // { name: 'Kakade', role: 'admin' }
  ```
- ```javascript
  const defaults = { theme: 'light', lang: 'en' };
  const settings = { ...defaults, theme: 'dark' };
  console.log(settings); // { theme: 'dark', lang: 'en' }
  ```

## Interview traps
- `{ age = 30 }` does not replace `null`; use `age ?? 30` when null should fall back.
- Nested pattern with no intermediate default → `TypeError`; add `= {}`.
- Rest must be last: `[...rest, last]` is a syntax error.
- `add(nums)` instead of `add(...nums)` passes one array → `1,2,3undefinedundefined`.
- Spread copies references: editing `copy.address.city` changes the original.
- Object spread drops the prototype; it is not a class instance clone.
- Assignment destructuring without parentheses `({ a } = obj)` is a `SyntaxError`.
- `JSON.parse(JSON.stringify(obj))` is not a safe deep clone (drops `undefined`/functions, stringifies `Date`, loses `Map`/`Set`).
- Destructuring `null` or `undefined` throws; default the whole source (`fn({} = {})`) if it can be missing.

## 3 things to remember
1. **Shape, not magic:** arrays bind by position, objects by key.
2. **Defaults are narrow:** only `undefined` triggers them.
3. **`...` changes job by side:** left = collect (rest), right = expand (spread).
