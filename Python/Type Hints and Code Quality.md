# Type Hints and Code Quality

Type hints don't change runtime — they catch bugs **before** runtime, in your editor and CI.

The core idea:

```text
Hints describe shape
    ↓
mypy + Ruff check it
    ↓
Black formats it
```

Install:

```bash
pip install mypy ruff black
```

---

# 1. Function Type Hints

`name: type -> return type`. Editor autocompletes, FastAPI/Pydantic enforce.

```python
def greet(name: str) -> str:
    return f"Hello {name}"


print(greet("Kakade"))
```

Output:

```text
Hello Kakade
```

`greet(5)` still runs (hints aren't enforced) — but `mypy` flags it. Hints are documentation machines check.

---

# 2. Variable Type Hints

Rarely needed — inference covers most. Use for empty containers or clarity.

```python
name: str = "Kakade"
count: int = 0
users: list = []
```

Output:

```text
Kakade
```

```python
print(name)
```

Rule: hint function signatures always, variables only when empty or ambiguous.

---

# 3. list[str]

Lowercase generics (3.9+). No `typing.List` needed.

```python
def upper_all(names: list[str]) -> list[str]:
    return [n.upper() for n in names]


print(upper_all(["Kakade", "Asha"]))
```

Output:

```text
['KAKADE', 'ASHA']
```

| Old | New |
| --- | --- |
| `List[str]` | `list[str]` |
| `Dict[str, int]` | `dict[str, int]` |

Interview trap: old `typing.List` still works but new style is preferred.

---

# 4. dict[str, int]

```python
def ages(users: dict[str, int]) -> dict[str, int]:
    return {k: v + 1 for k, v in users.items()}


print(ages({"Kakade": 25}))
```

Output:

```text
{'Kakade': 26}
```

Nesting reads inside-out: `list[dict[str, int]]` = list of dicts mapping str to int.

---

# 5. Optional

`Optional[X]` = `X | None`. Use for nullable fields, defaults to `None`.

```python
from typing import Optional


def find_user(name: str) -> Optional[str]:
    if name == "Kakade":
        return "found"
    return None


print(find_user("Kakade"))
print(find_user("Nobody"))
```

Output:

```text
found
None
```

Modern spelling: `str | None`. Same meaning, shorter. `Optional` still appears in old codebases.

---

# 6. Union

`Union[A, B]` = either type. Use sparingly — often a sign to simplify.

```python
from typing import Union


def to_str(v: Union[int, str]) -> str:
    return str(v)


print(to_str(5))
print(to_str("Kakade"))
```

Output:

```text
5
Kakade
```

Modern: `int | str`. Prefer `Optional`/`Literal` over broad `Union` when you can.

---

# 7. Literal

Restricts to exact values — perfect for modes, roles, statuses.

```python
from typing import Literal


def set_role(role: Literal["admin", "user"]) -> str:
    return f"role={role}"


print(set_role("admin"))
```

Output:

```text
role=admin
```

`set_role("owner")` fails `mypy` before it fails production. Better than free-form `str`.

---

# 8. TypedDict

Dict with fixed keys + types. Lightweight alternative to dataclass for JSON.

```python
from typing import TypedDict


class User(TypedDict):
    name: str
    age: int


def greet(u: User) -> str:
    return f"Hello {u['name']}"


print(greet({"name": "Kakade", "age": 25}))
```

Output:

```text
Hello Kakade
```

Wrong key or type flagged by `mypy`. Use `TypedDict` for JSON shapes, `dataclass`/`Pydantic` for behavior.

---

# 9. mypy

Static checker — finds type lies without running code.

```bash
mypy myapp.py
```

```python
def add(a: int, b: int) -> int:
    return a + b

add("Kakade", 5)
```

Output:

```text
error: Argument 1 has incompatible type "str"; expected "int"
```

Add to CI: `mypy --strict src/`. Start non-strict, tighten over time.

---

# 10. Ruff

Linter + formatter in one fast binary. Replaces flake8 + isort.

```bash
ruff check .
ruff format .
```

```python
import os  # unused → flagged

x = 1
```

Output:

```text
F401 'os' imported but unused
```

Config in `pyproject.toml`:

```toml
[tool.ruff]
line-length = 100
```

Run `ruff check --fix` to auto-fix imports and simple issues.

---

# 11. Black

Opinionated formatter — no debates, just consistent code.

```bash
black myapp.py
```

```python
# before
def greet(name:str):return f"Hello {name}"
```

```python
# after
def greet(name: str):
    return f"Hello {name}"
```

Black and Ruff-format overlap — pick one formatter (Ruff preferred now), not both.

---

# 12. PEP 8

Style guide Black/Ruff enforce: `snake_case` funcs, `CapWords` classes, 79-100 cols, two blank lines between top-level defs.

```python
# Good
user_name = "Kakade"


def get_user(user_id: int):
    pass


class UserService:
    pass
```

Bad: `UserName`, `getUser`, single-letter vars outside loops. Interviews check you *know* it exists, not memorize numbers.

---

# 13. Code Formatting

Pipeline order matters:

```text
Ruff/Black format
    ↓
Ruff check
    ↓
mypy
    ↓
pytest
```

Pre-commit hook runs all four before push. Format is automatic — never hand-format in reviews.

---

# 14. Linting

Linting = "suspicious code" detector: unused imports, undefined names, broad `except`.

```bash
ruff check src/
```

```python
try:
    open("x.txt")
except Exception:  # flagged: too broad
    pass
```

Fix: `except FileNotFoundError`. Linting catches bugs formatting never will.

---

# Real-World Uses

```python
CI pipeline → ruff → black --check → mypy → pytest --cov
```

FastAPI + Pydantic already use hints at runtime — clean hints mean free validation.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Function hints** | `def f(x: str) -> str` |
| **Variable hints** | Only when empty/ambiguous |
| **list[str]** | New lowercase generics |
| **dict[str, int]** | Inside-out nesting |
| **Optional** | `X \| None` |
| **Union** | `A \| B`, use rarely |
| **Literal** | Fixed string choices |
| **TypedDict** | Typed JSON dict |
| **mypy** | Type check in CI |
| **Ruff** | Lint + format, fast |
| **Black** | Opinionated format |
| **PEP 8** | `snake_case`, `CapWords` |
| **Formatting** | Auto, before check |
| **Linting** | Bug patterns, not style |

## The Mental Model

```text
        HINTS
 def f(x: str) -> str
   list[str] dict[str,int]
 Optional Union Literal TypedDict
            │
            ↓
    CHECK (mypy + Ruff)
            │
            ↓
    FORMAT (Black/Ruff)
            │
            ↓
     TEST (pytest)
```

### The 3 things to remember

**1. Hint signatures:**

```python
def greet(name: str) -> str:
    return f"Hello {name}"
```

**2. Narrow types:**

```python
def set_role(role: Literal["admin", "user"]) -> str: ...
def find(name: str) -> str | None: ...
```

**3. Quality gate:**

```bash
ruff check . && black --check . && mypy . && pytest -q
```

> **Hints describe intent, mypy verifies it, Ruff and Black enforce style — automate all four so reviews discuss logic, not formatting.**
