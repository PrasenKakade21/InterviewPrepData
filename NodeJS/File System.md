# File System

fs clicks as: `promises by default` → `paths with path.join` → `check before you trust`. `node:fs/promises` plus `await` covers 95% of file work.

The core idea:

```text
Build path (path.join)
    ↓
await read/write/append
    ↓
Kakade persists notes, configs, uploads
```

Use `fs/promises` for control flow; streams for big files; `watch` for dev tools, not prod logic.

---

# 1. fs Promises Read and Write, Append and mkdir, and Exists and Stat Checks

`fs promises read and write` (`readFile`/`writeFile` with `utf8`) handle text configs and small data. `Append and mkdir` (`appendFile`, `mkdir({ recursive: true })`) grow logs and ensure folders. `Exists and stat checks` (`stat`, `access`) verify before acting — but expect races.

```js
import { writeFile, readFile } from "node:fs/promises";

await writeFile("kakade.txt", "Hi Kakade");
const text = await readFile("kakade.txt", "utf8");
console.log(text);
```

Output:

```text
Hi Kakade
```

Mkdir + append sketch:

```js
// import { mkdir, appendFile, stat } from "node:fs/promises";
// await mkdir("data", { recursive: true });
// await appendFile("data/log.txt", "Kakade logged in\n");
// const info = await stat("data/log.txt"); // info.size, info.isFile()
```

Trap: check-then-act races (file deleted between `stat` and `read`) — just try/catch the operation instead.

---

# 2. fs.watch Basics, JSON Store Pattern, and Paths with path.join

`fs.watch basics` notifies on file changes — flaky across platforms, fine for dev reload, poor for prod logic. `JSON store pattern` (read → parse → modify → write) persists tiny datasets without a database. `Paths with path.join` builds cross-platform paths; never concatenate with `/`.

```js
import { writeFile, readFile } from "node:fs/promises";

const store = { users: ["Kakade"] };
await writeFile("store.json", JSON.stringify(store, null, 2));
const back = JSON.parse(await readFile("store.json", "utf8"));
console.log(back.users);
```

Output:

```text
[ 'Kakade' ]
```

Watch sketch:

```js
// import { watch } from "node:fs";
// watch("store.json", () => console.log("Changed for Kakade"));
```

Rule: JSON-store for toys and tests; concurrent writers corrupt it — graduate to SQLite/Postgres early.

---

# Real-World Uses

Kakade's CLI scaffolds folders with recursive mkdir, appends to daily logs, keeps test fixtures as JSON files, and reloads config on `watch` during development.

```js
import { appendFile, readFile } from "node:fs/promises";

await appendFile("visits.txt", "Kakade\n");
const lines = (await readFile("visits.txt", "utf8")).trim().split("\n");
console.log(`${lines.length} visit(s)`);
```

Output:

```text
1 visit(s)
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **fs promises read and write** | `readFile`/`writeFile` + utf8 |
| **Append and mkdir** | Logs grow, folders ensured |
| **Exists and stat checks** | Verify, but try/catch wins races |
| **fs.watch basics** | Dev reload, not prod logic |
| **JSON store pattern** | Parse-modify-write for tiny data |
| **Paths with path.join** | Cross-platform, no `/` concat |

## The Mental Model

```text
Path (path.join)
  │
  ├── small? → read/write/append (promises)
  │
  ├── big? → streams (see Streams)
  │
  └── tiny DB? → JSON store → graduate early
```

### The 3 things to remember

**1. Text round-trip:**

```js
await writeFile("kakade.txt", "Hi Kakade");
```

**2. Ensure + grow:**

```js
await mkdir("data", { recursive: true });
```

**3. JSON persist:**

```js
await writeFile("store.json", JSON.stringify(store, null, 2));
```

> **Promises for files, join for paths, JSON-store for toys — try/catch beats check-then-act.**
