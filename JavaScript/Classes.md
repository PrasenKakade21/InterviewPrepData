# Classes

A class is a **template for objects**: it packages data and behaviour, and `new` builds instances from it. Under the syntax, JavaScript still uses prototypes — **class → prototype methods → instance objects**.

Think:

```text
class User { ... }
        ↓
new User('Kakade')
        ↓
instance ──► User.prototype ──► Object.prototype ──► null
   name          greet()            toString()
```

---

# 1. Class Syntax

A class is declared with `class Name { ... }` or created as an expression. The body contains methods, fields, and at most one constructor.

### Declaring a class

```javascript
class User {
  greet() {
    return 'Hi from User';
  }
}

const kakade = new User();

console.log(kakade.greet());
console.log(typeof User);
console.log(User.name);
```

Output:

```text
Hi from User
function
User
```

`typeof` a class is `"function"` because a class **is** a constructor function with extra rules. Class declarations live in the **temporal dead zone** until evaluated, class bodies are always strict mode, and calling a class without `new` throws `TypeError`.

---

# 2. The `constructor`

The constructor is the special method that runs on `new`. It receives the arguments and sets up instance state. A class can have **only one** constructor.

### Running setup once per instance

```javascript
class User {
  constructor(name, role) {
    this.name = name;
    this.role = role;
  }
}

const kakade = new User('Kakade', 'admin');

console.log(kakade.name, kakade.role);
console.log(Object.keys(kakade));
```

Output:

```text
Kakade admin
[ 'name', 'role' ]
```

If you omit the constructor, JavaScript supplies an empty one. For a derived class the default is `constructor(...args) { super(...args); }`.

> **Trap:** returning an **object** from a constructor replaces the new instance (a returned primitive is ignored). Avoid it — it breaks `instanceof`.

---

# 3. Instance Methods

Methods declared in the class body are **not** copied onto each instance. They live once on `Class.prototype` and every instance reaches them through the prototype chain.

### Methods live on the prototype

```javascript
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hi ${this.name}`;
  }
}

const kakade = new User('Kakade');
const asha = new User('Asha');

console.log(kakade.greet(), asha.greet());
console.log(kakade.greet === asha.greet);
console.log(Object.hasOwn(kakade, 'greet'));
```

Output:

```text
Hi Kakade Hi Asha
true
false
```

One shared function, many receivers. `this` is whichever instance is before the dot. Class methods are **non-enumerable** (`Object.keys(User.prototype)` is `[]`), so `for...in` over an instance does not list them; constructor-function methods assigned with `= function` are enumerable.

---

# 4. Static Methods

`static` members belong to the **class itself**, not to instances. They are useful for factories, validation, and helpers that do not need a particular object.

### Static methods and fields

```javascript
class User {
  static count = 0;

  constructor(name) {
    this.name = name;
    User.count += 1;
  }

  static create(name) {
    return new this(name);
  }
}

const kakade = User.create('Kakade');

console.log(kakade.name, User.count);
console.log(typeof kakade.create);
```

Output:

```text
Kakade 1
undefined
```

Inside a static method, `this` is the class (or the subclass when inherited), which is why `new this(name)` works for subclasses too. A `static {}` block runs once when the class is evaluated and can initialise private static state.

---

# 5. Getters and Setters

`get` and `set` define **accessor properties**. Reading runs the getter; assigning runs the setter. They look like fields but are functions on the prototype.

### Accessor pairs

```javascript
class Temperature {
  #celsius = 0;

  get celsius() {
    return this.#celsius;
  }

  set celsius(value) {
    this.#celsius = value;
  }

