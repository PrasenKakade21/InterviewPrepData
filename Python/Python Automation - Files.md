# Python Automation - Files

File automation looks boring until you rename 500 files in 2 seconds. Once you know **find with glob → transform Path → act with shutil**, any folder becomes scriptable.

The core idea:

```text
Locate files (glob/rglob)
    ↓
filter + compute new names
    ↓
copy/move/rename/delete safely
```

Stdlib only: `pathlib`, `os`, `shutil`, `glob`. No pip needed.

---

# 1. pathlib Automation

`Path` is the base of all file scripts.

```python
from pathlib import Path

base = Path("demo_files")
base.mkdir(exist_ok=True)

(Path("demo_files") / "report.txt").write_text("Hello Kakade")

print(list(base.iterdir()))
```

Output:

```text
[WindowsPath('demo_files/report.txt')]
```

Useful in loops:

```python
for f in Path("demo_files").iterdir():
    print(f.name, f.suffix, f.stat().st_size)
```

Think:

```text
Path.iterdir() → one level
Path.rglob() → recursive
```

---

# 2. os Automation

`os` fills gaps: cwd, env, walk, sizes.

```python
import os

print(os.getcwd())
print(os.environ.get("USER", "kakade"))
print(os.path.getsize("demo_files/report.txt"))
```

`os.walk()` for manual recursion:

```python
for root, dirs, files in os.walk("demo_files"):
    print(root, files)
```

Prefer `Path.rglob()` for new code — shorter. Use `os` when you need env vars or low-level stats.

---

# 3. shutil

High-level actions: copy, move, delete trees, disk usage.

```python
import shutil

print(shutil.disk_usage("."))
```

Output:

```text
usage(total=..., used=..., free=...)
```

Copy/move/delete covered next with safe patterns.

---

# 4. glob

Pattern matching refresher (see Standard Library for basics).

```python
from pathlib import Path

for f in Path("demo_files").glob("*.txt"):
    print(f)
```

Patterns:

```text
*.txt → txt in folder
*.log → logs in folder
report_* → starts with report_
**/*.txt → recursive with rglob
```

---

# 5. File Searching

Find all large logs, all PDFs, all TODOs.

```python
from pathlib import Path

base = Path("demo_files")
base.mkdir(exist_ok=True)
(base / "a.txt").write_text("hello")
(base / "b.log").write_text("error here")

matches = list(base.rglob("*.log"))

print(matches)
```

Output:

```text
[WindowsPath('demo_files/b.log')]
```

Search by content:

```python
for f in Path("demo_files").rglob("*.log"):
    if "error" in f.read_text():
        print(f)
```

Think:

```text
rglob → by name
read_text + in → by content
```

---

# 6. File Renaming

Rename safely: build new name, check collision.

```python
from pathlib import Path

f = Path("demo_files/report.txt")
new = f.with_name("report_2026.txt")
f.rename(new)

print(new.exists())
```

Output:

```text
True
```

Bulk rename with numbering:

```python
for i, f in enumerate(Path("demo_files").glob("*.txt"), start=1):
    f.rename(f.with_name(f"doc_{i:02d}{f.suffix}"))
```

`{i:02d}` gives `01, 02`. Always dry-run print first before real rename.

---

# 7. Moving Files

Move = rename across folders.

```python
from pathlib import Path
import shutil

src = Path("demo_files/doc_01.txt")
dst_dir = Path("demo_files/archive")
dst_dir.mkdir(exist_ok=True)

shutil.move(str(src), str(dst_dir / src.name))

print(list(dst_dir.iterdir()))
```

Use `shutil.move()` over `Path.rename()` when crossing drives — `rename` fails there, `move` handles it.

---

# 8. Copying Files

```python
import shutil
from pathlib import Path

shutil.copy("demo_files/archive/doc_01.txt", "demo_files/doc_01_copy.txt")
shutil.copytree("demo_files/archive", "demo_files/archive_backup", dirs_exist_ok=True)
```

