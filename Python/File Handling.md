##  `open()`, File Modes, & The `with` Statement

### Clear Explanation

Files store persistent data on disk. In Python, the built-in `open()` function creates a **file object** (a stream handle) that connects your application code to the operating system's file system.

When working with files, managing system resources is critical. Leaving open file handles consumes OS file descriptors and can lead to data corruption (because writes are buffered).

The **`with` statement** creates a **Context Manager** around `open()`. It guarantees that the file stream is automatically closed as soon as code execution leaves the `with` block—even if an exception occurs or a `return` statement is executed inside.

---

### File Modes Summary

| Mode | Name | Description | Pointer Position | File Exists? | File Missing? |
| --- | --- | --- | --- | --- | --- |
| **`"r"`** | Read (default) | Opens file for reading only. | Start of file | Reads file | Raises `FileNotFoundError` |
| **`"w"`** | Write | Opens file for writing. **Truncates (erases)** content! | Start of file | Erases existing data | Creates new file |
| **`"a"`** | Append | Opens file for writing. Appends data to end. | End of file | Preserves existing data | Creates new file |
| **`"x"`** | Exclusive Create | Opens file for writing exclusively. | Start of file | Raises `FileExistsError` | Creates new file |
| **`"r+"`** | Read/Write | Opens for both reading and updating. | Start of file | Preserves existing data | Raises `FileNotFoundError` |

*Append `"b"` (e.g., `"rb"`, `"wb"`) to work with binary files instead of text.*

---

### Basic Syntax

```python
# Modern, safe approach using Context Manager
with open(file_path, mode="r", encoding="utf-8") as file_handle:
    # Perform operations on file_handle
    data = file_handle.read()
# File stream is automatically closed here

```

### Examples

```python
# Standard safe context manager usage
with open("server.log", mode="w", encoding="utf-8") as log_file:
    log_file.write("2026-09-09 INFO: Application initialized successfully.\n")

# Verifying the file closed automatically
print(f"Is file stream closed? {log_file.closed}")
# Output:
# Is file stream closed? True

```

### Explain the Code

* `open("server.log", mode="w", encoding="utf-8")`: Opens or creates `"server.log"` in text write mode. Specifying `encoding="utf-8"` is crucial for cross-platform reliability (Windows uses non-UTF-8 defaults like `cp1252`).
* `as log_file`: Assigns the resulting file object handle to variable `log_file`.
* `log_file.closed`: Evaluates to `True` because leaving the `with` block auto-triggers `.close()`.

### Important Edge Cases / Gotchas

* **Mode `"w"` Truncates Silently:** Opening an existing file with `"w"` instantly erases its entire contents before you write a single byte! Use `"a"` if you want to keep existing data.
* **Manual `.close()` vs `with` statement:**

```python
# ❌ DANGEROUS: If error happens between open and close, file descriptor leaks!
f = open("data.txt", "w")
f.write("test")
# Exception raised here prevents f.close() from running!
f.close()

# ✅ SAFE: Always use with statement
with open("data.txt", "w") as f:
    f.write("test")

```

### Practical Usage (Backend / Production Application Logging)

```python
import datetime

def write_audit_log(event_name, user_id, log_path="audit.log"):
    timestamp = datetime.datetime.now().isoformat()
    log_entry = f"[{timestamp}] EVENT: {event_name} | USER: {user_id}\n"
    
    # Mode "a" appends new lines without modifying past audit trails
    with open(log_path, mode="a", encoding="utf-8") as file:
        file.write(log_entry)

write_audit_log("USER_LOGIN", 8841)

```

### Mental Model

The `with` statement is like a **security guard holding open a vault door**: You step inside (`with`), do your read/write tasks, and when you step out, the guard automatically locks the vault door (`.close()`) behind you so you don't forget.

---

##  Text Files: Reading, Writing, & Iterating Line-by-Line

### Clear Explanation

Text files store human-readable characters mapped to bytes using an encoding schema (e.g., UTF-8).

Python provides multiple methods to read data from text files:

 `.read()`: Reads the **entire file contents** into a single string.
 `.readline()`: Reads a **single line** up to the next newline (`\n`).
 `.readlines()`: Reads all lines into a **list of strings**.
 **File Object Iteration (`for line in file`):** Memory-efficient line-by-line streaming.

For large files, iterating directly over the file object is preferred because it streams lines one at a time into memory rather than loading the entire file into RAM.

---

### Reading Methods Comparison

