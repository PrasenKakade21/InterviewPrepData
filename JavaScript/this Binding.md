# this Binding

`this` is not a variable you assign — it is a **context handed to a function at call time**. It does not mean "the function" or "the object where it was written". The rule is: **where the function is defined → how it is called → what `this` becomes**.

```text
Function definition
        ↓
Call site  (what is before the dot? / new? / call, apply, bind?)
        ↓
this value for this one call
        ↓
Arrow function? → ignore all of it, inherit the outer this
```

---

# 1. Global and undefined `this`

`this` is fixed **per call**, not per function. The same body can see different values on different lines.

### Top-level `this`

At the top of a browser classic script, `this` is `window`; in an ES module it is `undefined`; in a Node CommonJS file it is `module.exports`. Use `globalThis` when you genuinely need the global object.

```javascript
console.log(this === undefined);
```

Output (ES module):

```text
true
```

### Plain function calls

A plain call has nothing before the function name — no dot, no `new`, no `call`. In **strict mode** `this` is `undefined`; in **sloppy mode** it is replaced by the global object.

```javascript
'use strict';

function plain() {
  return this;
}

console.log(plain());
console.log(plain.call(globalThis) === globalThis);
```

Output:

```text
undefined
true
```

Without `'use strict'`, the same call prints `true` for `this === globalThis`.

Notice:

```text
plain()  strict → undefined
plain()  sloppy → globalThis
```

---

# 2. Method Calls

When you call `object.method()`, the object **before the final dot** becomes `this`. This is **implicit binding**.

### The receiver is the object before the dot

```javascript
const kakade = {
  name: 'Kakade',
  greet() {
    return `Hi ${this.name}`;
  },
};

console.log(kakade.greet());
```

Output:

```text
Hi Kakade
```

For `company.team.report()`, `this` is `company.team`, never `company` — only the object before the **final** dot counts.

---

# 3. Arrow Functions

Arrow functions have **no `this` of their own**. They look it up where they were **written**, like any other variable. This is **lexical `this`**.

### Arrows capture the enclosing `this`

```javascript
const kakade = {
  name: 'Kakade',
  greetLater() {
    const inner = () => `Hi ${this.name}`;
    return inner();
  },
};

console.log(kakade.greetLater());
```

Output:

```text
Hi Kakade
```

A normal nested function gets its own `this` instead, so `inner()` above would print `Hi nobody` — the function is called without a receiver.

### Bad vs Good: arrow as an object method

Bad — the arrow captures module/global `this`, never the object:

```javascript
const bad = { name: 'Rohan', greet: () => this?.name ?? 'nobody' };
console.log(bad.greet());
```

Output:

```text
nobody
```

Good — use method shorthand when you want the receiver:

```javascript
const good = { name: 'Rohan', greet() { return this.name; } };
console.log(good.greet());
```

Output:

```text
Rohan
```

### Arrows ignore `call`, `apply` and `bind`

```javascript
const kakade = {
  name: 'Kakade',
  makeArrow() {
    return () => this.name;
  },
};

const arrow = kakade.makeArrow();

console.log(arrow.call({ name: 'Asha' }));
console.log(arrow.bind({ name: 'Rohan' })());
```

Output:

```text
Kakade
Kakade
```

The arrow is permanently tied to `kakade`, because that was `this` when it was created.

> Use arrows for callbacks that should inherit `this`; use methods when the function needs its own receiver.

---

# 4. Explicit Binding: `call`, `apply` and `bind`

Every normal function has three methods that let you **choose** `this`.

### `call` and `apply`

```javascript
function introduce(city, role) {
  return `${this.name} from ${city} is a ${role}`;
}

const kakade = { name: 'Kakade' };
const asha = { name: 'Asha' };

console.log(introduce.call(kakade, 'Pune', 'developer'));
console.log(introduce.apply(asha, ['Mumbai', 'designer']));
```

Output:

```text
Kakade from Pune is a developer
Asha from Mumbai is a designer
```

`call` takes arguments one by one; `apply` takes one array.

### `bind` returns a new function

```javascript
const rohan = { name: 'Rohan' };
const rohanIntro = introduce.bind(rohan, 'Nagpur');

console.log(rohanIntro('tester'));
console.log(rohanIntro('mentor'));
```

Output:

