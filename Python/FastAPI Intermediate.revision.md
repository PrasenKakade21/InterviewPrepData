# FastAPI Intermediate — Revision Notes (Intermediate)
> One-line mental model: APIRouter splits features → Depends injects shared logic → JWT secures, background defers.
## Core recall
- `APIRouter(prefix="/users", tags=[...])` mini-app; `app.include_router()` mounts; `/v1`, `/v2` for versioning.
- `Depends(fn)`: pagination/auth/DB injected; runs before route; chainable `get_token → get_user → route`.
- Yield deps: code before `yield` setup, after cleanup (DB close); one session per request.
- Middleware `@app.middleware("http")`: every request (timing/logging); Depends = opt-in only.
- Handlers: `@app.exception_handler(ValueError)` → custom JSON; custom `TaskNotFound` + handler → 404.
- CORS: `CORSMiddleware(allow_origins=["http://localhost:3000"])`; no CORS → browser blocks, Postman works.
- `BackgroundTasks.add_task(send_email, email)`: respond 201 now, work after; no retries (use Celery).
- Auth: `Header(Authorization)` dependency → 401; protect routes with `Depends(get_current_user)`.
- OAuth2: `OAuth2PasswordBearer(tokenUrl="login")`, `OAuth2PasswordRequestForm` form-data login, Swagger Authorize button.
- JWT: `header.payload.signature`, `jwt.encode/decode(SECRET, HS256)`; wrong secret fails.
- Access 15m every request; refresh 7d only to mint new access; check `type=="refresh"`.
- `response_model=UserOut` strips `password_hash`; structure: router → service → schemas.
## Must-remember code
```python
from fastapi import APIRouter, FastAPI
r = APIRouter(prefix="/users")
@r.get("")
def list_users(): return [{"name": "Kakade"}]
app = FastAPI(); app.include_router(r)
```
```python
from fastapi import Depends
def get_db():
    db = {"connected": True}
    try: yield db
    finally: print("close")
```
```python
from fastapi import BackgroundTasks
def send_email(e: str): print(e)
@app.post("/users", status_code=201)
def create(email: str, bg: BackgroundTasks):
    bg.add_task(send_email, email); return {"created": email}
```
```python
from jose import jwt
from datetime import datetime, timedelta, timezone
SECRET = "super-secret-key"
exp = datetime.now(timezone.utc) + timedelta(minutes=15)
tok = jwt.encode({"sub": "kakade", "exp": exp}, SECRET, algorithm="HS256")
```
## Interview traps
- `rename` vs middleware: middleware = all routes, Depends = declared routes only.
- OAuth2 form expects `username/password` form-data, not JSON → 422 if JSON sent.
- `allow_origins=["*"]` with credentials insecure → list explicit frontend origin.
- BackgroundTasks fire-and-forget, no retry → Celery/APScheduler for critical jobs.
- Returning dict with `password_hash` leaks → always `response_model=UserOut`.
## 3 things to remember
1. Split with APIRouter, inject with Depends (yield for cleanup).
2. Access 15m + refresh 7d; JWT verified by secret.
3. Router validates HTTP, service holds logic, schemas shape I/O; version `/v1`.
