# Python Standard Library — Revision Notes (Intermediate)
> One-line mental model: Batteries included — `os/pathlib/shutil` system+files, `datetime` time, `collections/itertools/functools` data, `re` text, `subprocess` commands.
## Core recall
- `os`: `getcwd()`, `listdir()`, `makedirs("r/2026", exist_ok=True)`, `os.path.join/exists/isfile/isdir/basename/dirname`.
- `os.environ.get("API_KEY")` → None safe; `os.environ["X"]` → KeyError; setting only affects current process; fail-fast `if not DB: raise`.
- `pathlib`: `Path("r")/"2026"`, `cwd()/home()`, `.exists()/.is_file()/.mkdir(parents=True,exist_ok=True)`, `.read_text()/.write_text()`, `.name/.suffix/.stem/.parent`; prefer over `os.path`.
- `shutil`: `copy/move/copytree(dirs_exist_ok=True)/rmtree()` (no trash!); file delete `Path.unlink()`, empty dir `rmdir()`.
- `glob`: `Path("Python").glob("*.md")`, `rglob("*.md")` recursive; `**/*.md` all subfolders.
- `datetime`: `date.today()`, `datetime.now()`, `strftime` (dt→str, f=format) vs `strptime` (str→dt, p=parse); `%Y %m %d %H %M %S`; `timedelta(days=7)` arithmetic; aware `datetime.now(timezone.utc)`.
- `re`: raw `r"\d+"`, `search` (anywhere, first) vs `match` (start only), `findall`, groups `(\w+)@(\w+)`, `sub(r"\D","",s)`; `\d \D \w \s + * ^ $`.
- `collections`: `Counter(marks).most_common(2)`, `defaultdict(int/list)` avoids KeyError, `deque` `appendleft/popleft(maxlen=3)` + sliding window.
- `itertools`: `count/cycle/combinations(order-free)/permutations/product(nested loops)`; `functools`: `lru_cache/partial/reduce/wraps`.
- `subprocess.run(["echo","Hi"], capture_output=True, text=True, check=True)`; list args, avoid `shell=True` (injection); `returncode 0` = ok.
## Must-remember code
- ```python
  from pathlib import Path
  p = Path("reports") / "2026"; p.mkdir(parents=True, exist_ok=True)
  Path("hello.txt").write_text("Hello Kakade")
  print(Path("Python/roadmap.json").suffix)
  ```
- ```python
  import re
  print(re.findall(r"\d+", "Kakade scored 90, 85 and 95"))
  print(re.sub(r"\D", "", "Phone: 987-654-3210"))
  ```
- ```python
  from collections import Counter, defaultdict, deque
  print(Counter(["python","sql","python"]).most_common(1))
  d = defaultdict(list); d["backend"].append("Kakade"); print(dict(d))
  ```
- ```python
  from datetime import datetime, timezone, timedelta
  now = datetime.now(timezone.utc); print((now + timedelta(days=7)).strftime("%Y-%m-%d"))
  ```
## Interview traps
- String concat paths `"a"+"/"+"b"` breaks on Windows → `os.path.join` / `Path()/`.
- `re.match` vs `search`: match only start → `match(r"\d+","Kakade 123")` is None, use search.
- `strftime` vs `strptime` swap → f = to string, p = parse from string.
- `shutil.rmtree` + `shell=True` with user input = unrecoverable/injection → list args, no shell.
- Naive vs aware datetime compare fails → store UTC (`timezone.utc`), convert only for display.
## 3 things to remember
1. Files: `pathlib` modern, `shutil` bulk ops, `rglob` recursive.
2. Config safe: `os.environ.get` + fail-fast; secrets never hardcoded.
3. Data shortcuts: `Counter/defaultdict/deque/lru_cache` beat hand loops.