```text
Rohan from Nagpur is a tester
Rohan from Nagpur is a mentor
```

`bind` does not call anything. It locks `this` (and optionally leading arguments) permanently.

### Bound `this` cannot be rebound

```javascript
const bound = function () {
  return this.name;
}.bind({ name: 'Kakade' });

console.log(bound(), bound.call({ name: 'Asha' }), bound.apply({ name: 'Rohan' }));
```

Output:

```text
Kakade Kakade Kakade
```

| Method | First argument | Rest | Returns |
| --- | --- | --- | --- |
| `call` | `this` | listed | function result |
| `apply` | `this` | one array | function result |
| `bind` | `this` | listed | new function |

---

# 5. Constructor Calls

With `new`, JavaScript creates a fresh object and uses it as `this`. This is **`new` binding**, and it beats implicit binding.

### What `new` does

```javascript
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  return `Hi ${this.name}`;
};

const kakade = new User('Kakade');

console.log(kakade.name);
console.log(kakade.greet());
console.log(kakade instanceof User);
```

Output:

```text
Kakade
Hi Kakade
true
```

Think:

```text
new User('Kakade')
   ├── 1. create {}
   ├── 2. link {} → User.prototype
   ├── 3. run User with this = {}
   └── 4. return {} unless the constructor returns an object
```

### Returning an object overrides `this`

```javascript
function Weird() {
  this.name = 'Kakade';
  return { name: 'Asha' };
}

console.log(new Weird().name);
```

Output:

```text
Asha
```

A returned **primitive** is ignored; only an **object** replaces the new instance. Favourite interview trap.

### `new` ignores a bound `this` but keeps bound arguments

```javascript
function Point(x, y) {
  this.x = x;
  this.y = y;
}

const BoundPoint = Point.bind({ x: 99, y: 99 }, 1);
const p = new BoundPoint(2);

console.log(p.x, p.y, p instanceof Point);
```

Output:

```text
1 2 true
```

---

# 6. `this` in Classes

Class bodies are **always strict**, even in a sloppy file. A detached method sees `this === undefined` and usually throws.

### Methods are strict and live on the prototype

```javascript
class Counter {
  constructor() { this.count = 0; }
  increment() { this.count += 1; return this.count; }
}

const c = new Counter();
const inc = c.increment;

console.log(c.increment());
try {
  inc();
} catch (error) {
  console.log(error.constructor.name);
}
```

Output:

```text
1
TypeError
```

### Arrow class fields auto-bind

```javascript
class Button {
  label = 'Save';
  click = () => `${this.label} clicked`;
}

const b = new Button();
const handler = b.click;

console.log(handler());
```

Output:

```text
Save clicked
```

The arrow field is created **per instance** with `this` already bound, so it survives `setTimeout` and `addEventListener`.

### Static methods use the class as `this`

```javascript
class User {
  static role = 'member';
  static describe() { return `Role: ${this.role}`; }
}
console.log(User.describe());
```

Output:

```text
Role: member
```

Inside `static` methods `this` is the constructor itself, so `this.role` reads the static field, not an instance field.

---

# 7. Event Handlers

For a normal function listener, the DOM sets `this` to the element that received the event. An arrow listener keeps the outer `this`.

```html
<button id="save">Save</button>
<script>
  const button = document.querySelector('#save');

  button.addEventListener('click', function () {
    console.log(this === button); // true — DOM sets this
  });

  button.addEventListener('click', () => {
    console.log(this === button); // false — lexical this
  });
</script>
```

Output after two clicks:

```text
true
false
```

In real code prefer `event.currentTarget`. It works for arrows too and survives refactoring.

---

# 8. Lost `this`

"Lost `this`" happens whenever a method is **detached** and called by something else. It is the most common `this` bug in production.

```javascript
const kakade = {
  name: 'Kakade',
  greet() { return `Hi ${this.name}`; },
};

const greet = kakade.greet;

try {
  console.log(greet());
} catch (error) {
  console.log(error.constructor.name);
}
```

Output (strict):

```text
TypeError
```

### Bad vs Good with `setTimeout`

Bad — passing the method directly:

```javascript
const asha = { name: 'Asha', greet() { console.log(`Hi ${this.name}`); } };
setTimeout(asha.greet, 0);
```

Output:

```text
Hi undefined
```

