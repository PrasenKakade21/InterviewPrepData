# Tuples — Revision Notes (Intermediate)
> One-line mental model: Ordered + immutable fixed record; comma makes tuple, use when shape shouldn't change.
## Core recall
- Create `(10,20,30)`, mixed `("Kakade",25,True)`; comma defines: `10,20,30` tuple; `(10)` int vs `(10,)` tuple; `()` empty.
- Index/slice like lists (`t[0]`, `t[-1]`, `t[1:3]`, `[::-1]`); slicing returns new tuple.
- Unpack `name,age,role = ("Kakade",25,"Dev")` (count must match else ValueError); `first,*mid,last` starred → list; loop `for nid,name in users:`; swap `a,b = b,a`.
- Immutable: `t[0]=x / append / del` → TypeError/AttributeError; but mutable members can change (`data=([1,2],[3,4]); data[0].append(5)` ok, `data[0]=x` fails).
- List vs tuple: both ordered/indexed; tuple fixed, fewer methods, hashable if all elems hashable → dict key `{(lat,lon): "Mumbai"}`; list unhashable.
- Use tuple for fixed records (coord `(19.07,72.87)`, rgb, db config, `(id,name,role)` rows); list for changing collections.
- Funcs `return "Kakade",25` returns tuple; unpack immediately `n,a,r = get_user()`.
- Methods: `count(2)`, `index(20)`, `in`, `len()` only.
## Must-remember code
- ```python
  print(type((10)), type((10,)), type(10,))  # int, tuple, tuple
  ```
- ```python
  def get_user(): return "Kakade", 25, "Developer"
  name, age, role = get_user(); print(name, age, role)
  ```
- ```python
  locs = {(19.0760, 72.8777): "Mumbai"}; print(locs[(19.0760, 72.8777)])
  ```
- ```python
  users = [(101, "Kakade", "Dev"), (102, "Asha", "Designer")]
  for uid, name, role in users: print(f"{uid}: {name}-{role}")
  ```
## Interview traps
- `(10)` is int not tuple → single-element needs trailing comma `(10,)`.
- `*middle` from tuple unpack is list, not tuple → don't call tuple methods on it.
- Tuple with list `([1,2],)` appends inside fine — immutability is references, not contents.
- List as dict key → TypeError unhashable; tuple with list inside also unhashable → use all-hashable tuple.
- Count mismatch `a,b = (1,2,3)` → ValueError; use `a,*rest = t` when length varies.
## 3 things to remember
1. Comma = tuple; `(x,)` for single.
2. Immutable refs, mutable contents; fixed shape → tuple, growing → list.
3. Return + unpack multi-values; hashable tuples = dict keys.
