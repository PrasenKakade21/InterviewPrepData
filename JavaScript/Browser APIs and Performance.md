# Browser APIs and Performance

The browser offers navigation history, the clipboard, observers, frame-synced animation, geolocation, notifications and performance measurement. The chain is **navigate → observe → animate and measure → request permission → load lazily → clean up**. Interview questions focus on when callbacks fire, `requestAnimationFrame` vs `setTimeout`, debounce vs throttle, and listener leaks.

```text
Browser APIs
   ├── History / Clipboard      no reload
   ├── Observers                intersection / size / DOM changes
   ├── rAF / Performance        paint-synced work + measurement
   ├── Geolocation / Notification   permission-gated
   ├── Lazy loading             load only what is near
   └── Debounce / throttle      limit expensive handlers
```

---

# 1. Navigation and Clipboard

### History API

`history.pushState(state, title, url)` adds an entry without reloading; `replaceState` rewrites the current one. The `popstate` event fires when the user goes back or forward.

```javascript
// Browser only
history.pushState({ page: "quiz" }, "", "/quiz");
console.log(location.pathname, history.state.page);

window.addEventListener("popstate", (event) => {
  console.log("back/forward →", event.state?.page ?? "home");
});
```

Output:

```text
/quiz quiz
After Back: back/forward → home or the previous state
(no popstate from pushState itself)
```

`state` must be structured-cloneable, and the URL must be same-origin or the call throws `SecurityError`. A router listens to `popstate`, renders from `event.state`, and calls `pushState` on link clicks. `pushState` does **not** reload and does **not** fire `popstate`.

### Clipboard API

`navigator.clipboard` is promise-based and requires a **secure context**. Writing usually needs a user gesture; reading may show a permission prompt.

```javascript
// Browser only — must be triggered by a click
await navigator.clipboard.writeText("Kakade");
console.log(await navigator.clipboard.readText());
```

Output:

```text
Kakade
(the read may prompt for permission)
```

Failures reject with `NotAllowedError` (permission or no gesture) or `SecurityError` (insecure context). In insecure contexts `navigator.clipboard` is `undefined`.

---

# 2. Observers

### IntersectionObserver

`IntersectionObserver` tells you when an element enters or leaves the **viewport** (or another root). The callback receives `entries` and the observer, and it is the standard tool for lazy images and infinite scroll.

```javascript
// Browser only
const observer = new IntersectionObserver(
  (entries, obs) => {
    for (const entry of entries) {
      if (!entry.isIntersecting) continue;
      entry.target.src = entry.target.dataset.src;
      obs.unobserve(entry.target);
    }
  },
  { root: null, rootMargin: "200px", threshold: 0.25 }
);

document.querySelectorAll("img[data-src]").forEach((img) => observer.observe(img));
```

Output:

```text
Each image loads within 200px of the viewport at 25% visibility.
The callback can fire immediately on observe().
```

`isIntersecting` is the simple check; `intersectionRatio` gives the visible fraction. `rootMargin` grows or shrinks the root, which is how you preload before an element is actually visible. Call `unobserve` or `disconnect` when done.

### ResizeObserver

`ResizeObserver` reports changes to an element's **content box**, which is what you want when layout depends on the element rather than the window.

```javascript
// Browser only
const ro = new ResizeObserver((entries) => {
  const { width, height } = entries[0].contentRect;
  console.log("chart size", Math.round(width), Math.round(height));
});

ro.observe(document.querySelector(".chart"));
```

Output:

```text
chart size 320 180
chart size 480 240   (after resize)
```

Call `ro.disconnect()` on unmount. Writing to the observed element's size **inside** the callback can trigger the “ResizeObserver loop completed with undelivered notifications” error — schedule that work in `requestAnimationFrame`.

### MutationObserver

`MutationObserver` watches DOM changes: children, attributes, character data. Notice: callbacks are batched as a **microtask**, so synchronous changes arrive together.

```javascript
// Browser only
const list = document.querySelector("#list");

const mo = new MutationObserver((mutations) => {
  console.log(mutations.length, mutations[0].type);
});

mo.observe(list, { childList: true, subtree: true });
list.append(document.createElement("li"));
```

