# Prototypes and Inheritance

JavaScript does not copy properties from parent to child. Instead, every object holds a hidden link to another object called its **prototype**, and property lookups walk that link. The chain is: **object → its prototype → that prototype's prototype → `null`**. Inheritance in JavaScript is simply this lookup walk, and `class` is a cleaner syntax on top of it.

```text
kakade ──► User.prototype ──► Object.prototype ──► null
  │              │                    │
 own props     shared methods      toString, etc.
```

---

# 1. The Prototype Chain

Every ordinary object has an internal `[[Prototype]]` slot. When you read a property that the object does not own, JavaScript asks the prototype, then the prototype's prototype, until it finds the property or hits `null`.

### Every object has a hidden link

```javascript
const animal = {
  eats: true,
  walk() { return `${this.name} walks`; },
};

const dog = Object.create(animal);
dog.name = 'Rohan';

console.log(dog.name);
console.log(dog.eats);
console.log(dog.walk());
```

Output:

```text
Rohan
true
Rohan walks
```

`dog` owns only `name`. `eats` and `walk` are found one level up.

### Walking the chain

```javascript
const base = { level: 1 };
const middle = Object.create(base);
const leaf = Object.create(middle);

console.log(leaf.level);
console.log(Object.getPrototypeOf(leaf) === middle);
console.log(Object.getPrototypeOf(base) === Object.prototype);
console.log(Object.getPrototypeOf(Object.prototype));
```

Output:

```text
1
true
true
null
```

Think:

```text
leaf
 └── middle
      └── base
           └── Object.prototype
                └── null
```

A missing property is not an error — it is just `undefined` at the end of the chain. The chain is checked **at read time**, so adding a property to a prototype later makes it visible to every object that inherits from it.

---

# 2. `__proto__` vs `prototype`

These two names look alike but mean different things — the most common prototype interview question.

### Two different things

- `obj.__proto__` — the prototype **of an object** (a getter/setter on `Object.prototype`).
- `Fn.prototype` — a normal property **on a function**, used as the prototype for objects created with `new Fn()`.

```javascript
function User(name) {
  this.name = name;
}

const kakade = new User('Kakade');

console.log(Object.getPrototypeOf(kakade) === User.prototype);
console.log(User.prototype.constructor === User);
console.log(kakade.__proto__ === User.prototype);
console.log(typeof User.prototype);
```

Output:

```text
true
true
true
object
```

`Asha.__proto__` is `Function.prototype`, because `Asha` itself is a function object. `Asha.prototype` is the object instances inherit from. A function has **both** links, pointing to different places.

| Expression | Means |
| --- | --- |
| `obj.__proto__` | prototype link of that object |
| `Object.getPrototypeOf(obj)` | safe, standard way to read it |
| `Fn.prototype` | object used as prototype by `new Fn()` |
| `Fn.__proto__` | `Function.prototype` (functions are objects too) |

> **Trap:** `User.prototype` is not the prototype of `User`. It is the prototype of `new User()` instances.

---

# 3. `Object.create`

`Object.create(proto)` builds a new object with an exact prototype link and **no constructor call**. It is the cleanest way to demonstrate inheritance.

### Creating without a constructor

```javascript
const person = {
  greet() { return `Hi ${this.name}`; },
};

const kakade = Object.create(person);
kakade.name = 'Kakade';

console.log(kakade.greet());
console.log(Object.keys(kakade));
```

Output:

```text
Hi Kakade
[ 'name' ]
```

`greet` is inherited, so it is not an own key.

A null-prototype object (`Object.create(null)`) has no `toString`, `hasOwnProperty`, or `__proto__`. That makes it a safe dictionary for untrusted keys (the classic prototype-pollution defence).

---

# 4. Constructor Functions and `new`

Before `class`, inheritance was built with constructor functions plus manual prototype wiring.

### The constructor pattern

```javascript
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  return `Hi ${this.name}`;
};

const kakade = new User('Kakade');

console.log(kakade.greet());
console.log(Object.hasOwn(kakade, 'name'));
console.log(Object.hasOwn(kakade, 'greet'));
```

Output:

```text
Hi Kakade
true
false
```

Methods go on `User.prototype` so all instances share **one** function, not one copy each.

```text
new User('Kakade')
   ├── 1. create a fresh empty object {}
   ├── 2. set its [[Prototype]] to User.prototype
   ├── 3. call User with this = that object
   └── 4. return that object (unless User returns another object)
```

`kakade.constructor` works because `User.prototype.constructor` points back to `User`; it is inherited, not copied.

---

# 5. `instanceof` and `hasOwnProperty`

### `instanceof` walks the chain

`obj instanceof Fn` asks one question: **does `Fn.prototype` appear anywhere in `obj`'s prototype chain?**

```javascript
class Animal {}
class Dog extends Animal {}

const rohan = new Dog();

console.log(rohan instanceof Dog);
console.log(rohan instanceof Animal);
console.log(rohan instanceof Object);
console.log({} instanceof Dog);
```

Output:

```text
true
true
true
false
```

`instanceof` checks the **prototype chain**, not the constructor that ran. If you replace `Fn.prototype` after creating an instance, the old instance stops matching.

### Own vs inherited properties

```javascript
const person = { role: 'member' };
const kakade = Object.create(person);
kakade.name = 'Kakade';

console.log(kakade.name, kakade.role);
console.log(Object.hasOwn(kakade, 'name'));
console.log(Object.hasOwn(kakade, 'role'));
console.log(kakade.hasOwnProperty('role'));
```

Output:

```text
Kakade member
true
false
false
```

`hasOwnProperty` only checks the object itself. Prefer `Object.hasOwn(obj, key)` (ES2022) — it works on null-prototype objects and cannot be shadowed by a property named `hasOwnProperty`. Use `key in obj` when inherited properties should count, and avoid `obj[key] !== undefined` because it confuses a missing key with a key set to `undefined`.

---

# 6. Inheriting Methods

To inherit with constructor functions you chain the prototypes and then repair `constructor`.

### Setting up the chain

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function () {
  return `Hi ${this.name}`;
};

function Admin(name) {
  Person.call(this, name);
}

Admin.prototype = Object.create(Person.prototype);
Admin.prototype.constructor = Admin;
Admin.prototype.deleteUser = function () {
  return `${this.name} deleted a user`;
};

const asha = new Admin('Asha');

console.log(asha.greet());
console.log(asha.deleteUser());
console.log(asha instanceof Admin, asha instanceof Person);
```

Output:

```text
Hi Asha
Asha deleted a user
true true
```

**Why each line matters**

`Person.call(this, name)` runs the parent setup on the instance. `Admin.prototype = Object.create(Person.prototype)` links the child prototype to the parent. `Admin.prototype.constructor = Admin` repairs the constructor pointer.

Bad — assigning the parent prototype directly:

```javascript
Admin.prototype = Person.prototype; // same object!
Admin.prototype.deleteUser = function () {};
```

Now `Admin.prototype === Person.prototype`, so every `Person` instance can call `deleteUser`, and `asha.constructor === Person`. Both `instanceof` checks still pass — the chain is intact — but the two types are no longer separable. Good: keep the `Object.create(Person.prototype)` version above, which gives `Admin` its own prototype object.

> **Trap:** forgetting `Person.call(this, name)` means `this.name` is never set — methods still inherit, but instance data is missing.

---

# 7. Prototypal Inheritance

The same chain can be built without constructors. This is **prototypal inheritance**: objects inherit directly from objects.

### Object-to-object inheritance

```javascript
const speaker = {
  speak() { return `${this.name} speaks`; },
};

const kakade = Object.create(speaker);
kakade.name = 'Kakade';

const asha = Object.create(speaker);
asha.name = 'Asha';

console.log(kakade.speak());
console.log(asha.speak());
console.log(Object.getPrototypeOf(kakade) === speaker);
```

Output:

```text
Kakade speaks
Asha speaks
true
```

No classes, no `new`, no constructor. `speaker` is a shared method holder.

### Shadowing and overriding

```javascript
const parent = { greet: () => 'parent' };
const child = Object.create(parent);
child.greet = () => 'child';

