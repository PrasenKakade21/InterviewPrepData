# Decorators — Revision Notes (Intermediate)
> One-line mental model: Decorator = function taking function, returning wrapper; `@d` means `f = d(f)`.
## Core recall
- First-class funcs: assign/pass/return (`execute(greet)` not `execute(greet())`); nested func + closure (inner remembers outer var, e.g. `multiplier(2)`).
- Basic: `def my_decorator(fn): def wrapper(): ...; return wrapper`; `@my_decorator def greet()` wraps without editing.
- Generic wrapper MUST be `def wrapper(*args, **kwargs): result = fn(*args, **kwargs); return result`.
- `functools.wraps`: preserves `__name__/__doc__` (else becomes `wrapper`); always use it.
- Decorator with args = factory: `repeat(times) → decorator(fn) → wrapper(*a,**k)`; 3 layers = config / function / call-args.
- Stacking: `@first @second def g` = `first(second(g))`; applied bottom-up, executed top-down (First, Second, Hello Kakade).
- Can modify result (`return result.upper()`), log/time/auth/cache; Flask `@app.route` registers.
- Decoration time (import/definition) vs call time (each call) — `print` in decorator body runs once at def.
## Must-remember code
- ```python
  from functools import wraps
  def decorator(fn):
      @wraps(fn)
      def wrapper(*args, **kwargs):
          print("Before"); r = fn(*args, **kwargs); print("After"); return r
      return wrapper
  @decorator
  def greet(n): return f"Hello {n}"
  print(greet("Kakade"))
  ```
- ```python
  def repeat(times):
      def decorator(fn):
          @wraps(fn)
          def wrapper(*a, **k):
              for _ in range(times): fn(*a, **k)
          return wrapper
      return decorator
  ```
- ```python
  def uppercase(fn):
      @wraps(fn)
      def wrapper(*a, **k): return fn(*a, **k).upper()
      return wrapper
  ```
## Interview traps
- Wrapper without `*args/**kwargs` breaks on `greet(name)` → always use generic signature.
- Forgetting `return result` → decorated `add()` returns None → propagate return.
- Missing `@wraps` → `greet.__name__ == "wrapper"`, lost docstring → add `@wraps(fn)`.
- Order confusion: bottom decorator applied first → `@first/@second` prints First then Second.
- `@repeat` vs `@repeat(3)` confusion → latter needs extra factory layer.
## 3 things to remember
1. `@d` = `f = d(f)`; generic wrapper + return result.
2. Always `@wraps(fn)`; 3-layer pattern for `@deco(arg)`.
3. Bottom-up applied, top-down executed; decoration ≠ call time.
