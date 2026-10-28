# Event Loop — Revision Notes (Intermediate)

> One-line mental model: one task at a time → drain **all** microtasks → maybe render → next task.

## Core recall

- **Call stack**: LIFO frames; only the top runs; a blocking loop keeps the stack busy so no task or paint can start.
- **Task queue**: timers, I/O completions and UI events (macrotasks). The loop runs **one** per turn and never preempts it.
- **Microtask queue**: Promise reactions, `queueMicrotask`, `MutationObserver`. Drained until empty after every task — including microtasks queued during the drain.
- **Rendering**: happens between tasks, after microtasks. `requestAnimationFrame` runs before paint; long tasks drop frames.
- **setTimeout / setInterval**: delays are minimums; `0` still waits for the current task. Nested timers clamp (~4 ms); intervals drift; always `clearInterval`.
- **queueMicrotask**: explicit microtask, no Promise allocation. Runs before render and before timers; thrown errors are uncaught errors, not unhandled rejections.
- **Ordering**: sync code → microtasks (and nested) → tasks. `await` continuations are microtasks; a timer created inside `.then` runs later.
- **Blocking the loop**: a 200 ms `while` delays a 0 ms timer by 200 ms and freezes paint; chunk work or use a Worker.
- **Web Workers**: separate thread + loop, no DOM, talk via `postMessage`; structured clone copies data, transferables move buffers.
- **Node.js event loop**: timers → pending → poll → check (`setImmediate`) → close; `process.nextTick` before Promise microtasks; inside I/O, `setImmediate` beats `setTimeout(0)`.

## Must-remember code

```javascript
// canonical order
console.log("sync");
setTimeout(() => console.log("task"), 0);
Promise.resolve().then(() => console.log("microtask"));
// sync, microtask, task
```

```javascript
// microtasks queued during a task still run before the next task
Promise.resolve().then(() => {
  console.log("then");
  queueMicrotask(() => console.log("inner micro"));
  setTimeout(() => console.log("timer"), 0);
});
Promise.resolve().then(() => console.log("then 2"));
// then, then 2, inner micro, timer
```

```javascript
// yield so the browser can paint
for (let i = 0; i < rows.length; i += 100) {
  render(rows.slice(i, i + 100));
  await new Promise((r) => setTimeout(r, 0));
}
```

```javascript
// Node: deterministic inside an I/O callback
fs.readFile(file, () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
  process.nextTick(() => console.log("nextTick"));
  Promise.resolve().then(() => console.log("promise"));
});
// nextTick, promise, immediate, timeout
```

## Interview traps

- `setTimeout(fn, 0)` is **not** immediate; it queues a task behind the current script and all microtasks.
- Microtasks always beat timers, even a timer scheduled before the Promise.
- A microtask that queues another microtask keeps the loop in the same turn — infinite recursion starves rendering.
- `setInterval` does not queue missed ticks; slow callbacks cause drift or skipped work.
- Long synchronous code blocks timers, clicks and paint — async syntax does not fix CPU work.
- `setImmediate` and `process.nextTick` do not exist in browsers; `queueMicrotask` is the portable option.
- Top-level `setTimeout(0)` vs `setImmediate` in Node is nondeterministic; inside I/O, `setImmediate` wins.
- `requestAnimationFrame` is not a timer for logic — it is tied to the display refresh and pauses in background tabs.
- Worker messages are copied; mutating the sent object afterwards does not change the worker's copy.

## 3 things to remember

1. **One task, then every microtask.**
   ```javascript
   setTimeout(() => console.log("task"), 0);
   Promise.resolve().then(() => console.log("microtask"));
   // microtask, task
   ```

2. **Sync work blocks the loop.**
   ```javascript
   while (Date.now() - start < 200) {} // timers and paint wait
   ```

3. **Pick the right queue.**
   ```javascript
   queueMicrotask(fn);   // before render, after current work
   setTimeout(fn, 0);    // next task
   new Worker("w.js");   // another thread
   ```