```python
# Sample setup file creation
with open("sample.txt", "w", encoding="utf-8") as f:
    f.write("Line 1\nLine 2\nLine 3")

```

#### Approach 1: Read whole file (`.read()`)

```python
with open("sample.txt", "r", encoding="utf-8") as f:
    content = f.read()
    print(repr(content))
# Output:
# 'Line 1\nLine 2\nLine 3'

```

#### Approach 2: Read into list of lines (`.readlines()`)

```python
with open("sample.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()
    print(lines)
# Output:
# ['Line 1\n', 'Line 2\n', 'Line 3']

```

#### Approach 3: Streaming Line-by-Line (Memory Efficient - Idiomatic)

```python
with open("sample.txt", "r", encoding="utf-8") as f:
    for line in f:
        # line.strip() removes trailing newline '\n' and extra whitespace
        print(f"Processed: {line.strip()}")

# Output:
# Processed: Line 1
# Processed: Line 2
# Processed: Line 3

```

---

### Writing & Appending Methods

* `.write(string)`: Writes a raw string to the file stream. Returns character length. Does **NOT** auto-append `\n`.
* `.writelines(list_of_strings)`: Writes an iterable of strings to the file in sequence. Does **NOT** add newlines automatically between items.

```python
records = ["user1,active\n", "user2,suspended\n", "user3,active\n"]

with open("users.csv", "w", encoding="utf-8") as f:
    f.writelines(records)

```

### Important Edge Cases / Gotchas

* **Memory Exhaustion with `.read()` / `.readlines()`:** Calling `.read()` or `.readlines()` on a 10 GB server log file attempts to load 10 GB of data into RAM at once, causing MemoryError or triggering OS OOM killers. Always use `for line in file:` for files of unknown size.
* **Trailing Newlines:** Each line returned during iteration ends with `\n`. Calling `print(line)` without `strip()` prints double spaced blank lines because `print()` adds its own default newline.

---

##  Binary Files

### Clear Explanation

Binary files contain raw unformatted bytes (`0x00` - `0xFF`) rather than encoded text characters. Examples include images, PDFs, compiled executables, audio clips, and compressed archives (`.zip`, `.gz`).

To open binary files, append `"b"` to the mode string (`"rb"`, `"wb"`, `"ab"`). Operations on binary files process Python `bytes` objects instead of string (`str`) objects.

### Basic Syntax

```python
with open("image.jpg", mode="rb") as binary_file:
    byte_data = binary_file.read()

```

### Examples

```python
# Creating a dummy binary file containing raw bytes
raw_bytes = bytes([0x47, 0x49, 0x46, 0x38, 0x39, 0x61]) # 'GIF89a' header

with open("header.bin", "wb") as bin_file:
    bin_file.write(raw_bytes)

# Reading and checking byte signature
with open("header.bin", "rb") as bin_file:
    header = bin_file.read(6) # Read first 6 bytes
    print(f"Raw Bytes: {header}")
    print(f"Header Signature: {header.decode('ascii')}")

# Output:
# Raw Bytes: b'GIF89a'
# Header Signature: GIF89a

```

### Explain the Code

* `mode="wb"`: Opens stream for binary output. Passing standard string literals (`str`) to a binary stream raises a `TypeError`. You must pass `bytes` or `bytearray`.
* `header.decode('ascii')`: Converts raw byte values back to readable ASCII text representations.

### Practical Usage (Handling File Uploads in Web APIs)

Streaming file downloads or API asset uploads in chunks so large images or video buffers don't consume web server RAM.

```python
def copy_media_file(source_path, destination_path, chunk_size=4096):
    """Copies media assets safely in 4KB chunks."""
    with open(source_path, "rb") as src, open(destination_path, "wb") as dst:
        while True:
            chunk = src.read(chunk_size)
            if not chunk:
                break  # EOF (End Of File) reached
            dst.write(chunk)

```

---

##  Modern Path Operations: `pathlib`

### Clear Explanation

`pathlib` is a modern object-oriented library built into Python's standard library (Python 3.4+) that replaces legacy string-based path manipulations (`os.path`).

