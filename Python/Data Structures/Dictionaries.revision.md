# Dictionaries — Revision Notes (Intermediate)
> One-line mental model: A dict is a key-identified collection — JSON maps naturally to it; master access, iteration, nesting, and `**` unpacking.
## Core recall
- Structure: `{key: value}`; keys unique (later wins), must be hashable (str/int/tuple ok, list not).
- Access: `d[k]` raises KeyError if missing; `d.get(k)` → None; `d.get(k, default)` safe for API/JSON fields.
- Views: `keys()` (or `for k in d`), `values()`, `items()` → `for k, v in d.items():` most idiomatic.
- Mutate: `d[k]=v`, `update({...})` / `update(kw=)` (later wins), `pop(k)` removes+returns (KeyError unless `pop(k, default)`).
- Nested: `user["profile"]["city"]`; safe: `user.get("profile", {}).get("city")` → None not crash.
- Unpack: `{**a, **b}` merges without mutating (later wins); `f(**d)` passes dict as kwargs.
- Mental map: list=position, tuple=fixed, set=membership, dict=key lookup; dict ↔ JSON object.
## Must-remember code
- ```python
kakade = {"name": "Kakade", "age": 25}
print(kakade.get("email", "Not provided"))
kakade.update({"age": 26, "city": "Mumbai"}); print(kakade)
  ```
- ```python
kakade = {"name": "Kakade", "role": "Dev"}
for k, v in kakade.items(): print(k, "=", v)
print("Kakade" in kakade.values())
  ```
- ```python
user = {"name": "Kakade", "profile": {"city": "Mumbai"}}
print(user["profile"]["city"])
print(user.get("profile", {}).get("phone"))  # None, safe
  ```
- ```python
base = {"theme": "dark"}; extra = {"theme": "light", "lang": "en"}
print({**base, **extra})  # theme=light wins
  ```
- ```python
def create_user(name, age): print(name, age)
kakade = {"name": "Kakade", "age": 25}; create_user(**kakade)
  ```
## Interview traps
- `d[k]` → KeyError on missing; use `get(k, default)` for optional API fields.
- Duplicate keys silently overwrite: `{"name":"A","name":"B"}` → `"B"`.
- Unhashable keys: `{[1,2]: v}` → TypeError; use tuple instead.
- Nested `d["a"]["b"]` crashes if `d["a"]` missing → chain `.get(a, {})`.
- `update`/unpack order matters: later value wins — `settings.update(defaults)` overwrites user choice.
## 3 things to remember
1. `[]` raises, `.get()`/`.pop(k, default)` safe — use for flaky JSON.
2. `for k, v in d.items()` + nested `.get` chains for APIs.
3. `{**a, **b}` and `f(**d)` for non-mutating merges and framework calls.
