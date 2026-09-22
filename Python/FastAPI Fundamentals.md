# FastAPI Fundamentals

FastAPI looks like Flask at first, but once you understand **type hints → validation → docs**, it becomes much more powerful.

The core idea:

```text
Function signature with types
    ↓
FastAPI validates request
    ↓
runs your function
    ↓
validates response + generates docs
```

FastAPI is **typed and automatic**. You declare what you expect, it handles parsing, validation, and OpenAPI docs.

Install:

```bash
pip install fastapi uvicorn
```

Run:

```bash
uvicorn app:app --reload
```

---

# 1. FastAPI Application

Minimal app:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/health")
def health():
    return {"status": "ok"}
```

Compare with Flask:

```text
Flask:
  app = Flask(__name__)

FastAPI:
  app = FastAPI()
```

Visit:

```text
http://127.0.0.1:8000/health → your JSON
http://127.0.0.1:8000/docs → Swagger UI
http://127.0.0.1:8000/redoc → ReDoc
```

Docs come free. That is FastAPI's killer feature.

---

# 2. Routes

Same decorator style, different import.

```python
@app.get("/")
def home():
    return {"message": "Hello Kakade"}
```

Full CRUD shape:

```python
@app.get("/tasks")
def list_tasks():
    return {"tasks": []}


@app.post("/tasks")
def create_task():
    return {"created": True}
```

---

# 3. GET / POST / PUT / PATCH / DELETE

FastAPI has explicit decorators for each method.

```python
from fastapi import FastAPI

app = FastAPI()

tasks = {}


@app.get("/tasks")
def list_tasks():
    return list(tasks.values())


@app.post("/tasks", status_code=201)
def create_task(task: dict):
    task_id = len(tasks) + 1
    tasks[task_id] = task
    return {"id": task_id, **task}


@app.put("/tasks/{task_id}")
def replace_task(task_id: int, task: dict):
    tasks[task_id] = task
    return {"id": task_id, **task}


@app.patch("/tasks/{task_id}")
def update_task(task_id: int, updates: dict):
    tasks[task_id].update(updates)
    return tasks[task_id]


@app.delete("/tasks/{task_id}")
def delete_task(task_id: int):
    del tasks[task_id]
    return {"deleted": True}
```

Notice `status_code=201` on POST. Declarative, no tuple return needed.

---

# 4. Path Parameters

Declare them as function arguments with types.

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"id": user_id, "name": "Kakade"}
```

Visit `/users/5`:

```json
{"id": 5, "name": "Kakade"}
```

Visit `/users/abc`:

```json
{
  "detail": [
    {
      "loc": ["path", "user_id"],
      "msg": "Input should be a valid integer"
    }
  ]
}
```

Status `422`. FastAPI validated automatically. In Flask you would write manual checks.

Multiple params:

```python
@app.get("/orders/{year}/{month}")
def orders(year: int, month: int):
    return {"year": year, "month": month}
```

---

# 5. Query Parameters

Non-path arguments become query params automatically.

```python
@app.get("/tasks")
def list_tasks(active: bool = True, page: int = 1, limit: int = 10):
    return {"active": active, "page": page, "limit": limit}
```

Request:

```text
/tasks?active=false&page=2&limit=5
```

Response:

```json
{"active": false, "page": 2, "limit": 5}
```

Think:

```text
/path/{x} in URL → path param
other args with defaults → query params
```

Optional query param:

```python
@app.get("/users")
def list_users(role: str | None = None):
    if role:
        return {"filter": role}
    return {"filter": "all"}
```

---

# 6. Request Bodies

Dicts work, but Pydantic models are standard (next file).

Simple dict version:

```python
@app.post("/users")
def create_user(user: dict):
    return {"created": user}
```

Send:

```json
{"name": "Kakade"}
```

Response:

```json
{"created": {"name": "Kakade"}}
```

With Pydantic (preview):

```python
from pydantic import BaseModel


class UserIn(BaseModel):
    name: str
    email: str


@app.post("/users", status_code=201)
def create_user(user: UserIn):
    return {"id": 1, **user.model_dump()}
```

FastAPI parses JSON, validates types, and shows schema in docs.

---

# 7. Response Models

Response models filter and document output.

```python
from pydantic import BaseModel


class UserOut(BaseModel):
    id: int
    name: str


@app.get("/users/5", response_model=UserOut)
def get_user():
    return {"id": 5, "name": "Kakade", "password_hash": "secret"}
```

Output:

```json
{"id": 5, "name": "Kakade"}
```

`password_hash` is stripped. Critical for security interviews.

Think:

```text
Request model → what you accept
Response model → what you expose
```

Never return passwords, tokens, or internal fields.

---

# 8. Status Codes

Declare success codes in decorator.

