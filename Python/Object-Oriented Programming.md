# Python Object-Oriented Programming (OOP)

OOP is a way of organizing code around **objects** that contain both **data (attributes)** and **behavior (methods)**.

A simple mental model:

```text
Class → blueprint
Object → actual thing created from the blueprint

Attributes → what the object has
Methods    → what the object can do
```

For example, imagine a website called `kakade.dev`:

```python
class User:
    def __init__(self, name):
        self.name = name

    def greet(self):
        return f"Hello, {self.name}"
```

```python
user = User("Kakade")

print(user.name)
print(user.greet())
```

Output:

```text
Kakade
Hello, Kakade
```

---

# 1. Classes

A **class** is a blueprint for creating objects.

```python
class User:
    pass
```

You can put attributes and methods inside it:

```python
class User:
    def greet(self):
        print("Hello!")
```

A class itself doesn't necessarily represent one specific user. It defines what a `User` object should look like and do.

Think:

```text
Class User
│
├── attributes
│   ├── name
│   └── email
│
└── methods
    ├── login()
    └── logout()
```

---

# 2. Objects

An **object** is an instance of a class.

```python
class User:
    pass

user1 = User()
user2 = User()
```

Here:

```text
User → class
user1 → object
user2 → object
```

You can create many objects from the same class:

```python
user1 = User()
user2 = User()
user3 = User()
```

Each object is a separate instance.

---

# 3. `__init__`

`__init__` is used to **initialize an object when it is created**.

```python
class User:
    def __init__(self, name):
        self.name = name
```

Now:

```python
user = User("Kakade")
```

Python automatically calls:

```python
__init__(user, "Kakade")
```

So:

```python
print(user.name)
```

gives:

```text
Kakade
```

### Important

`__init__` is commonly called the constructor, although technically object creation occurs before `__init__`; `__init__` initializes the already-created object.

---

# 4. `self`

`self` refers to the **current object/instance**.

```python
class User:
    def __init__(self, name):
        self.name = name
```

When:

```python
user1 = User("Kakade")
```

`self` refers to `user1`.

When:

```python
user2 = User("Developer")
```

`self` refers to `user2`.

So:

```python
self.name
```

means:

> "The `name` belonging to this particular object."

### Example

```python
class User:
    def __init__(self, name):
        self.name = name

    def greet(self):
        print(f"Hello {self.name}")
```

```python
user1 = User("Kakade")
user2 = User("Developer")

user1.greet()
user2.greet()
```

Output:

```text
Hello Kakade
Hello Developer
```

### Important

`self` is not a special keyword like `this` in some other languages. It's the conventional parameter name.

This technically works:

```python
class User:
    def greet(current_object):
        print(current_object)
```

But **always use `self`** by convention.

---

# 5. Instance Attributes

Instance attributes belong to a **specific object**.

Usually they're created using `self`:

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

```python
user1 = User("Kakade", 25)
user2 = User("Developer", 30)
```

Each object has its own values:

```python
print(user1.name)
print(user2.name)
```

```text
Kakade
Developer
```

Changing one doesn't change the other:

```python
user1.name = "Python Developer"
```

```python
print(user1.name)
print(user2.name)
```

```text
Python Developer
Developer
```

---

# 6. Class Attributes

A class attribute belongs to the **class itself** and is shared by instances unless an instance overrides it.

```python
class User:
    platform = "kakade.dev"

    def __init__(self, name):
        self.name = name
```

```python
user1 = User("Kakade")
user2 = User("Developer")

print(user1.platform)
print(user2.platform)
print(User.platform)
```

All three can access:

```text
kakade.dev
```

### Instance vs class attribute

```python
class User:
    platform = "kakade.dev"   # class attribute

    def __init__(self, name):
        self.name = name       # instance attribute
```

Think:

```text
Class attribute
    ↓
Shared by objects

Instance attribute
    ↓
Specific to each object
```

---

# 7. Instance Methods

Instance methods operate on a particular object.

They take `self` as their first parameter.

```python
class User:
    def __init__(self, name):
        self.name = name

    def greet(self):
        return f"Hello {self.name}"
```

```python
user = User("Kakade")

print(user.greet())
```

Output:

```text
Hello Kakade
```

Because `greet()` uses `self.name`, it needs access to the particular object.

---

# 8. Class Methods

A class method operates on the **class rather than a particular instance**.

Use:

```python
@classmethod
```

and conventionally use `cls` as the first parameter.

```python
class User:
    platform = "kakade.dev"

    @classmethod
    def get_platform(cls):
        return cls.platform
```

Call it directly from the class:

```python
print(User.get_platform())
```

Output:

```text
kakade.dev
```

It can also be called from an instance:

