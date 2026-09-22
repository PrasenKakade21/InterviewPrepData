# Scheduling and Background Automation

A script run once is a demo. The same script on a schedule is infrastructure.

The core idea:

```text
Script + trigger (daily 9am)
    ↓
scheduler runs it
    ↓
log result, retry on failure
```

Install:

```bash
pip install apscheduler
```

---

# 1. cron

Linux scheduler. One line per job: `minute hour day month weekday command`.

```bash
0 9 * * * /usr/bin/python3 /home/kakade/report.py >> /tmp/report.log 2>&1
```

Reads: "9:00 AM daily, run report, append logs."

```text
* → every
0 9 → 09:00
*/15 → every 15 min
```

Check with `crontab -l`, edit with `crontab -e`. Always use absolute paths — cron has a minimal `PATH`.

---

# 2. Windows Task Scheduler

Windows equivalent: GUI or `schtasks`.

```bash
schtasks /create /tn "KakadeReport" /tr "python C:\jobs\report.py" /sc daily /st 09:00
```

Think:

```text
cron → text line, Linux
Task Scheduler → GUI/task, Windows
```

Both do the same: run command on calendar trigger, survive reboots.

---

# 3. APScheduler

In-process scheduler — no OS setup, runs inside your Python app.

```python
from apscheduler.schedulers.background import BackgroundScheduler
import time

ran = []

scheduler = BackgroundScheduler()
scheduler.add_job(lambda: ran.append("Kakade"), "date")
scheduler.start()
time.sleep(1)
scheduler.shutdown()

print(ran)
```

Output:

```text
['Kakade']
```

Three trigger types: `date` (once), `interval` (every N), `cron` (calendar). Great for "FastAPI app that also sends nightly digests".

---

# 4. Scheduled Scripts

Make scripts schedule-safe: idempotent, absolute paths, no prompts.

```python
from pathlib import Path

def daily_report():
    out = Path("/tmp/kakade_report.txt")
    out.write_text("report ok")
    return str(out)

print(daily_report())
```

Output:

```text
/tmp/kakade_report.txt
```

Checklist:

```text
absolute paths only
--date flag for reruns
writes to new file, then renames
exit 0 on success
```

Bad: `input()` prompts or relative `open("out.txt")` — cron hangs or writes to wrong folder.

---

# 5. Daily Jobs

APScheduler cron-style daily run:

```python
from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.triggers.cron import CronTrigger

def send_report():
    print("sending Kakade report")

scheduler = BackgroundScheduler()
scheduler.add_job(send_report, CronTrigger(hour=9, minute=0))
jobs = scheduler.get_jobs()

print(len(jobs))
```

Output:

```text
1
```

cron equivalent: `0 9 * * * ...`. Daily = reports, backups, digest emails.

---

# 6. Hourly Jobs

```python
from apscheduler.schedulers.background import BackgroundScheduler

scheduler = BackgroundScheduler()
scheduler.add_job(lambda: print("hourly check"), "interval", hours=1)

print(scheduler.get_jobs()[0].trigger)
```

Output:

```text
interval[1:00:00]
```

cron equivalent: `0 * * * * ...`. Hourly = health checks, queue drains, price polls.

---

# 7. Recurring Jobs

General pattern: every N minutes/seconds, with jitter to avoid thundering herd.

```python
from apscheduler.schedulers.background import BackgroundScheduler

scheduler = BackgroundScheduler()
scheduler.add_job(lambda: None, "interval", minutes=15, jitter=30)

print("every 15 min + 30s jitter")
```

Output:

```text
every 15 min + 30s jitter
```

Store last-run state in a file/DB so restarts don't duplicate work.

---

# 8. Background Tasks

Two flavors — don't confuse them in interviews:

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()

@app.post("/report")
def make_report(background: BackgroundTasks):
    background.add_task(lambda: print("Kakade report queued"))
    return {"status": "queued"}
```

```text
FastAPI BackgroundTasks → after response, same process, lost on restart
APScheduler/Celery → true scheduler/queue, survives restarts
```

Use FastAPI tasks for "send email after signup". Use APScheduler/Celery for "every night at 2 AM".

---

# 9. Job Failure Handling

Jobs fail. Plan: catch, log, alert, retry.

```python
import logging

logging.basicConfig(level=logging.INFO)
log = logging.getLogger("jobs")

def flaky():
    try:
        raise ValueError("API down")
    except ValueError as e:
        log.error("job failed for Kakade: %s", e)
        return "notified"

print(flaky())
```

Output:

```text
notified
```

APScheduler options:

```python
scheduler.add_job(
    flaky,
    "interval",
    hours=1,
    max_instances=1,
    misfire_grace_time=300,
    coalesce=True,
)
```

```text
coalesce → skip backlog pileup
misfire_grace_time → still run if 5 min late
max_instances=1 → no overlapping runs
```

Add Sentry/email on exception for production.

---

# Real-World Uses

```python
cron 0 9 * * * → report.py → Excel → email
APScheduler hourly → poll API → append CSV
FastAPI BackgroundTasks → resize upload after response
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **cron** | `m h dom mon dow cmd`, absolute paths |
| **Task Scheduler** | `schtasks`, Windows cron |
| **APScheduler** | In-app, `date/interval/cron` |
| **Scheduled scripts** | Idempotent, no prompts |
| **Daily** | `CronTrigger(hour=9)` |
| **Hourly** | `"interval", hours=1` |
| **Recurring** | `minutes=15`, jitter, state file |
| **Background** | FastAPI after-response vs real queue |
| **Failures** | Log, `coalesce`, `misfire_grace`, alert |

## The Mental Model

```text
        SCRIPT (idempotent CLI)
            │
   ┌────────┼────────┐
   ↓        ↓        ↓
 cron   Task Sched  APScheduler
 daily    daily    interval/cron
   │        │        │
   └────────┼────────┘
            ↓
   log → alert → retry
   (coalesce, no overlap)
```

### The 3 things to remember

**1. Daily at 9:**

```bash
0 9 * * * /usr/bin/python3 /home/kakade/report.py
```

**2. In-app interval:**

```python
scheduler.add_job(job, "interval", hours=1)
```

**3. Safe job flags:**

```python
scheduler.add_job(job, "interval", hours=1, max_instances=1, coalesce=True)
```

> **Scheduling turns scripts into systems: write idempotent CLIs, trigger with cron or APScheduler, and handle failures with logging, coalescing, and alerts.**
