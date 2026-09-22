# Projects

Tutorials teach syntax. Projects teach **wiring** — CLI → logic → API → DB → schedule. Build these ten in order; each reuses the last.

The core idea:

```text
Problem → spec (inputs/outputs)
    ↓
build slice → test → commit
    ↓
demo with real data
```

Every project below lists stack, spec, and the one file to start with.

---

# 1. File Organizer Automation

Sorts a messy folder by extension. Tests `pathlib` + `shutil` + CLI.

```python
from pathlib import Path
import shutil

def organize(folder: str):
    base = Path(folder)
    moved = 0
    for f in base.iterdir():
        if f.is_file() and f.suffix:
            dest = base / f.suffix.lstrip(".")
            dest.mkdir(exist_ok=True)
            shutil.move(str(f), str(dest / f.name))
            moved += 1
    return moved


print(organize("demo_data"))
```

Output:

```text
5
```

Extend: `--dry-run` flag, log moves, schedule nightly. Interview demo in under 2 minutes.

---

# 2. API Data Fetching and Reporting Tool

Fetches JSON, saves CSV + Excel summary. Tests `httpx` + `pandas` + CLI.

```python
import httpx
import pandas as pd

r = httpx.get("https://jsonplaceholder.typicode.com/users", timeout=10)
df = pd.DataFrame(r.json())

print(df.shape)
print(df.columns.tolist()[:3])
```

Output:

```text
(10, 8)
['id', 'name', 'username']
```

Extend: `report.py fetch --out report.xlsx` with Raw + Summary sheets, `--date` flag, retry on timeout.

---

# 3. Flask REST API

Classic CRUD with Blueprints + app factory. Tests routes, JSON, error handlers.

```python
from flask import Flask, jsonify

app = Flask(__name__)
tasks = [{"id": 1, "title": "Prep for Kakade"}]

@app.get("/tasks")
def list_tasks():
    return jsonify(tasks)

print(app.url_map)
```

Output:

```text
Map([<Rule '/tasks' (GET, HEAD, OPTIONS)>])
```

Extend: `POST /tasks` with validation, `404` handler, Blueprint `tasks_bp`, factory `create_app()` reading `APP_ENV`.

---

# 4. FastAPI Task Management API

Same domain, modern stack: Pydantic validation + auto docs + `TestClient`.

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()

@app.get("/tasks")
def list_tasks():
    return [{"id": 1, "title": "Prep for Kakade"}]

client = TestClient(app)
print(client.get("/tasks").json())
```

Output:

```text
[{'id': 1, 'title': 'Prep for Kakade'}]
```

Extend: `TaskIn`/`TaskOut` models, `POST` → `201`, `GET /tasks/{id}` → `404`, `APIRouter(prefix="/tasks")`.

---

# 5. FastAPI + PostgreSQL Backend

Adds persistence: SQLAlchemy models + session-per-request + Alembic.

```python
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy import String
from sqlalchemy.orm import Mapped, mapped_column


class Base(DeclarativeBase):
    pass


class Task(Base):
    __tablename__ = "tasks"
    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(100))


print(Task.__tablename__)
```

Output:

```text
tasks
```

Extend: `get_db` dependency, `select(Task)` queries, `alembic revision --autogenerate`, `RETURNING id` on Postgres.

---

# 6. JWT Authentication API

Login → access + refresh tokens → protected routes. Tests OAuth2 + `Depends`.

```python
import jwt
import datetime

SECRET = "kakade-dev-secret-key-min-32-bytes-long"
payload = {
    "sub": "kakade",
    "exp": datetime.datetime.now(datetime.timezone.utc)
    + datetime.timedelta(minutes=15),
}
token = jwt.encode(payload, SECRET, algorithm="HS256")

print(jwt.decode(token, SECRET, algorithms=["HS256"])["sub"])
```

Output:

```text
kakade
```

Extend: `/login` issues pair, `get_current_user` dependency verifies, refresh rotation, `401` on expiry. Needs `pip install pyjwt`.

---

# 7. Web Scraping Automation

Nightly price/post tracker: requests → BeautifulSoup → CSV.

```python
from bs4 import BeautifulSoup

