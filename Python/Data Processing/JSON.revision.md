# JSON — Revision Notes (Intermediate)
> One-line mental model: `s` = String in memory, no-`s` = file Stream on disk; json bridges Python dicts and JSON text.
## Core recall
- 4 fns: `loads(str)→obj`, `dumps(obj)→str` (Strings); `load(f)→obj`, `dump(obj,f)` (File objects).
- Type map: dict↔object, list/tuple↔array, str↔string, int/float↔number, True/False↔true/false, None↔null.
- `dumps(obj, indent=2, sort_keys=True)` for pretty logs; `open(..., encoding="utf-8")` for files.
- Nested access: chain `payload["data"]["users"][1]["profile"]["email"]`; use `.get("users", [])` + comprehension for safe filtering.
- FastAPI/Flask: parse body with `loads`, filter lists with comprehensions on `.get()` fields.
## Must-remember code
- ```python
import json
s = '{"user_id": 101, "is_active": true, "quota": null}'
d = json.loads(s); print(d["user_id"], d["is_active"], d["quota"])
print(json.dumps({"status": 200, "debug_mode": False}, indent=2, sort_keys=True))
  ```
- ```python
import json
kakade = {"name": "Kakade", "roles": ["admin"]}
with open("kakade.json", "w", encoding="utf-8") as f: json.dump(kakade, f, indent=4)
with open("kakade.json", encoding="utf-8") as f: print(json.load(f)["name"])
  ```
- ```python
import json
def active_emails(raw):
    data = json.loads(raw)
    return [u["email"] for u in data.get("users", []) if u.get("is_active")]
  ```
- ```python
import json, datetime
print(json.dumps({"ts": datetime.datetime.now().isoformat()}))  # serialize via str
  ```
## Interview traps
- `datetime`/`set`/custom objects not serializable → TypeError; fix with `.isoformat()` or `default=str`.
- JSON needs double quotes: `json.loads("{'k':'v'}")` → JSONDecodeError; valid is `'{"k": "v"}'`.
- Confusing `load` vs `loads`: `load(f)` takes file handle, `loads(s)` takes string (same for dump/dumps).
- `tuple` becomes JSON array/list on round-trip — type not preserved.
## 3 things to remember
1. `s` = String, no-`s` = Stream (file handle).
2. Pretty-print with `indent` + `sort_keys`; always `encoding="utf-8"`.
3. Pre-convert unserializable types; enforce double quotes.
