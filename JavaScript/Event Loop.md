# Event Loop

JavaScript runs your code on **one thread**, yet it handles timers, clicks and network responses without freezing. The trick is the event loop: **call stack → task queue → microtask queue → event loop picks the next task**. Once you can trace that chain, every "predict the output order" question becomes mechanical.

```text
call stack        runs one thing at a time
    ↓
task queue        timers, I/O, UI events (macrotasks)
    ↓
microtask queue   promise callbacks, queueMicrotask
    ↓
event loop        one task → drain ALL microtasks → maybe render
    ↓
repeat
```

The rule that answers most interview questions: **after every task, the microtask queue is drained completely before the next task runs.**

---

# 1. The Stack and the Queues

### Call stack

The call stack is a LIFO list of function calls. A call pushes a frame; a `return` pops it. Only the top frame runs. When the stack is empty, the event loop can pick up queued work.

```javascript
function second() {
  console.log("second");
}

function first() {
  second();
  console.log("first done");
}

first();
console.log("script done");
```

Output:

```text
second
first done
script done
```

Think:

```text
first()        → [ first ]
second()       → [ first, second ]
console.log    → [ first, second, log ]
second returns → [ first ]
first returns  → [ ]
script done    → [ log ]
```

`console.log("script done")` runs after `first()` finishes because the stack must empty first. A stack trace is simply this stack printed at the moment of an error.

### Task queue

The **task queue** (also called the macrotask queue) holds callbacks waiting for their turn: expired `setTimeout`/`setInterval`, I/O completions, and user events such as clicks. The loop takes **one task**, runs it to completion, then drains microtasks before taking the next task.

```javascript
setTimeout(() => console.log("task: Kakade"), 0);
setTimeout(() => console.log("task: Rohan"), 0);
console.log("sync: Asha");
```

Output:

```text
sync: Asha
task: Kakade
task: Rohan
```

Notice: `setTimeout(..., 0)` does not mean "now". It means "put this in the task queue after at least 0 ms". The current script is itself a task, so it must finish first.

### Microtask queue

The **microtask queue** holds Promise reactions, `queueMicrotask` callbacks and `MutationObserver` callbacks. It has priority over the task queue, and it is drained **until empty** — including microtasks queued by other microtasks.

```javascript
Promise.resolve().then(() => console.log("microtask"));
console.log("sync");

setTimeout(() => console.log("timeout"), 0);
```

Output:

```text
sync
microtask
timeout
```

Think:

```text
current task ends
    ↓
microtask queue: [ then callback ]
    ↓
run it — queue any new microtasks → run them too
    ↓
queue empty → next task (timeout)
```

| Queue | Holds | Priority |
| --- | --- | --- |
| **Call stack** | currently running frames | highest |
| **Microtask** | Promise `.then`, `queueMicrotask` | after each task, drains fully |
| **Task (macro)** | timers, I/O, UI events | one per loop turn |

---

# 2. Timers and Microtasks

### setTimeout and setInterval

`setTimeout(fn, delay)` schedules **one** task; `setInterval(fn, delay)` keeps scheduling until you `clearInterval`. Delays are minimums, not guarantees, and intervals drift because the callback's own runtime pushes the next tick back.

```javascript
let count = 0;

const id = setInterval(() => {
  count++;
  console.log("tick", count);
  if (count === 3) clearInterval(id);
}, 10);

console.log("scheduled");
```

Output:

```text
scheduled
tick 1
tick 2
tick 3
```

Notice: browsers clamp deeply nested timers to 4 ms after five levels, and background tabs can throttle them to once per second. `setInterval` does not queue up missed ticks; if the callback takes longer than the delay, ticks are skipped or bunched. For animation use `requestAnimationFrame`; for repeated polling prefer a recursive `setTimeout` so the next run starts after the previous one finishes.

### queueMicrotask

`queueMicrotask(fn)` adds a callback directly to the microtask queue. It behaves like `Promise.resolve().then(fn)` for ordering, but it is more explicit and does not create a Promise. An error thrown inside it becomes a global uncaught error, not an unhandled rejection.

```javascript
queueMicrotask(() => console.log("microtask A"));
Promise.resolve().then(() => console.log("microtask B"));
console.log("sync");
```

Output:

```text
sync
microtask A
microtask B
```

Both callbacks run after the script, in the order they were queued. Use `queueMicrotask` when you need to defer work until after the current operation but **before** rendering or the next task — for example, batching notifications after a state change.

---

# 3. Ordering, Rendering and Blocking

### Ordering examples

This is the classic interview snippet. Trace it in layers: all synchronous code first, then microtasks (including nested ones), then tasks.

```javascript
console.log("1 script start");
setTimeout(() => console.log("2 timeout"), 0);
queueMicrotask(() => console.log("3 microtask"));
Promise.resolve().then(() => {
  console.log("4 promise");
  queueMicrotask(() => console.log("5 nested microtask"));
});
console.log("6 script end");
```

Output:

```text
1 script start
6 script end
3 microtask
4 promise
5 nested microtask
2 timeout
```

Think:

```text
sync:       1, 6
microtasks: 3, 4 → 5 (queued while draining, still runs now)
tasks:      2
```

Add `await` and the same rule holds: everything before the first `await` is synchronous, and everything after it is a microtask continuation. A `setTimeout` inside a `.then` goes to the **task** queue and runs after all current microtasks.

### Rendering

Between tasks, the browser may render: recalculate style, layout, paint, composite. Microtasks run before rendering, so a long microtask chain delays the frame. `requestAnimationFrame` callbacks run just before paint, which makes them the right place for visual updates.