Output:

```text
1 childList
```

Use it for third-party widgets or content you do not control, never to implement your own render loop — it easily creates an infinite cycle. Call `disconnect()` when finished.

---

# 3. Timing and Measurement

### requestAnimationFrame

`requestAnimationFrame(callback)` schedules work **before the next paint**. The callback receives a high-resolution timestamp, and the browser pauses or throttles it in background tabs.

```javascript
// Browser only
const box = document.querySelector(".box");
let x = 0;

function move() {
  x = Math.min(x + 4, 200);
  box.style.transform = `translateX(${x}px)`;
  if (x < 200) requestAnimationFrame(move);
}

requestAnimationFrame(move);
```

Output:

```text
The box slides 200px over roughly 0.8 seconds at 60fps.
Each callback runs once per frame, just before the browser paints.
```

`rAF` is synced to paint and pauses in background tabs; `setTimeout(fn, 16)` is not paint-synced. Cancel with `cancelAnimationFrame(id)`, and never read layout and write styles repeatedly in the same frame — batch reads, then writes.

### Performance API

`performance.now()` is a monotonic millisecond clock. `mark`, `measure` and `PerformanceObserver` give you named timings instead of ad-hoc `Date.now()` subtraction.

```javascript
// Browser only
performance.mark("start");
for (let i = 0; i < 100000; i++) {
  // simulated work
}
performance.mark("end");
performance.measure("loop", "start", "end");

const [entry] = performance.getEntriesByName("loop");
console.log(entry.duration > 0);
```

Output:

```text
true (duration varies by device)
```

Use `performance.getEntriesByType("navigation")` for page-load timings and a `PerformanceObserver` for Core Web Vitals such as LCP and CLS. Measurements are per-page and are cleared on navigation.

---

# 4. Device APIs

### Geolocation

`navigator.geolocation.getCurrentPosition(success, error, options)` asks for permission and returns a position asynchronously. It requires a secure context and user consent.

```javascript
// Browser only
navigator.geolocation.getCurrentPosition(
  (pos) => console.log(pos.coords.latitude.toFixed(3)),
  (err) => console.log(err.code, err.message),
  { enableHighAccuracy: true, timeout: 5000, maximumAge: 60000 }
);
```

Output:

```text
18.520  (or 1 User denied Geolocation)
```

Error codes: `1` permission denied, `2` position unavailable, `3` timeout. `watchPosition` returns an id for continuous updates; stop it with `clearWatch(id)`. Always handle denial — the page must still work without location.

### Notifications

The Notifications API shows system notifications. Ask for permission from a user gesture, and check the result before constructing one.

```javascript
// Browser only
const permission = await Notification.requestPermission();

if (permission === "granted") {
  new Notification("Revision time", { body: "Closures, then fetch." });
}

console.log(permission);
```

Output:

```text
granted
A system notification appears: "Revision time — Closures, then fetch."
```

States are `default`, `granted` and `denied`; a denial is permanent, and mobile often requires a service worker.

---

# 5. Loading and Interaction

### Lazy loading

Native lazy loading defers offscreen images and iframes. Always set `width` and `height` (or an aspect ratio) so the reserved space prevents layout shift.

```html
<img
  src="/gallery/kakade.webp"
  alt="Kakade at the conference"
  loading="lazy"
  width="640"
  height="360"
  decoding="async"
>
```

Output:

```text
The image is fetched only near the viewport; the 640×360 box is reserved, so no CLS.
```

`loading="lazy"` works for `<img>` and `<iframe>`. For CSS backgrounds or custom behaviour, use `IntersectionObserver` and swap `data-src` into `src`. Do not lazy-load the hero image — it delays the largest contentful paint.

### Debounce and throttle

Both limit how often a handler runs. **Debounce** waits until the input has been quiet for `delay`; **throttle** runs at most once per interval.

```javascript
// Browser only
function debounce(fn, delay = 300) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

const search = debounce((q) => console.log("search", q), 300);
search("k");
search("ka");
search("kakade");
```

Output:

```text
After 300ms of silence: search kakade
```

```javascript
// Browser only
function throttle(fn, wait = 200) {
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= wait) {
      last = now;
      fn(...args);
    }
  };
}

const onScroll = throttle(() => console.log(scrollY), 200);
window.addEventListener("scroll", onScroll);
```

Output:

```text
At most one log every 200ms while scrolling (leading edge).
```

Both return a wrapper; keep a reference for cleanup, and add `.cancel()` if needed. Use debounce when only the final value matters (search, resize end); use throttle when you need steady updates (scroll, mousemove, drag).

---

# 6. Memory

### Memory leaks

A leak is memory still reachable but no longer needed. Common causes: timers never cleared, listeners never removed, observers never disconnected, detached DOM nodes kept in variables, and caches that grow forever.

```javascript
// Browser only — bad: listener captures big and is never removed
function mount() {
  const big = new Array(1_000_000).fill("x");
  window.addEventListener("resize", () => console.log(big.length));
}
mount();

// Good: remove the listener with an AbortSignal
const controller = new AbortController();
window.addEventListener("resize", () => console.log("resize"), {
  signal: controller.signal,
});
controller.abort();
```

Output:

```text
Bad: every mount adds another listener and keeps its own big array alive.
Good: abort() removes the listener and its closure can be collected.
```

Cleanup checklist: clear intervals/timeouts and `cancelAnimationFrame`; remove listeners or use `{ signal }` with `AbortController`; `disconnect()` the three observers; use `WeakMap`/`WeakSet` for object-keyed caches; drop detached DOM nodes. Find leaks with DevTools → Memory → heap snapshot.

---

# Real-World Uses

- **SPA routing** — `pushState` on navigation, render on `popstate`.
- **Infinite gallery** — `IntersectionObserver` loads the next page near the bottom.
- **Responsive chart** — `ResizeObserver` redraws when the container changes.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **History API** | `pushState`/`replaceState` change URL without reload; `popstate` on back/forward |
| **Clipboard API** | Promise-based, secure context, user gesture; `NotAllowedError` on denial |
| **IntersectionObserver** | Fires on threshold crossings; `isIntersecting`, `threshold`, `rootMargin` |
| **ResizeObserver** | Reports element content-box size; avoid resizing inside the callback |
| **MutationObserver** | Batched DOM-change callbacks in a microtask; `disconnect()` when done |
| **requestAnimationFrame** | Runs before paint, pauses in background, timestamp argument |
| **Geolocation** | `getCurrentPosition`; error codes 1/2/3; secure context + consent |
| **Notifications** | `requestPermission` → `granted`/`denied`/`default`; gesture required |
| **Performance API** | `performance.now`, `mark`, `measure`, `PerformanceObserver` |
| **Lazy loading** | `loading="lazy"` + width/height; IO for custom cases |
| **Debounce and throttle** | Debounce waits for quiet; throttle limits rate |
| **Memory leaks** | Timers, listeners, observers, detached DOM, unbounded caches |

## The Mental Model

Think:

```text
                    PAGE
                      │
   ┌──────────┬───────┼────────┬──────────┐
   ↓          ↓       ↓        ↓          ↓
navigate   observe  timing   device    load
history    IO/Ro/Mo rAF/perf geo/notif lazy/debounce
   └──────────┴───────┼────────┴──────────┘
                      ↓
                   cleanup
        abort · disconnect · clear · drop
```

### The 3 things to remember

**1. `rAF` is for pixels, timers are for delays.**

```javascript
requestAnimationFrame(function step(time) {
  // runs before the next paint
  requestAnimationFrame(step);
});
```

**2. Debounce waits, throttle limits.**

```javascript
const search = debounce(runSearch, 300); // after typing stops
const onScroll = throttle(update, 200);  // every 200ms max
```

**3. Every observer and listener needs an exit.**

```javascript
const controller = new AbortController();
window.addEventListener("scroll", onScroll, { signal: controller.signal });
controller.abort();
```

> **Browser APIs are powerful but stateful: observe only what you need, animate in the paint cycle, ask permission gracefully, and clean up everything you start.**
