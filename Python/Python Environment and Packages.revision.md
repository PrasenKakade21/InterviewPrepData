# Python Environment and Packages — Revision Notes (Intermediate)
> One-line mental model: `venv` isolates, `pip` installs, `requirements.txt` reproduces, env vars configure.
## Core recall
- `python -m venv venv` → create; `venv\Scripts\activate` (Win) / `source venv/bin/activate` (*nix); `deactivate`; gitignore `venv/`.
- `pip install requests`, `==2.32.3` pinned, `>=2.30` minimum, multiple pkgs at once; `-U/--upgrade`; `pip install -r requirements.txt`.
- `pip uninstall requests` (confirm Y); `pip list` (human view) vs `pip freeze` (pinned `Pkg==x.y.z` for requirements).
- `pip freeze > requirements.txt` to snapshot; commit txt, not `venv/`.
- Env vars: `os.getenv("API_KEY")` → None if missing; `os.getenv("PORT","8000")` default; config changes without code change.
- `.env` file (`API_KEY=abc123`) NOT auto-loaded; needs `python-dotenv`: `load_dotenv()` then `os.getenv`.
- Poetry: `poetry new/add/install/remove/update/show`, `poetry run python app.py`, deps in `pyproject.toml`.
- uv: `uv init/add/sync/run/venv`, `uv pip install`, lockfile `uv.lock`; fast unified toolchain.
## Must-remember code
- ```python
  # bash workflow
  # python -m venv venv; venv\Scripts\activate; pip install flask; pip freeze > requirements.txt
  ```
- ```python
  import os
  port = os.getenv("PORT", "8000")
  api_key = os.getenv("API_KEY")
  print(port, api_key)
  ```
- ```python
  from dotenv import load_dotenv
  import os
  load_dotenv()
  print(os.getenv("APP_NAME"))  # Kakade.dev from .env
  ```
- ```python
  # fail-fast config pattern
  import os
  db = os.environ.get("DATABASE_URL")
  if not db: raise ValueError("DATABASE_URL is not set")
  ```
## Interview traps
- Forgetting `load_dotenv()` → `os.getenv` returns None even though `.env` exists → always call it first.
- Committing `venv/` or `.env` secrets → gitignore both (`venv/`, `.env`).
- `pip list` vs `pip freeze` confusion → list = "what's installed?", freeze = "exact versions to reproduce".
- `os.environ["KEY"]` raises KeyError if missing → prefer `os.environ.get("KEY")` / `os.getenv`.
- Installing globally without venv → version clash (Django 4 vs 5) → one venv per project.
## 3 things to remember
1. venv isolates, pip installs, freeze records, dotenv loads.
2. Never commit `venv/` or `.env`; commit `requirements.txt`.
3. Use `os.getenv` with defaults + fail-fast check for required config.