```javascript
// Browser console
const box = document.createElement("div");
document.body.append(box);
box.textContent = "Asha";
console.log("DOM updated:", box.textContent);

requestAnimationFrame(() => {
  console.log("frame for:", box.textContent);
});
```

Output:

```text
DOM updated: Asha
frame for: Asha
```

Notice: changing `textContent` updates the DOM immediately, but pixels change at the next rendering opportunity. Reading layout properties such as `offsetHeight` right after a write forces the browser to compute layout early — that is the "layout thrashing" covered in the DOM topic.

### Blocking the loop

Any synchronous work blocks everything: timers, clicks, rendering. There is no preemption. A 200 ms loop delays a 0 ms timer by at least 200 ms.

```javascript
const start = Date.now();

setTimeout(() => {
  console.log("timer ran late:", Date.now() - start >= 200);
}, 0);

while (Date.now() - start < 200) {
  // simulate heavy synchronous work
}

console.log("sync work done");
```

Output:

```text
sync work done
timer ran late: true
```

Bad: parsing a 20 MB JSON file or sorting a huge array on the main thread → frozen UI. Good: move heavy computation to a Web Worker, chunk work with `setTimeout`/`requestIdleCallback`, or reduce the data before rendering.

---

# 4. Offloading Work

### Web Workers

A **Web Worker** runs JavaScript on a separate thread with its own event loop. It cannot touch the DOM, but it can compute, parse and fetch, then send results back with `postMessage`. The main thread stays responsive.

```javascript
// Browser: main page
const source = `
  onmessage = (event) => {
    let total = 0;
    for (let i = 0; i < 1e7; i++) total += i;
    postMessage(total + event.data);
  };
`;

const worker = new Worker(
  URL.createObjectURL(new Blob([source], { type: "text/javascript" }))
);

worker.onmessage = (event) => console.log("worker result:", event.data);
worker.postMessage(1);
console.log("main thread still responsive");
```

Output:

```text
main thread still responsive
worker result: 49999995000001
```

Notice: the main thread logs first because the worker starts asynchronously. Messages are copied with the structured clone algorithm — functions and DOM nodes cannot be sent. `postMessage` is the only channel; the worker cannot read your variables. Terminate with `worker.terminate()` when done.

---

# 5. Node.js

### Node.js event loop

Node uses the same stack + microtask model, but adds phases: **timers → pending callbacks → poll (I/O) → check (`setImmediate`) → close**, with `process.nextTick` and Promise microtasks drained between phases. `nextTick` has priority over Promise callbacks.

```javascript
const fs = require("node:fs");

fs.readFile(__filename, () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
  process.nextTick(() => console.log("nextTick"));
  Promise.resolve().then(() => console.log("promise"));
});
```

Output:

```text
nextTick
promise
immediate
timeout
```

Inside an I/O callback, `setImmediate` always runs before a `setTimeout(..., 0)`, because the loop is in the poll phase and moves to check next. At the **top level** that order is not deterministic — it depends on process startup timing, so never assert it in a test.

| API | Where it runs | Browser? |
| --- | --- | --- |
| `setTimeout` | timers phase / task queue | yes |
| `setImmediate` | check phase | no |
| `process.nextTick` | nextTick queue, before Promises | no |
| `queueMicrotask` | microtask queue | yes |

---

# Real-World Uses

- **Smooth UIs** — keep tasks short so the browser can paint at 60 fps.
- **Debounced search** — reset a timer on each keystroke instead of firing a request per key.
- **Chunked rendering** — yield with `setTimeout` between batches of 100 rows.
- **Heavy computation** — parsing, image processing and crypto in a Worker.
- **Server throughput** — Node handles many connections because I/O waits in the loop, not in threads.
- **Debugging "why is my page frozen"** — look for long tasks blocking the stack.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Call stack** | LIFO frames; one thing runs at a time; empty stack → next task |
| **Task queue** | Timers, I/O, UI events; one macrotask per loop turn |
| **Microtask queue** | Promises, `queueMicrotask`; drains fully after every task |
| **Rendering** | Between tasks; microtasks first; `requestAnimationFrame` before paint |
| **setTimeout and setInterval** | Delays are minimums; intervals drift; clear them |
| **queueMicrotask** | Explicit microtask; runs before render and before timers |
| **Ordering examples** | sync → microtasks (and nested) → tasks |
| **Blocking the loop** | Synchronous work freezes timers, events and paint |
| **Web Workers** | Separate thread, no DOM, `postMessage` + structured clone |
| **Node.js event loop** | Phases + `nextTick` before Promises; `setImmediate` in check |

## The Mental Model

```text
                 EVENT LOOP
                     │
        ┌────────────┴────────────┐
        ↓                         ↓
   CALL STACK                TASK QUEUE
   (run now)          timers · I/O · clicks
        │                         │
        └───────────┬─────────────┘
                    ↓
             MICROTASK QUEUE
        promises · queueMicrotask
                    │
             drain until empty
                    │
                    ↓
              RENDER (browser)
                    │
                    └────────→ next task
```

### The 3 things to remember

**1. One task, then all microtasks.**

```javascript
setTimeout(() => console.log("task"), 0);
Promise.resolve().then(() => console.log("microtask"));
// microtask, then task
```

**2. Sync code blocks everything.**

```javascript
while (Date.now() - start < 200) {} // timers and paint wait
```

**3. Use the right queue.**

```javascript
queueMicrotask(fn);            // before render, after current work
setTimeout(fn, 0);             // next task
new Worker("worker.js");       // another thread
```

> **The event loop runs one task at a time, drains every microtask after it, then renders — so keep tasks short and push heavy work off the main thread.**
