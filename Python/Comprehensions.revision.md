# Comprehensions — Revision Notes (Intermediate)
> One-line mental model: `[WHAT for ITEM in COLL if FILTER]` — bracket type decides result container.

## Core recall
- Forms: `[...]` list, `{k:v}` dict, `{x}` set, `(...)` generator (lazy).
- Basic: `[n**2 for n in nums]`; transform `[p*1.18 for p in prices]`; filter `[n for n in nums if n%2==0]`.
- Filter vs ternary: `[n for n in nums if even]` drops items; `["even" if e else "odd" for n in nums]` keeps all, changes value.
- Nested: `[x for row in matrix for x in row]` (order = nested loops); flatten matrix; add `if` at end to filter.
- Dict: `{n:n**2 for n in nums}`, `{k:v*0.9 for k,v in prices.items() if v>5000}`; set dedups `{x**2 for x in [1,2,2]}`.
- Generators: `(x**2 for x in range(10**7))` lazy, memory-light; direct in `sum(x**2 for x in nums)`, `any()`, `all()`.
- Readability rule: if 2+ loops/conditions or complex expr, use explicit loop.

## Must-remember code
```python
nums = [1, 2, 3, 4, 5, 6]
print([n**2 for n in nums if n % 2 == 0])  # [4,16,36]
print(["even" if n % 2 == 0 else "odd" for n in nums[:4]])
```
```python
matrix = [[1, 2, 3], [4, 5, 6]]
print([x for row in matrix for x in row])  # flat
users = [{"name": "Kakade", "age": 25, "active": True}, {"name": "Rahul", "age": 17, "active": False}]
print({u["name"]: u["age"] for u in users})  # name->age
```
```python
print({n**2 for n in [1, 2, 2, 3]})  # {1,4,9} deduped
print(sum(n**2 for n in range(5)))  # 30, no list built
```
```python
names = ["kakade", "rahul"]
print([n.upper() for n in names])  # ['KAKADE','RAHUL']
print([u for u in ["Kakade", "", "Amit"] if u])  # falsy filter
```

## Interview traps
- `if` at end filters; `if/else` at front transforms — swapping changes length of result.
- `{x for x in ...}` is set (dedups, unordered) vs `{x:x**2 ...}` dict vs `[x ...]` list — one colon changes type.
- `(...)` is generator: prints `<generator object>`, single-use; `list(gen)` exhausts it.
- Nested order matters: `for row ... for x in row` ≠ reversed; over-nesting kills readability.
- `map/filter` + `lambda` works but comprehensions preferred: `[f(x) for x in xs if p(x)]`.

## 3 things to remember
1. Brackets decide type: `[]` list, `{k:v}` dict, `{x}` set, `()` lazy generator.
2. Trailing `if` filters; leading `if/else` transforms every item.
3. Generators for big data / `sum(any/all)`; loops when complex.
