# FastAPI Intermediate

FastAPI Fundamentals gets you routes. Intermediate gets you **real project structure: routers → dependencies → auth → background work**.

The core idea:

```text
Small app (one file)
    ↓
APIRouter splits by feature
    ↓
Depends() injects shared logic
    ↓
middleware + handlers + auth wrap everything
```

This file assumes you know path/query/body, response models, and `/docs` from `FastAPI Fundamentals`.

Install:

```bash
pip install fastapi uvicorn python-jose[cryptography] passlib[bcrypt] python-multipart
```

---

# 1. APIRouter

`APIRouter` is a mini-app for one feature. Mount many routers on one `FastAPI()`.

```python
from fastapi import APIRouter, FastAPI

users_router = APIRouter(prefix="/users", tags=["users"])
tasks_router = APIRouter(prefix="/tasks", tags=["tasks"])


@users_router.get("")
def list_users():
    return [{"name": "Kakade"}]


@tasks_router.get("")
def list_tasks():
    return [{"title": "Learn routers"}]


app = FastAPI()
app.include_router(users_router)
app.include_router(tasks_router)
```

Think:

```text
users_router (prefix /users)
    ↓
"" → /users
"/{id}" → /users/{id}

app.include_router()
    ↓
mounts all routes at once
```

Real structure:

```text
app/
    main.py
    users/
        router.py
        service.py
    tasks/
        router.py
```

Each `router.py` holds an `APIRouter`. `main.py` only includes routers.

---

# 2. Dependency Injection

Dependencies are reusable functions that run before your route. Auth, DB sessions, pagination.

```python
from fastapi import Depends, FastAPI

app = FastAPI()


def get_page(page: int = 1, limit: int = 10):
    return {"page": page, "limit": limit}


@app.get("/tasks")
def list_tasks(paging: dict = Depends(get_page)):
    return paging
```

Request `/tasks?page=2&limit=5`:

```json
{"page": 2, "limit": 5}
```

Think:

```text
Request
    ↓
get_page() runs first
    ↓
result injected into route
    ↓
route runs
```

Why? Pagination logic written once, reused in 20 routes.

---

# 3. Depends()

`Depends()` can inject DB sessions, current users, settings.

DB pattern (SQLite preview, full in SQLAlchemy topic):

```python
from fastapi import Depends


def get_db():
    db = {"connected": True}
    try:
        yield db
    finally:
        print("close connection")


@app.get("/users")
def list_users(db: dict = Depends(get_db)):
    return {"db": db}
```

Notice `yield`. Code before `yield` runs before route, code after runs after. This is how sessions open/close safely.

Chained dependencies:

```python
def get_token(token: str = "abc"):
    return token


def get_user(token: str = Depends(get_token)):
    return {"name": "Kakade", "token": token}


@app.get("/me")
def me(user: dict = Depends(get_user)):
    return user
```

```text
get_token → get_user → route
```

Each layer builds on the previous.

---

# 4. Middleware

Middleware runs on **every request**. Logging, timing, custom headers.

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()


@app.middleware("http")
async def add_timing(request: Request, call_next):
    start = time.time()
    response = await call_next(request)
    response.headers["X-Process-Time"] = str(time.time() - start)
    return response
```

Think:

```text
Request
    ↓
middleware (before)
    ↓
route
    ↓
middleware (after)
    ↓
Response
```

Compare:

```text
Middleware → every route automatically
Depends() → only routes that declare it
```

Use middleware for cross-cutting concerns, `Depends()` for opt-in logic.

---

# 5. Exception Handlers

Default errors are JSON already, but you can customize shape.

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()


@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    return JSONResponse(
        status_code=400,
        content={"error": str(exc)}
    )
```

Trigger:

```python
@app.get("/divide")
def divide(a: int, b: int):
    if b == 0:
        raise ValueError("b cannot be zero")
    return {"result": a / b}
```

---

# 6. Custom Exceptions

Define your own error types for business logic.

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()


class TaskNotFound(Exception):
    def __init__(self, task_id: int):
        self.task_id = task_id


@app.exception_handler(TaskNotFound)
async def task_not_found_handler(request: Request, exc: TaskNotFound):
    return JSONResponse(
        status_code=404,
        content={"error": f"Task {exc.task_id} not found"}
    )


