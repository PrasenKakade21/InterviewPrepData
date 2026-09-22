# CSV — Revision Notes (Intermediate)
> One-line mental model: reader/writer = positional lists; DictReader/DictWriter = named dicts keyed by header — prefer dict style.
## Core recall
- Styles: `csv.reader/writer` → rows as lists (`row[1]`); `csv.DictReader/DictWriter` → rows as dicts (`row["email"]`), header auto-mapped.
- Write: `open(..., "w", newline="", encoding="utf-8")`; `writerows(rows)` / `writerow(row)`; DictWriter needs `fieldnames=[...]` + `writeheader()`.
- Read: `reader` needs manual `next(reader)` to skip header; DictReader maps first row automatically.
- DictReader wins on refactor safety (column reorder safe); reader slightly lighter memory.
- Gotchas: all reads are strings → cast `int()/float()`; custom delimiter via `delimiter="\t"`; CSV→JSON: DictReader + type-clean + `json.dumps`.
## Must-remember code
- ```python
import csv
with open("k.csv", "w", newline="", encoding="utf-8") as f:
    w = csv.writer(f); w.writerows([["id", "name"], [1, "Kakade"]])
with open("k.csv", encoding="utf-8") as f:
    r = csv.reader(f); print(next(r)); print(next(r)[1])
  ```
- ```python
import csv
with open("u.csv", "w", newline="", encoding="utf-8") as f:
    w = csv.DictWriter(f, fieldnames=["id", "name"]); w.writeheader()
    w.writerows([{"id": "1", "name": "Kakade"}])
with open("u.csv", encoding="utf-8") as f:
    for row in csv.DictReader(f): print(row["id"], row["name"])
  ```
- ```python
import csv
with open("s.csv", encoding="utf-8") as f:
    for row in csv.DictReader(f): total = float(row["price"]) + 10
  ```
- ```python
import csv
with open("d.tsv", encoding="utf-8") as f:
    r = csv.DictReader(f, delimiter="\t"); print(next(r))
  ```
## Interview traps
- Missing `newline=""` on Windows → blank rows between every line (double-spaced CSV).
- All values read as strings → `row["price"] + 10` TypeError; cast first.
- `csv.reader` index code breaks on column reorder → use DictReader + header names.
- DictWriter without `writeheader()` → missing header; wrong `fieldnames` → missing/extra columns.
## 3 things to remember
1. Always `newline=""` when writing CSV.
2. Prefer DictReader/DictWriter (name-based, reorder-safe).
3. Cast strings to int/float; specify `delimiter` for TSV/pipe files.
