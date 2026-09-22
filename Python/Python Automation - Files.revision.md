# Python Automation - Files — Revision Notes (Intermediate)
> One-line mental model: Find with glob → transform with Path → act with shutil, dry-run first.
## Core recall
- `Path`: `iterdir()` one level, `rglob("*.log")` recursive, `name/suffix/stat()`, `read_text/write_text`.
- `os`: `getcwd()`, `environ.get()`, `walk()` manual recursion; `Path.rglob` shorter for new code.
- `shutil`: `copy/copy2/copytree`, `move`, `rmtree`, `disk_usage()`.
- Glob: `*.txt`, `report_*`, `**/*.txt` recursive; content search via `"error" in f.read_text()`.
- Rename: `with_name()` + `rename()`; bulk `f"doc_{i:02d}{suffix}"`; dry-run print first.
- Move: `shutil.move()` cross-drive safe (`rename` fails across drives); `mkdir(exist_ok=True)` dest.
- Copy: `copy` file, `copy2` keeps timestamps, `copytree(dirs_exist_ok=True)` folder.
- Delete: file `unlink(missing_ok=True)`, empty `rmdir()`, tree `rmtree()`; no trash — rerunnable with guards.
- Mkdir: `mkdir(parents=True, exist_ok=True)`; date folders `reports/{date.today()}`.
- Organize: sort by `suffix.lstrip(".")` into subfolders; guard extensionless.
- Batch: `find → read → transform → write`; verify in new folder before replacing originals.
## Must-remember code
```python
from pathlib import Path
for f in Path("demo_files").rglob("*.log"):
    if "error" in f.read_text(): print(f)
```
```python
from pathlib import Path
for i, f in enumerate(Path("demo_files").glob("*.txt"), start=1):
    print(f"doc_{i:02d}{f.suffix}")  # dry-run then f.rename(...)
```
```python
from pathlib import Path
import shutil
p = Path("demo_files/a.txt"); d = Path("demo_files/txt"); d.mkdir(exist_ok=True)
shutil.move(str(p), str(d / p.name))
```
```python
from pathlib import Path
total = sum(len(f.read_text().splitlines()) for f in Path("demo_files").rglob("*.txt"))
print(total)
```
## Interview traps
- Real rename without dry-run → mass misname; always print first.
- `rename` across drives fails → use `shutil.move`.
- `rmtree/unlink` no recovery → `missing_ok`, archive before delete.
- Overwriting originals in batch → write to new folder, verify, then replace.
- `iterdir` misses nested → `rglob` for recursive.
## 3 things to remember
1. Find recursively: `Path(".").rglob("*.log")`.
2. Act safely: `missing_ok`, `parents+exist_ok`, dry-run.
3. Batch loop: find → transform → write.
