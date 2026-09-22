# Python Standard Library

The Python Standard Library is one of those topics that looks huge at first, but once you understand **os → pathlib → datetime → collections → itertools/functools → re/subprocess**, it becomes much easier.

The core idea:

```text
Python installation
    ↓
comes with built-in modules
    ↓
no pip install needed
    ↓
import and use directly
```

The standard library lets you **do operating system tasks, handle files, work with dates, process data, and run external commands without installing anything extra**.

You already used parts of it:

```text
json, csv, pathlib, contextlib, functools
```

This topic connects those pieces and adds the interview-frequent modules: `os`, `datetime`, `collections`, `itertools`, `re`, and `subprocess`.

---

# 1. `os`

The `os` module lets you talk to the operating system.

Common uses:

* Get current directory
* List files
* Create directories
* Join paths
* Get environment variables

### Current directory and listing

```python
import os

print(os.getcwd())
print(os.listdir("."))
```

Output:

```text
D:\Kakade.dev Network\Interview prep
['Python', 'React', 'NodeJS']
```

`getcwd()` means **get current working directory**.

---

## Creating directories

```python
import os

os.makedirs("reports/2026", exist_ok=True)
```

Notice:

```python
exist_ok=True
```

not:

```python
os.makedirs("reports/2026")
```

Because without `exist_ok=True`, Python raises an error if the directory already exists.

Think:

```text
makedirs(path)
    │
    ├── creates parent folders too
    │
    └── exist_ok=True avoids crash on rerun
```

---

## Joining paths

Bad:

```python
path = "reports" + "/" + "2026"
```

This breaks on Windows vs Linux.

Correct:

```python
import os

path = os.path.join("reports", "2026", "summary.txt")

print(path)
```

Output on Windows:

```text
reports\2026\summary.txt
```

Output on Linux/macOS:

```text
reports/2026/summary.txt
```

`os.path.join()` handles the separator for you.

Other useful `os.path` helpers:

```python
import os

print(os.path.exists("reports"))
print(os.path.isfile("Python/roadmap.json"))
print(os.path.isdir("Python"))
print(os.path.basename("Python/roadmap.json"))
print(os.path.dirname("Python/roadmap.json"))
```

Output:

```text
True
True
True
roadmap.json
Python
```

---

# 2. `os.environ`

Environment variables store configuration **outside your code**.

Examples:

```text
API_KEY
DATABASE_URL
DEBUG
```

### Reading environment variables

```python
import os

api_key = os.environ.get("API_KEY")

print(api_key)
```

Output if not set:

```text
None
```

Why `.get()`?

Because:

```python
os.environ["API_KEY"]
```

raises `KeyError` if the variable does not exist, while:

```python
os.environ.get("API_KEY")
```

returns `None`.

With a default:

```python
import os

debug = os.environ.get("DEBUG", "False")

print(debug)
```

Output:

```text
False
```

---

## Setting environment variables

```python
import os

os.environ["APP_USER"] = "Kakade"

print(os.environ["APP_USER"])
```

Output:

```text
Kakade
```

Important distinction:

```text
os.environ["X"] = "..."
    ↓
only affects current Python process
    ↓
does NOT permanently change your system
```

This is why real projects use `.env` files plus `python-dotenv` to load them. You will cover that in Python Environment and Packages.

### Interview pattern

```python
import os

DATABASE_URL = os.environ.get("DATABASE_URL")

if not DATABASE_URL:
    raise ValueError("DATABASE_URL is not set")
```

This **fail fast** pattern is very common in backend interviews.

---

# 3. `pathlib`

`pathlib` is the modern way to work with file paths.

Old style:

```python
import os

path = os.path.join("reports", "2026")
```

New style:

```python
from pathlib import Path

path = Path("reports") / "2026"

print(path)
```

Output:

```text
reports\2026
```

The `/` operator joins paths. It looks strange at first, but it is extremely readable once you get used to it.

