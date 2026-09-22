# Object-Oriented Programming — Revision Notes (Intermediate)
> One-line mental model: Class = blueprint, object = instance; `self` = this object, `cls` = this class.
## Core recall
- Class `class User: pass`; object `u = User()`; `__init__(self,name)` initializes (not strictly constructor).
- `self` = current instance (convention, not keyword); `self.name` per-object state.
- Instance attr (`self.name`) per-object vs class attr (`platform="kakade.dev"`) shared; `User.platform` accessible via class + instances.
- Instance method `def greet(self)` needs object; `@classmethod def get_platform(cls)` works on class, used as alt constructor (`from_string`); `@staticmethod` no self/cls (utility, e.g. `is_valid_name`).
- `@property` → method as attr (`user.email`); `@email.setter` validates on assignment.
- Encapsulation: `_balance` = internal-use convention; `__password` name-mangled, not true privacy; expose controlled methods.
- Inheritance `class Dog(Animal)` reuses/extends; Polymorphism same `speak()` different behavior (duck typing); Abstraction `ABC + @abstractmethod` forces `pay()` impl, can't instantiate base.
- `super().__init__(name)` inits parent; `super().speak()` extends parent method.
- Composition "has-a" (`Car.engine = Engine()`) vs Inheritance "is-a" (`Dog is Animal`); prefer composition for flexibility.
## Must-remember code
- ```python
  class User:
      platform = "kakade.dev"
      def __init__(self, name): self.name = name
      @classmethod
      def from_string(cls, s):
          n, a = s.split(","); return cls(n, int(a))
      @staticmethod
      def is_valid_name(n): return len(n) >= 3
  print(User.from_string("Kakade,25").name)
  ```
- ```python
  class User:
      def __init__(self, email): self._email = email
      @property
      def email(self): return self._email
      @email.setter
      def email(self, v):
          if "@" not in v: raise ValueError("Invalid email")
          self._email = v
  ```
- ```python
  from abc import ABC, abstractmethod
  class Payment(ABC):
      @abstractmethod
      def pay(self, amount): pass
  class CardPayment(Payment):
      def pay(self, amount): print(f"Paid {amount} using card")
  ```
- ```python
  class Engine:
      def start(self): print("Engine started")
  class Car:
      def __init__(self): self.engine = Engine()
      def start(self): self.engine.start()
  ```
## Interview traps
- Mutating class attr via instance shadows instead of sharing → set shared state via `User.platform` / `cls`.
- Forgetting `super().__init__(name)` in child → parent attrs (`self.name`) missing.
- Calling `user.email()` when it's a `@property` → TypeError; use `user.email`.
- Instantiating `Payment()` with abstractmethod → TypeError; must subclass + implement.
- `__init__` with mutable/class-attr confusion: instance assignment doesn't affect siblings.
## 3 things to remember
1. self = instance, cls = class, static = neither — pick by what you need.
2. is-a = inheritance, has-a = composition; encapsulation controls access, abstraction hides how.
3. Use `@property` for validated attrs, `super()` for parent init, `ABC` for contracts.
