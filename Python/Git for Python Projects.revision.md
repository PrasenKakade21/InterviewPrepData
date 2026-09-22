# Git for Python Projects — Revision Notes (Intermediate)
> One-line mental model: Edit → stage → snapshot → share; branches isolate, merges combine.
## Core recall
- `git init` once at root (or `clone`); `git status` / `--short` shows `A/M/??`.
- `git add app.py` stages intentionally; `add .` blind; working dir → stage → history.
- `git commit -m "Add Kakade greeting"` imperative, one idea per commit, early/often.
- Branches: `git switch -c feature-login`, `feature/`/`fix/` prefixes; `main` deployable.
- Merge: `switch main + merge feature-login`; fast-forward vs merge commit; resolve markers → add → commit.
- `pull = fetch + merge` (sync down, do often); `push` publishes (`-u` first time sets upstream).
- `.gitignore`: `__pycache__/ *.pyc .venv/ .env .pytest_cache/ dist/`; commit it first.
- `.env` never committed; share `.env.example`; `requirements.txt` via `freeze`, install `-r`.
- README: what/setup/run/test/env in 2 min; layout `src/myapp tests/ root config`.
## Must-remember code
- ```bash
  git pull origin main
  git add src/
  git commit -m "Add Kakade feature"
  git push
  ```
- ```bash
  git switch -c feature-login
  git push -u origin feature-login
  ```
- ```bash
  git switch main
  git merge feature-login
  ```
- ```python
  import os
  print(os.getenv("API_TOKEN", "missing") == "missing")
  ```
- ```bash
  pip freeze > requirements.txt
  pip install -r requirements.txt
  ```
## Interview traps
- `git add .` with scratch files → junk committed → stage `src/` intentionally.
- "added stuff" mega-commit → unreviewable → atomic imperative commits.
- Committed `.env`/`.venv` → leak/bloat → gitignore first; `git rm --cached` to fix.
- Push rejected → forgot pull → pull before work and before push.
- No README/structure → unusable → `src/ tests/ .env.example requirements.txt README`.
## 3 things to remember
1. Daily loop: pull → add → commit → push.
2. Feature branch → PR → merge to main.
3. Never commit `.venv/ __pycache__/ .env`.
