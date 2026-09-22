# FastAPI Fundamentals — Revision Notes (Intermediate)
> One-line mental model: Type hints in the signature → auto validation + response filtering + free OpenAPI docs.
## Core recall
- App: `FastAPI()`, `uvicorn app:app --reload`; free docs: `/docs` (try it), `/redoc` (read), `/openapi.json`.
- Routes: `@app.get/post/put/patch/delete`; POST success via `status_code=201` in decorator (no tuple needed).
- Path: `def f(user_id: int)` from `/users/{user_id}`; bad type → 422 auto (Flask needs manual checks).
- Query: non-path args with defaults (`active: bool=True, page: int=1`); optional `role: str | None = None`.
- Body: `user: dict` simple, Pydantic `user: UserIn` standard (validates + docs); response filter `response_model=UserOut` strips secrets; errors `raise HTTPException(404)`.
- Extras: `Header()` (underscores→hyphens), `Cookie()`, `Response` for custom/set-cookie headers, `UploadFile = File(...)` (streams, not `.bytes`).
## Must-remember code
- ```python
from fastapi import FastAPI
app = FastAPI()
@app.get("/users/{user_id}")
def get_user(user_id: int, active: bool = True): return {"id": user_id, "active": active}
# /users/abc -> 422 auto
  ```
- ```python
from pydantic import BaseModel
class UserIn(BaseModel): name: str; email: str
class UserOut(BaseModel): id: int; name: str
@app.post("/users", status_code=201, response_model=UserOut)
def create(user: UserIn): return {"id": 1, "name": user.name}
  ```
- ```python
from fastapi import Header, Response
@app.get("/tasks")
def t(authorization: str | None = Header(default=None), response: Response = None):
    response.headers["X-App"] = "1.0"; return {"auth": authorization}
  ```
- ```python
from fastapi import File, UploadFile
@app.post("/upload")
async def up(file: UploadFile = File(...)):
    return {"filename": file.filename, "size": len(await file.read())}
  ```
## Interview traps
- `/users/abc` for `int` → 422, not 404/500 — FastAPI validates before your code runs.
- No `response_model` may leak `password_hash` — always filter output models.
- `Header`/`Cookie` names: `authorization` matches `Authorization`; forgetting `status_code=201` returns 200 on create.
- `bytes` loads whole file in RAM → use `UploadFile` for large uploads.
## 3 things to remember
1. Types do work: path `{id}` + query defaults from signature.
2. Models validate in, filter out (`response_model` strips secrets).
3. Docs free: `/docs` test, `/redoc` read, from OpenAPI spec.
