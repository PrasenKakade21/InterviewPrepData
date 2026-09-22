# Debugging

Debugging clicks as: `read the stack` → `reproduce small` → `watch + inspect`. The error message names the line — trust it before adding logs everywhere.

The core idea:

```text
Crash → first frame of stack (your code)
    ↓
minimal repro → --watch iterate
    ↓
inspector for Kakade's tricky ones
```

Reproduce first, hypothesize second.

---

# 1. Reading Stack Traces, Unhandled Rejections, and Common Crash Causes

`Reading stack traces`: top frame is where it threw, first *your-file* frame is where you fix. `Unhandled rejections` (a promise with no `.catch`) crash modern Node — every async path needs handling. `Common crash causes`: missing env, `undefined` property reads, forgotten `await`, and port conflicts.

```js
function greet(user) {
  return user.name.toUpperCase();
}
try {
  greet({});
} catch (err) {
  console.log(err.constructor.name, "|", err.message.includes("name") || err.message.includes("undefined"));
}
```

Output:

```text
TypeError | true
```

Rejection guard:

```js
process.on("unhandledRejection", (err) => {
  console.error("Crash for Kakade:", err.message);
  process.exit(1);
});
```

Interview line: `Cannot read properties of undefined` means walk one frame up — who passed the empty object?

---

# 2. node --watch and node Inspector Basics

`node --watch` restarts on file saves — the tightest repro loop. `node inspector basics` (`node --inspect`, then `chrome://inspect`) adds breakpoints, stepping, and variable peeks for state Kakade can't `console.log` easily.

```bash
node --watch server.js
```

```text
Restarting for Kakade… (every save reruns).
```

Inspect shape:

```bash
node --inspect server.js
```

```text
Debugger listening on ws://127.0.0.1:9229… open chrome://inspect for Kakade.
```

Loop: `--watch` for fast crashes, inspector for wrong-value bugs — logs for what happened, breakpoints for why.

---

# Real-World Uses

Kakade reproduces a 500 from the stack's first own-frame, guards rejections to crash loudly, iterates under `--watch`, and only then attaches the inspector.

```js
try {
  JSON.parse("{bad json for Kakade");
} catch (err) {
  console.log(err.constructor.name);
}
```

Output:

```text
SyntaxError
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Reading stack traces** | Top throws, own-frame fixes |
| **Unhandled rejections** | Guard or crash loudly |
| **Common crash causes** | Env, undefined, await, ports |
| **node --watch** | Restart-on-save loop |
| **node inspector basics** | Breakpoints via chrome://inspect |

## The Mental Model

```text
Crash for Kakade
  │
  ├── stack → own first frame → repro small
  │
  ├── --watch → iterate fast
  │
  └── stuck? → --inspect → breakpoints
        └── guard rejections always
```

### The 3 things to remember

**1. Read down to yours:**

```js
err.stack.split("\n")[1];
```

**2. Loud failures:**

```js
process.on("unhandledRejection", (err) => process.exit(1));
```

**3. Fast loop:**

```bash
node --watch server.js
```

> **Read the stack to your frame, reproduce small, and guard every rejection.**
