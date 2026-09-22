# Testing

Testing looks like extra work until your first refactor breaks login silently. Then `pytest` becomes non-negotiable.

The core idea:

```text
Write function
    ↓
assert expected behavior
    ↓
fixtures + mocks isolate it
```

Install:

```bash
pip install pytest httpx
```

---

# 1. pytest

`pytest` finds `test_*.py` files and runs `test_*` functions. No boilerplate class needed.

```python
def add(a, b):
    return a + b


def test_add():
    assert add(2, 3) == 5
```

Run:

```bash
pytest -q
```

Output:

```text
1 passed in 0.02s
```

Think:

```text
test file → test function → assert
    ↓
green = safe to refactor
```

---

# 2. Test Functions

One behavior per function, name says what breaks.

```python
def is_adult(age: int):
    return age >= 18


def test_adult_true():
    assert is_adult(25) is True


def test_child_false():
    assert is_adult(10) is False
```

```text
test_<what>_<case>
```

Keep tests tiny. If a test needs "and" in its name, split it into two.

---

# 3. Assertions

`assert` is the whole check. `pytest` rewrites it to show values on failure.

```python
def greet(name: str):
    return f"Hello {name}"


def test_greet():
    assert greet("Kakade") == "Hello Kakade"
    assert "Kakade" in greet("Kakade")
    assert len(greet("Kakade")) > 0
```

Failed output shows both sides:

```text
assert 'Hello Asha' == 'Hello Kakade'
```

Prefer direct `==` over `assertTrue(x == y)` — better diffs.

---

# 4. Fixtures

Fixtures build reusable setup (DB, client, sample data). `yield` gives teardown.

```python
import pytest


@pytest.fixture
def user():
    return {"name": "Kakade", "age": 25}


def test_user_name(user):
    assert user["name"] == "Kakade"
```

Output:

```text
1 passed
```

DB pattern:

```python
@pytest.fixture
def db():
    conn = {"open": True}
    yield conn
    conn["open"] = False
```

Think: fixture = setup → `yield` → cleanup. Just name it as a parameter.

---

# 5. Parametrization

One test, many inputs. No copy-paste.

```python
import pytest


@pytest.mark.parametrize("age,expected", [
    (25, True),
    (18, True),
    (10, False),
])
def test_is_adult(age, expected):
    assert (age >= 18) == expected
```

Output:

```text
3 passed
```

Use for validation tables: emails, ages, status codes. Three cases minimum before parametrizing.

---

# 6. Mocking

Mock replaces slow/real things (API, DB, email) with fakes you control.

```python
from unittest.mock import Mock

api = Mock()
api.get_user.return_value = {"name": "Kakade"}

print(api.get_user(1))
print(api.get_user.called)
```

Output:

```text
{'name': 'Kakade'}
True
```

Patch where it's used, not where it's defined:

```python
from unittest.mock import patch

with patch("myapp.send_email") as fake:
    fake.return_value = True
    print(fake("hi"))
```

Output:

```text
True
```

---

# 7. Unit Testing

Tests one function in isolation. Fast, no network/DB.

```python
def total(items):
    return sum(i["price"] for i in items)


def test_total():
    cart = [{"price": 100}, {"price": 200}]
    assert total(cart) == 300
```

Rules:

```text
one function, mocked deps
milliseconds each
run on every save
```

If it touches DB/API, it's not a unit test — it's integration.

---

# 8. Integration Testing

Tests pieces wired together: route → DB → response.

```python
import sqlite3


def test_user_saved():
    db = sqlite3.connect(":memory:")
    db.execute("CREATE TABLE users (name TEXT)")
    db.execute("INSERT INTO users VALUES (?)", ("Kakade",))
    rows = db.execute("SELECT * FROM users").fetchall()
    assert rows == [("Kakade",)]
```

Slower, fewer than unit tests. Pyramid: many unit, few integration, very few end-to-end.

---

# 9. API Testing

Hit real endpoints, assert status + JSON shape.

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()

@app.get("/users/{name}")
def get_user(name: str):
    return {"name": name}

client = TestClient(app)
r = client.get("/users/Kakade")

print(r.status_code)
print(r.json())
```

Output:

```text
200
{'name': 'Kakade'}
```

Pattern: status first, then keys, then values. Test `404` and `422` paths too.

---

# 10. FastAPI TestClient

`TestClient` runs your app in-process — no server needed.

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()

@app.post("/users")
def create_user(name: str):
    return {"name": name, "id": 1}

client = TestClient(app)
r = client.post("/users", params={"name": "Asha"})

print(r.status_code)
print(r.json()["id"])
```

Output:

```text
200
1
```

Override DB dependency in tests:

```python
app.dependency_overrides[get_db] = lambda: fake_db
```

---

# 11. httpx Testing

Newer async-safe way: `ASGITransport` talks to app without network.

```python
import httpx
from fastapi import FastAPI

app = FastAPI()

@app.get("/ping")
def ping():
    return {"ping": "pong"}
```

Usage (async test):

```python
transport = httpx.ASGITransport(app=app)
# async with httpx.AsyncClient(transport=transport, base_url="http://test") as c:
#     r = await c.get("/ping")
```

`TestClient` = sync simplicity. `httpx` transport = async correctness. Both assert the same JSON.

---

# 12. Test Coverage

Coverage shows untested lines. Aim 80%+, 100% on critical paths.

```bash
pip install pytest-cov
pytest --cov=myapp --cov-report=term-missing -q
```

Output:

```text
Name       Stmts   Miss  Cover
myapp.py       5      1    80%
```

Missing lines point to untested `except` branches. Don't chase 100% blindly — test behavior, not lines.

---

# Real-World Uses

```python
PR check → pytest -q → coverage gate → merge
```

Signup flow: unit (password hash) + API (POST /users → 201) + integration (row in DB).

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **pytest** | `test_*.py`, `pytest -q` |
| **Test functions** | One behavior, `test_what_case` |
| **Assertions** | Plain `assert`, rich diffs |
| **Fixtures** | Setup via `yield` |
| **Parametrization** | `@mark.parametrize` tables |
| **Mocking** | `Mock` / `patch` where used |
| **Unit** | One func, mocked, fast |
| **Integration** | Wired pieces, slower |
| **API testing** | Status + JSON shape |
| **TestClient** | In-process FastAPI |
| **httpx** | `ASGITransport` async tests |
| **Coverage** | `--cov`, 80%+ gate |

## The Mental Model

```text
        PYRAMID
   many unit (fast, mocked)
            │
   fewer integration (DB/API)
            │
   few end-to-end (full flow)
            │
   ┌────────┼────────┐
   ↓        ↓        ↓
fixtures parametrize mocks
   │        │        │
   └────────┼────────┘
            ↓
 TestClient/httpx + coverage gate
```

### The 3 things to remember

**1. Basic test:**

```python
def test_add():
    assert add(2, 3) == 5
```

**2. Table test:**

```python
@pytest.mark.parametrize("age,expected", [(25, True), (10, False)])
def test_is_adult(age, expected):
    assert (age >= 18) == expected
```

**3. API test:**

```python
client = TestClient(app)
assert client.get("/users/Kakade").status_code == 200
```

> **Testing is assert-isolate-cover: write small asserts, isolate with fixtures and mocks, and gate merges with API tests plus coverage.**
