# Scheduling and Background Automation — Revision Notes (Intermediate)
> One-line mental model: Idempotent script + trigger (cron/APScheduler) + log/retry.
## Core recall
- cron: `m h dom mon dow cmd`; `0 9 * * *` daily 9am, `*/15` every 15min; `crontab -l/-e`; absolute paths.
- Windows: `schtasks /create /tn "KakadeReport" /sc daily /st 09:00`; same idea as cron.
- APScheduler in-process: `BackgroundScheduler`; triggers `date` once, `interval` every N, `cron` calendar.
- Schedule-safe scripts: idempotent, absolute paths, `--date` rerun flag, no `input()`, exit 0.
- Daily: `CronTrigger(hour=9, minute=0)`; hourly: `"interval", hours=1`; recurring: `minutes=15, jitter=30`.
- Background flavors: FastAPI `BackgroundTasks` (after response, same process, lost on restart) vs APScheduler/Celery (true schedule, survives).
- Failures: try/except + `log.error`, `coalesce=True`, `misfire_grace_time=300`, `max_instances=1` + alerts.
## Must-remember code
- ```python
  # cron daily 9am
  # 0 9 * * * /usr/bin/python3 /home/kakade/report.py >> /tmp/report.log 2>&1
  ```
- ```python
  from apscheduler.schedulers.background import BackgroundScheduler
  s = BackgroundScheduler()
  s.add_job(lambda: print("Kakade"), "interval", hours=1)
  print(s.get_jobs()[0].trigger)
  ```
- ```python
  from apscheduler.triggers.cron import CronTrigger
  from apscheduler.schedulers.background import BackgroundScheduler
  s = BackgroundScheduler()
  s.add_job(lambda: print("Kakade report"), CronTrigger(hour=9, minute=0))
  ```
- ```python
  from fastapi import BackgroundTasks, FastAPI
  app = FastAPI()
  @app.post("/report")
  def make_report(background: BackgroundTasks):
      background.add_task(lambda: print("Kakade report queued"))
      return {"status": "queued"}
  ```
- ```python
  scheduler.add_job(flaky, "interval", hours=1, max_instances=1, misfire_grace_time=300, coalesce=True)
  ```
## Interview traps
- Relative `open("out.txt")` in cron → wrong cwd → use absolute `Path("/tmp/...")`.
- `input()` prompt in scheduled script → hangs forever → use flags/defaults.
- FastAPI BackgroundTasks for nightly job → lost on restart → use APScheduler/Celery.
- Overlapping long jobs pile up → set `max_instances=1, coalesce=True`.
- No state file → restart duplicates work → persist last-run in file/DB.
## 3 things to remember
1. cron line = calendar + absolute command + log redirect.
2. APScheduler: `date`/`interval`/`cron`; `CronTrigger(hour=9)` for daily.
3. Safe jobs: idempotent + no overlap + log/alert/retry.
