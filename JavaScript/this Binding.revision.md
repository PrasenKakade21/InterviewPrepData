# this Binding — Revision Notes (Intermediate)
> One-line mental model: `this` is chosen by the call site — `obj.fn()` → `obj`, `fn()` → `undefined` (strict) or global, `new` → fresh object, arrow → outer lexical value.

## Core recall
- **Call-time, not definition-time.** The same function body can have different `this` on different calls.
- **Top level:** browser script → `window`; ES module → `undefined`; Node CommonJS → `module.exports`. Use `globalThis` explicitly.
- **Plain call:** strict → `undefined`; sloppy → global object. Arrow functions never get their own `this`.
- **Method call:** `obj.fn()` → `obj`; `a.b.fn()` → `a.b`. Detaching (`const f = obj.fn`) loses the receiver.
- **Explicit binding:** `call(thisArg, ...args)` invokes with args listed; `apply(thisArg, [args])` invokes with an array; `bind(thisArg, ...leading)` returns a new function without invoking.
- **`bind` is permanent.** Later `call`/`apply` cannot override it. `new` on a bound function ignores the bound `this` but keeps bound args.
- **Constructor call:** `new` creates `{}`, links it to `Fn.prototype`, runs the body with `this = {}`, returns the object unless the constructor returns another object.
- **Classes are always strict.** Detached method → `TypeError`. Arrow class fields auto-bind per instance. Static methods get the class as `this`.
- **Lost `this`:** `setTimeout(obj.method)`, `array.map(obj.method)`, promise callbacks, IIFEs.
- **Fixes:** `fn.bind(obj)`, `() => obj.fn()`, arrow class fields, `event.currentTarget`, `map`'s `thisArg`.
- **Arrows ignore call/apply/bind** — they use the lexical `this` where they were written.

## Must-remember code
- ```javascript
  const obj = {
    name: 'Kakade',
    greet() { return `Hi ${this.name}`; },
  };
  const lost = obj.greet;
  console.log(obj.greet());              // Hi Kakade
  console.log(lost.bind(obj)());          // Hi Kakade
  ```
- ```javascript
  function introduce(city) { return `${this.name} from ${city}`; }
  console.log(introduce.call({ name: 'Asha' }, 'Pune'));
  console.log(introduce.apply({ name: 'Rohan' }, ['Mumbai']));
  const bound = introduce.bind({ name: 'Kakade' }, 'Nagpur');
  console.log(bound(), bound.call({ name: 'Asha' }));
  ```
- ```javascript
  class Button {
    label = 'Save';
    click = () => `${this.label} clicked`; // survives detaching
  }
  const b = new Button();
  const handler = b.click;
  console.log(handler()); // Save clicked
  ```

## Interview traps
- "What is `this` in a function?" — ask about call site + strict mode first; there is no single answer.
- `setTimeout(obj.method, 0)` runs with `this = undefined`/global, not `obj` → use `() => obj.method()` or `bind`.
- A nested `function` callback inside a method does **not** inherit `this`; an arrow does.
- `const f = obj.method; f()` breaks; `f.call(obj)` and `obj.method()` work.
- Arrow object methods never see the object; arrows cannot be rebound with `call`/`apply`/`bind`.
- `new` returning an object discards the constructed instance; returning a primitive does not.
- Class methods are strict even in sloppy files, so a detached method throws instead of silently using the global.
- Top-level `this` differs between classic scripts, ES modules, and CommonJS.

## 3 things to remember
1. **Read the call site:** `obj.fn()` → `obj`; `fn()` → `undefined` (strict); `new Fn()` → new object.
2. **Arrows inherit:** `const inner = () => this.name;` — no own `this`, ever.
3. **Freeze it when passing:** `setTimeout(obj.fn.bind(obj), 0)` or use an arrow class field.