  get fahrenheit() {
    return (this.#celsius * 9) / 5 + 32;
  }
}

const t = new Temperature();
t.celsius = 25;

console.log(t.celsius, t.fahrenheit);
```

Output:

```text
25 77
```

### Bad vs Good: setter recursion

Bad — the setter assigns to the same property it defines:

```javascript
class Bad {
  set name(value) {
    this.name = value; // RangeError: Maximum call stack size exceeded
  }
}
```

Good — store in a different backing field:

```javascript
class Good {
  set name(value) {
    this._name = value.trim();
  }
  get name() {
    return this._name;
  }
}

const good = new Good();
good.name = '  Kakade  ';
console.log(good.name);
```

Output:

```text
Kakade
```

A getter/setter pair is **one property**. A same-named class field is allowed but silently **shadows** it, so the accessor never runs — use a private backing field instead.

---

# 6. Private Fields

A `#` prefix makes a field or method truly private: it is enforced by the language, not by convention.

### `#` fields and brand checks

```javascript
class Account {
  #balance = 0;

  constructor(owner) {
    this.owner = owner;
  }

  deposit(amount) {
    this.#balance += amount;
    return this.#balance;
  }

  static isAccount(value) {
    return #balance in value;
  }
}

const asha = new Account('Asha');
asha.deposit(100);

console.log(asha.owner);
console.log(Account.isAccount(asha), Account.isAccount({}));
```

Output:

```text
Asha
true false
```

`account.#balance` outside the class is a **syntax error**, not a runtime lookup, and `account['#balance']` is just an ordinary missing string key (`undefined`). Private names are lexically scoped to the class body; `#balance in obj` is the supported brand check. A private field must be **declared** in the class body — `this.#x = 1` without a declaration is a `SyntaxError`.

---

# 7. `extends` and `super`

`extends` sets up prototype inheritance: the subclass prototype links to the parent prototype, and the subclass constructor links to the parent constructor.

### Constructor chaining

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
}

const rohan = new Dog('Rohan', 'lab');

console.log(rohan.name, rohan.breed);
console.log(rohan instanceof Dog, rohan instanceof Animal);
```

Output:

```text
Rohan lab
true true
```

In a derived constructor you **must** call `super()` before touching `this`. JavaScript will not create the instance until the parent constructor runs.

```javascript
class Bad extends Animal {
  constructor(name) {
    this.name = name; // ReferenceError
    super(name);
  }
}
```

### `super.method()`

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  speak() {
    return `${super.speak()} — woof`;
  }
}

console.log(new Dog('Asha').speak());
```

Output:

```text
Asha makes a sound — woof
```

`super.speak()` looks the method up on the **parent prototype** but still calls it with the current `this`. Static members are inherited the same way, with `this` set to the subclass that was called.

---

# 8. `instanceof`

`instanceof` checks whether `Class.prototype` appears in the object's prototype chain. It confirms the **lineage**, not how the object was built.

### Chain check

```javascript
class Animal {}
class Dog extends Animal {}

const kakade = new Dog();

console.log(kakade instanceof Dog);
console.log(kakade instanceof Animal);
console.log(kakade instanceof Object);
console.log({} instanceof Dog);
```

Output:

```text
true
true
true
false
```

`Symbol.hasInstance` lets a class customise the answer: `class Even { static [Symbol.hasInstance](v) { return Number.isInteger(v) && v % 2 === 0; } }` makes `4 instanceof Even` true and `5 instanceof Even` false.

> **Trap:** after `Dog.prototype = {}`, old `Dog` instances fail `instanceof Dog`. Never replace a prototype after instances exist.

---

# 9. Class Fields

Class fields are declared directly in the body. Instance fields are created **per object**, in declaration order, before the constructor body runs (after `super()` in a derived class).

### Field order and arrow binding

```javascript
class User {
  role = 'member';
  active = true;

  constructor(name) {
    this.name = name;
  }
}

console.log(Object.keys(new User('Kakade')));
```

Output:

```text
[ 'role', 'active', 'name' ]
```

Fields are initialised before constructor assignments, so `role` and `active` appear first.

An arrow function field is created per instance with `this` already bound:

```javascript
class Button {
  label = 'Save';
  click = () => `${this.label} clicked`;
}

const handler = new Button().click;
console.log(handler());
```

Output:

```text
Save clicked
```

