# Pydantic

Pydantic looks like simple classes at first, but once you understand **declare types → automatic validation → serialization**, it becomes the backbone of FastAPI.

The core idea:

```text
class User(BaseModel):
    name: str
    age: int
    ↓
Pydantic validates input
    ↓
gives you typed object + JSON
```

Pydantic lets you **define data shape once and trust it everywhere**. No manual `if "name" not in data` checks.

Install:

```bash
pip install pydantic
```

FastAPI already includes it.

---

# 1. BaseModel

All models inherit from `BaseModel`.

```python
from pydantic import BaseModel


class User(BaseModel):
    name: str
    age: int


user = User(name="Kakade", age=25)

print(user)
print(user.name)
```

Output:

```text
name='Kakade' age=25
Kakade
```

Think:

```text
BaseModel
    │
    ├── declares fields + types
    │
    └── validates on creation
```

If you come from dataclasses: Pydantic looks similar but **validates**.

---

# 2. Type Validation

Pydantic rejects wrong types.

```python
from pydantic import BaseModel


class User(BaseModel):
    name: str
    age: int


user = User(name="Kakade", age="not-a-number")
```

Output:

```text
pydantic_core._pydantic_core.ValidationError: 1 validation error for User
age
  Input should be a valid integer ...
```

Valid coercion works:

```python
user = User(name="Kakade", age="25")

print(user.age)
```

Output:

```text
25
```

Pydantic converts `"25"` to `25`. Strict invalid data raises `ValidationError`.

Common types:

```text
str, int, float, bool
list[str], dict[str, int]
datetime, UUID, EmailStr
```

In FastAPI, invalid input becomes `422 Unprocessable Entity` automatically.

---

# 3. Serialization

Serialization means **model → dict / JSON**.

```python
from pydantic import BaseModel


class User(BaseModel):
    name: str
    age: int


user = User(name="Kakade", age=25)

print(user.model_dump())
print(user.model_dump_json())
```

Output:

```text
{'name': 'Kakade', 'age': 25}
{"name":"Kakade","age":25}
```

Think:

```text
model_dump() → dict
model_dump_json() → JSON string
```

Use `model_dump()` when returning from FastAPI with extra fields, or saving to DB.

Exclude fields:

```python
print(user.model_dump(exclude={"age"}))
```

Output:

```text
{'name': 'Kakade'}
```

---

# 4. Deserialization

Deserialization means **dict / JSON → model**.

```python
from pydantic import BaseModel


class User(BaseModel):
    name: str
    age: int


user = User.model_validate({"name": "Kakade", "age": 25})

print(user)
```

Output:

```text
name='Kakade' age=25
```

From JSON string:

```python
user = User.model_validate_json('{"name": "Kakade", "age": 25}')

print(user.name)
```

Output:

```text
Kakade
```

Flow:

```text
JSON string / dict
    ↓
model_validate()
    ↓
typed object

typed object
    ↓
model_dump()
    ↓
dict / JSON
```

Old Pydantic v1 used `.dict()` and `.parse_obj()`. In v2 use `model_dump()` and `model_validate()`.

---

# 5. Nested Models

Models can contain other models.

```python
from pydantic import BaseModel


class Address(BaseModel):
    city: str
    pin: str


class User(BaseModel):
    name: str
    address: Address


user = User(
    name="Kakade",
    address={"city": "Pune", "pin": "411001"}
)

print(user)
print(user.address.city)
```

Output:

```text
name='Kakade' address=Address(city='Pune', pin='411001')
Pune
```

You passed a dict for `address`, Pydantic converted it to `Address`.

Nested validation is automatic. If `pin` is missing, you get a clear error pointing to `address.pin`.

Use for:

```text
user → address
order → items[]
post → author
```

---

# 6. Optional Fields

Optional means **can be None or missing**.

```python
from pydantic import BaseModel


class User(BaseModel):
    name: str
    nickname: str | None = None


print(User(name="Kakade"))
print(User(name="Kakade", nickname="KK"))
```

Output:

```text
name='Kakade' nickname=None
name='Kakade' nickname='KK'
```

Old style:

```python
from typing import Optional

nickname: Optional[str] = None
```

New style (Python 3.10+):

```python
nickname: str | None = None
```

Important: `str | None` alone still requires the field. You need `= None` to make it truly optional.

```python
# Required but nullable
nickname: str | None

# Optional with default None
nickname: str | None = None
```

---

# 7. Default Values

Defaults make fields optional.

```python
from pydantic import BaseModel


class Task(BaseModel):
    title: str
    active: bool = True
    priority: int = 1


print(Task(title="Learn Pydantic"))
```

Output:

```text
title='Learn Pydantic' active=True priority=1
```

Mutable defaults are safe in Pydantic (unlike dataclasses):

```python
class User(BaseModel):
    name: str
    tags: list[str] = []
```

Each instance gets its own list.

Use `Field()` for descriptions and constraints (next section).

---

# 8. Field Validation

`Field()` adds metadata and constraints.

```python
from pydantic import BaseModel, Field


class User(BaseModel):
    name: str = Field(min_length=2, max_length=50)
    age: int = Field(ge=0, le=120)
    email: str = Field(pattern=r"^[\w\.-]+@[\w\.-]+\.\w+$")
```

Test:

```python
User(name="K", age=25, email="kakade@example.com")
```

Output:

```text
ValidationError: String should have at least 2 characters
```

Common constraints:

| Constraint | Meaning |
| ---------- | ------- |
| `min_length` | Min string/list length |
| `max_length` | Max string/list length |
| `ge` | Greater or equal |
| `le` | Less or equal |
| `gt` / `lt` | Strict greater/less |
| `pattern` | Regex must match |
| `description` | Docs text |

With description for OpenAPI:

```python
class User(BaseModel):
    name: str = Field(description="Full name of user")
```

Shows in `/docs` automatically.

---

# 9. Custom Validation

For logic `Field()` cannot express, use validators.

Pydantic v2 style:

```python
from pydantic import BaseModel, field_validator


class User(BaseModel):
    name: str
    age: int

    @field_validator("name")
    @classmethod
    def name_must_not_be_empty(cls, value):
        if not value.strip():
            raise ValueError("name cannot be empty")
        return value.strip()
```

Test:

```python
print(User(name="  Kakade  ", age=25))
```

Output:

```text
name='Kakade' age=25
```

Validator stripped spaces.

Multi-field check with `model_validator`:

```python
from pydantic import BaseModel, model_validator


class PasswordChange(BaseModel):
    password: str
    confirm: str

    @model_validator(mode="after")
    def passwords_match(self):
        if self.password != self.confirm:
            raise ValueError("passwords do not match")
        return self
```

Think:

```text
field_validator → one field
model_validator → whole model / cross-fields
```

---

# 10. Enums

Enums restrict a field to fixed choices.

```python
from enum import Enum
from pydantic import BaseModel


class Role(str, Enum):
    ADMIN = "admin"
    USER = "user"
    GUEST = "guest"


class User(BaseModel):
    name: str
    role: Role


print(User(name="Kakade", role="admin"))
```

Output:

```text
name='Kakade' role=<Role.ADMIN: 'admin'>
```

Invalid:

```python
User(name="Kakade", role="superuser")
```

Output:

```text
ValidationError: Input should be 'admin', 'user' or 'guest'
```

Why inherit `str`?

```python
class Role(str, Enum):
```

makes JSON output `"admin"` instead of `1` or object. Always use `str, Enum` for APIs.

---

# 11. Model Configuration

Config controls model behavior.

```python
from pydantic import BaseModel, ConfigDict


class User(BaseModel):
    model_config = ConfigDict(
        extra="forbid",
        str_strip_whitespace=True
    )

    name: str
```

Options:

```text
extra="forbid"
    ↓
reject unknown fields
    ↓
{"name": "Kakade", "hacker": 1} → error
```

```text
extra="ignore" (default)
    ↓
silently drop unknown fields
```

```text
str_strip_whitespace=True
    ↓
"  Kakade  " → "Kakade"
```

Useful for strict APIs:

```python
class UserIn(BaseModel):
    model_config = ConfigDict(extra="forbid")

    name: str
    email: str
```

Prevents typos like `emial` from silently passing.

---

# Real-World Example: Request + Response

```python
from pydantic import BaseModel, Field


class UserIn(BaseModel):
    name: str = Field(min_length=2)
    email: str
    age: int = Field(ge=0, le=120)


class UserOut(BaseModel):
    id: int
    name: str
    email: str


def create_user(data: UserIn):
    # validated input
    # save to DB, get id 5
    user = {"id": 5, **data.model_dump()}
    # filter output
    return UserOut.model_validate(user)


print(create_user({"name": "Kakade", "email": "k@example.com", "age": 25}))
```

Output:

```text
id=5 name='Kakade' email='k@example.com'
```

Pattern:

```text
UserIn → validate request
UserOut → filter response, hide secrets
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **BaseModel** | Parent of all models |
| **Validation** | Auto on creation, raises `ValidationError` |
| **Serialize** | `model_dump()` → dict, `model_dump_json()` → JSON |
| **Deserialize** | `model_validate()` / `model_validate_json()` |
| **Nested** | Model inside model, validated recursively |
| **Optional** | `str \| None = None` |
| **Defaults** | `active: bool = True` |
| **Field** | `Field(ge=, le=, min_length=, pattern=)` |
| **Custom** | `@field_validator`, `@model_validator` |
| **Enums** | `class Role(str, Enum)` |
| **Config** | `model_config = ConfigDict(extra="forbid")` |

## The Mental Model

```text
         PYDANTIC
             │
             ↓
   class X(BaseModel)
             │
   ┌─────────┼─────────┐
   ↓         ↓         ↓
 Types    Fields     Config
   │         │         │
   ↓         ↓         ↓
str/int   Field()   extra=forbid
Optional  validator Enums
Nested    defaults
   │         │         │
   └─────────┼─────────┘
             ↓
   validate → object → dump
             │
             ↓
   FastAPI: 422 on fail,
   /docs from schema
```

### The 3 things to remember

**1. Declare once:**

```python
class User(BaseModel):
    name: str
    age: int = Field(ge=0)
```

**2. Convert both ways:**

```python
User.model_validate(data)
user.model_dump()
```

**3. Separate in/out:**

```python
UserIn  # what you accept
UserOut # what you expose
```

> **Pydantic is a validation boundary: declare models with types, let it parse and check input, and serialize safe output for APIs.**
