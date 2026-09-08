
## CSV Data Processing

### Clear Explanation

**CSV** (Comma-Separated Values) is a plain-text tabular format where each line represents a data record, and fields in that record are separated by delimiters (commonly commas `,` or tabs `\t`).

Python’s built-in `csv` module provides two main styles for reading and writing CSV files:

1. **Positional / Sequence-based (`csv.reader` & `csv.writer`):** Interacts with CSV rows as **Lists of strings**.
2. **Key-value / Mapping-based (`csv.DictReader` & `csv.DictWriter`):** Interacts with CSV rows as **Dictionaries**, automatically using the header row as keys.

---

### Basic Syntax

```python
import csv

# Positional style
reader = csv.reader(file_handle)
writer = csv.writer(file_handle)

# Dictionary style (Preferred for tabular data with headers)
dict_reader = csv.DictReader(file_handle)
dict_writer = csv.DictWriter(file_handle, fieldnames=["col1", "col2"])

```

---

### A. List-Based Processing: `csv.reader` & `csv.writer`

#### Examples

```python
import csv

# 1. Writing rows using csv.writer
rows = [
    ["ID", "Name", "Role"],
    [101, "Alice", "Admin"],
    [102, "Bob", "Developer"]
]

# Note: newline="" is REQUIRED when opening files for writing with csv module!
with open("users_list.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerows(rows)  # Writes all rows at once

# 2. Reading rows using csv.reader
with open("users_list.csv", "r", encoding="utf-8") as f:
    reader = csv.reader(f)
    header = next(reader)  # Skip / extract header row
    print(f"Header: {header}")
    
    for row in reader:
        # 'row' is a list of strings
        print(f"User: {row[1]} (ID: {row[0]})")

# Output:
# Header: ['ID', 'Name', 'Role']
# User: Alice (ID: 101)
# User: Bob (ID: 102)

```

#### Explain the Code

* `csv.writer(f)` creates a writer wrapper around file object `f`.
* `writer.writerows(rows)` writes all records in a single call. (Use `writer.writerow(row)` for single rows).
* `next(reader)` advances the reader iterator by one step to consume and extract the header row before entering the `for` loop.

---

### B. Dictionary-Based Processing: `csv.DictReader` & `csv.DictWriter`

`DictReader` and `DictWriter` make code significantly cleaner and less error-prone because fields are accessed by **header column name** instead of hardcoded list indices (e.g., `row["Email"]` instead of `row[3]`).

#### Examples

```python
import csv

data = [
    {"user_id": "1", "username": "sara_dev", "status": "active"},
    {"user_id": "2", "username": "mike_admin", "status": "suspended"}
]

fieldnames = ["user_id", "username", "status"]

# 1. Writing with csv.DictWriter
with open("users_dict.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    
    writer.writeheader()  # Explicitly write header row
    writer.writerows(data)

# 2. Reading with csv.DictReader
with open("users_dict.csv", "r", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    
    for row in reader:
        # 'row' is an OrderedDict/dict where keys are header column names
        print(f"ID: {row['user_id']} | Name: {row['username']} | Status: {row['status']}")

# Output:
# ID: 1 | Name: sara_dev | Status: active
# ID: 2 | Name: mike_admin | Status: suspended

```

---

### Important Edge Cases / Gotchas

* **The Blank Line Bug on Windows (`newline=""`):** When writing CSV files in Python, if you do **not** specify `newline=""` in `open()`, Python's default newline translation on Windows converts `\n` to `\r\n`, resulting in extra **blank empty rows** between every line in the output CSV!

```python
# ❌ INCORRECT: Produces double-spaced empty rows on Windows
with open("data.csv", "w") as f:
    writer = csv.writer(f)

# ✅ CORRECT: Always pass newline="" when opening files for csv writer/DictWriter
with open("data.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)

```

* **All Read CSV Values Are Strings:** `csv.reader` and `csv.DictReader` read **ALL values as strings**. Numeric calculations require explicit type casting (`int()`, `float()`).

```python
with open("sales.csv", "r", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        # ❌ TypeError: string + int
        # total = row["price"] + 10 
        
        # ✅ Correct casting
        total = float(row["price"]) + 10

```

* **Custom Delimiters (TSV files or Pipe-delimited data):** CSV reader/writer default to commas `,`. Pass `delimiter` parameter for non-comma files.

