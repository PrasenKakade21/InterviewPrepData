# Type Hints and Code Quality — Revision Notes (Intermediate)
> One-line mental model: Hints describe shape → mypy/Ruff check → Black formats.
## Core recall
- Funcs: `def greet(name: str) -> str`; hints not enforced at runtime, `mypy` flags lies.
- Vars: only when empty/ambiguous (`users: list = []`); signatures always hinted.
- Generics: `list[str]`, `dict[str, int]` (3.9+ lowercase); nesting reads inside-out.
- `Optional[X]` = `X | None` (modern `str | None`); `Union[int, str]` = `int | str`, use rarely.
- `Literal["admin","user"]` locks modes/roles; `TypedDict` types JSON dicts with fixed keys.
- mypy: `mypy myapp.py` / `--strict src/` in CI; Ruff: `ruff check .` + `ruff format .` (F401 unused).
- Black: opinionated formatter; pick Ruff-format OR Black, not both; PEP 8 `snake_case`/`CapWords`.
- Pipeline: format → `ruff check` → `mypy` → `pytest`; pre-commit runs all four.
## Must-remember code
- ```python
  def greet(name: str) -> str: return f"Hello {name}"
  print(greet("Kakade"))
  ```
- ```python
  def upper_all(names: list[str]) -> list[str]: return [n.upper() for n in names]
  print(upper_all(["Kakade", "Asha"]))
  ```
- ```python
  from typing import Literal
  def set_role(role: Literal["admin", "user"]) -> str: return f"role={role}"
  print(set_role("admin"))
  ```
- ```python
  from typing import TypedDict
  class User(TypedDict): name: str; age: int
  def greet(u: User) -> str: return f"Hello {u['name']}"
  print(greet({"name": "Kakade", "age": 25}))
  ```
- ```python
  def find_user(name: str) -> str | None: return "found" if name == "Kakade" else None
  ```
## Interview traps
- `greet(5)` runs fine → hints aren't runtime checks → need mypy/CI to catch.
- Old `typing.List/Dict` works but dated → prefer `list[str]`/`dict[str,int]`.
- Broad `Union` for everything → simplify; prefer `Optional`/`Literal` when possible.
- `except Exception: pass` → flagged by Ruff → narrow to `FileNotFoundError` etc.
- Running Black + Ruff-format both → conflicts → choose one formatter.
## 3 things to remember
1. Hint signatures always; `str | None` over `Optional`.
2. `Literal`/`TypedDict` narrow JSON and modes.
3. Gate: `ruff check && (black --check) && mypy && pytest -q`.
