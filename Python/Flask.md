# Flask

Flask looks minimal at first, but once you understand **app → route → request → JSON response**, you can build full REST APIs.

The core idea:

```text
Flask application
    ↓
route maps URL to function
    ↓
function reads request
    ↓
returns JSON response
```

Flask is **micro and explicit**. It gives you routing and requests, but leaves validation, docs, and structure to you. That makes it perfect for learning how backends actually work.

Install:

```bash
pip install flask flask-cors
```

---

# 1. Flask Application

Minimal app:

```python
from flask import Flask

app = Flask(__name__)


@app.get("/health")
def health():
    return {"status": "ok"}


if __name__ == "__main__":
    app.run(debug=True)
```

Run:

```bash
python app.py
```

Output in browser at `http://127.0.0.1:5000/health`:

```json
{"status": "ok"}
```

Think:

```text
Flask(__name__)
    │
    ↓
creates app
    │
    ↓
@app.get() registers routes
    │
    ↓
app.run() starts server
```

---

# 2. Routes

A route connects URL to function.

```python
@app.get("/users")
def list_users():
    return {"users": ["Kakade", "Asha"]}
```

Old style also works:

```python
@app.route("/users", methods=["GET"])
def list_users():
    ...
```

Prefer `@app.get()` / `@app.post()` for clarity.

Multiple routes, same file:

```python
@app.get("/")
def home():
    return {"message": "Hello Kakade"}


@app.get("/about")
def about():
    return {"app": "Interview Prep API"}
```

---

# 3. HTTP Methods

Same function can handle REST methods.

```python
from flask import Flask, request

app = Flask(__name__)


@app.get("/tasks")
def list_tasks():
    return {"tasks": []}


@app.post("/tasks")
def create_task():
    data = request.get_json()
    return {"created": data}, 201
```

Or one route, many methods:

```python
@app.route("/tasks", methods=["GET", "POST"])
def tasks():
    if request.method == "GET":
        return {"tasks": []}
    return {"created": True}, 201
```

First style is cleaner for interviews.

---

# 4. Request Object

`request` holds everything the client sent.

```python
from flask import request

@app.post("/users")
def create_user():
    print(request.method)
    print(request.path)
    print(request.args)
    print(request.headers)
    print(request.get_json())
    return {"ok": True}
```

Common attributes:

| Attribute | Gives you |
| --------- | --------- |
| `request.method` | GET, POST, ... |
| `request.path` | `/users` |
| `request.args` | Query params |
| `request.headers` | Headers dict |
| `request.get_json()` | Parsed JSON body |
| `request.cookies` | Cookies |

---

# 5. Response Object

Flask is flexible. These all work:

```python
@app.get("/a")
def a():
    return {"name": "Kakade"}
```

```python
@app.get("/b")
def b():
    return {"name": "Kakade"}, 201
```

```python
from flask import jsonify

@app.get("/c")
def c():
    return jsonify(name="Kakade"), 200
```

Think:

```text
return dict
    ↓
Flask converts to JSON, status 200

return dict, status
    ↓
custom status code

return jsonify()
    ↓
explicit JSON response
```

For errors, return message + code:

```python
@app.get("/users/<int:user_id>")
def get_user(user_id):
    return {"error": "User not found"}, 404
```

---

# 6. JSON Responses

REST APIs speak JSON.

```python
@app.get("/users/5")
def get_user():
    return {
        "id": 5,
        "name": "Kakade",
        "active": True
    }
```

Flask automatically sets:

```text
Content-Type: application/json
```

For lists, same:

```python
@app.get("/users")
def list_users():
    return [
        {"id": 1, "name": "Kakade"},
        {"id": 2, "name": "Asha"}
    ]
```

---

# 7. Path Parameters

Dynamic URL parts use `<type:name>`.

```python
@app.get("/users/<int:user_id>")
def get_user(user_id):
    return {"id": user_id, "name": "Kakade"}
```

Visit `/users/5`:

```json
{"id": 5, "name": "Kakade"}
```

Types:

```text
<int:user_id>   → integer
<string:name>   → string (default)
<float:price>   → float
<path:subpath>  → with slashes
```

Multiple params:

```python
@app.get("/orders/<int:year>/<int:month>")
def orders(year, month):
    return {"year": year, "month": month}
```

---

# 8. Query Parameters

Query params live in `request.args`.

URL:

```text
/tasks?active=true&page=2
```

Code:

```python
@app.get("/tasks")
def list_tasks():
    active = request.args.get("active", "true")
    page = request.args.get("page", "1")
    return {"active": active, "page": page}
```

With types:

```python
page = request.args.get("page", 1, type=int)
limit = request.args.get("limit", 10, type=int)
```

Compare:

```text
Path param → /users/5 → which resource
Query param → ?active=true → filter/how many
```

---

# 9. Request Body

Read JSON with `get_json()`.

```python
from flask import request

@app.post("/users")
def create_user():
    data = request.get_json()

    if not data or "name" not in data:
        return {"error": "name is required"}, 400

    return {"id": 1, "name": data["name"]}, 201
```

Test with requests:

```python
import requests

requests.post(
    "http://127.0.0.1:5000/users",
    json={"name": "Kakade"}
)
```

Important: if client forgets `Content-Type: application/json`, `get_json()` returns `None`. Always validate.

---

# 10. Headers

Read headers like a dict:

```python
@app.get("/tasks")
def list_tasks():
    token = request.headers.get("Authorization")
    if token != "Bearer abc123":
        return {"error": "Unauthorized"}, 401
    return {"tasks": []}
```

Send custom headers back:

```python
@app.get("/users")
def list_users():
    return {"users": []}, 200, {"X-App-Version": "1.0"}
```

Or with `make_response`:

```python
from flask import make_response

@app.get("/users")
def list_users():
    response = make_response({"users": []})
    response.headers["X-App-Version"] = "1.0"
    return response
```

---

# 11. Blueprints

Blueprints split a large app into modules. Without them, everything lives in one file.

Structure:

```text
app.py
users/
    __init__.py
    routes.py
tasks/
    __init__.py
    routes.py
```

`users/routes.py`:

```python
from flask import Blueprint

users_bp = Blueprint("users", __name__, url_prefix="/users")


@users_bp.get("")
def list_users():
    return {"users": ["Kakade"]}


@users_bp.get("/<int:user_id>")
def get_user(user_id):
    return {"id": user_id}
```

`app.py`:

```python
from flask import Flask
from users.routes import users_bp

app = Flask(__name__)
app.register_blueprint(users_bp)
```

Think:

```text
Blueprint
    ↓
mini-app with routes
    ↓
registered on main app
    ↓
/users + "" = /users
```

---

# 12. Application Factory

The factory pattern creates the app inside a function. This makes testing much easier.

Bad for growth:

```python
app = Flask(__name__)

@app.get("/health")
def health():
    ...
```

Good:

```python
from flask import Flask


def create_app():
    app = Flask(__name__)

    @app.get("/health")
    def health():
        return {"status": "ok"}

    return app
```

Run:

```python
app = create_app()
app.run(debug=True)
```

Why? Tests can create fresh apps with different config:

```python
app = create_app()
app.config["TESTING"] = True
```

---

# 13. Configuration

Config lives in `app.config`.

```python
def create_app():
    app = Flask(__name__)
    app.config["DEBUG"] = True
    app.config["API_KEY"] = "secret"
    return app
```

Better with environments:

```python
import os


def create_app():
    app = Flask(__name__)
    app.config["ENV"] = os.environ.get("FLASK_ENV", "development")
    app.config["SECRET_KEY"] = os.environ.get("SECRET_KEY", "dev-key")
    return app
```

Load from object for larger apps:

```python
class Config:
    DEBUG = False
    TESTING = False


app.config.from_object(Config)
```

---

# 14. Environment Variables

Never hardcode secrets.

```python
import os
from flask import Flask

app = Flask(__name__)

DATABASE_URL = os.environ.get("DATABASE_URL")
if not DATABASE_URL:
    raise ValueError("DATABASE_URL is not set")
```

With `.env` files (needs `python-dotenv`):

```bash
pip install python-dotenv
```

```python
from dotenv import load_dotenv

load_dotenv()
```

`.env`:

```text
DATABASE_URL=postgresql://localhost/mydb
SECRET_KEY=super-secret
```

---

# 15. Error Handlers

Custom JSON errors instead of HTML pages.

```python
@app.errorhandler(404)
def not_found(error):
    return {"error": "Not found"}, 404


@app.errorhandler(500)
def server_error(error):
    return {"error": "Server failed"}, 500
```

Manual abort:

```python
from flask import abort

@app.get("/admin")
def admin():
    abort(403)
```

Combined with handler:

```python
@app.errorhandler(403)
def forbidden(error):
    return {"error": "No permission"}, 403
```

---

# 16. Middleware / Hooks

Flask calls them hooks: `before_request` and `after_request`.

```python
from flask import request

@app.before_request
def log_request():
    print(f"{request.method} {request.path}")


@app.after_request
def add_headers(response):
    response.headers["X-App"] = "InterviewPrep"
    return response
```

Use cases:

