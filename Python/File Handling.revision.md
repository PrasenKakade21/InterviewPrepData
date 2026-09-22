# File Handling — Revision Notes (Intermediate)
> One-line mental model: `with open()` borrows the file; modes decide read vs clobber; stream big files, don't load them.

## Core recall
- Always `with open(path, mode, encoding="utf-8") as f:` — auto-close even on error/`return`; manual open/close leaks on exception.
- Modes: `r` read (missing → `FileNotFoundError`), `w` truncate/create, `a` append/create, `x` exclusive (exists → `FileExistsError`), `r+` read-write; +`b` (`rb/wb`) for bytes.
- Read: `read()` whole str, `readline()` one line, `readlines()` list, `for line in f:` streaming (use for GB logs; `line.strip()`).
- Write: `write(s)` (no auto `\n`, returns len), `writelines([...])` (no auto newlines — include `\n`).
- Binary: `bytes` not `str` (`b"..."`/`.encode()`); chunk copy `src.read(4096)` until `b""`; decode with `.decode("ascii"/"utf-8")`.
- `pathlib`: `Path("a")/"b.txt"`, `.name/.suffix/.stem/.parent`, `.exists()/.is_file()/.is_dir()`, `mkdir(parents=True, exist_ok=True)`, `glob/rglob`, `read_text/write_text(encoding=...)`.

## Must-remember code
```python
from pathlib import Path
p = Path("logs/app.log")
p.parent.mkdir(parents=True, exist_ok=True)
p.write_text("Kakade started\n", encoding="utf-8")
print(p.read_text(encoding="utf-8"))
```
```python
with open("users.csv", "w", encoding="utf-8") as f:
    f.writelines(["kakade,active\n", "rahul,suspended\n"])
with open("users.csv", encoding="utf-8") as f:
    for line in f:
        print(f"Processed: {line.strip()}")
```
```python
def copy_media(src, dst, chunk=4096):
    with open(src, "rb") as s, open(dst, "wb") as d:
        while chunk_data := s.read(chunk):
            d.write(chunk_data)
```
```python
import datetime
def audit(event, uid, path="audit.log"):
    ts = datetime.datetime.now().isoformat()
    with open(path, "a", encoding="utf-8") as f:
        f.write(f"[{ts}] {event} | {uid}\n")
audit("USER_LOGIN", "Kakade")
```

## Interview traps
- `w` silently erases existing file → use `a` to preserve; `r` on missing → `FileNotFoundError`, `x` on existing → `FileExistsError`.
- `read()/readlines()` on 10 GB log → OOM → stream `for line in f:` / chunks.
- `print(line)` double-spaces (`line` already ends `\n`) → `print(line.strip())` or `print(line, end="")`.
- Binary `write("str")` → `TypeError`; forgetting `encoding="utf-8"` breaks on Windows (`cp1252`).
- `os.path.join/splitext/exists` legacy — prefer `Path / .suffix / .exists()`.

## 3 things to remember
1. `with` + explicit `encoding="utf-8"` always.
2. Stream large files line-by-line / chunk-by-chunk.
3. `pathlib` for paths; `w` clobbers, `a` appends.