---

## Creating Path objects

```python
from pathlib import Path

current = Path.cwd()
home = Path.home()
file = Path("Python/roadmap.json")

print(current)
print(home)
print(file)
```

Output:

```text
D:\Kakade.dev Network\Interview prep\InterviewPrepData
C:\Users\Kakade
Python/roadmap.json
```

---

## Checking and creating

```python
from pathlib import Path

data_dir = Path("data")

data_dir.mkdir(parents=True, exist_ok=True)

print(data_dir.exists())
print(data_dir.is_dir())
```

Output:

```text
True
True
```

Think:

```text
Path("data")
    │
    ├── .exists()
    ├── .is_file()
    ├── .is_dir()
    │
    └── .mkdir(parents=True, exist_ok=True)
```

`parents=True` is like `makedirs`. It creates missing parent folders.

---

## Reading and writing files

```python
from pathlib import Path

report = Path("hello.txt")

report.write_text("Hello Kakade")

print(report.read_text())
```

Output:

```text
Hello Kakade
```

No `open()` needed for simple cases.

Other useful properties:

```python
from pathlib import Path

file = Path("Python/roadmap.json")

print(file.name)
print(file.suffix)
print(file.stem)
print(file.parent)
```

Output:

```text
roadmap.json
.json
roadmap
Python
```

### `os` vs `pathlib`

```text
os.path.join()
    ↓
string-based, older style

Path() / "folder"
    ↓
object-based, modern style
```

For interviews and automation, prefer `pathlib` unless the question specifically asks for `os`.

---

# 4. `shutil`

`shutil` means **shell utilities**. It handles high-level file operations.

While `pathlib` is good for single files, `shutil` is better for:

```text
copying files
moving files
deleting folders
copying entire directories
checking disk space
```

### Copying and moving

```python
import shutil

shutil.copy("hello.txt", "hello_backup.txt")
shutil.move("hello_backup.txt", "reports/hello_backup.txt")
```

Think:

```text
shutil.copy(src, dst)
    │
    └── copies file content + permissions

shutil.move(src, dst)
    │
    └── moves or renames file
```

---

## Copying directories

```python
import shutil

shutil.copytree("reports", "reports_backup", dirs_exist_ok=True)
```

Notice:

```python
dirs_exist_ok=True
```

Without it, `copytree()` fails if the destination already exists.

---

## Deleting directories

```python
import shutil

shutil.rmtree("reports_backup")
```

Be careful. Unlike normal file deletion:

```text
shutil.rmtree()
    ↓
deletes folder + everything inside
    ↓
no recycle bin / trash recovery
```

For interviews, remember:

| Task | Use |
| ---- | --- |
| Delete a file | `Path("x.txt").unlink()` |
| Delete empty dir | `Path("dir").rmdir()` |
| Delete full dir | `shutil.rmtree("dir")` |
| Copy file | `shutil.copy()` |
| Copy folder | `shutil.copytree()` |
| Move file | `shutil.move()` |

---

# 5. `glob`

`glob` finds files using patterns.

```python
from pathlib import Path

for file in Path("Python").glob("*.md"):
    print(file)
```

Output:

```text
Python\Comprehensions.md
Python\Decorators.md
Python\Functions.md
```

Common patterns:

```text
*.md
    ↓
all .md files in folder

**/*.md
    ↓
all .md files recursively

report_*.txt
    ↓
report_1.txt, report_final.txt, ...
```

---

## Recursive search

```python
from pathlib import Path

for file in Path("Python").rglob("*.md"):
    print(file)
```

`rglob()` means **recursive glob**. It searches subfolders too.

The older `glob` module works similarly:

```python
import glob

print(glob.glob("Python/*.md"))
```

Output:

```text
['Python\\Comprehensions.md', 'Python\\Decorators.md']
```

Best practice:

```python
from pathlib import Path
```

Use `Path.glob()` and `Path.rglob()` for new code.

---

