# Events — Revision Notes (Beginner)

> One-line mental model: events travel down (capture) and back up (bubble); listen wisely, then choose between stopping travel and cancelling the default.

## Core recall

- **addEventListener**: multiple listeners, run in registration order; the same function reference + type + phase is added once; `onclick` is a single overwritable slot.
- **Event object**: `type`, `target` (origin), `currentTarget` (listener's element), `timeStamp`; mouse/keyboard add coordinates and key data.
- **Bubbling/capturing**: capture goes window → target, then bubble target → window. Default is bubble; `true` or `{ capture: true }` opts into capture.
- **Focus/load do not bubble**: `focus`, `blur`, `mouseenter`, `mouseleave`, `load`; use `focusin`/`focusout` for delegation.
- **stopPropagation**: stops travel to other elements; `stopImmediatePropagation()` also blocks remaining listeners on the same element; neither affects the default action.
- **preventDefault**: cancels the default action (link, submit, checkbox, contextmenu); the event still bubbles. Works only on `cancelable` events and is ignored in passive listeners.
- **Delegation**: one listener on an ancestor + `event.target.closest(selector)` + null guard; survives new children and re-renders.
- **Common events**: `input` on every edit vs `change` on commit; `submit`; `keydown`; `DOMContentLoaded` after parsing vs `load` after resources.
- **Custom events**: `new CustomEvent(name, { detail, bubbles })`; `detail` is any value; `bubbles` defaults `false`; `dispatchEvent` is synchronous.
- **Passive**: `{ passive: true }` promises no `preventDefault`, letting scroll/touch start immediately; a call there is ignored with a warning.
- **once / AbortSignal**: `{ once: true }` auto-removes after one call; `{ signal }` + `controller.abort()` removes a whole group.
- **Keyboard/mouse**: `key` (layout-aware) vs `code` (physical); `button` (0 left, 2 right); `clientX/Y`, `pageX/Y`, `offsetX/Y`.
- **Performance**: delegate, passive scroll, throttle `scroll`/`resize`/`mousemove`, avoid layout reads in handlers, clean up listeners.

## Must-remember code

```javascript
// delegation
list.addEventListener("click", (event) => {
  const item = event.target.closest("li");
  if (!item) return;
  console.log(item.dataset.id);
});
```

```javascript
// prevent default without stopping propagation
form.addEventListener("submit", (event) => {
  event.preventDefault();
  console.log("handled, still bubbles");
});
```

```javascript
// once + abortable listener
const controller = new AbortController();
btn.addEventListener("click", onClick, { once: true });
btn.addEventListener("click", onOther, { signal: controller.signal });
controller.abort(); // removes onOther
```

```javascript
// custom event
el.dispatchEvent(new CustomEvent("cart:add", {
  detail: { name: "Kakade", qty: 2 },
  bubbles: true,
}));
```

## Interview traps

- `preventDefault` does **not** stop bubbling; `stopPropagation` does **not** cancel navigation or submit.
- Defensive `stopPropagation` silently breaks delegated handlers higher in the tree.
- `focus`/`blur` do not bubble; use `focusin`/`focusout` or capture.
- `return false` works only for inline `on*` attributes, not `addEventListener`.
- A passive listener's `preventDefault` is ignored; `defaultPrevented` stays `false`.
- `event.target` can be a nested span; always `closest` before reading `dataset`.
- Removing a listener requires the same function reference (or an `AbortSignal`).
- Custom events do not bubble unless you set `bubbles: true`.

## 3 things to remember

1. **Target vs currentTarget.**
   ```javascript
   list.addEventListener("click", (e) => {
     e.target;        // clicked child
     e.currentTarget; // list
   });
   ```

2. **Travel vs default.**
   ```javascript
   event.stopPropagation(); // no parent handlers
   event.preventDefault();  // no navigation/submit
   ```

3. **Delegate and clean up.**
   ```javascript
   list.addEventListener("click", (e) => e.target.closest("li")?.click());
   const c = new AbortController();
   window.addEventListener("resize", onResize, { signal: c.signal });
   c.abort();
   ```