tasks = {1: {"title": "Learn FastAPI"}}


@app.get("/tasks/{task_id}")
def get_task(task_id: int):
    if task_id not in tasks:
        raise TaskNotFound(task_id)
    return tasks[task_id]
```

Think:

```text
raise TaskNotFound(99)
    ↓
handler catches it
    ↓
returns 404 JSON
```

Pattern: one exception class per business error, one handler per class.

---

# 7. CORS

Browsers block `fetch()` to a different origin unless server allows it. Your React app on `:3000` calling FastAPI on `:8000` needs CORS.

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

For interviews:

```text
No CORS → browser blocks frontend, Postman still works
With CORS → browser allows listed origins
```

Never use `allow_origins=["*"]` with credentials in production.

---

# 8. Background Tasks

Some work should happen **after** responding: send email, generate report.

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()


def send_email(email: str):
    print(f"Sending email to {email}")


@app.post("/users", status_code=201)
def create_user(email: str, background: BackgroundTasks):
    background.add_task(send_email, email)
    return {"created": email}
```

Flow:

```text
POST /users
    ↓
respond 201 immediately
    ↓
send_email() runs after
```

Use for fire-and-forget jobs. For retries and schedules, use APScheduler/Celery (later topic).

---

# 9. Authentication

Standard pattern: token in `Authorization` header, dependency verifies it.

```python
from fastapi import Depends, FastAPI, Header, HTTPException

app = FastAPI()


def get_current_user(authorization: str | None = Header(default=None)):
    if authorization != "Bearer abc123":
        raise HTTPException(status_code=401, detail="Unauthorized")
    return {"name": "Kakade", "role": "admin"}


@app.get("/me")
def me(user: dict = Depends(get_current_user)):
    return user
```

Test:

```python
import requests

requests.get(
    "http://127.0.0.1:8000/me",
    headers={"Authorization": "Bearer abc123"}
)
```

All protected routes just add `Depends(get_current_user)`.

---

# 10. OAuth2

OAuth2 is a standard for delegated login ("Login with Google"). FastAPI has built-in helpers.

Password flow shape:

```python
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

app = FastAPI()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="login")


@app.post("/login")
def login(form: OAuth2PasswordRequestForm = Depends()):
    # verify form.username / form.password
    return {"access_token": "fake-jwt", "token_type": "bearer"}


@app.get("/me")
def me(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

`OAuth2PasswordRequestForm` expects form data (`username`, `password`), not JSON. Swagger UI renders a nice Authorize button automatically.

---

# 11. JWT

JWT (JSON Web Token) is a signed token carrying user data. Server signs it, client sends it back, server verifies signature.

Structure:

```text
header.payload.signature
```

Create (needs `python-jose`):

```python
from jose import jwt

SECRET = "super-secret-key"

token = jwt.encode({"sub": "kakade"}, SECRET, algorithm="HS256")

print(token)
```

Output:

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJkYWthZGUifQ...
```

Verify:

```python
data = jwt.decode(token, SECRET, algorithms=["HS256"])

print(data)
```

Output:

```text
{'sub': 'kakade'}
```

Wrong secret raises error. That is the whole security model: only server knows `SECRET`.

---

# 12. Access Tokens

Short-lived tokens for API calls (e.g. 15 minutes).

```python
from datetime import datetime, timedelta, timezone
from jose import jwt

SECRET = "super-secret-key"


def create_access_token(user: str):
    expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    return jwt.encode(
        {"sub": user, "exp": expire, "type": "access"},
        SECRET,
        algorithm="HS256"
    )


print(create_access_token("kakade"))
```

FastAPI dependency decodes it per request. Expired token → 401, client must log in again.

---

# 13. Refresh Tokens

Long-lived tokens (e.g. 7 days) used only to get new access tokens.

Flow:

```text
1. POST /login → access (15m) + refresh (7d)
       ↓
2. Access expires → POST /refresh with refresh token
       ↓
3. Server returns new access token
       ↓
4. Refresh expires → login again
```