# 6. `datetime`, `date`, `timedelta`, and `timezone`

Dates and times appear in almost every backend and automation interview.

The main classes:

```text
date
    ↓
year, month, day

datetime
    ↓
date + hour, minute, second

timedelta
    ↓
difference / duration

timezone
    ↓
UTC and offsets
```

---

## `date`

```python
from datetime import date

today = date.today()

print(today)
print(today.year)
print(today.month)
print(today.day)
```

Output:

```text
2026-09-16
2026
9
16
```

Create a specific date:

```python
from datetime import date

dob = date(2000, 5, 17)

print(dob)
```

Output:

```text
2000-05-17
```

---

## `datetime`

```python
from datetime import datetime

now = datetime.now()

print(now)
```

Output:

```text
2026-09-16 10:30:45.123456
```

Format it as a string:

```python
from datetime import datetime

now = datetime.now()

print(now.strftime("%Y-%m-%d"))
print(now.strftime("%Y-%m-%d %H:%M:%S"))
print(now.strftime("%d/%m/%Y"))
```

Output:

```text
2026-09-16
2026-09-16 10:30:45
16/09/2026
```

Common format codes:

| Code | Meaning |
| ---- | ------- |
| `%Y` | 4-digit year |
| `%m` | Month |
| `%d` | Day |
| `%H` | Hour |
| `%M` | Minute |
| `%S` | Second |

---

## Parsing strings into dates

This is the reverse operation:

```python
from datetime import datetime

birthday = datetime.strptime("17-05-2000", "%d-%m-%Y")

print(birthday)
```

Output:

```text
2000-05-17 00:00:00
```

Think:

```text
strftime()
    ↓
datetime → string

strptime()
    ↓
string → datetime
```

A common mistake is mixing them up. Remember **f = format (to string), p = parse (from string)**.

---

## `timedelta`

`timedelta` represents a duration.

```python
from datetime import datetime, timedelta

now = datetime.now()
reminder = now + timedelta(days=7)

print(reminder)
```

Output:

```text
2026-09-23 10:30:45.123456
```

Difference between dates:

```python
from datetime import date

today = date(2026, 9, 16)
event = date(2026, 9, 23)

difference = event - today

print(difference)
print(difference.days)
```

Output:

```text
7 days, 0:00:00
7
```

Practical example:

```python
from datetime import datetime, timedelta

created = datetime(2026, 9, 10, 12, 0, 0)
now = datetime(2026, 9, 16, 12, 0, 0)

age = now - created

print(age.days)
```

Output:

```text
6
```

---

## `timezone`

Naive datetimes have no timezone info. Aware datetimes do.

```python
from datetime import datetime, timezone

now_utc = datetime.now(timezone.utc)

print(now_utc)
```

Output:

```text
2026-09-16 10:30:45.123456+00:00
```

The `+00:00` means UTC.

Best practice for backends and APIs:

```python
from datetime import datetime, timezone

created_at = datetime.now(timezone.utc)
```

Store and compare times in UTC, convert to local time only for display.

---

# 7. `re`

The `re` module handles **regular expressions**. It searches and matches text patterns.

You do not need to memorize every regex symbol for most interviews. Focus on:

```text
searching
matching emails / numbers
extracting parts
splitting and replacing
```

### `re.search()`

```python
import re

text = "Hello Kakade, your order 12345 is ready"

match = re.search(r"\d+", text)

print(match.group())
```

Output:

```text
12345
```

`\d+` means **one or more digits**.

Notice the `r` prefix:

```python
r"\d+"
```

not:

```python
"\d+"
```

Raw strings prevent Python from interpreting backslashes as escape characters. Always use `r"..."` for regex.

---

## `re.findall()`

`search()` finds the first match. `findall()` finds all matches.

```python
import re

text = "Kakade scored 90, 85 and 95"

print(re.findall(r"\d+", text))
```

Output:

```text
['90', '85', '95']
```