```python
user = User()
print(user.get_platform())
```

### Why use `cls`?

`cls` refers to the class.

```text
self → instance
cls  → class
```

---

## Class method as an alternative constructor

One of the most useful applications:

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    @classmethod
    def from_string(cls, data):
        name, age = data.split(",")
        return cls(name, int(age))
```

Now:

```python
user = User.from_string("Kakade,25")

print(user.name)
print(user.age)
```

Output:

```text
Kakade
25
```

---

# 9. Static Methods

A static method doesn't need access to either:

* the instance (`self`)
* the class (`cls`)

Use:

```python
@staticmethod
```

Example:

```python
class MathUtils:

    @staticmethod
    def add(a, b):
        return a + b
```

Call it:

```python
print(MathUtils.add(10, 20))
```

Output:

```text
30
```

There's no `self` or `cls`.

### When to use?

Use a static method when a function logically belongs to a class but doesn't need class or instance data.

```python
class User:

    @staticmethod
    def is_valid_name(name):
        return len(name) >= 3
```

```python
print(User.is_valid_name("Kakade"))
```

---

# 10. Properties

Properties allow you to access a method **like an attribute**.

Instead of:

```python
user.get_email()
```

you can write:

```python
user.email
```

Use `@property`:

```python
class User:
    def __init__(self, email):
        self._email = email

    @property
    def email(self):
        return self._email
```

```python
user = User("dev@kakade.dev")

print(user.email)
```

Notice:

```python
user.email
```

not:

```python
user.email()
```

---

## Setter

You can also control assignment:

```python
class User:
    def __init__(self, email):
        self._email = email

    @property
    def email(self):
        return self._email

    @email.setter
    def email(self, value):
        if "@" not in value:
            raise ValueError("Invalid email")

        self._email = value
```

Now:

```python
user = User("dev@kakade.dev")

user.email = "new@kakade.dev"
```

But:

```python
user.email = "invalid"
```

raises:

```text
ValueError: Invalid email
```

Properties are useful for **validation and controlled access**.

---

# 11. Encapsulation

Encapsulation means **bundling data and behavior together while controlling how internal data is accessed or modified**.

Python doesn't enforce private fields as strictly as languages like Java.

By convention:

```python
self._balance
```

means:

> "This is intended for internal use."

Example:

```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance

    def deposit(self, amount):
        if amount > 0:
            self._balance += amount

    def get_balance(self):
        return self._balance
```

Instead of letting everything modify `_balance` arbitrarily, you provide controlled methods.

---

## Double underscore

Python also supports name mangling:

```python
class User:
    def __init__(self):
        self.__password = "secret"
```

`__password` gets name-mangled internally.

It's not truly private security, but it helps prevent accidental access/name collisions.

---

# 12. Inheritance

Inheritance allows one class to **reuse and extend another class**.

```python
class Animal:
    def speak(self):
        print("Animal speaks")


class Dog(Animal):
    pass
```

Now:

```python
dog = Dog()

dog.speak()
```

Output:

```text
Animal speaks
```

`Dog` inherits from `Animal`.

Think:

```text
Animal
   │
   ├── Dog
   ├── Cat
   └── Bird
```

---

## Extending the parent

```python
class Animal:
    def speak(self):
        print("Animal speaks")


class Dog(Animal):
    def bark(self):
        print("Woof!")
```

Now:

```python
dog = Dog()

dog.speak()
dog.bark()
```

The child gets the parent's behavior and can add its own.

---

# 13. Polymorphism

Polymorphism means **the same interface can behave differently depending on the object**.

Example:

```python
class Dog:
    def speak(self):
        return "Woof"


class Cat:
    def speak(self):
        return "Meow"
```

Now:

```python
def make_sound(animal):
    print(animal.speak())
```

Both work:

```python
make_sound(Dog())
make_sound(Cat())
```

Output:

```text
Woof
Meow
```

The function doesn't care whether it's given a `Dog` or `Cat`.

It simply expects the object to provide:

```python
speak()
```

This is closely related to **duck typing** in Python:

> If an object behaves like what you need, you can often use it without caring about its exact type.

---

# 14. Abstraction

Abstraction means **hiding implementation details and exposing only the essential interface**.

Python provides abstract base classes through `abc`.

```python
from abc import ABC, abstractmethod

class Payment(ABC):

    @abstractmethod
    def pay(self, amount):
        pass
```

A subclass must implement `pay()`:

```python
class CardPayment(Payment):

    def pay(self, amount):
        print(f"Paid {amount} using card")
```

You cannot normally instantiate the abstract class:

```python
payment = Payment()
```

This results in an error because `Payment` contains an abstract method.

But:

```python
payment = CardPayment()
payment.pay(500)
```

works.

### Mental model

```text
Abstraction
     ↓