Good — wrapper or `bind`:

```javascript
const rohan = { name: 'Rohan', greet() { console.log(`Hi ${this.name}`); } };
setTimeout(() => rohan.greet(), 0);
setTimeout(rohan.greet.bind(rohan), 0);
```

Output:

```text
Hi Rohan
Hi Rohan
```

| Situation | Fix |
| --- | --- |
| Callback needs the object | `fn.bind(obj)` |
| One-off inline callback | `() => obj.fn()` |
| DOM listener | `event.currentTarget` |

---

# 9. Common Traps

### Trap 1: nested `function` inside a method

```javascript
const team = {
  name: 'Platform',
  members: ['Kakade', 'Asha'],
  list() {
    return this.members.map(function (member) {
      return `${member} on ${this?.name ?? 'nothing'}`;
    });
  },
};

console.log(team.list());
```

Output:

```text
[ 'Kakade on nothing', 'Asha on nothing' ]
```

`map` calls the callback as a plain function. Fix with an arrow:

```javascript
list() {
  return this.members.map((member) => `${member} on ${this.name}`);
}
```

### Trap 2: IIFE resets `this`

```javascript
const counter = {
  count: 0,
  start() {
    (function () { this.count += 1; })();
    return this.count;
  },
};

console.log(counter.start());
```

Output (strict):

```text
0
```

The IIFE has its own `this` (`undefined` in strict, global in sloppy). Use an arrow IIFE or capture `const self = this`.

### Trap 3: `this` inside a `promise.then` callback

```javascript
const loader = {
  status: 'idle',
  load() {
    return Promise.resolve().then(function () {
      this.status = 'done'; // sloppy: this = globalThis
      return this.status;
    });
  },
};

console.log(loader.load());
console.log(loader.status);
```

Output (sloppy CommonJS):

```text
Promise { <pending> }
idle
```

The promise eventually resolves to `'done'` because the callback wrote to `globalThis.status`, while `loader.status` never changed. In strict mode (or an ES module) the same callback throws `TypeError`. Use an arrow:

```javascript
load() {
  return Promise.resolve().then(() => {
    this.status = 'done';
    return this.status;
  });
}
```

---

# Real-World Uses

- **Event handlers** — `this` is the element for `function` listeners; prefer `event.currentTarget`.
- **Controllers** — arrow class fields keep `this` when handlers are passed down.
- **Partial application** — `const vat = addTax.bind(null, 0.2);`.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| Global and undefined `this` | Top-level module and strict plain calls give `undefined` |
| Method calls | Object before the final dot wins |
| Plain function calls | No receiver → `undefined` (strict) or global (sloppy) |
| Arrow functions | No own `this`; capture the enclosing lexical value |
| `call`, `apply`, `bind` | Args list, args array, new function with locked `this` |
| Constructor calls | `new` creates the object and makes it `this` |
| Event handlers | DOM binds `this` to the element for normal functions |
| Lost `this` | Detached method loses its receiver; `bind` or wrap it |
| Explicit binding | `bind` cannot be overridden by later `call`/`apply` |
| `this` in classes | Always strict; detached method throws; arrow fields bind |
| Common traps | Nested callbacks, IIFEs, `setTimeout(obj.method)`, top-level `this` |

## The Mental Model

```text
                    this
                      │
        ┌─────────────┼──────────────┐
        ↓             ↓              ↓
   where written   how called     new Fn()
        │             │              │
   arrow? yes     obj.fn()       new object
        │             │
   outer this       obj
        │
        └── no own this, ever
                      │
              call / apply / bind
                      │
                 explicit this
                      │
                 bind = permanent
```

### The 3 things to remember

**1. Read the call site, not the definition:**

```javascript
const fn = kakade.greet; // detached
fn();                    // this = undefined (strict)
kakade.greet();          // this = kakade
```

**2. Arrows inherit, they do not bind:**

```javascript
const obj = { name: 'Kakade', greet() { return (() => this.name)(); } };
console.log(obj.greet()); // Kakade
```

**3. Freeze `this` when passing a method around:**

```javascript
setTimeout(kakade.greet.bind(kakade), 0);
```

> **`this` is decided by the call site — `obj.fn()` gives the object, plain calls give `undefined` in strict mode, `new` gives a fresh object, and arrows borrow the outer `this` instead.**