---

## `re.match()` vs `re.search()`

This confuses many beginners.

```python
import re

print(re.match(r"\d+", "123 Kakade"))
print(re.match(r"\d+", "Kakade 123"))
```

Output:

```text
<re.Match object; span=(0, 3), match='123'>
None
```

Think:

```text
re.match()
    ↓
only checks start of string

re.search()
    ↓
checks anywhere in string
```

---

## Extracting groups

Parentheses create groups:

```python
import re

text = "Email: kakade@example.com"

match = re.search(r"(\w+)@(\w+)\.com", text)

print(match.group(0))
print(match.group(1))
print(match.group(2))
```

Output:

```text
kakade@example.com
kakade
example
```

`\w+` means **one or more letters, digits, or underscore**.

---

## Replace with `re.sub()`

```python
import re

text = "Phone: 987-654-3210"

cleaned = re.sub(r"\D", "", text)

print(cleaned)
```

Output:

```text
9876543210
```

`\D` means **non-digit**. So this removes everything except digits.

### Useful patterns

| Pattern | Meaning |
| ------- | ------- |
| `\d` | Digit |
| `\D` | Non-digit |
| `\w` | Letter, digit, underscore |
| `\s` | Whitespace |
| `+` | One or more |
| `*` | Zero or more |
| `^` | Start of string |
| `$` | End of string |

---

# 8. `collections`

The `collections` module provides specialized data structures beyond lists and dicts.

The three most interview-important ones:

```text
Counter
defaultdict
deque
```

---

## `Counter`

`Counter` counts how often items appear.

```python
from collections import Counter

marks = ["python", "sql", "python", "api", "python", "sql"]

counts = Counter(marks)

print(counts)
print(counts["python"])
print(counts.most_common(2))
```

Output:

```text
Counter({'python': 3, 'sql': 2, 'api': 1})
3
[('python', 3), ('sql', 2)]
```

Practical example:

```python
from collections import Counter

text = "hello kakade"

print(Counter(text))
```

Output:

```text
Counter({'l': 2, 'h': 1, 'e': 1, 'o': 1, ' ': 1, 'k': 1, 'a': 1, 'd': 1})
```

Think:

```text
list / string
    ↓
Counter()
    ↓
{item: count}
```

---

## `defaultdict`

Normal dicts raise errors for missing keys:

```python
scores = {}

print(scores["Kakade"])
```

Output:

```text
KeyError: 'Kakade'
```

`defaultdict` automatically creates a default value:

```python
from collections import defaultdict

scores = defaultdict(int)

scores["Kakade"] += 1

print(scores["Kakade"])
print(scores["Asha"])
```

Output:

```text
1
0
```

`int` as the factory means missing keys start at `0`.

With lists:

```python
from collections import defaultdict

groups = defaultdict(list)

groups["backend"].append("Kakade")
groups["backend"].append("Asha")

print(groups)
```

Output:

```text
defaultdict(<class 'list'>, {'backend': ['Kakade', 'Asha']})
```

Think:

```text
defaultdict(int)
    ↓
missing key → 0

defaultdict(list)
    ↓
missing key → []
```

This avoids repetitive `if key not in dict` checks.

---

## `deque`

`deque` means **double-ended queue**. It efficiently adds/removes from both ends.

Lists are slow when inserting at the beginning:

```python
items = [2, 3, 4]
items.insert(0, 1)
```

`deque` is designed for this:

```python
from collections import deque

queue = deque(["Kakade", "Asha"])

queue.append("Rohan")
queue.appendleft("Meera")
queue.popleft()

print(queue)
```

Output:

```text
deque(['Kakade', 'Asha', 'Rohan'])
```

Common methods:

```text
append()
appendleft()
pop()
popleft()
```

Practical use:

```python
from collections import deque

recent = deque(maxlen=3)

for user in ["A", "B", "C", "D"]:
    recent.append(user)

print(recent)
```

