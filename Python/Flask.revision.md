# Flask — Revision Notes (Intermediate)
> One-line mental model: App → route maps URL to function → read `request` → return JSON + status; scale with blueprints + factory.
## Core recall
- App: `Flask(__name__)`, `@app.get/post(...)` (prefer over `@app.route(..., methods=)`), `app.run(debug=True)` — port 5000.
- Request: `method/path/args(headers)/get_json()/cookies`; response: `return dict` (200 JSON), `return dict, status`, `jsonify()`, `make_response` for headers; errors `return {"error":...}, 404`.
- Params: path `<int:user_id>` (`int/string/float/path`); query `request.args.get("page", 1, type=int)`; body `request.get_json()` + validate `if not data or "name" not in data: 400`.
- Structure: Blueprints = mini-apps (`Blueprint(..., url_prefix=...)` + `register_blueprint`); factory `def create_app()` for tests/config; config via `app.config` + `os.environ`/dotenv.
- Cross-cutting: `@app.errorhandler(404/500)` + `abort(403)`; `before_request` (return stops req) / `after_request` for auth/log/headers; `flask-cors` `CORS(app)` for browser calls; token decorator with `@wraps`.
## Must-remember code
- ```python
from flask import Flask, request
app = Flask(__name__)
@app.get("/users/<int:user_id>")
def get_user(user_id): return {"id": user_id, "name": "Kakade"}
@app.post("/users")
def create():
    d = request.get_json()
    if not d or "name" not in d: return {"error": "name required"}, 400
    return {"id": 1, "name": d["name"]}, 201
  ```
- ```python
from flask import request
@app.get("/tasks")
def tasks():
    page = request.args.get("page", 1, type=int)
    tok = request.headers.get("Authorization")
    if tok != "Bearer abc123": return {"error": "Unauthorized"}, 401
    return {"page": page}
  ```
- ```python
from flask import Blueprint
users_bp = Blueprint("users", __name__, url_prefix="/users")
@users_bp.get("")
def lst(): return {"users": ["Kakade"]}
# app.register_blueprint(users_bp)
  ```
- ```python
from flask import Flask
def create_app():
    app = Flask(__name__)
    @app.get("/health")
    def h(): return {"status": "ok"}
    return app
  ```
- ```python
@app.errorhandler(404)
def nf(e): return {"error": "Not found"}, 404
@app.before_request
def log(): print(request.method, request.path)
  ```
## Interview traps
- `get_json()` → None if client missed `Content-Type: application/json` — always validate before `data["x"]`.
- `return dict` always 200; POST-create must return `, 201`; DELETE often 200/204.
- One-file app doesn't scale/tests poorly → use blueprints + factory + env secrets.
- CORS missing → React on :3000 blocked calling Flask on :5000; restrict `origins=` in prod.
## 3 things to remember
1. Route → function → JSON: read path/query/body from `request`.
2. Validate input; return `dict, status` (201 create, 404/400 errors).
3. Blueprints split routes, factory creates app, env vars hold secrets.