* Logging
* Auth checks
* Timing
* CORS headers

Auth example:

```python
@app.before_request
def check_auth():
    if request.path.startswith("/admin"):
        token = request.headers.get("Authorization")
        if token != "Bearer abc123":
            return {"error": "Unauthorized"}, 401
```

Returning a response from `before_request` stops the request.

---

# 17. CORS

Browsers block frontend JS from calling a different-domain API unless the server allows it.

Enable with Flask-CORS:

```python
from flask import Flask
from flask_cors import CORS

app = Flask(__name__)
CORS(app)
```

Restrict in production:

```python
CORS(app, origins=["https://kakade.dev"])
```

Without CORS, your React app on `localhost:3000` cannot call Flask on `localhost:5000` from the browser.

---

# 18. Authentication

Simple token check:

```python
from functools import wraps
from flask import request


def require_token(view):
    @wraps(view)
    def wrapper(*args, **kwargs):
        token = request.headers.get("Authorization")
        if token != "Bearer abc123":
            return {"error": "Unauthorized"}, 401
        return view(*args, **kwargs)
    return wrapper


@app.get("/tasks")
@require_token
def list_tasks():
    return {"tasks": []}
```

This is the decorator pattern from earlier, applied to web routes. JWT details come in FastAPI Intermediate.

---

# 19. Database Integration

Flask does not include a database. You add one.

Conceptual pattern (full SQLAlchemy in later topic):

```python
@app.get("/users")
def list_users():
    # 1. open DB session
    # 2. query users table
    # 3. convert rows to dicts
    # 4. return JSON
    return {"users": [{"id": 1, "name": "Kakade"}]}
```

For interviews, know this separation:

```text
Route → validates request
Service → business logic
DB layer → queries database
```

Do not put raw SQL inside route functions in real projects.

---

# 20. Building REST APIs with Flask

Complete in-memory CRUD example combining everything.

```python
from flask import Flask, request

app = Flask(__name__)

tasks = {}
next_id = 1


@app.get("/tasks")
def list_tasks():
    return {"tasks": list(tasks.values())}


@app.post("/tasks")
def create_task():
    global next_id
    data = request.get_json()
    if not data or "title" not in data:
        return {"error": "title is required"}, 400
    task = {"id": next_id, "title": data["title"]}
    tasks[next_id] = task
    next_id += 1
    return task, 201


@app.get("/tasks/<int:task_id>")
def get_task(task_id):
    task = tasks.get(task_id)
    if not task:
        return {"error": "Not found"}, 404
    return task


@app.delete("/tasks/<int:task_id>")
def delete_task(task_id):
    if task_id not in tasks:
        return {"error": "Not found"}, 404
    del tasks[task_id]
    return {"deleted": True}
```

Test flow:

```text
POST /tasks {"title": "Learn Flask"} → 201
GET /tasks → 200, list with 1 item
GET /tasks/1 → 200
DELETE /tasks/1 → 200
GET /tasks/1 → 404
```

This is the exact shape interviewers ask you to build.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **App** | `Flask(__name__)`, `app.run()` |
| **Routes** | `@app.get()`, `@app.post()` |
| **Request** | `request.args`, `get_json()`, `headers` |
| **Response** | `return dict, status` |
| **Path params** | `/users/<int:id>` |
| **Query params** | `request.args.get()` |
| **Body** | `request.get_json()` + validate |
| **Blueprints** | Split routes, `register_blueprint()` |
| **Factory** | `def create_app():` for tests |
| **Config** | `app.config`, env vars |
| **Errors** | `@app.errorhandler()` |
| **Hooks** | `before_request`, `after_request` |
| **CORS** | `flask-cors` for browser access |
| **Auth** | Header check / decorator |

## The Mental Model

```text
        FLASK APP
            │
            ↓
    Route maps URL
            │
    ┌───────┼───────┐
    ↓       ↓       ↓
  Path    Query    Body
 /u/5   ?a=true  JSON
    │       │       │
    └───────┼───────┘
            ↓
    function logic
            ↓
    return dict + status
            │
            ↓
   Blueprints + Factory
   scale the app
```

### The 3 things to remember

**1. Route → function → JSON:**

```python
@app.get("/users/<int:user_id>")
def get_user(user_id):
    return {"id": user_id}
```

**2. Validate input:**

```python
data = request.get_json()
if not data or "name" not in data:
    return {"error": "name required"}, 400
```

**3. Structure for growth:**

```text
Blueprints split routes, factory creates app, env vars hold secrets
```

> **Flask maps URLs to functions; read path/query/body from `request`, return JSON with status, and scale with blueprints and the app factory.**
