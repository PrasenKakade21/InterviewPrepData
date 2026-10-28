# Prototypes and Inheritance — Revision Notes (Intermediate)
> One-line mental model: every object links to a prototype; reads walk that chain to `null`, and `class` is syntax over the same mechanism.

## Core recall
- **Chain:** own props → `[[Prototype]]` → parent prototype → `Object.prototype` → `null`. Missing key = `undefined`.
- **`__proto__` vs `prototype`:** `obj.__proto__` / `Object.getPrototypeOf(obj)` is the object's parent link. `Fn.prototype` is what `new Fn()` installs as the instance's parent. `Fn.__proto__` is usually `Function.prototype`.
- **`Object.create(proto)`:** exact prototype, no constructor run. `Object.create(null)` has no inherited methods — safe dictionary.
- **Constructor functions:** instance data in `this`, shared methods on `Fn.prototype`. Don't create methods inside the constructor.
- **`new`:** create `{}` → link to `Fn.prototype` → run body with `this = {}` → return it unless another **object** is returned.
- **`instanceof`:** true if `Fn.prototype` is in the chain. Reassigning `Fn.prototype` orphans existing instances.
- **`hasOwnProperty`:** own only. Prefer `Object.hasOwn`; `key in obj` includes inherited; `obj[key] !== undefined` confuses missing with explicit `undefined`.
- **Inheriting:** `Child.prototype = Object.create(Parent.prototype)`, reset `constructor`, call `Parent.call(this, ...)`.
- **Prototypal inheritance:** objects inherit from objects; own props **shadow** inherited; `delete` uncovers the inherited value.
- **Mixins:** `Object.assign(target, ...sources)` copies own enumerable props by reference; class factories compose behaviour.
- **Performance:** short chains, shared prototype methods, don't mutate built-ins, avoid `setPrototypeOf` after creation, `delete` deoptimises shapes.

## Must-remember code
- ```javascript
  const parent = { greet() { return `Hi ${this.name}`; } };
  const child = Object.create(parent);
  child.name = 'Kakade';
  console.log(child.greet());            // Hi Kakade
  console.log(Object.hasOwn(child, 'greet')); // false
  ```
- ```javascript
  function Person(name) { this.name = name; }
  Person.prototype.greet = function () { return `Hi ${this.name}`; };
  function Admin(name) { Person.call(this, name); }
  Admin.prototype = Object.create(Person.prototype);
  Admin.prototype.constructor = Admin;
  console.log(new Admin('Asha') instanceof Person); // true
  ```
- ```javascript
  const canFly = { fly() { return `${this.name} flies`; } };
  const duck = Object.assign({ name: 'Duck' }, canFly);
  console.log(duck.fly(), Object.hasOwn(duck, 'fly')); // Duck flies true
  ```

## Interview traps
- `User.prototype` is **not** the prototype of `User`; it is the prototype of `new User()` instances.
- Replacing `User.prototype` after creating instances breaks `instanceof` and hides newly added methods from old instances.
- `Child.prototype = Parent.prototype` merges the two types — adding a child method leaks to every parent instance.
- Forgetting `Child.prototype.constructor = Child` makes `instance.constructor` point at the parent.
- Forgetting `Parent.call(this, ...)` leaves instance data unset even though methods inherit.
- Object spread `{ ...obj }` copies own enumerable props only and drops the prototype.
- `instanceof` can be fooled by `Symbol.hasInstance`; it is not a type tag.
- Adding to `Object.prototype` / `Array.prototype` pollutes every object and can break `for...in`.

## 3 things to remember
1. **Reads walk the chain:** `object → prototype → … → null`.
2. **Link, don't copy:** `Child.prototype = Object.create(Parent.prototype)`.
3. **Check ownership correctly:** `Object.hasOwn(obj, key)` vs `key in obj`.
