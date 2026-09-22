# Projects — Revision Notes (Intermediate)
> One-line mental model: Spec inputs/outputs → slice → test → commit; CLI → API → DB → schedule.
## Core recall
- Order: organizer → API reporter → Flask → FastAPI tasks → Postgres → JWT → scraper → Excel → scheduler.
- Organizer: `pathlib.iterdir` + `shutil.move` by suffix; `--dry-run`, log moves, nightly schedule.
- Reporter: `httpx.get` → `pandas.DataFrame` → CSV/Excel Raw + Summary; `--date`, retry timeout.
- Flask: Blueprints + `create_app()` factory + error handlers; FastAPI: Pydantic `TaskIn/Out`, `APIRouter`, `TestClient`.
- Postgres: `DeclarativeBase` model, `get_db` session dep, `select(Task)`, `alembic revision --autogenerate`.
- JWT: `/login` issues access (15min) + refresh; `Depends(get_current_user)` verifies; 401 on expiry.
- Scraper: `select("li.post")` + `urljoin` + paginate + `sleep(1)`; Playwright fallback for JS + `storage_state`.
- Excel: `groupby().sum()` → `ExcelWriter` multi-sheet + bold/freeze + dated filename; scheduler: idempotent CLI + cron.
## Must-remember code
- ```python
  from pathlib import Path
  import shutil
  def organize(folder: str) -> int:
      n = 0
      for f in Path(folder).iterdir():
          if f.is_file() and f.suffix:
              d = Path(folder) / f.suffix.lstrip("."); d.mkdir(exist_ok=True)
              shutil.move(str(f), str(d / f.name)); n += 1
      return n
  ```
- ```python
  import pandas as pd
  df = pd.DataFrame([{"city": "Pune", "amount": 100}, {"city": "Pune", "amount": 200}])
  print(df.groupby("city")["amount"].sum().reset_index().to_dict(orient="records"))
  ```
- ```python
  from fastapi import FastAPI
  from fastapi.testclient import TestClient
  app = FastAPI()
  @app.get("/tasks")
  def list_tasks(): return [{"id": 1, "title": "Prep for Kakade"}]
  assert TestClient(app).get("/tasks").status_code == 200
  ```
- ```python
  import jwt, datetime
  SECRET = "kakade-dev-secret-key-min-32-bytes-long"
  tok = jwt.encode({"sub": "kakade"}, SECRET, algorithm="HS256")
  print(jwt.decode(tok, SECRET, algorithms=["HS256"])["sub"])
  ```
- ```python
  from bs4 import BeautifulSoup
  soup = BeautifulSoup("<ul><li class='post'>Kakade post</li></ul>", "html.parser")
  print([li.get_text(strip=True) for li in soup.select("li.post")])
  ```
## Interview traps
- Giant "build all" start → no demo → smallest demoable slice + extend flags.
- No contract test → breaks silently → `TestClient` assert status + 201/404 first.
- `pytest/ruff` skipped → dirty PR → ship loop `pytest -q && ruff check . && git push`.
- Secrets/DB URL hardcoded → unleakable → env + `.env.example`.
- Non-idempotent nightly → duplicates → `--date` rerun + atomic rename + `coalesce`.
## 3 things to remember
1. Slice: `def organize(folder) -> int` demo in 2 min.
2. Contract: TestClient status + JSON before features.
3. Ship: test → lint → commit → push; schedule idempotent CLI.