"What should this object do?"

Implementation
     ↓
"How exactly does it do it?"
```

---

# 15. `super()`

`super()` allows you to access functionality from a parent class.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name


class Admin(User):
    def __init__(self, name, permissions):
        super().__init__(name)
        self.permissions = permissions
```

Now:

```python
admin = Admin("Kakade", ["manage_users"])
```

`super().__init__(name)` calls the parent's:

```python
User.__init__()
```

Without it, `self.name` wouldn't be initialized by the parent constructor.

---

## Calling parent methods

```python
class Animal:
    def speak(self):
        print("Animal sound")


class Dog(Animal):
    def speak(self):
        super().speak()
        print("Woof!")
```

```python
dog = Dog()
dog.speak()
```

Output:

```text
Animal sound
Woof!
```

---

# 16. Composition

Composition means building a class using **objects of other classes**.

Instead of saying:

> "A `Car` is an `Engine`"

we say:

> "A `Car` has an `Engine`."

Example:

```python
class Engine:
    def start(self):
        print("Engine started")


class Car:
    def __init__(self):
        self.engine = Engine()

    def start(self):
        self.engine.start()
```

Now:

```python
car = Car()

car.start()
```

Output:

```text
Engine started
```

The `Car` **contains** an `Engine`.

That's composition.

---

# 17. Composition vs Inheritance

This is an important interview concept.

### Inheritance = "is-a"

```python
class Dog(Animal):
    pass
```

A:

```text
Dog IS AN Animal
```

### Composition = "has-a"

```python
class Car:
    def __init__(self):
        self.engine = Engine()
```

A:

```text
Car HAS AN Engine
```

---

## Comparison

| Inheritance                         | Composition                        |
| ----------------------------------- | ---------------------------------- |
| "is-a" relationship                 | "has-a" relationship               |
| Child extends parent                | Object contains another object     |
| Creates tighter coupling            | Usually more flexible              |
| Useful for genuine type hierarchies | Useful for combining behaviors     |
| Uses `class Child(Parent)`          | Uses attributes containing objects |

### Example

Inheritance:

```python
class Animal:
    pass

class Dog(Animal):
    pass
```

Composition:

```python
class Engine:
    pass

class Car:
    def __init__(self):
        self.engine = Engine()
```

---

# Putting It All Together

Here's a small example combining several OOP concepts:

```python
from abc import ABC, abstractmethod


class User(ABC):
    platform = "kakade.dev"

    def __init__(self, name):
        self.name = name

    @property
    def username(self):
        return self.name.lower()

    @abstractmethod
    def get_role(self):
        pass

    @classmethod
    def platform_name(cls):
        return cls.platform

    @staticmethod
    def is_valid_name(name):
        return len(name) >= 3


class Developer(User):
    def __init__(self, name, language):
        super().__init__(name)
        self.language = language

    def get_role(self):
        return "Developer"
```

Using it:

```python
developer = Developer("Kakade", "Python")

print(developer.name)
print(developer.username)
print(developer.language)
print(developer.get_role())
print(developer.platform_name())
```

Output:

```text
Kakade
kakade
Python
Developer
kakade.dev
```


# Quick Revision

| Concept                        | Remember it as                           |
| ------------------------------ | ---------------------------------------- |
| **Class**                      | Blueprint                                |
| **Object**                     | Instance of a class                      |
| **`__init__`**                 | Initializes an object                    |
| **`self`**                     | Current instance                         |
| **Instance attribute**         | Data belonging to one object             |
| **Class attribute**            | Data shared by the class/instances       |
| **Instance method**            | Works with an instance                   |
| **Class method**               | Works with the class                     |
| **Static method**              | Utility function inside a class          |
| **Property**                   | Method accessed like an attribute        |
| **Encapsulation**              | Control access to internal state         |
| **Inheritance**                | Child gets/extends parent behavior       |
| **Polymorphism**               | Same interface, different behavior       |
| **Abstraction**                | Expose what matters, hide implementation |
| **`super()`**                  | Access parent implementation             |
| **Composition**                | Build objects using other objects        |
| **Inheritance vs composition** | **is-a** vs **has-a**                    |

### The 4 major OOP pillars

```text
             OOP
              │
     ┌────────┼────────┐
     ↓        ↓        ↓
Encapsulation Inheritance Polymorphism
              │
              ↓
         Abstraction
```

The most important distinction to remember is:

> **Encapsulation** controls access to data.
> **Abstraction** hides implementation complexity.
> **Inheritance** reuses/extends behavior.
> **Polymorphism** lets different objects respond to the same interface differently.
