# JSON and CSV — Revision Notes (Intermediate)
> One-line mental model: JSON for nested APIs (`json.load/dump`), CSV for flat tables (`csv.DictReader/Writer`).

## Core recall
- JSON: `json.loads(s)` str→obj, `json.load(f)` file→obj; `json.dumps(obj, indent=2)` obj→str, `json.dump(obj,f)` obj→file; `ensure_ascii=False` keeps Unicode.
- JSON types map: `null→None, true→True, {...}→dict, [...]→list`; `JSONDecodeError` on bad JSON — wrap in try/except.
- CSV: `open(..., newline="", encoding="utf-8")`; `csv.reader(f)` rows as lists; `csv.DictReader(f)` rows as dicts via header.
- Write: `csv.writer(f).writerow([...])`, `DictWriter(f, fieldnames=[...])` + `writeheader()`; quote handling via `quoting`.
- `pathlib` + `with` for both; validate keys with `.get()`/EAFP; convert types (`int(row["age"])`) after read — all CSV values are str.
- Pandas-free stdlib only; large files → stream row-by-row, not `list(reader)` fully.

## Must-remember code
```python
import json
s = '{"name": "Kakade", "age": 25}'
print(json.loads(s))  # {'name':'Kakade','age':25}
print(json.dumps({"name": "Kakade"}, indent=2, ensure_ascii=False))
```
```python
import csv
with open("kakade.csv", "w", newline="", encoding="utf-8") as f:
    w = csv.DictWriter(f, fieldnames=["name", "age"])
    w.writeheader(); w.writerow({"name": "Kakade", "age": 25})
with open("kakade.csv", newline="", encoding="utf-8") as f:
    print(list(csv.DictReader(f)))
```
```python
import json
try:
    json.loads('{"name": "Kakade",}')
except json.JSONDecodeError as e:
    print(f"bad JSON line {e.lineno}: {e.msg}")
```
```python
import csv
with open("kakade.csv", newline="", encoding="utf-8") as f:
    for row in csv.DictReader(f):
        print(row["name"], int(row["age"]))
```

## Interview traps
- Trailing comma `'{"a":1,}'` invalid JSON → `JSONDecodeError`; JSON needs double quotes, not single.
- CSV `open` without `newline=""` → blank rows on Windows; all reads are `str` — `int()` needed.
- `json.dump` without `indent` unreadable; forgetting `ensure_ascii=False` mangles non-ASCII names.
- `DictWriter` without `writeheader()` loses header; fieldnames mismatch → `ValueError`.
- Loading whole 1 GB CSV with `list()` → OOM — iterate streaming.

## 3 things to remember
1. `loads/load` in, `dumps/dump` out; catch `JSONDecodeError`.
2. CSV needs `newline=""`; `DictReader/Writer` for named columns.
3. CSV values always strings — convert; stream large files.
