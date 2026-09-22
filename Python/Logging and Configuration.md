# Logging and Configuration

`print` debugging dies in production. `logging` plus env-based config survives it.

The core idea:

```text
Code logs events (DEBUG→CRITICAL)
    ↓
config picks level + destination
    ↓
secrets stay in env, not code
```

Stdlib only, plus `python-dotenv` for `.env` files.

```bash
pip install python-dotenv
```

---

# 1. logging Module

Never `print` in servers — use `getLogger(__name__)`.

```python
import logging

logging.basicConfig(level=logging.INFO)
log = logging.getLogger(__name__)

log.info("Hello Kakade")
```

Output:

```text
INFO:__main__:Hello Kakade
```

One `basicConfig` at entrypoint. Library code only creates loggers, never configures.

---

# 2. DEBUG

Verbose internals: variable dumps, loop steps. Off in production.

```python
import logging

logging.basicConfig(level=logging.DEBUG)
log = logging.getLogger(__name__)

log.debug("user Kakade id=%s", 5)
```

Output:

```text
DEBUG:__main__:user Kakade id=5
```

Use `%s` args, not f-strings — skipped when level disabled, faster.

---

# 3. INFO

Normal events: server started, job finished, user signed up.

```python
import logging

logging.basicConfig(level=logging.INFO)
log = logging.getLogger(__name__)

log.info("report built for Kakade")
```

Output:

```text
INFO:__main__:report built for Kakade
```

Default production level. If it's not worth paging anyone, it's INFO.

---

# 4. WARNING

Something odd, but continuing: deprecated API, retrying, disk 80% full.

```python
import logging

logging.basicConfig(level=logging.WARNING)
log = logging.getLogger(__name__)

log.warning("API slow, retrying for Kakade")
log.info("this is hidden")
```

Output:

```text
WARNING:__main__:API slow, retrying for Kakade
```

INFO hidden because WARNING > INFO. Levels filter upward.

---

# 5. ERROR

A request/job failed, app continues: payment declined, file missing.

```python
import logging

logging.basicConfig(level=logging.ERROR)
log = logging.getLogger(__name__)

try:
    open("missing.txt")
except FileNotFoundError as e:
    log.error("failed for Kakade: %s", e)
```

Output:

```text
ERROR:__main__:failed for Kakade: [Errno 2] ...
```

Log exception with `log.exception("...")` inside `except` — adds traceback free.

---

# 6. CRITICAL

App can't continue: DB unreachable, no disk, corrupt config.

```python
import logging
import sys

logging.basicConfig(level=logging.CRITICAL)
log = logging.getLogger(__name__)

log.critical("database down, exiting")
```

Output:

```text
CRITICAL:__main__:database down, exiting
```

```text
DEBUG 10 < INFO 20 < WARNING 30 < ERROR 40 < CRITICAL 50
```

Set level once via env: `LOG_LEVEL=DEBUG python app.py`.

---

# 7. Log Formatting

Timestamps + names make logs greppable.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s: %(message)s",
)
log = logging.getLogger("kakade")

log.info("job done")
```

Output:

```text
2026-09-16 10:00:00,000 INFO kakade: job done
```

JSON format in production (for Loki/CloudWatch). Human format locally.

---

# 8. File Logging

Console locally, file + rotation in servers.

```python
import logging
from logging.handlers import RotatingFileHandler

handler = RotatingFileHandler("demo_data/app.log", maxBytes=10000, backupCount=3)
logging.basicConfig(level=logging.INFO, handlers=[handler])

logging.getLogger(__name__).info("Hello Kakade file")

print(open("demo_data/app.log").read().strip()[-20:])
```

Output:

```text
Hello Kakade file
```

`RotatingFileHandler` prevents 10 GB log files. Never `open("log.txt", "a")` manually.

---

# 9. Environment-Based Configuration

Same code, different envs via `APP_ENV` + env vars.

```python
import os

ENV = os.getenv("APP_ENV", "dev")
DEBUG = ENV == "dev"

print(ENV)
print(DEBUG)
```

Output:

```text
dev
True
```

```text
dev → DEBUG, SQLite, verbose logs
prod → WARNING, Postgres, JSON logs
```

Read config at startup, fail fast if required vars missing.

---

# 10. Secrets Management

Secrets in env, never in git. `.env` locally, vault in prod.

```python
import os

token = os.getenv("API_TOKEN", "missing")

print("set" if token != "missing" else "missing")
```

Output:

```text
missing
```

`.env` file:

```text
API_TOKEN=secret123
```

Load with `python-dotenv` in dev only. Prod injects real secrets via platform env.

---

# 11. Configuration Classes

One object per env — no scattered `os.getenv` calls.

```python
import os


class Config:
    ENV: str = os.getenv("APP_ENV", "dev")
    DB_URL: str = os.getenv("DB_URL", "sqlite:///app.db")
    LOG_LEVEL: str = os.getenv("LOG_LEVEL", "INFO")


print(Config.ENV)
print(Config.DB_URL)
```

Output:

```text
dev
sqlite:///app.db
```

FastAPI pattern: `@lru_cache def get_settings()` returns `Settings(BaseSettings)` from Pydantic — typed, validated, cached.

---

# Real-World Uses

```python
uvicorn app:app → INFO console locally, JSON file in prod, DB_URL from env
```

Cron job logs `ERROR` with traceback → alert channel picks it up.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **logging** | `getLogger(__name__)`, config once |
| **DEBUG** | Internals, `%s` args |
| **INFO** | Normal events, prod default |
| **WARNING** | Odd but continuing |
| **ERROR** | Failed task, `log.exception` |
| **CRITICAL** | App can't continue |
| **Formatting** | Time + level + name |
| **File logging** | `RotatingFileHandler` |
| **Env config** | `APP_ENV` switches behavior |
| **Secrets** | Env only, never git |
| **Config classes** | One typed object |

## The Mental Model

```text
        LEVELS
 DEBUG < INFO < WARNING < ERROR < CRITICAL
            │
            ↓
   logger.info("event for Kakade")
            │
     ┌──────┴──────┐
     ↓             ↓
  console       rotating file
  (dev)         (prod JSON)
            │
            ↓
  Config class ← env + .env
  (DB_URL, LOG_LEVEL, secrets)
```

### The 3 things to remember

**1. Log, don't print:**

```python
log = logging.getLogger(__name__)
log.info("Hello Kakade")
```

**2. Format once:**

```python
logging.basicConfig(level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s: %(message)s")
```

**3. Config from env:**

```python
DB_URL = os.getenv("DB_URL", "sqlite:///app.db")
```

> **Production code logs leveled events with context and reads all config plus secrets from the environment through one validated object.**