```python
@app.post("/refresh")
def refresh(refresh_token: str):
    data = jwt.decode(refresh_token, SECRET, algorithms=["HS256"])
    if data.get("type") != "refresh":
        raise HTTPException(status_code=401, detail="Invalid token")
    return {"access_token": create_access_token(data["sub"])}
```

Why two tokens? Access token stolen → expires fast. Refresh stored securely, sent rarely.

---

# 14. Response Models

Recap with auth context: never leak hashes or tokens.

```python
from pydantic import BaseModel


class UserOut(BaseModel):
    id: int
    name: str


@app.get("/me", response_model=UserOut)
def me():
    return {"id": 1, "name": "Kakade", "password_hash": "secret"}
```

Output:

```json
{"id": 1, "name": "Kakade"}
```

`password_hash` stripped. Use `response_model=` on every route returning user/order data in interviews.

---

# 15. API Versioning

When mobile apps depend on your API, you cannot break old responses. Version them.

```python
from fastapi import APIRouter, FastAPI

v1 = APIRouter(prefix="/v1")
v2 = APIRouter(prefix="/v2")


@v1.get("/users")
def list_users_v1():
    return [{"name": "Kakade"}]


@v2.get("/users")
def list_users_v2():
    return [{"id": 1, "name": "Kakade", "active": True}]


app = FastAPI()
app.include_router(v1)
app.include_router(v2)
```

```text
/v1/users → old shape, frozen
/v2/users → new shape
```

---

# 16. Project Structure

Interview-ready layout:

```text
app/
    main.py          → creates app, includes routers, middleware, handlers
    core/
        config.py    → env vars, secrets
        security.py  → JWT create/verify
    users/
        router.py    → APIRouter
        schemas.py   → Pydantic In/Out
        service.py   → business logic
    tasks/
        router.py
        schemas.py
        service.py
```

`main.py`:

```python
from fastapi import FastAPI
from users.router import router as users_router
from tasks.router import router as tasks_router

app = FastAPI(title="Interview Prep API")
app.include_router(users_router)
app.include_router(tasks_router)
```

Rule:

```text
router → validates HTTP
service → business logic
schemas → Pydantic shapes
```

Never put SQL inside router functions.

---

# Real-World Uses

```python
APIRouter + Depends + JWT
```

Standard backend: versioned routers, `get_current_user` dependency, access/refresh flow, background email on signup, CORS for React frontend.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **APIRouter** | Split by feature, `prefix=`, `include_router()` |
| **Depends()** | Inject paging, DB, user; `yield` for cleanup |
| **Middleware** | Every request, timing/logging/headers |
| **Handlers** | `@app.exception_handler()` custom JSON |
| **Custom exc** | Business error class + handler |
| **CORS** | `CORSMiddleware`, allow frontend origin |
| **Background** | `BackgroundTasks.add_task()` after response |
| **Auth** | Header check via dependency, 401 on fail |
| **OAuth2** | `OAuth2PasswordBearer`, form login, Swagger button |
| **JWT** | `header.payload.signature`, sign with secret |
| **Access** | 15m, sent every request |
| **Refresh** | 7d, only to mint new access |
| **Response model** | Strip secrets |
| **Versioning** | `/v1`, `/v2` routers |
| **Structure** | router/service/schemas split |

## The Mental Model

```text
        FASTAPI APP
             │
             ↓
   APIRouter per feature (/v1/users)
             │
             ↓
   Depends() injects (db, user, paging)
             │
   ┌─────────┼─────────┐
   ↓         ↓         ↓
Middleware Handlers  Background
 timing    404/400    email
             │
             ↓
   Auth: OAuth2 → JWT access + refresh
             │
             ↓
   Response model filters output
```

### The 3 things to remember

**1. Split and inject:**

```python
router = APIRouter(prefix="/users")
@app.get("/me")
def me(user: dict = Depends(get_current_user)):
    ...
```

**2. Tokens have lifetimes:**

```text
access 15m for calls, refresh 7d to renew
```

**3. Structure scales:**

```text
router → service → schemas, version with /v1
```

> **Intermediate FastAPI is routers plus dependencies: split features with APIRouter, share logic with Depends(), secure with JWT, and run side work in background tasks.**
