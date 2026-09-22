# Git for Python Projects

Git looks like memorized commands until you think **working dir → stage → commit → remote**. Then every command has a home.

The core idea:

```text
Edit files
    ↓
stage (add) → snapshot (commit) → share (push)
    ↓
branches isolate work, merge combines
```

No install block — `git` ships with the OS or `git --version` confirms.

---

# 1. git init

Creates a repo — a `.git` folder tracking history.

```bash
git init
git status
```

Output:

```text
Initialized empty Git repository
On branch main, nothing to commit
```

Run once per project, at root. Clone with `git clone <url>` instead when joining existing work.

---

# 2. git add

Stages changes — picks what goes into the next commit.

```bash
git add app.py
git add .
git status --short
```

Output:

```text
A  app.py
M  notes.txt
```

```text
working dir → (add) → staging → (commit) → history
```

Stage intentionally: `git add src/` beats blind `git add .` when scratch files exist.

---

# 3. git commit

Snapshots staged changes with a message.

```bash
git commit -m "Add Kakade greeting"
```

Output:

```text
[main abc123] Add Kakade greeting
 1 file changed
```

Good messages: imperative, short — "Add login", not "added stuff". Commit early, commit often; each commit = one idea.

---

# 4. git branch

Branches isolate work without breaking `main`.

```bash
git branch feature-login
git switch feature-login
git branch --show-current
```

Output:

```text
feature-login
```

```text
main ──●──●
            \
             ●──● feature-login
```

Name pattern: `feature/`, `fix/`, `chore/` prefixes. `main` always deployable.

---

# 5. git merge

Combines branches. Fast-forward when linear, merge commit when diverged.

```bash
git switch main
git merge feature-login
```

Output:

```text
Updating abc123..def456
Fast-forward
```

Conflict? Git marks files, you pick lines, then `git add` + `git commit`. Pull often to stay mergeable.

---

# 6. git pull

Fetches remote + merges into current branch.

```bash
git pull origin main
```

Output:

```text
Already up to date.
```

Think:

```text
pull = fetch + merge
push = publish my commits
```

Pull before starting work and before pushing — avoids "rejected" surprises.

---

# 7. git push

Publishes commits to GitHub.

```bash
git push origin feature-login
```

Output:

```text
To github.com:kakade/prep.git
 * [new branch] feature-login -> feature-login
```

First push sets upstream: `git push -u origin feature-login`, then bare `git push` works.

---

# 8. .gitignore

Lists files Git must never track: venvs, caches, env secrets.

```python
# demo: what a Python .gitignore contains
print(open(".gitignore").read() if False else "__pycache__/\n.venv/\n.env\n*.pyc")
```

Output:

```text
__pycache__/
.venv/
.env
*.pyc
```

Minimal Python set:

```text
__pycache__/ *.pyc
.venv/ venv/
.env
.pytest_cache/ .ruff_cache/
dist/ build/ *.egg-info/
```

Commit `.gitignore` first — untracking later needs `git rm --cached`.

---

# 9. .env Handling

`.env` holds secrets locally, never reaches GitHub.

```bash
echo ".env" >> .gitignore
```

```python
import os

print(os.getenv("API_TOKEN", "missing") == "missing")
```

Output:

```text
True
```

Share shape via `.env.example` (keys, fake values). Prod secrets come from platform env, not files.

---

# 10. requirements.txt

Freezes deps so others reproduce your env.

```bash
pip freeze > requirements.txt
cat requirements.txt
```

Output:

```text
fastapi==0.136.0
requests==2.33.0
```

Install: `pip install -r requirements.txt`. Commit it always. Modern alternative: `pyproject.toml` + `uv.lock` — same idea, better resolver.

---

# 11. README.md

Tells strangers how to run your project in 2 minutes.

```markdown
# Kakade Prep API

## Setup
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

## Run
uvicorn app:app --reload

## Test
pytest -q
```

Sections: what, setup, run, test, env vars. No README = no users.

---

# 12. Project Structure

Standard layout interviews expect:

```text
kakade-api/
├── src/
│   └── myapp/
│       ├── __init__.py
│       ├── main.py
│       ├── routers/
│       ├── services/
│       └── models.py
├── tests/
├── .env.example
├── .gitignore
├── pyproject.toml
├── requirements.txt
└── README.md
```

Rules: source under `src/`, tests mirror source, config at root, secrets never committed.

---

# Real-World Uses

```python
feature → branch → pytest → commit → push → PR → merge to main
```

Every project in topic 35 follows this loop.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **init** | Once per project |
| **add** | Stage intentionally |
| **commit** | One idea, imperative msg |
| **branch** | Isolate, `feature/` prefix |
| **merge** | Combine, resolve conflicts |
| **pull** | `fetch + merge`, do often |
| **push** | Publish, `-u` first time |
| **.gitignore** | venv, cache, `.env` |
| **.env** | Local secrets + `.example` |
| **requirements** | `freeze` + `-r` install |
| **README** | Setup/run/test in 2 min |
| **Structure** | `src/ tests/ root config` |

## The Mental Model

```text
      WORKING DIR (edits)
            │
       git add (stage)
            │
      git commit (snapshot)
            │
     ┌──────┴──────┐
     ↓             ↓
 branch        git push (share)
 (isolate)         │
     │        git pull (sync)
     └──────┬──────┘
            ↓
       git merge (combine)
```

### The 3 things to remember

**1. Daily loop:**

```bash
git pull origin main
git add src/
git commit -m "Add Kakade feature"
git push
```

**2. Feature branch:**

```bash
git switch -c feature-login
git push -u origin feature-login
```

**3. Never commit:**

```text
.venv/ __pycache__/ .env *.pyc
```

> **Git is stage-snapshot-share: edit, add intentionally, commit atomically on branches, and push through reviewed merges with secrets ignored.**