It abstracts operating system differences (Windows backslashes `\` vs. Linux/macOS forward slashes `/`) by treating paths as rich **Path objects** instead of plain text strings.

### Basic Syntax

```python
from pathlib import Path

# Instantiating a Path object
path = Path("directory") / "subdirectory" / "filename.txt"

```

### Key `pathlib` Properties & Operations

```python
from pathlib import Path

# Define path relative to current working directory
config_path = Path("config") / "settings.json"

print(f"Raw Path: {config_path}")
print(f"File Name: {config_path.name}")
print(f"File Extension: {config_path.suffix}")
print(f"File Stem (Name w/o ext): {config_path.stem}")
print(f"Parent Directory: {config_path.parent}")

# Output:
# Raw Path: config/settings.json
# File Name: settings.json
# File Extension: .json
# File Stem: settings
# Parent Directory: config

```

---

### Common `pathlib` Utilities

####  Checking Existence & Path Types

```python
file_path = Path("data/users.json")

print(file_path.exists())       # Returns True if file/dir exists
print(file_path.is_file())      # Returns True if path is a file
print(file_path.is_dir())       # Returns True if path is a directory

```

####  Creating Directories Safely

```python
log_dir = Path("logs/app/2026")

# create parents if missing, don't crash if dir exists
log_dir.mkdir(parents=True, exist_ok=True)

```

####  Searching Files with Glob Patterns

```python
project_dir = Path(".")

# Find all python files in current directory
py_files = list(project_dir.glob("*.py"))

# Recursively find all log files in all subdirectories
all_logs = list(project_dir.rglob("*.log"))

```

####  Shortcut Read/Write Operations (No `open()` required for small files!)

```python
data_file = Path("notes.txt")

# Quick write
data_file.write_text("Fast path writing!", encoding="utf-8")

# Quick read
content = data_file.read_text(encoding="utf-8")
print(content)
# Output:
# Fast path writing!

```

---

### Comparisons: `os.path` vs `pathlib`

| Task | Legacy `os.path` (String-based) | Modern `pathlib` (Object-Oriented) |
| --- | --- | --- |
| **Join Paths** | `os.path.join("dir", "file.txt")` | `Path("dir") / "file.txt"` |
| **Get Absolute Path** | `os.path.abspath("file.txt")` | `Path("file.txt").resolve()` |
| **Check Exists** | `os.path.exists("file.txt")` | `Path("file.txt").exists()` |
| **Get Extension** | `os.path.splitext("file.txt")[1]` | `Path("file.txt").suffix` |
| **Create Folder** | `os.makedirs("a/b", exist_ok=True)` | `Path("a/b").mkdir(parents=True, exist_ok=True)` |

---

## Revision Cheat Sheet

###  File Handling Best Practices

```python
#  ALWAYS use 'with' statement for file handling
#  ALWAYS specify explicit encoding="utf-8" for text files
#  Stream large text files line-by-line: for line in file:
#  Stream large binary files in fixed byte chunks: f.read(4096)

```

###  File Mode Decision Flowchart

* Need to read text? ➔ `"r"`
* Need to overwrite/create file with text? ➔ `"w"`
* Need to append to existing text log? ➔ `"a"`
* Reading binary media/PDF/zip? ➔ `"rb"`
* Writing binary stream? ➔ `"wb"`

###  Quick `pathlib` Reference

```python
from pathlib import Path

p = Path("logs/app.log")
p.parent.mkdir(parents=True, exist_ok=True) # Ensure dir exists

if not p.exists():
    p.write_text("Created log stream", encoding="utf-8")

print(p.read_text(encoding="utf-8"))

```

---

## Self-Assessment Practice Problems

 **The Dangerous Mode Bug:**
What will happen to the contents of `credentials.json` when the following code block executes? How should it be changed if the developer's intention was to append a new API key without disturbing existing content?
```python
with open("credentials.json", mode="w", encoding="utf-8") as f:
    f.write('{"API_KEY": "secret_xyz123"}')

```


 **Memory Efficiency Refactoring:**
Refactor this log parser script so that it can process a 15 GB access log without crashing a server with only 2 GB of RAM:
```python
def count_error_logs(log_file_path):
    with open(log_file_path, "r", encoding="utf-8") as f:
        lines = f.readlines()
        error_count = len([line for line in lines if "ERROR" in line])
    return error_count

```


 **`pathlib` Conversion:**
Rewrite the following legacy `os.path` script using modern Python `pathlib` syntax:
```python
import os

dir_name = "exports"
file_name = "data.csv"
full_path = os.path.join(dir_name, file_name)

if not os.path.exists(dir_name):
    os.makedirs(dir_name)

if os.path.exists(full_path):
    print("Extension:", os.path.splitext(full_path)[1])

```