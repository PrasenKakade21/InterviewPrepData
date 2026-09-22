# Python Automation - Data

File automation moves files. Data automation reads what's **inside** them — filter rows, fix columns, spit out a clean report.

The core idea:

```text
Read CSV/Excel → DataFrame
    ↓
filter → sort → group → transform
    ↓
write CSV/Excel report
```

Install:

```bash
pip install pandas openpyxl
```

---

# 1. pandas Basics

`pandas` is Excel in Python. Everything revolves around `Series` (one column) and `DataFrame` (table).

```python
import pandas as pd

s = pd.Series(["Kakade", "Asha", "Rohan"])

print(s)
```

Output:

```text
0    Kakade
1      Asha
2     Rohan
dtype: object
```

Think:

```text
list → Series (labeled)
dict of lists → DataFrame (table)
```

---

# 2. DataFrames

A `DataFrame` is rows + named columns.

```python
import pandas as pd

df = pd.DataFrame([
    {"name": "Kakade", "age": 25, "city": "Pune"},
    {"name": "Asha", "age": 30, "city": "Mumbai"},
])

print(df)
print(df.columns.tolist())
print(df.shape)
```

Output:

```text
     name  age    city
0  Kakade   25    Pune
1    Asha   30  Mumbai
['name', 'age', 'city']
(2, 3)
```

`shape` = `(rows, columns)`. `df.head()` previews first 5 rows — use it in every script before transforming.

---

# 3. Reading CSV

```python
import pandas as pd
from pathlib import Path

Path("demo_data").mkdir(exist_ok=True)
Path("demo_data/users.csv").write_text("name,age,city\nKakade,25,Pune\nAsha,30,Mumbai\n")

df = pd.read_csv("demo_data/users.csv")

print(df)
```

Output:

```text
     name  age    city
0  Kakade   25    Pune
1    Asha   30  Mumbai
```

Common options: `pd.read_csv("f.csv", encoding="utf-8", nrows=100)` for a quick peek at big files.

---

# 4. Writing CSV

```python
import pandas as pd

df = pd.DataFrame([
    {"name": "Kakade", "age": 25},
    {"name": "Asha", "age": 30},
])

df.to_csv("demo_data/out.csv", index=False)

print(open("demo_data/out.csv").read())
```

Output:

```text
name,age
Kakade,25
Asha,30
```

Always `index=False` unless you want a stray `Unnamed: 0` column in the report. Interview trap.

---

# 5. Reading Excel

Same API, different reader (`openpyxl` engine).

```python
import pandas as pd

df = pd.DataFrame([{"name": "Kakade", "age": 25}])
df.to_excel("demo_data/users.xlsx", index=False, sheet_name="Users")

print(pd.read_excel("demo_data/users.xlsx", sheet_name="Users"))
```

Output:

```text
     name  age
0  Kakade   25
```

Pick sheet by name or index: `sheet_name=0` or `sheet_name=None` for all sheets as dict.

---

# 6. Writing Excel

```python
import pandas as pd

with pd.ExcelWriter("demo_data/report.xlsx") as writer:
    pd.DataFrame([{"name": "Kakade"}]).to_excel(writer, sheet_name="Users", index=False)
    pd.DataFrame([{"city": "Pune", "count": 1}]).to_excel(writer, sheet_name="Summary", index=False)

print(pd.ExcelFile("demo_data/report.xlsx").sheet_names)
```

Output:

```text
['Users', 'Summary']
```

One writer → many sheets. This is how multi-tab reports are built.

---

# 7. Filtering Data

Boolean mask keeps matching rows.

```python
import pandas as pd

df = pd.DataFrame([
    {"name": "Kakade", "age": 25},
    {"name": "Asha", "age": 30},
    {"name": "Rohan", "age": 22},
])

print(df[df["age"] >= 25])
```

Output:

```text
     name  age
0  Kakade   25
1    Asha   30
```

Multiple conditions need parens + `&`:

```python
print(df[(df["age"] >= 25) & (df["name"] != "Asha")])
```

Output:

```text
     name  age
0  Kakade   25
```

Bad: `df[df.age >= 25 and df.name != "Asha"]` crashes — use `&`, not `and`.

---

# 8. Sorting Data

```python
import pandas as pd

df = pd.DataFrame([
    {"name": "Kakade", "age": 25},
    {"name": "Asha", "age": 30},
])

print(df.sort_values("age", ascending=False))
```

Output:

```text
     name  age
1    Asha   30
0  Kakade   25
```

Multi-column: `df.sort_values(["city", "age"])`. Add `ignore_index=True` to reset row numbers after sorting.

---

# 9. Grouping Data

Split rows into buckets, aggregate each bucket.