```python
@app.post("/users", status_code=201)
def create_user():
    return {"id": 1}
```

```python
@app.delete("/users/{user_id}", status_code=204)
def delete_user(user_id: int):
    return None
```

Common:

```text
GET → 200
POST → 201
DELETE → 200 or 204
```

For errors, raise `HTTPException` (full details in FastAPI Intermediate):

```python
from fastapi import HTTPException

@app.get("/users/{user_id}")
def get_user(user_id: int):
    raise HTTPException(status_code=404, detail="User not found")
```

---

# 9. Headers

Read headers with `Header()`.

```python
from fastapi import Header


@app.get("/tasks")
def list_tasks(authorization: str | None = Header(default=None)):
    return {"auth": authorization}
```

Send:

```text
Authorization: Bearer abc123
```

Note: FastAPI converts underscores to hyphens. `authorization` matches `Authorization` header.

Custom response headers:

```python
from fastapi import Response


@app.get("/users")
def list_users(response: Response):
    response.headers["X-App-Version"] = "1.0"
    return {"users": []}
```

---

# 10. Cookies

Similar to headers, with `Cookie()`.

```python
from fastapi import Cookie


@app.get("/prefs")
def prefs(session: str | None = Cookie(default=None)):
    return {"session": session}
```

Set cookies on response:

```python
from fastapi import Response


@app.post("/login")
def login(response: Response):
    response.set_cookie(key="session", value="abc123")
    return {"logged_in": True}
```

For APIs, prefer `Authorization` headers over cookies. Cookies matter more for browser flows.

---

# 11. File Uploads

Two ways: `bytes` for small files, `UploadFile` for real use.

```python
from fastapi import File, UploadFile


@app.post("/upload")
def upload(file: UploadFile = File(...)):
    return {
        "filename": file.filename,
        "content_type": file.content_type
    }
```

Read content:

```python
@app.post("/upload")
async def upload(file: UploadFile = File(...)):
    content = await file.read()
    return {"size": len(content)}
```

Why `UploadFile`?

```text
.bytes → loads whole file in memory
UploadFile → streams, handles large files
```

Always use `UploadFile` in interviews unless asked otherwise.

---

# 12. OpenAPI

OpenAPI is a machine-readable spec of your API. FastAPI generates it from your code.

```text
Your type hints
    ↓
OpenAPI JSON at /openapi.json
    ↓
Swagger UI + ReDoc render it
```

No extra YAML needed. Change a model, docs update instantly.

---

# 13. Swagger UI

Interactive docs at `/docs`.

Features:

* Try endpoints from browser
* See request schemas
* See response schemas
* Copy curl commands

This is why teams love FastAPI. Frontend devs can test without Postman.

---

# 14. ReDoc

Cleaner read-only docs at `/redoc`.

Use:

```text
/docs → interactive testing
/redoc → clean reading/sharing
```

Both come from the same OpenAPI spec.

---

# Flask vs FastAPI

| Aspect | Flask | FastAPI |
| ------ | ----- | ------- |
| Style | Micro, manual | Typed, automatic |
| Validation | Manual `if` checks | Type hints + Pydantic |
| Docs | Manual | `/docs` free |
| Speed | Sync by default | Async-ready |
| Learning | Shows HTTP basics | Faster production APIs |

Learn Flask first for concepts, use FastAPI for real typed APIs.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **App** | `FastAPI()`, `uvicorn app:app` |
| **Routes** | `@app.get/post/put/patch/delete` |
| **Path** | `def f(user_id: int)` |
| **Query** | Other args with defaults |
| **Body** | Dict or Pydantic model |
| **Response model** | `response_model=` filters output |
| **Status** | `status_code=201` in decorator |
| **Headers** | `Header()` |
| **Cookies** | `Cookie()` |
| **Uploads** | `UploadFile = File(...)` |
| **Docs** | `/docs`, `/redoc`, `/openapi.json` |

## The Mental Model

```text
        FASTAPI
            │
            ↓
   Type hints in signature
            │
  ┌─────────┼─────────┐
  ↓         ↓         ↓
 Path     Query      Body
 {id}    ?page=1    JSON model
            │
            ↓
  Auto validation → 422 on fail
            │
            ↓
  Response model filters output
            │
            ↓
  OpenAPI → /docs + /redoc
```

### The 3 things to remember

**1. Types do work:**

```python
@app.get("/users/{user_id}")
def get_user(user_id: int, active: bool = True):
    ...
```

Path + query from signature.

**2. Models validate:**

```python
@app.post("/users", status_code=201, response_model=UserOut)
def create_user(user: UserIn):
    ...
```

**3. Docs are free:**

```text
/docs for testing, /redoc for reading
```

> **FastAPI turns type hints into validation and docs: declare path, query, and body types, filter with response models, and get OpenAPI for free.**