html = "<ul><li class='post'>Kakade post</li><li class='post'>Asha post</li></ul>"
soup = BeautifulSoup(html, "html.parser")

print([li.get_text(strip=True) for li in soup.select("li.post")])
```

Output:

```text
['Kakade post', 'Asha post']
```

Extend: paginate `?page=N`, `urljoin` links, append to `pandas` CSV, `time.sleep(1)` politeness, Playwright fallback for JS.

---

# 8. Playwright Browser Automation

Login + download + screenshot for JS sites.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.set_content("<h1>Hi Kakade</h1>")
    page.wait_for_selector("h1")
    print(page.get_by_role("heading").text_content())
    browser.close()
```

Output:

```text
Hi Kakade
```

Extend: fill login form, `storage_state("auth.json")`, screenshot evidence, `all_text_contents()` → CSV.

---

# 9. Excel Report Generator

Ops-facing output: `pandas` aggregate → multi-sheet → `openpyxl` polish.

```python
import pandas as pd

df = pd.DataFrame([
    {"city": "Pune", "amount": 100},
    {"city": "Pune", "amount": 200},
])
summary = df.groupby("city")["amount"].sum().reset_index()

print(summary.to_dict(orient="records"))
```

Output:

```text
[{'city': 'Pune', 'amount': 300}]
```

Extend: `ExcelWriter` Raw + Summary sheets, bold headers, freeze panes, dated filename `report_2026-09-16.xlsx`.

---

# 10. Scheduled Automation Tool

Puts it together: CLI + APScheduler/cron + logging + failure alerts.

```python
from apscheduler.schedulers.background import BackgroundScheduler

jobs = []


def nightly():
    jobs.append("Kakade report done")


scheduler = BackgroundScheduler()
scheduler.add_job(nightly, "interval", hours=24)

print(len(scheduler.get_jobs()))
```

Output:

```text
1
```

Extend: `tool.py run --date X` idempotent, `RotatingFileHandler` logs, `coalesce=True max_instances=1`, cron `0 9 * * *` in prod.

---

# Real-World Uses

```python
Portfolio order → organizer → API reporter → FastAPI tasks → Postgres → JWT → scraper → Excel → scheduler
```

Each project = one interview story: problem, stack, tradeoff, demo.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Organizer** | `pathlib` + `shutil`, `--dry-run` |
| **API reporter** | `httpx` → `pandas` → Excel |
| **Flask API** | Blueprint + factory |
| **FastAPI tasks** | Pydantic + `TestClient` |
| **Postgres backend** | Session dep + Alembic |
| **JWT API** | Access + refresh, `Depends` |
| **Scraper** | Selectors + pagination + CSV |
| **Playwright** | Login state + waits |
| **Excel gen** | Group + multi-sheet + polish |
| **Scheduler** | Idempotent CLI + cron |

## The Mental Model

```text
        AUTOMATE        BUILD API        PERSIST
   organizer/scheduler  Flask→FastAPI   Postgres+Alembic
            │                │               │
            └────────┬───────┴───────┬───────┘
                     ↓               ↓
              SCRAPE/REPORT      SECURE (JWT)
              pandas/Excel      Depends/auth
                     │               │
                     └───────┬───────┘
                             ↓
                    TEST + LOG + GIT
                    (pytest, levels, PRs)
```

### The 3 things to remember

**1. Smallest demoable slice:**

```python
def organize(folder: str) -> int: ...
```

**2. API contract first:**

```python
client = TestClient(app)
assert client.get("/tasks").status_code == 200
```

**3. Ship loop:**

```bash
pytest -q && ruff check . && git push
```

> **Finish the roadmap by shipping: automate files and data, serve them over tested Flask/FastAPI with Postgres and JWT, scrape what has no API, and schedule it all with logs.**