| Function | Use |
| -------- | --- |
| `shutil.copy()` | One file |
| `shutil.copytree()` | Whole folder |
| `shutil.copy2()` | Keep timestamps |

---

# 9. Deleting Files

Delete with care — no trash recovery.

```python
from pathlib import Path
import shutil

Path("demo_files/doc_01_copy.txt").unlink(missing_ok=True)
Path("demo_files/empty_dir").mkdir(exist_ok=True)
Path("demo_files/empty_dir").rmdir()
shutil.rmtree("demo_files/archive_backup", ignore_errors=True)
```

| Target | Use |
| ------ | --- |
| File | `.unlink()` |
| Empty dir | `.rmdir()` |
| Full dir | `shutil.rmtree()` |

`missing_ok=True` avoids crash if already deleted. Ideal for rerunnable scripts.

---

# 10. Directory Creation

```python
from pathlib import Path

Path("demo_files/2026/09").mkdir(parents=True, exist_ok=True)
```

`parents=True` creates intermediates. `exist_ok=True` makes reruns safe. Same as `os.makedirs(..., exist_ok=True)`.

Date-based folders for reports:

```python
from datetime import date
from pathlib import Path

today = date.today().isoformat()
Path(f"reports/{today}").mkdir(parents=True, exist_ok=True)
```

---

# 11. File Organization

Classic interview task: sort downloads by extension.

```python
from pathlib import Path
import shutil

downloads = Path("demo_files")
for f in downloads.iterdir():
    if f.is_file():
        folder = downloads / f.suffix.lstrip(".")
        folder.mkdir(exist_ok=True)
        shutil.move(str(f), str(folder / f.name))
```

Before:

```text
a.pdf b.jpg c.pdf
```

After:

```text
pdf/a.pdf pdf/c.pdf jpg/b.jpg
```

Add `if f.suffix` guard to skip extensionless files.

---

# 12. Batch Processing

Apply one action to many files: convert, compress, hash, count lines.

```python
from pathlib import Path

total_lines = 0
for f in Path("demo_files").rglob("*.txt"):
    total_lines += len(f.read_text().splitlines())

print(total_lines)
```

Output:

```text
5
```

Template for any batch job:

```python
for f in Path("demo_files").rglob("*.txt"):
    text = f.read_text()
    cleaned = text.strip().lower()
    f.write_text(cleaned)
```

```text
find → read → transform → write
```

Always process to new folder first, verify, then replace. Never overwrite originals blindly.

---

# Real-World Uses

```python
Nightly job → rglob logs → move old to archive/ → delete temp → report counts
```

Onboarding script, download organizer, log rotator — same 20 lines reshaped.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **pathlib** | `Path`, `iterdir`, `mkdir(parents, exist_ok)` |
| **os** | `getcwd`, `environ`, `walk` |
| **shutil** | `copy/move/rmtree/disk_usage` |
| **glob** | `glob(*.txt)`, `rglob` recursive |
| **Search** | Name via glob, content via `in` |
| **Rename** | `with_name` + `rename`, dry-run |
| **Move** | `shutil.move` cross-drive safe |
| **Copy** | `copy` file, `copytree` dir |
| **Delete** | `unlink/rmdir/rmtree`, `missing_ok` |
| **Mkdir** | Date folders for reports |
| **Organize** | Sort by `suffix` |
| **Batch** | Find → transform → write |

## The Mental Model

```text
        FIND
   glob / rglob / walk
            │
            ↓
        FILTER
   suffix / size / content
            │
            ↓
          ACT
 copy / move / rename / delete
            │
            ↓
     Batch loop + dry-run first
```

### The 3 things to remember

**1. Find recursively:**

```python
list(Path(".").rglob("*.log"))
```

**2. Act safely:**

```python
Path("x.txt").unlink(missing_ok=True)
Path("out").mkdir(parents=True, exist_ok=True)
```

**3. Batch template:**

```python
for f in Path(".").rglob("*.txt"):
    f.write_text(f.read_text().strip())
```

> **File automation is find-filter-act: locate with glob, compute with pathlib, execute with shutil — always dry-run destructive steps.**
