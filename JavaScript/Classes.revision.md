# Classes — Revision Notes (Intermediate)
> One-line mental model: a class is a strict, non-hoisted constructor whose methods live on `Class.prototype`; `new` links instances, `extends` links prototypes.

## Core recall
- **Syntax:** `class User { ... }`; `typeof User === "function"`; TDZ, always strict, no call without `new`.
- **`constructor`:** one per class; runs per `new`; omitted → empty (derived: `super(...args)`). Returning an **object** replaces the instance.
- **Instance methods:** one shared function on `Class.prototype`, non-enumerable, `this` = receiver.
- **Static members:** on the constructor, inherited by subclasses; `this` = called class. `static {}` runs once.
- **Getters/setters:** accessor pair on the prototype; use a private backing field to avoid recursion. A same-named field silently shadows the accessor.
- **Private fields:** `#x` must be declared; language-enforced; invisible to `Object.keys`/JSON/bracket access; `#x in obj` brand check.
- **`extends` / `super`:** `super(...)` before `this`; `super.method()` calls the parent method with current `this`; statics inherit.
- **`instanceof`:** true if `Class.prototype` is in the chain; `Symbol.hasInstance` can override; replacing a prototype orphans old instances.
- **Class fields:** per instance, initialised in declaration order before the constructor body (after `super()`). Arrow fields capture `this`.
- **Classes vs prototypes:** classes add strictness, TDZ, non-enumerable methods, `super`, and real `#` privacy.
- **Traps:** detached method loses `this`; field shadows accessor; private brand errors; `this` before `super()`.

## Must-remember code
- ```javascript
  class User {
    static count = 0;
    #role = 'member';
    constructor(name) { this.name = name; User.count += 1; }
    get role() { return this.#role; }
    set role(v) { this.#role = v; }
    greet() { return `Hi ${this.name} (${this.#role})`; }
  }
  const k = new User('Kakade');
  k.role = 'admin';
  console.log(k.greet(), User.count); // Hi Kakade (admin) 1
  ```
- ```javascript
  class Animal {
    constructor(name) { this.name = name; }
    speak() { return `${this.name} makes a sound`; }
  }
  class Dog extends Animal {
    speak() { return `${super.speak()} — woof`; }
  }
  console.log(new Dog('Rohan').speak());
  ```
- ```javascript
  class Button {
    label = 'Save';
    click = () => `${this.label} clicked`;
  }
  const handler = new Button().click;
  console.log(handler()); // Save clicked
  ```
- ```javascript
  class HttpError extends Error {
    constructor(status, message) { super(message); this.name = 'HttpError'; this.status = status; }
  }
  ```

## Interview traps
- `handler = instance.method; handler()` → `this` is `undefined` (strict) → `TypeError`.
- `class Bad { name = 'x'; get name() { ... } }` → field shadows the accessor and construction throws.
- `#x` is not `this['#x']`; bracket access returns `undefined`, dot access outside the class is a syntax error.
- Derived constructor using `this` before `super()` → `ReferenceError`.
- `Child.prototype = Parent.prototype` merges both types; use `Object.create(Parent.prototype)`.
- Replacing `Class.prototype` after instances exist breaks `instanceof` for those instances.
- Static `this` is the class, not an instance — `static create() { return new this(); }`.
- `Object.keys(instance)` shows fields only, never prototype methods.

## 3 things to remember
1. **Methods are shared:** they live on `Class.prototype`, not on each instance.
2. **`super()` first:** derived constructors cannot touch `this` until the parent runs.
3. **Bind at definition:** arrow class fields keep `this` when methods are passed around.