Output:

```text
deque(['B', 'C', 'D'], maxlen=3)
```

`maxlen` automatically removes old items. Useful for recent history, logs, and sliding windows.

---

# 9. `itertools`

`itertools` provides fast tools for loops and combinations.

You do not need the whole module. Learn these four first:

```text
count()
cycle()
combinations()
product()
```

### `count()` and `cycle()`

```python
import itertools

counter = itertools.count(start=1)

print(next(counter))
print(next(counter))
print(next(counter))
```

Output:

```text
1
2
3
```

`count()` creates an infinite counter. It pairs well with `zip()` or `islice()` to avoid infinite loops.

```python
import itertools

colors = itertools.cycle(["red", "green", "blue"])

print(next(colors))
print(next(colors))
print(next(colors))
print(next(colors))
```

Output:

```text
red
green
blue
red
```

---

## `combinations()`

```python
import itertools

users = ["Kakade", "Asha", "Rohan"]

for pair in itertools.combinations(users, 2):
    print(pair)
```

Output:

```text
('Kakade', 'Asha')
('Kakade', 'Rohan')
('Asha', 'Rohan')
```

Important:

```text
combinations()
    ↓
order does NOT matter
    ↓
('A', 'B') same as ('B', 'A'), included once
```

For order-sensitive pairs, use `permutations()`.

---

## `product()`

```python
import itertools

sizes = ["S", "M"]
colors = ["red", "blue"]

for item in itertools.product(sizes, colors):
    print(item)
```

Output:

```text
('S', 'red')
('S', 'blue')
('M', 'red')
('M', 'blue')
```

Think:

```text
product()
    ↓
nested for loops
    ↓
every combination
```

---

# 10. `functools`

`functools` provides helpers for functions. The most useful ones:

```text
lru_cache
partial
reduce
wraps
```

You already saw `wraps` in Decorators. Here are the others.

---

## `lru_cache`

Caches function results so repeated calls are faster.

```python
from functools import lru_cache
import time


@lru_cache(maxsize=None)
def expensive_calculation(n):
    time.sleep(1)
    return n * n
```

First call:

```python
print(expensive_calculation(10))
```

takes 1 second.

Second call:

```python
print(expensive_calculation(10))
```

returns instantly from cache.

Output both times:

```text
100
```

Think:

```text
function call
    │
    ├── result cached?
    │     ├── yes → return cached value
    │     └── no → compute + store
```

Useful for recursive problems like Fibonacci in interviews.

---

## `partial`

Creates a new function with some arguments pre-filled.

```python
from functools import partial

def greet(greeting, name):
    return f"{greeting} {name}"


say_hello = partial(greet, "Hello")

print(say_hello("Kakade"))
print(say_hello("Asha"))
```

Output:

```text
Hello Kakade
Hello Asha
```

Think:

```text
greet(greeting, name)
    ↓
partial(greet, "Hello")
    ↓
new function(name)
```

---

## `reduce()`

Applies a function cumulatively.

```python
from functools import reduce

numbers = [1, 2, 3, 4]

result = reduce(lambda x, y: x + y, numbers)

print(result)
```

Output:

```text
10
```

Step by step:

```text
((1 + 2) + 3) + 4
    ↓
10
```

---

# 11. `subprocess`

`subprocess` runs external system commands from Python.

Example:

```python
import subprocess

result = subprocess.run(
    ["echo", "Hello Kakade"],
    capture_output=True,
    text=True
)

print(result.stdout)
print(result.returncode)
```

Output:

```text
Hello Kakade
0
```

Important parts:

```python
["echo", "Hello Kakade"]
```

is a list, not a single string. Each argument is a separate item.

```python
capture_output=True
```

captures stdout instead of printing directly.

```python
text=True
```

returns output as string instead of bytes.

---

## Checking return codes

`0` usually means success:

```python
import subprocess

result = subprocess.run(["python", "--version"], capture_output=True, text=True)

print(result.returncode)
```

