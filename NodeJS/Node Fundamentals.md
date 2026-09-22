# Node Fundamentals

Node clicks as: `V8 runs JS` → `libuv handles I/O` → `your code stays non-blocking`. It is JavaScript outside the browser, built for I/O-heavy servers, CLIs, and scripts.

The core idea:

```text
JavaScript (V8 engine)
    ↓
Node runtime (libuv + core modules)
    ↓
servers, scripts, tools for Kakade
```

One install gives the runtime, REPL, and npm — check with `node -v`.

---

# 1. What is Node.js, V8 and libuv Overview, and Node vs Browser JS

`What is Node.js`: a runtime that executes JavaScript outside the browser using the V8 engine. `V8 and libuv overview`: V8 compiles JS fast; libuv provides the event loop, thread pool, and async I/O. `Node vs browser JS`: Node has files, processes, and servers — no DOM or `window`.

```js
console.log("Hi Kakade from Node");
console.log(process.version);
```

Output:

```text
Hi Kakade from Node
v24.15.0
```

Think:

```text
Browser: JS + DOM + fetch
Node:    JS + fs + http + process
```

Interview trap: Node is not a language or framework — it is a runtime. The language is still JavaScript.

---

# 2. REPL Basics, Globals, and Checking Versions

`REPL basics` (`node` with no args) evaluates JS line-by-line — ideal for quick experiments. `Globals` (`process`, `__dirname`, `console`, `setTimeout`) exist without imports. `Checking versions` (`node -v`, `process.version`) pins debugging context.

```js
const user = "Kakade";
console.log(`Hello ${user}`);
console.log(process.platform);
```

Output:

```text
Hello Kakade
win32
```

REPL session:

```text
$ node
> const a = "Kakade"
> a.toUpperCase()
'KAKADE'
> .exit
```

Rule: `process` is your window into the runtime (args, env, version, exit) — learn it before any framework.

---

# 3. Use Cases and Limits

`Use cases`: APIs, realtime servers, CLIs, build tools, and automation scripts — anything I/O-heavy with lots of concurrent connections. `Limits`: CPU-bound work (video encoding, heavy math) blocks the single thread; huge in-memory state does not scale across cores without clustering.

```js
const started = Date.now();
setTimeout(() => {
  console.log(`Kakade waited ${Date.now() - started}ms without blocking`);
}, 100);
console.log("Kakade moves on immediately");
```

Output:

```text
Kakade moves on immediately
Kakade waited 100ms without blocking
```

| Good fit | Poor fit |
| -------- | -------- |
| **REST APIs for Kakade** | **Video encoding on one thread** |
| **Chat / realtime** | **Heavy ML training** |
| **CLIs and scripts** | **Giant single-process state** |

Motto: Node juggles thousands of waiting connections; it does not sprint through heavy computation.

---

# Real-World Uses

Kakade ships an Express API on Node, prototypes queries in the REPL, checks `process.version` in crash reports, and moves image resizing to a worker thread instead of blocking requests.

```js
console.log(`API for Kakade on ${process.platform} ${process.version}`);
```

Output:

```text
API for Kakade on win32 v24.15.0
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **What is Node.js** | JS runtime outside browser |
| **V8 and libuv overview** | V8 runs JS, libuv does async I/O |
| **REPL basics** | `node` → live JS playground |
| **Globals** | `process`, `__dirname`, no import |
| **Node vs browser JS** | fs/http/process vs DOM/window |
| **Use cases and limits** | I/O great, CPU-bound poor |
| **Checking versions** | `node -v` / `process.version` |

## The Mental Model

```text
V8 (runs JS fast)
  │
  ├── libuv → event loop + async I/O
  │
  └── Node API → fs, http, process (no DOM)
        ├── REPL for Kakade experiments
        └── great at I/O, weak at CPU Sprint
```

### The 3 things to remember

**1. Runtime check:**

```js
console.log(process.version);
```

**2. REPL probe:**

```text
$ node
> "Kakade".toUpperCase()
```

**3. Non-blocking proof:**

```js
setTimeout(() => console.log("Kakade waited"), 100);
```

> **Node runs JavaScript on V8 + libuv: learn the REPL, process, and non-blocking I/O first.**
