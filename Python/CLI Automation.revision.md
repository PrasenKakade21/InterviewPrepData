# CLI Automation — Revision Notes (Intermediate)
> One-line mental model: Parse → validate → run pure function; fail with usage + exit code.
## Core recall
- `argparse`: stdlib, auto `--help`; `add_argument("src")` positional vs `--flag` optional.
- Flags: `action="store_true"`, `type=int`, `choices=[...]`, `default=...`; bad input exits 2.
- Subcommands: `add_subparsers(dest="cmd", required=True)` → `git`-style verbs; dispatch on `args.cmd`.
- Typer: hints → CLI (`def greet(name: str = "Kakade")`), less boilerplate, needs install.
- Structure: thin `main()` + importable pure function; `if __name__ == "__main__": main()`.
- Validation: `type` fn raising `ArgumentTypeError`, or Typer `Option(min=, max=)`; fail fast.
- Errors: `parser.error(msg)` → usage + exit 2; `sys.exit("msg")` → stderr; never raw traceback.
## Must-remember code
- ```python
  import argparse
  p = argparse.ArgumentParser(description="Greet Kakade")
  p.add_argument("--name", default="Kakade")
  print(p.parse_args(["--name", "Asha"]))
  ```
- ```python
  import argparse
  p = argparse.ArgumentParser()
  p.add_argument("--count", type=int, default=1)
  p.add_argument("--city", choices=["pune", "mumbai"], default="pune")
  print(p.parse_args(["--count", "3"]))
  ```
- ```python
  import argparse
  p = argparse.ArgumentParser()
  subs = p.add_subparsers(dest="cmd", required=True)
  subs.add_parser("add").add_argument("name")
  print(p.parse_args(["add", "Kakade"]))
  ```
- ```python
  import typer
  app = typer.App()
  @app.command()
  def greet(name: str = "Kakade", count: int = 1):
      for _ in range(count):
          print(f"Hello {name}")
  ```
- ```python
  import argparse
  def age(v: str):
      i = int(v)
      if not 0 <= i <= 120:
          raise argparse.ArgumentTypeError("age must be 0-120")
      return i
  ```
## Interview traps
- Positional order matters, `--opt` order free → don't make required things optional flags.
- Forgetting `required=True` on subparsers → silently runs with no command → set it.
- Manual `int()` + try/except → redundant → use `type=` / `choices=` for free errors.
- Business logic inside `main()` → untestable → keep pure function importable.
- `print(traceback)` for missing file → bad UX → use `parser.error("file not found...")`.
## 3 things to remember
1. `store_true` / `type` / `choices` validate for free with exit code 2.
2. Subcommands route verbs; dispatch on `args.cmd`.
3. Typer = hints-driven nicety; argparse = stdlib default — know both.