```python
import pandas as pd

df = pd.DataFrame([
    {"city": "Pune", "amount": 100},
    {"city": "Pune", "amount": 200},
    {"city": "Mumbai", "amount": 50},
])

print(df.groupby("city")["amount"].agg(["count", "sum", "mean"]))
```

Output:

```text
        count  sum   mean
city
Mumbai      1   50   50.0
Pune        2  300  150.0
```

Think:

```text
groupby(city)
    ↓
Pune bucket, Mumbai bucket
    ↓
count/sum/mean per bucket
```

Same idea as SQL `GROUP BY` — just Python syntax.

---

# 10. Data Transformation

Add, rename, fix columns.

```python
import pandas as pd

df = pd.DataFrame([{"name": " kakade ", "age": 25}])

df["name"] = df["name"].str.strip().str.title()
df["is_adult"] = df["age"] >= 18
df = df.rename(columns={"age": "years"})

print(df)
```

Output:

```text
     name  years  is_adult
0  Kakade     25      True
```

Common cleaners: `.str.strip()`, `.str.lower()`, `.fillna(0)`, `.astype(int)`, `pd.to_datetime(df["date"])`.

---

# 11. openpyxl

`pandas` writes values. `openpyxl` styles them — bold headers, widths, number formats.

```python
import openpyxl

wb = openpyxl.Workbook()
ws = wb.active
ws.title = "Users"
ws.append(["Name", "Age"])
ws.append(["Kakade", 25])

ws["A1"].font = openpyxl.styles.Font(bold=True)
ws.column_dimensions["A"].width = 15
wb.save("demo_data/styled.xlsx")

print(ws["A1"].font.bold)
```

Output:

```text
True
```

Rule: `pandas` for data, `openpyxl` for polish. Write with pandas, then open with openpyxl to format.

---

# 12. Excel Formatting

Typical polish pass:

```python
import openpyxl

wb = openpyxl.load_workbook("demo_data/styled.xlsx")
ws = wb["Users"]

for cell in ws[1]:
    cell.font = openpyxl.styles.Font(bold=True)
    cell.fill = openpyxl.styles.PatternFill("solid", fgColor="DDDDDD")

ws.freeze_panes = "A2"
ws.auto_filter.ref = ws.dimensions
wb.save("demo_data/styled.xlsx")

print(ws.dimensions)
```

Output:

```text
A1:B2
```

Freeze header + autofilter = every manager-readable report.

---

# 13. Generating Reports

Full pipeline in one function:

```python
import pandas as pd
from pathlib import Path

def build_report():
    df = pd.read_csv("demo_data/users.csv")
    summary = df.groupby("city")["age"].agg(count="count", avg_age="mean").reset_index()
    out = Path("demo_data/final_report.xlsx")
    with pd.ExcelWriter(out) as writer:
        df.to_excel(writer, sheet_name="Raw", index=False)
        summary.to_excel(writer, sheet_name="Summary", index=False)
    return str(out)

print(build_report())
```

Output:

```text
demo_data/final_report.xlsx
```

```text
read → transform → group → multi-sheet write
```

Add a date to the filename (`report_2026-09-16.xlsx`) for real jobs.

---

# Real-World Uses

```python
Ops dump → read CSV → filter failures → group by city → styled Excel → email
```

Weekly sales CSV, attendance sheet, API export cleanup — same 30 lines reshaped.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **pandas basics** | `Series` column, `DataFrame` table |
| **DataFrames** | `head()`, `shape`, `columns` |
| **Reading CSV** | `pd.read_csv()` |
| **Writing CSV** | `to_csv(index=False)` |
| **Reading Excel** | `pd.read_excel(sheet_name=)` |
| **Writing Excel** | `ExcelWriter` multi-sheet |
| **Filtering** | `df[mask]` with `&`, not `and` |
| **Sorting** | `sort_values()` |
| **Grouping** | `groupby().agg()` |
| **Transformation** | `str` methods, `fillna`, `astype` |
| **openpyxl** | Styling engine |
| **Excel formatting** | Bold, widths, freeze, filter |
| **Reports** | Read → group → multi-sheet |

## The Mental Model

```text
        READ
  read_csv / read_excel
            │
            ↓
       TRANSFORM
 filter → sort → group → clean
            │
            ↓
         WRITE
 to_csv / ExcelWriter (multi-sheet)
            │
            ↓
         POLISH
  openpyxl bold/freeze/filter
```

### The 3 things to remember

**1. Read and inspect:**

```python
df = pd.read_csv("demo_data/users.csv")
df.head()
```

**2. Filter and group:**

```python
df[df["age"] >= 25].groupby("city")["age"].mean()
```

**3. Report without stray index:**

```python
df.to_csv("out.csv", index=False)
df.to_excel("report.xlsx", sheet_name="Raw", index=False)
```

> **Data automation is read-filter-write: load into a DataFrame, transform with vectorized ops, and export a clean multi-sheet report.**
