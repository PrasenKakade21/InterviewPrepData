# Logging and Configuration — Revision Notes (Intermediate)
> One-line mental model: Log leveled events; configure level/destination and secrets via env.
## Core recall
- `log = logging.getLogger(__name__)`; one `basicConfig` at entrypoint; libs never configure.
- Levels: DEBUG 10 internals, INFO 20 normal, WARNING 30 odd-but-ok, ERROR 40 failed task, CRITICAL 50 exit.
- `%s` args not f-strings (lazy); `log.exception()` inside `except` adds traceback.
- Format: `"%(asctime)s %(levelname)s %(name)s: %(message)s"`; JSON in prod, human locally.
- File: `RotatingFileHandler("app.log", maxBytes=..., backupCount=3)`; never manual `open("log.txt","a")`.
- Env config: `APP_ENV` switches dev/prod; `os.getenv("DB_URL", "sqlite:///app.db")`; fail fast if missing.
- Secrets: env only, `.env` + `python-dotenv` dev-only; share `.env.example`; `Config` class centralizes.
## Must-remember code
- ```python
  import logging
  logging.basicConfig(level=logging.INFO)
  log = logging.getLogger(__name__)
  log.info("Hello %s", "Kakade")
  ```
- ```python
  import logging
  logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(name)s: %(message)s")
  logging.getLogger("kakade").info("job done")
  ```
- ```python
  import logging
  try: open("missing.txt")
  except FileNotFoundError: logging.getLogger(__name__).exception("failed for Kakade")
  ```
- ```python
  import os
  ENV = os.getenv("APP_ENV", "dev")
  DB_URL = os.getenv("DB_URL", "sqlite:///app.db")
  print(ENV, DB_URL)
  ```
- ```python
  from logging.handlers import RotatingFileHandler
  import logging
  h = RotatingFileHandler("demo_data/app.log", maxBytes=10000, backupCount=3)
  logging.basicConfig(level=logging.INFO, handlers=[h])
  ```
## Interview traps
- `print()` in servers → unfilterable → use leveled `log.info/warning/error`.
- f-string in debug → always evaluated → use `log.debug("id=%s", 5)` lazy args.
- `log.error(...)` without traceback → mystery → use `log.exception` in `except`.
- Secrets in code/git → leak → env + `.gitignore` + `.env.example`.
- `basicConfig` in library + app → duplicate/noisy → configure once at entrypoint.
## 3 things to remember
1. DEBUG < INFO < WARNING < ERROR < CRITICAL; prod default INFO/WARNING.
2. Format once with time + level + name; rotate files.
3. One `Config` from env; secrets never committed.
