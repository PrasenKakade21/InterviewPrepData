# Testing — Revision Notes (Intermediate)
> One-line mental model: Assert behavior → isolate with fixtures/mocks → gate with coverage.
## Core recall
- pytest: `test_*.py` + `test_*` fns, `pytest -q`; plain `assert` with rich diffs.
- Naming: `test_<what>_<case>`, one behavior each; split on "and".
- Fixtures: `@pytest.fixture` setup, `yield` teardown; inject by param name.
- Parametrize: `@pytest.mark.parametrize("age,expected", [...])` tables, no copy-paste.
- Mocks: `Mock(return_value=...)`, `patch("myapp.send_email")` where used; check `.called`.
- Pyramid: many fast unit (mocked) → fewer integration (sqlite/DB) → few e2e.
- API: `TestClient(app)` sync in-process, assert status → keys → values + 404/422; `httpx.ASGITransport` for async.
- Coverage: `pytest --cov=myapp --cov-report=term-missing`; 80%+ gate, 100% critical paths.
## Must-remember code
- ```python
  def add(a, b): return a + b
  def test_add(): assert add(2, 3) == 5
  ```
- ```python
  import pytest
  @pytest.mark.parametrize("age,expected", [(25, True), (18, True), (10, False)])
  def test_is_adult(age, expected): assert (age >= 18) == expected
  ```
- ```python
  import pytest
  @pytest.fixture
  def user(): return {"name": "Kakade", "age": 25}
  def test_user_name(user): assert user["name"] == "Kakade"
  ```
- ```python
  from unittest.mock import Mock
  api = Mock()
  api.get_user.return_value = {"name": "Kakade"}
  assert api.get_user(1) == {"name": "Kakade"}
  ```
- ```python
  from fastapi import FastAPI
  from fastapi.testclient import TestClient
  app = FastAPI()
  @app.get("/users/{name}")
  def get_user(name: str): return {"name": name}
  assert TestClient(app).get("/users/Kakade").status_code == 200
  ```
## Interview traps
- `assertTrue(x == y)` → weak diffs → use plain `assert x == y`.
- `patch` where defined vs used → no-op mock → patch import location (`myapp.send_email`).
- DB/API in "unit" test → slow/flaky → that's integration; mock deps in unit.
- Only happy path → misses 404/422 → always test error shapes.
- Chasing 100% lines → vanity → test behavior; missing lines hint untested `except`.
## 3 things to remember
1. Fixture = setup → `yield` → cleanup.
2. Parametrize tables; mock at usage point.
3. TestClient status + JSON; coverage gates merges.
