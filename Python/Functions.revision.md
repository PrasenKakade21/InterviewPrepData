# Functions — Revision Notes (Intermediate)
> One-line mental model: Factory machine — params in, `return` out; `*args`/`**kwargs` are flexible hoppers.

## Core recall
- `def f(params): ... return x`; missing/bare `return` → `None`; early `return` guards beat deep nesting; multi-return packs tuple.
- Positional by order, keyword by name; positional must precede keyword in calls; signature order: `a, b=1, *args, key=None, **kwargs`.
- Defaults evaluated once — never `def f(lst=[])`; use `None` sentinel then `lst = [] if lst is None else lst`.
- `*args` → tuple of extra positionals; `**kwargs` → dict of extra keywords; `f(*lst, **d)` unpacks at call; dict keys must match param names.
- Scope LEGB: Local → Enclosing → Global → Built-in; assign creates local unless `global`/`nonlocal` declared; don't shadow `list/str/sum`.
- `lambda x: expr` single-expression anon func for `sorted(key=lambda u: u["age"])`; prefer `def` for named funcs (PEP8, tracebacks).
- `map(f,it)` transform, `filter(p,it)` keep-if (both iterators), `reduce(f,it,init)` aggregate; comprehensions usually clearer.

## Must-remember code
```python
def add_kakade(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item); return lst
print(add_kakade("Apple"), add_kakade("Banana"))  # no leak
```
```python
def build_log(level, *msgs, **meta):
    return {"level": level, "msg": " ".join(msgs), "meta": meta}
print(build_log("INFO", "Kakade login", user_id=42))
def pt(x, y, z): return f"{x},{y},{z}"
print(pt(*[1, 2, 3]), pt(**{"x": 1, "y": 2, "z": 3}))
```
```python
def outer():
    n = 0
    def inner():
        nonlocal n; n += 1; return n
    return inner
t = outer(); print(t(), t())  # 1 2 (closure)
```
```python
users = [{"name": "Kakade", "age": 25}]
print(sorted(users, key=lambda u: u["age"]))
from functools import reduce
print(reduce(lambda a, b: a + b, [1, 2, 3], 0))  # 6
```

## Interview traps
- Mutable default `[]/{}` persists across calls → `None` sentinel.
- `func(pos, kw=1, 2)` SyntaxError; `func(**{"altitude":1})` TypeError if name mismatches.
- `counter += 1` without `global counter` → `UnboundLocalError`; nested reassign needs `nonlocal`.
- `add = lambda ...` discouraged; lambda can't hold statements/assignments.
- `map/filter` return iterators — must `list(...)`; forgetting `from functools import reduce`.

## 3 things to remember
1. Defaults once at def-time — use `None` for mutables.
2. `*args` tuple, `**kwargs` dict; mirror with `*`/`**` unpacking at calls.
3. LEGB + `global`/`nonlocal` for writes; early returns flatten logic.
