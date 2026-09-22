# Event Loop

The loop clicks as: `sync runs first` → `microtasks drain` → `timers/poll/check in phases`. Understanding order predicts output — the favorite Node interview question.

The core idea:

```text
Sync code
    ↓
microtasks (promises, nextTick)
    ↓
macrotasks (timers, I/O, immediates)
```

If Kakade can recite the order of one snippet, the event loop topic is done.

---

# 1. Event Loop Phases, Microtasks vs Macrotasks, and Timers Basics

`Event loop phases` (timers → pending → poll → check → close) process different callback queues per tick. `Microtasks vs macrotasks`: promise callbacks run before the next phase; timer/I/O callbacks wait their phase. `Timers basics` (`setTimeout`, `setInterval`) schedule macrotasks with minimum delays, not guarantees.

```js
console.log("1 sync for Kakade");
setTimeout(() => console.log("4 timeout"), 0);
Promise.resolve().then(() => console.log("3 microtask"));
console.log("2 sync end");
```

Output:

```text
1 sync for Kakade
2 sync end
3 microtask
4 timeout
```

Think:

```text
sync ──→ microtasks ──→ timers ──→ poll ──→ check
  1,2        3              4
```

Trap: `setTimeout(…, 0)` is not immediate — microtasks and earlier phases always win.

---

# 2. process.nextTick vs setImmediate, Blocking the Loop Demo, and Offloading Heavy Work Concept

`process.nextTick vs setImmediate`: `nextTick` runs before any I/O phases (highest priority, starvation risk); `setImmediate` runs in the check phase after poll. `Blocking the loop demo`: a long sync loop freezes all callbacks. `Offloading heavy work concept`: move CPU sprints to worker threads, child processes, or queues.

```js
setImmediate(() => console.log("3 immediate for Kakade"));
process.nextTick(() => console.log("2 tick"));
console.log("1 sync");
```

Output:

```text
1 sync
2 tick
3 immediate for Kakade
```

Blocking demo:

```js
// const end = Date.now() + 50;
// while (Date.now() < end) {} // Kakade's server answers NOTHING here.
// console.log("loop was frozen");
```

| API | Runs | Risk |
| --- | ---- | ---- |
| **`process.nextTick`** | **Before I/O phases** | **Starves loop if recursive** |
| **`setImmediate`** | **Check phase** | **Safe deferral** |
| **Worker threads** | **Separate thread** | **Complexity cost** |

Rule: never block — chunk, defer, or offload. A 50ms freeze at scale drops thousands of requests.

---

# Real-World Uses

Kakade debugs ordering bugs with the sync→micro→macro chant, defers analytics with `setImmediate`, and moves PDF rendering to a worker so the API loop never freezes.

```js
process.nextTick(() => console.log("tick for Kakade"));
console.log("sync first");
```

Output:

```text
sync first
tick for Kakade
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Event loop phases** | Timers → poll → check → close |
| **Microtasks vs macrotasks** | Promises drain before next phase |
| **process.nextTick vs setImmediate** | Pre-I/O vs check phase |
| **Timers basics** | Minimum delay, not exact |
| **Blocking the loop demo** | Sync freeze stops everything |
| **Offloading heavy work** | Workers/queues for CPU sprints |

## The Mental Model

```text
Tick for Kakade
  │
  ├── sync code (now)
  │
  ├── nextTick + promises (before phases)
  │
  └── phases: timers → poll → check (immediate)
        └── blocked? → offload to workers
```

### The 3 things to remember

**1. Order chant:**

```js
console.log("sync");
Promise.resolve().then(() => console.log("micro"));
```

**2. Safe defer:**

```js
setImmediate(() => console.log("later for Kakade"));
```

**3. Never block:**

```js
// while (...) {} // freezes Kakade's server
```

> **Sync, then microtasks, then phases — defer safely and never block the loop.**