This is the standard fix for passing methods as callbacks, at the cost of one function per instance.

---

# 10. Classes vs Prototypes

Classes do not replace prototypes — they **are** prototypes with clearer syntax and stricter rules.

### Same mechanism, cleaner syntax

```javascript
class User {
  greet() { return 'hi'; }
}

console.log(Object.getPrototypeOf(User.prototype) === Object.prototype);
console.log(User.prototype.greet === new User().greet);
```

Output:

```text
true
true
```

The class method lives on `User.prototype` and instances reach it through the chain — exactly what assigning `User.prototype.greet = function () {}` does manually.

| Feature | `class` | Constructor function |
| --- | --- | --- |
| Hoisting | TDZ, not usable before declaration | Function declarations hoist |
| Strict mode | Always | Follows the file |
| Call without `new` | `TypeError` | Runs with global/undefined `this` |
| Methods | Non-enumerable, on prototype | Enumerable when assigned |
| `super` | Built in | Manual `.call` and prototype wiring |
| Private fields | `#x` | Closures or `WeakMap` |

---

# 11. Common Traps

### Trap 1: detached method loses `this`

```javascript
class Counter {
  count = 0;
  increment() {
    this.count += 1;
    return this.count;
  }
}

const counter = new Counter();
const inc = counter.increment;

try {
  inc();
} catch (error) {
  console.log(error.constructor.name);
}
```

Output:

```text
TypeError
```

Fix with an arrow field (`increment = () => ...`) or `counter.increment.bind(counter)`.

A field and an accessor with the same name do not throw — the field silently **shadows** the accessor, so the getter/setter never run. Use a private backing field (`#name`) or rename one of them.

---

# Real-World Uses

- **Domain models**: `class User`, `class Order`, `class Invoice` with validation in the constructor.
- **Framework base classes**: React class components, `Error` subclasses.
- **Custom errors**: `class HttpError extends Error { constructor(status, message) { super(message); this.status = status; } }`.
- **Bound UI handlers**: arrow class fields passed to `addEventListener`.
- **Encapsulation**: `#` fields for invariants that must not be mutated from outside.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| Class syntax | `class User {}`; `typeof` is `"function"`; TDZ, no hoisting |
| `constructor` | Runs once per `new`; only one per class; returning an object replaces the instance |
| Instance methods | Shared on `Class.prototype`, non-enumerable, `this` = receiver |
| Static methods | On the class itself; `this` is the class/subclass; `static {}` runs once |
| Getters and setters | Accessor pair on the prototype; use a backing field to avoid recursion |
| Private fields | `#x` must be declared; brand-checked; invisible to `Object.keys` |
| `extends` and `super` | `super()` before `this`; `super.method()` keeps `this` |
| `instanceof` | Checks `Class.prototype` in the chain; customisable via `Symbol.hasInstance` |
| Class fields | Per instance, initialised in order before constructor body |
| Classes vs prototypes | Same mechanism; classes add strictness, TDZ, non-enumerable methods |
| Common traps | Detached method, accessor/field collision, private brand, `this` before `super()` |

## The Mental Model

```text
                    class
                      │
        ┌─────────────┼──────────────┐
        ↓             ↓              ↓
   constructor    prototype       static
        │             │              │
    instance      shared methods   class helpers
        │             │
        └──────► instance ──► Class.prototype ──► Parent.prototype ──► null
                                      │
                              getters, #private
```

### The 3 things to remember

**1. Methods live on the prototype:**

```javascript
class User { greet() { return 'hi'; } }
console.log(Object.hasOwn(new User(), 'greet')); // false
```

**2. `super()` comes before `this`:**

```javascript
class Dog extends Animal {
  constructor(name) { super(name); this.name = name; }
}
```

**3. Arrow fields keep `this`:**

```javascript
class Button { click = () => this.label; }
```

> **A class is a strict, non-hoisted constructor function whose methods live on `Class.prototype` — `new` links instances to it, `extends` links prototypes, and `#` fields are real encapsulation.**
