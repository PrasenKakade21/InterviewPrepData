# Pydantic — Revision Notes (Intermediate)
> One-line mental model: Declare types once → Pydantic validates input → serialize safe output.
## Core recall
- `class User(BaseModel)`: declares fields, validates on creation, raises `ValidationError`.
- Types: `str/int/float/bool`, `list[str]`, `datetime`, coercion `"25"`→`25`, invalid → 422 in FastAPI.
- Serialize: `model_dump()` → dict, `model_dump_json()` → JSON, `exclude={...}` to hide fields.
- Deserialize: `User.model_validate(dict)`, `User.model_validate_json(str)`; v1 `.dict()/.parse_obj()` deprecated.
- Nested: `address: Address` auto-converts dict → model, errors point to `address.pin`.
- Optional: `str | None = None` truly optional; `str | None` alone = required-but-nullable.
- Defaults: `active: bool=True`; mutable `list[str]=[]` safe (per-instance).
- `Field(min_length, max_length, ge/le/gt/lt, pattern, description)` → constraints + OpenAPI docs.
- Custom: `@field_validator("name")` one field, `@model_validator(mode="after")` cross-field.
- Enums: `class Role(str, Enum)` → JSON as `"admin"`; invalid raises error.
- Config: `model_config = ConfigDict(extra="forbid", str_strip_whitespace=True)`; `forbid` catches typos.
- Pattern: `UserIn` validates request, `UserOut` filters response.
## Must-remember code
```python
from pydantic import BaseModel, Field
class User(BaseModel):
    name: str = Field(min_length=2)
    age: int = Field(ge=0, le=120)
u = User(name="Kakade", age="25"); print(u.model_dump())
```
```python
u = User.model_validate({"name": "Kakade", "age": 25})
print(u.model_dump_json(exclude={"age"}))
```
```python
from pydantic import field_validator
class U(BaseModel):
    name: str
    @field_validator("name")
    @classmethod
    def clean(cls, v): return v.strip()
```
```python
from enum import Enum
class Role(str, Enum): ADMIN="admin"; USER="user"
class U2(BaseModel): name: str; role: Role
print(U2(name="Kakade", role="admin"))
```
## Interview traps
- `str | None` without `= None` still required → add `= None`.
- Using v1 `.dict()` → use `model_dump()` / `model_validate()` in v2.
- `extra="ignore"` default silently drops typos → use `extra="forbid"` for strict APIs.
- Forgetting `str` in `class Role(Enum)` → JSON serializes badly; always `str, Enum`.
- Exposing secrets in response → separate `UserIn`/`UserOut`, use `exclude` + `response_model`.
## 3 things to remember
1. Declare once with types + `Field()` constraints.
2. Convert both ways: `model_validate()` in, `model_dump()` out.
3. Separate In/Out models to validate input and hide secrets.