console.log(child.greet(), parent.greet());
delete child.greet;
console.log(child.greet());
```

Output:

```text
child parent
parent
```

An own property **shadows** an inherited one. `delete` removes the own property and uncovers the inherited version again.

---

# 8. Mixins

A mixin copies behaviour from one or more source objects into a target. JavaScript has single prototype inheritance, so mixins are the usual way to share extra capabilities.

### Object.assign mixins

```javascript
const canFly = { fly() { return `${this.name} flies`; } };
const canSwim = { swim() { return `${this.name} swims`; } };

const duck = { name: 'Duck' };
Object.assign(duck, canFly, canSwim);

console.log(duck.fly());
console.log(duck.swim());
console.log(Object.hasOwn(duck, 'fly'));
```

Output:

```text
Duck flies
Duck swims
true
```

`Object.assign` copies **own enumerable** properties by value. Methods become own properties of `duck`, not prototype methods. Class factories (`(Base) => class extends Base {}`) compose the same way for classes.

> **Trap:** `Object.assign` copies references. A shared array on a mixin is shared by every target unless you clone it per instance.

---

# 9. `Object.getPrototypeOf` and Performance

### Inspecting prototypes and performance

```javascript
const shared = { greet() { return 'hi'; } };
const a = Object.create(shared);
const b = Object.create(shared);

console.log(Object.getPrototypeOf(a) === shared);
console.log(a.greet === b.greet);
```

Output:

```text
true
true
```

`Object.getPrototypeOf` is the standard read; `obj.__proto__` is legacy and slower. `Object.setPrototypeOf` changes the link after creation, which can deoptimise the object.

- Property lookup walks the chain, so **short chains are faster**.
- Put shared methods on the prototype: both instances above share one function.
- Avoid adding properties to built-in prototypes and avoid `delete`, which deoptimises object shapes.

---

# Real-World Uses

- **`class`** is prototype inheritance with syntax sugar: methods go on `Class.prototype`.
- **Library extension**: `Dog.prototype = Object.create(Animal.prototype)` is what `extends` compiles down to.
- **Safe dictionaries**: `Object.create(null)` for maps keyed by user input, preventing prototype pollution.
- **Feature detection**: `'fetch' in globalThis` walks the prototype chain.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| Prototype chain | Read walks `obj → proto → proto… → null` |
| `__proto__` vs `prototype` | `__proto__` is an object's link; `prototype` is a function's template for `new` |
| `Object.create` | New object with an exact prototype, no constructor run |
| Constructor functions | Set instance data in the function, methods on `.prototype` |
| `new` operator | Create, link, run with `this`, return object unless another object is returned |
| `instanceof` | True if `Fn.prototype` is anywhere in the chain |
| `hasOwnProperty` | Own only; prefer `Object.hasOwn` |
| Inheriting methods | Chain prototypes and reset `constructor` |
| Prototypal inheritance | Objects inherit directly from objects; own props shadow inherited |
| Mixins | Copy behaviour in; `Object.assign` or class factories |
| `Object.getPrototypeOf` | Standard read; `setPrototypeOf` sparingly |
| Performance notes | Short chains, shared prototype methods, avoid mutating built-ins |

## The Mental Model

```text
                 Object.create(proto)
                          │
                          ↓
   instance ──► Child.prototype ──► Parent.prototype ──► Object.prototype ──► null
      │               │                    │                    │
   own data      shared methods      shared methods        toString, etc.
      │
   new Child()
      │
   instanceof walks this line →
```

### The 3 things to remember

**1. Lookup walks the chain:**

```javascript
const child = Object.create({ inherited: true });
console.log(child.inherited); // true
```

**2. `prototype` belongs to functions, `__proto__` to objects:**

```javascript
function User() {}
const u = new User();
console.log(u.__proto__ === User.prototype); // true
```

**3. `instanceof` checks the chain, not the constructor:**

```javascript
console.log(new User() instanceof User); // true
```

> **JavaScript inheritance is a chain of objects: reads walk up to `null`, `Fn.prototype` is the link `new` installs, and `instanceof` simply searches that chain.**
