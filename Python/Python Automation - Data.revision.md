# Python Automation - Data — Revision Notes (Intermediate)
> One-line mental model: Read into DataFrame → filter/sort/group → write clean multi-sheet report.
## Core recall
- `Series` = one column, `DataFrame` = table; `shape`, `columns`, `head()` inspect first.
- `pd.read_csv(path, encoding, nrows)`; `to_csv(index=False)` avoids `Unnamed: 0` trap.
- Excel: `read_excel(sheet_name=)`; `ExcelWriter` → multi-sheet `to_excel(writer, sheet_name=...)`.
- Filter: `df[df["age"]>=25]`; multi `df[(a>=25) & (b!="Asha")]` — `&` not `and`, parens required.
- Sort: `sort_values("age", ascending=False)`, multi `["city","age"]`, `ignore_index=True`.
- Group: `df.groupby("city")["amount"].agg(["count","sum","mean"])` = SQL GROUP BY.
- Transform: `str.strip/title/lower`, `fillna(0)`, `astype(int)`, `to_datetime`, `rename(columns=...)`, `df["is_adult"]=df["age"]>=18`.
- openpyxl: pandas writes values, openpyxl polishes — bold, widths, `freeze_panes="A2"`, `auto_filter`.
- Report: `read_csv → groupby → ExcelWriter(Raw+Summary)`; date-stamp filename.
## Must-remember code
```python
import pandas as pd
df = pd.read_csv("demo_data/users.csv"); print(df.head(), df.shape)
df[df["age"] >= 25].to_csv("demo_data/out.csv", index=False)
```
```python
print(df[(df["age"] >= 25) & (df["name"] != "Asha")])
print(df.sort_values("age", ascending=False, ignore_index=True))
```
```python
print(df.groupby("city")["amount"].agg(["count", "sum", "mean"]))
df["name"] = df["name"].str.strip().str.title()
```
```python
with pd.ExcelWriter("demo_data/report.xlsx") as w:
    df.to_excel(w, sheet_name="Users", index=False)
```
```python
import openpyxl
wb = openpyxl.load_workbook("demo_data/styled.xlsx"); ws = wb["Users"]
ws.freeze_panes = "A2"; ws.auto_filter.ref = ws.dimensions
```
## Interview traps
- `to_csv` without `index=False` → stray `Unnamed: 0` column.
- `and` in mask crashes (`truth value ambiguous`) → `&` with parens.
- Skipping `head()` → transform wrong cols on big file; peek with `nrows=100`.
- Styling with pandas only fails → write pandas, polish openpyxl.
- Single-sheet dump for manager → multi-sheet Raw+Summary + freeze/filter.
## 3 things to remember
1. Read + `head()` before transforming.
2. Filter/group with vectorized ops (`&`, `groupby.agg`).
3. Export `index=False`, multi-sheet + polish.
