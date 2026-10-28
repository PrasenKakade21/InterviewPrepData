# Browser APIs and Performance — Revision Notes (Intermediate)

> One-line mental model: observe only what you need, animate in the paint cycle, ask permission gracefully, and clean up everything you start.

## Core recall

- **History API**: `pushState` adds an entry, `replaceState` overwrites; neither reloads nor fires `popstate`. `popstate` fires on Back/Forward. State must be structured-cloneable; URL same-origin.
- **Clipboard API**: `navigator.clipboard` is promise-based, secure-context only, and writes need a user gesture. `NotAllowedError` on denial; `undefined` on HTTP.
- **IntersectionObserver**: fires on visibility **threshold crossings** (plus initial `observe()`), not each scroll frame. `isIntersecting`, `intersectionRatio`, `rootMargin`.
- **ResizeObserver**: reports element content-box size; resizing inside the callback can trigger the “loop completed with undelivered notifications” error.
- **MutationObserver**: batches DOM changes as a microtask; `childList`, `subtree`, `attributes`; `disconnect()` when done.
- **requestAnimationFrame**: runs before paint, gets a timestamp, pauses in background tabs; `setTimeout` is not paint-synced.
- **Geolocation**: `getCurrentPosition(success, error, options)`; errors 1 denied, 2 unavailable, 3 timeout; secure context + consent.
- **Notifications**: `requestPermission()` → `default`/`granted`/`denied`; denial is permanent; mobile usually needs a service worker.
- **Performance API**: `performance.now()` monotonic; `mark`, `measure`, `getEntriesByName`; `PerformanceObserver` for vitals.
- **Lazy loading**: `loading="lazy"` on `<img>`/`<iframe>` plus width/height; IntersectionObserver for custom cases.
- **Debounce and throttle**: debounce waits for quiet, throttle limits rate. Debounce for search, throttle for scroll.
- **Memory leaks**: uncleared timers, unremoved listeners, undisconnected observers, detached DOM, unbounded caches.

## Must-remember code

```javascript
// history + popstate
history.pushState({ page: "quiz" }, "", "/quiz");
window.addEventListener("popstate", (e) => render(e.state ?? { page: "home" }));
```

```javascript
// cleanup with AbortController
const controller = new AbortController();
window.addEventListener("scroll", onScroll, { signal: controller.signal });
controller.abort();
```

```javascript
// debounce, trailing edge
const search = debounce((q) => fetchResults(q), 300);
```

```javascript
// paint-synced animation
requestAnimationFrame(function step(time) {
  // update using time
  requestAnimationFrame(step);
});
```

## Interview traps

- `pushState` does **not** fire `popstate`; only Back/Forward does.
- `pushState` with a cross-origin URL throws `SecurityError`; a deep-link refresh needs server rewrites.
- IntersectionObserver does not fire continuously while an element stays visible.
- `rootMargin` changes the effective viewport; `threshold` is a ratio, not pixels.
- ResizeObserver writing layout in its callback can cause the loop error.
- MutationObserver callbacks are batched microtasks.
- `requestAnimationFrame` pauses in background tabs; `setTimeout` does not guarantee frame timing.
- Clipboard requires HTTPS/localhost and usually a click; `navigator.clipboard` is `undefined` otherwise.
- Notification permission cannot be re-prompted after denial.
- An anonymous listener cannot be removed; keep a reference or use `AbortSignal`.
- Debounce is not throttle: debounce can starve updates during continuous input.

## 3 things to remember

1. **rAF for pixels, timers for delays.**
   ```javascript
   requestAnimationFrame(step); // before paint, timestamped
   ```

2. **Debounce waits, throttle limits.**
   ```javascript
   const search = debounce(run, 300);   // after quiet
   const onScroll = throttle(run, 200); // every 200ms max
   ```

3. **Everything you start needs cleanup.**
   ```javascript
   controller.abort();      // listeners
   observer.disconnect();   // observers
   clearInterval(timerId);  // timers
   ```