```python
# Reading tab-separated values (.tsv)
with open("data.tsv", "r", encoding="utf-8") as f:
    reader = csv.DictReader(f, delimiter="\t")

```

---

### Comparisons: `csv.reader` vs. `csv.DictReader`

| Feature | `csv.reader` | `csv.DictReader` |
| --- | --- | --- |
| **Row Data Type** | `list` of strings | `dict` (Key = Header name) |
| **Element Access** | Index-based (`row[0]`, `row[1]`) | Header-based (`row["id"]`, `row["email"]`) |
| **Header Dependency** | Manual (`next(reader)`) | Automatic mapping from first row |
| **Refactoring Safety** | Low (Reordering CSV columns breaks indices) | High (Column reordering does not break code) |
| **Memory Footprint** | Slightly lower | Slightly higher |

---

## Practical End-to-End Task: CSV Data to JSON API Payload Converter

In real-world backend automation, converting incoming CSV spreadsheet reports into structured JSON API payloads is a daily requirement.

```python
import csv
import json
from pathlib import Path

def convert_csv_to_json_payload(csv_filepath):
    """Reads a CSV file and converts records into a structured JSON string."""
    csv_path = Path(csv_filepath)
    if not csv_path.exists():
        raise FileNotFoundError(f"Source file {csv_filepath} not found.")

    records = []
    with open(csv_path, mode="r", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for row in reader:
            # Clean and type-cast incoming row fields
            processed_record = {
                "id": int(row["id"]),
                "sku": row["sku"].strip().upper(),
                "price": float(row["price"]),
                "in_stock": row["in_stock"].lower() == "true"
            }
            records.append(processed_record)

    payload = {
        "total_records": len(records),
        "products": records
    }
    
    return json.dumps(payload, indent=2)

# --- Simulation setup ---
sample_csv = "inventory.csv"
Path(sample_csv).write_text("id,sku,price,in_stock\n1, widget-a,19.99,true\n2, gadget-b,49.50,false", encoding="utf-8")

# Execute conversion
json_result = convert_csv_to_json_payload(sample_csv)
print(json_result)

# Output:
# {
#   "products": [
#     {
#       "id": 1,
#       "in_stock": true,
#       "price": 19.99,
#       "sku": "WIDGET-A"
#     },
#     {
#       "id": 2,
#       "in_stock": false,
#       "price": 49.5,
#       "sku": "GADGET-B"
#     }
#   ],
#   "total_records": 2
# }

```

---

## Revision Cheat Sheet

### 1. JSON Quick Function Matrix

* **String Operations:**
* `json.loads(str)` ➔ String ➔ Dict/List
* `json.dumps(obj)` ➔ Dict/List ➔ String


* **File Operations:**
* `json.load(f)` ➔ File Handle ➔ Dict/List
* `json.dump(obj, f)` ➔ Dict/List ➔ File Handle



### 2. CSV Quick Function Matrix

* **Positional (Lists):** `csv.reader(f)` / `csv.writer(f)`
* **Named (Dicts - Preferred):** `csv.DictReader(f)` / `csv.DictWriter(f, fieldnames=[...])`
* **Golden Rule for CSV Writing:** Always open file with `open("file.csv", "w", newline="", encoding="utf-8")`.

---

## Self-Assessment Revision Exercises

1. **Debugging Unserializable Types:**
Why will execution crash on the second line of this snippet? How would you fix `custom_data` so that it serializes to valid JSON without error?
```python
import json
custom_data = {"user_id": 99, "permissions": {"admin", "read", "write"}}
json_output = json.dumps(custom_data)

```


2. **Refactoring CSV Indices to Keys:**
Refactor this brittle `csv.reader` code to use `csv.DictReader` so that changing column positions in `metrics.csv` won't break calculation logic:
```python
import csv

total_revenue = 0.0
with open("metrics.csv", "r", encoding="utf-8") as f:
    reader = csv.reader(f)
    next(reader) # skip header
    for row in reader:
        total_revenue += float(row[2]) # Revenue is in 3rd column

```


3. **Missing `newline=""` Impact:**
What bug occurs on Windows OS when exporting query dataset tables into `.csv` files if `newline=""` is omitted during file creation?