Output:

```text
0
```

If the command fails, `returncode` is non-zero.

To automatically raise on failure:

```python
import subprocess

subprocess.run(["python", "--version"], check=True)
```

If the command fails, this raises:

```text
subprocess.CalledProcessError
```

### Security warning

Bad:

```python
subprocess.run("rm -rf " + user_input, shell=True)
```

If `user_input` comes from a user, they can inject dangerous commands.

Best practice:

```python
subprocess.run(["command", "arg1", "arg2"])
```

Pass arguments as a list and avoid `shell=True` unless absolutely necessary.

---

# Real-World Uses

You will see these modules constantly in real Python work.

### Backend APIs

```python
from datetime import datetime, timezone
from collections import Counter
```

Timestamps, request counting, rate limiting.

### Automation scripts

```python
from pathlib import Path
import shutil
import glob
```

File organization, backups, batch processing.

### Configuration

```python
import os

token = os.environ.get("API_TOKEN")
```

Secrets and environment-based settings.

### Data processing

```python
from collections import defaultdict, Counter
import itertools
```

Grouping, counting, combinations.

### System tooling

```python
import subprocess
```

Running tests, builds, deployments, CLI tools.

---

# Quick Revision

| Module | Remember |
| ------ | -------- |
| **`os`** | OS interaction, `getcwd()`, `makedirs()`, `os.path.join()` |
| **`os.environ`** | Config outside code, use `.get()` with defaults |
| **`pathlib`** | Modern paths, `Path() / "folder"`, `.read_text()`, `.write_text()` |
| **`shutil`** | Copy/move/delete files and folders, `copytree()`, `rmtree()` |
| **`glob`** | Pattern search, `*.md`, `Path.glob()`, `Path.rglob()` |
| **`date`** | Year/month/day only |
| **`datetime`** | Date + time, `strftime()` vs `strptime()` |
| **`timedelta`** | Durations, date arithmetic |
| **`timezone`** | UTC-aware times, use `timezone.utc` |
| **`re`** | Patterns, `search()`, `findall()`, `sub()` |
| **`Counter`** | Count items, `most_common()` |
| **`defaultdict`** | Auto default values, avoids `KeyError` |
| **`deque`** | Fast ends, `appendleft()`, `popleft()`, `maxlen` |
| **`itertools`** | `count()`, `cycle()`, `combinations()`, `product()` |
| **`functools`** | `lru_cache`, `partial`, `reduce`, `wraps` |
| **`subprocess`** | Run commands, list args, `capture_output=True`, avoid `shell=True` |

## The Mental Model

```text
        STANDARD LIBRARY
                │
                ↓
     No install needed
                │
      ┌─────────┼─────────┐
      ↓         ↓         ↓
   SYSTEM     FILES      TIME
      │         │         │
      ↓         ↓         ↓
     os      pathlib   datetime
 os.environ  shutil     date
 subprocess  glob       timedelta
                           timezone
      ┌─────────┼─────────┐
      ↓         ↓         ↓
    TEXT      DATA     FUNCTIONS
      │         │         │
      ↓         ↓         ↓
     re    collections  itertools
           Counter      functools
           defaultdict
           deque
```

### The 3 things to remember

**1. Files and system:**

```python
from pathlib import Path
import os
import shutil
```

`pathlib` for paths, `os` for system info, `shutil` for copy/move/delete.

**2. Safe configuration:**

```python
import os

value = os.environ.get("KEY", "default")
```

Never hardcode secrets.

**3. Data helpers save code:**

```python
from collections import Counter, defaultdict, deque
from functools import lru_cache
```

Counting, grouping, queues, and caching without writing manual loops.

> **The standard library is Python's built-in toolbox: use `os`/`pathlib`/`shutil` for system and files, `datetime` for time, `collections`/`itertools`/`functools` for data and functions, `re` for patterns, and `subprocess` for external commands.**
