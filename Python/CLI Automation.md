# CLI Automation

A script becomes a tool when someone else can run it with `--help` and sensible errors. `argparse` ships with Python; `Typer` makes it prettier.

The core idea:

```text
python tool.py --name Kakade --count 2
    ↓
parse → validate → run
    ↓
clean output or clear error
```

Install for second half:

```bash
pip install typer
```

---

# 1. argparse

Stdlib CLI parser. Define arguments, it handles `--help` free.

```python
import argparse

parser = argparse.ArgumentParser(description="Greet Kakade")
parser.add_argument("--name", default="Kakade")

print(parser.parse_args(["--name", "Asha"]))
```

Output:

```text
Namespace(name='Asha')
```

Run real: `python greet.py --name Asha` → `Namespace(name='Asha')`. `--help` prints usage automatically.

---

# 2. Command-Line Arguments

Positional = required, order matters. Optional = `--flag`, order free.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("src")
parser.add_argument("dst")
parser.add_argument("--overwrite", action="store_true")

print(parser.parse_args(["a.txt", "b.txt", "--overwrite"]))
```

Output:

```text
Namespace(src='a.txt', dst='b.txt', overwrite=True)
```

Think:

```text
src dst → positionals
--overwrite → optional switch
```

---

# 3. Flags

`store_true` makes `--verbose` a boolean switch. `type=` + `choices=` constrain values.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--verbose", action="store_true")
parser.add_argument("--count", type=int, default=1)
parser.add_argument("--city", choices=["pune", "mumbai"], default="pune")

print(parser.parse_args(["--verbose", "--count", "3"]))
```

Output:

```text
Namespace(verbose=True, count=3, city='pune')
```

Bad input (`--count abc`) exits with a clear error — no manual `int()` try/except needed.

---

# 4. Subcommands

Like `git push` / `git pull`: one tool, many verbs. `add_subparsers` routes them.

```python
import argparse

parser = argparse.ArgumentParser()
subs = parser.add_subparsers(dest="cmd", required=True)
subs.add_parser("add").add_argument("name")
subs.add_parser("list")

print(parser.parse_args(["add", "Kakade"]))
```

Output:

```text
Namespace(cmd='add', name='Kakade')
```

Dispatch:

```python
args = parser.parse_args(["add", "Kakade"])
if args.cmd == "add":
    print(f"adding {args.name}")
```

---

# 5. Typer

`Typer` turns type hints into a CLI — less boilerplate, auto help, colors.

```python
import typer

app = typer.App()

@app.command()
def greet(name: str = "Kakade", count: int = 1):
    for _ in range(count):
        print(f"Hello {name}")

if __name__ == "__main__":
    app()
```

Run: `python tool.py --name Asha --count 2`

Output:

```text
Hello Asha
Hello Asha
```

`argparse` = stdlib, no deps. `Typer` = nicer UX, needs install. Know both for interviews.

---

# 6. Building Reusable CLI Tools

Structure every tool the same way: `parse → validate → pure function → print`.

```python
import argparse
from pathlib import Path

def rename_files(folder: str, prefix: str):
    return [f"{prefix}_{p.name}" for p in Path(folder).glob("*.txt")]

parser = argparse.ArgumentParser(description="Bulk rename .txt files")
parser.add_argument("folder")
parser.add_argument("--prefix", default="doc")

args = parser.parse_args(["demo_data", "--prefix", "kakade"])
print(rename_files(args.folder, args.prefix)[:2])
```

Output:

```text
['kakade_out.csv', 'kakade_users.csv']
```

Keep logic importable (`rename_files()` testable without CLI). `if __name__ == "__main__":` calls `main()`.

---

# 7. Input Validation

Validate early with `type`, `choices`, or custom function raising `ArgumentTypeError`.

```python
import argparse

def age(value: str):
    v = int(value)
    if not 0 <= v <= 120:
        raise argparse.ArgumentTypeError("age must be 0-120")
    return v

parser = argparse.ArgumentParser()
parser.add_argument("--age", type=age, default=25)

print(parser.parse_args(["--age", "30"]))
```

Output:

```text
Namespace(age=30)
```

Typer equivalent: `age: int = typer.Option(25, min=0, max=120)`. Invalid input → clean usage error, not traceback.

---

# 8. CLI Error Handling

Exit codes tell scripts what happened: `0` success, non-zero failure.

```python
import argparse
import sys

parser = argparse.ArgumentParser()
parser.add_argument("src")

args = parser.parse_args(["missing.txt"])
try:
    open(args.src).read()
except FileNotFoundError:
    parser.error(f"file not found: {args.src}")
```

Output:

```text
usage: ... 
error: file not found: missing.txt
(exit code 2)
```

Rules:

```text
parser.error() → usage + exit 2
raise SystemExit(1) → silent fail for scripts
print to stderr → sys.exit("message")
```

Never let raw tracebacks reach end users for expected errors.

---

# Real-World Uses

```python
organize.py --folder ~/Downloads --dry-run → preview moves
report.py build --date 2026-09-16 → dated Excel
```

Every automation in this roadmap ends as a CLI + scheduler entry.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **argparse** | Stdlib, auto `--help` |
| **Arguments** | Positional required, `--opt` free order |
| **Flags** | `store_true`, `type`, `choices` |
| **Subcommands** | `add_subparsers`, `git`-style verbs |
| **Typer** | Hints → CLI, less code |
| **Reusable tools** | Logic separate, `main()` thin |
| **Validation** | Fail fast with clear message |
| **Errors** | `parser.error`, exit codes |

## The Mental Model

```text
        CLI CALL
 python tool.py <cmd> --flags
            │
            ↓
    argparse / Typer parses
            │
     ┌──────┴──────┐
     ↓             ↓
  valid → run    invalid → usage + exit 2
     │
     ↓
 pure function + clean output
```

### The 3 things to remember

**1. Basic flag:**

```python
parser.add_argument("--name", default="Kakade")
```

**2. Subcommand:**

```python
subs = parser.add_subparsers(dest="cmd", required=True)
subs.add_parser("add")
```

**3. Typer shortcut:**

```python
@app.command()
def greet(name: str = "Kakade"):
    print(f"Hello {name}")
```

> **Good CLIs parse with argparse or Typer, validate early with types and choices, and fail with usage plus exit codes — logic stays importable and testable.**
