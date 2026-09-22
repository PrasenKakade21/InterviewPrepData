# Path and OS

Path/OS clicks as: `join builds` → `parse inspects` → `os describes the box`. They make file code portable — Windows backslashes stop leaking into Kakade's codebase.

The core idea:

```text
Segments → path.join (portable)
    ↓
parse/basename/extname (inspect)
    ↓
os (cores, memory, platform)
```

Never hardcode separators; `join` for paths, `resolve` for absolutes.

---

# 1. path.join vs Resolve, Basename Dirname Extname, and ESM Dirname Pattern

`path.join vs resolve`: `join` concatenates normalized segments (may stay relative); `resolve` builds an absolute path from the right, falling back to cwd. `Basename dirname extname` dissect filenames. `ESM dirname pattern` rebuilds `__dirname` from `import.meta.url`.

```js
import path from "node:path";

console.log(path.join("data", "users", "kakade.json"));
console.log(path.basename("/data/users/kakade.json"));
console.log(path.extname("kakade.json"));
```

Output:

```text
data\users\kakade.json
kakade.json
.json
```

Resolve vs join:

```js
console.log(path.join("/app", "data", "x.json"));
console.log(path.dirname("/app/data/x.json"));
```

Output:

```text
\app\data\x.json
/app/data
```

Think: `join("a", "b")` glues; `resolve("a", "b")` anchors at cwd — use `resolve(__dirname, …)` for file locations that must not depend on where Node was launched.

---

# 2. os Cpus and Memory, Platform Checks, and Cross-Platform Habits

`os cpus and memory` (`cpus().length`, `freemem`/`totalmem`) size pools and guard large jobs. `Platform checks` (`os.platform()`, `path.sep`) branch Windows-vs-rest behavior. Habits: `join` everywhere, `EOL` for newlines, no hardcoded `/tmp`.

```js
import os from "node:os";

console.log(os.platform());
console.log(os.EOL === "\r\n" ? "windows lines" : "unix lines");
```

Output:

```text
win32
windows lines
```

Pool sizing sketch:

```js
// const workers = os.cpus().length; // Kakade sizes cluster to cores.
// const ok = os.freemem() > 512 * 1024 * 1024; // 512MB guard.
```

Rule: gate platform-specific code (`if (os.platform() === "win32")`) and test on both — CI matrices exist for this.

---

# Real-World Uses

Kakade's uploader resolves storage under `__dirname`, validates `.png` via `extname`, sizes the thumbnail pool from `cpus()`, and writes logs with `os.EOL` so files open cleanly everywhere.

```js
import path from "node:path";

const file = path.join("uploads", "kakade.png");
console.log(path.extname(file));
```

Output:

```text
.png
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **path.join vs resolve** | Glue segments vs anchor absolute |
| **basename dirname extname** | Dissect filenames |
| **ESM dirname pattern** | From `import.meta.url` |
| **os cpus and memory** | Size pools, guard jobs |
| **Platform checks** | Branch win32 vs rest |

## The Mental Model

```text
File location for Kakade
  │
  ├── build → join (relative) / resolve (absolute)
  │
  ├── inspect → basename / extname / dirname
  │
  └── adapt → os.platform / cpus / EOL
```

### The 3 things to remember

**1. Portable build:**

```js
path.join("data", "users", "kakade.json");
```

**2. Inspect:**

```js
path.extname("kakade.png");
```

**3. Adapt:**

```js
os.platform();
```

> **Join paths, dissect names, and let os adapt Kakade's code to the machine.**
