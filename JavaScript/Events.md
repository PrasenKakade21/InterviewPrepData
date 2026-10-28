# Events

An event is a **message from the browser**: a click, key press, form submission or network change. The chain is **something happens → the browser creates an event object → listeners run in capture/bubble order → your handler decides what to change**. Most bugs come from those steps: which element received the event, and whether you stopped propagation or the default action.

```text
user action / browser signal
        ↓
event object created (type, target, detail)
        ↓
capture: window → document → … → target
        ↓
target listeners
        ↓
bubble: target → … → document → window
        ↓
your handler runs → preventDefault? stopPropagation?
```

All examples are **browser** snippets; they need an HTML page, not Node.

---

# 1. Listening to Events

### addEventListener

`addEventListener(type, handler, options)` attaches a listener without overwriting existing ones. Several handlers can listen to the same event and run in registration order.

```html
<button id="save">Save</button>
<script>
  const save = document.getElementById("save");
  save.addEventListener("click", () => console.log("saved"));
  save.addEventListener("click", () => console.log("second handler"));
  save.click();
</script>
```

Output:

```text
console: saved
console: second handler
```

Notice: the same function reference added twice runs once; different arrow functions both run. `onclick` is a single slot.

### Event object

Every handler receives an event object with `type`, `target` (where the event originated) and `currentTarget` (the element whose listener is running).

```html
<button id="delete" data-id="7">Delete Rohan</button>
<script>
  document.getElementById("delete").addEventListener("click", (event) => {
    console.log(event.type, event.target.dataset.id);
    console.log(event.currentTarget.id, typeof event.timeStamp);
  });
  document.getElementById("delete").click();
</script>
```

Output:

```text
console: click 7
console: delete number
```

`target` stays the same; `currentTarget` changes per listener.

---

# 2. Propagation

### Bubbling and capturing

An event travels down from `window` to the target (**capture**), fires on the target, then travels back up (**bubble**). Listeners default to the bubble phase; pass `true` or `{ capture: true }` for capture.

```html
<div id="outer"><button id="inner">Kakade</button></div>
<script>
  const outer = document.getElementById("outer");
  const inner = document.getElementById("inner");

  document.addEventListener("click", () => console.log("document capture"), true);
  outer.addEventListener("click", () => console.log("outer capture"), true);
  inner.addEventListener("click", () => console.log("target"));
  outer.addEventListener("click", () => console.log("outer bubble"));
  document.addEventListener("click", () => console.log("document bubble"));

  inner.click();
</script>
```

Output:

```text
console: document capture
console: outer capture
console: target
console: outer bubble
console: document bubble
```

Think:

```text
capture ↓  window → document → target
bubble  ↑  target → document → window
```

`focus`, `blur`, `mouseenter`, `mouseleave` and `load` do not bubble; `click`, `input`, `change`, `keydown` and `submit` do.

### stopPropagation

`stopPropagation()` stops the event from moving to the next element in the path. It does **not** prevent the default action, and not other listeners on the same element — use `stopImmediatePropagation()` for that.

```html
<div id="outer"><button id="inner">Kakade</button></div>
<script>
  const inner = document.getElementById("inner");
  inner.addEventListener("click", (event) => {
    event.stopPropagation();
    console.log("inner handler");
  });
  document.getElementById("outer").addEventListener("click", () => console.log("outer handler"));
  inner.click();
</script>
```

Output:

```text
console: inner handler
```

Bad: calling this everywhere breaks delegation. Good: stop only when a component owns it, such as a modal backdrop. It does not stop form submission or navigation — that is `preventDefault`.

### preventDefault

`preventDefault()` cancels the browser's **default action** for a cancelable event: following a link, submitting a form or toggling a checkbox. It does not stop bubbling.

```html
<a id="link" href="https://example.com">Kakade</a>
<script>
  document.getElementById("link").addEventListener("click", (event) => {
    event.preventDefault();
    console.log("defaultPrevented:", event.defaultPrevented);
  });
</script>
```

Output:

```text
console: defaultPrevented: true
```

The page does not navigate. `event.cancelable` says whether prevention is allowed; passive listeners ignore it, and `return false` works only in inline `onclick`.

---

# 3. Patterns

### Event delegation

Instead of one listener per child, put **one listener on a common ancestor** and use `event.target.closest(selector)`. This works for future children and uses less memory.

```html
<ul id="team">
  <li data-id="1">Kakade</li>
  <li data-id="2"><strong>Asha</strong></li>
</ul>
<script>
  document.getElementById("team").addEventListener("click", (event) => {
    const item = event.target.closest("li");
    if (!item) return;
    console.log("selected:", item.dataset.id);
  });
  document.querySelector("strong").click();
</script>
```

Output:

```text
console: selected: 2
```

Notice: the click lands on `<strong>`, but `closest("li")` finds the row. The guard handles clicks on padding or the list itself. Use delegation for lists, tables and menus.

### Common events

| Event | Fires when | Bubbles |
| --- | --- | --- |
| `input` | every edit | yes |
| `change` | value committed | yes |
| `submit` | form sent | yes |
| `focusin` / `focusout` | focus moves | yes |
| `DOMContentLoaded` / `load` | parsed / resources loaded | yes / no |

```html
<input id="name" value="">
<script>
  const input = document.getElementById("name");
  input.addEventListener("input", (event) => console.log("input:", event.target.value));
  input.addEventListener("change", (event) => console.log("change:", event.target.value));

  input.value = "Kakade";
  input.dispatchEvent(new Event("input", { bubbles: true }));
  input.dispatchEvent(new Event("change", { bubbles: true }));
</script>
```

Output:

```text
console: input: Kakade
console: change: Kakade
```

Notice: `input` fires on every edit, `change` only when the value is committed — use `input` for live search, `change` for validation on blur.

### Custom events

`new CustomEvent(name, { detail, bubbles })` lets components communicate without tight coupling. `detail` holds any value; `bubbles` defaults to `false`, so the event stays on the dispatching element unless you opt in.

```html
<div id="cart">Cart</div>
<script>
  const cart = document.getElementById("cart");

  cart.addEventListener("cart:add", (event) => {
    console.log("added:", event.detail.name, event.detail.qty);
  });

  cart.dispatchEvent(new CustomEvent("cart:add", {
    detail: { name: "Kakade", qty: 2 },
    bubbles: true,
  }));
</script>
```

Output:

```text
console: added: Kakade 2
```

Notice: event names are arbitrary; `namespace:action` avoids collisions. `dispatchEvent` runs listeners **synchronously**, so the log appears immediately.

---

# 4. Listener Options

### Passive listeners

`{ passive: true }` promises that the handler will **not** call `preventDefault()`. For `touchmove`, `wheel` and `scroll`, this lets the browser scroll immediately instead of waiting for your code.

```html
<div id="page" style="height: 100px; overflow: auto">
  <div style="height: 500px">Kakade</div>
</div>
<script>
  const page = document.getElementById("page");
  page.addEventListener("scroll", () => console.log("scroll handler"), { passive: true });
  page.scrollTop = 20;
</script>
```

Output:

```text
console: scroll handler
```

Bad: a non-passive `touchmove` handler makes the browser wait even if it never calls `preventDefault`. Good: mark it passive. A `preventDefault` call there is ignored and warns; use passive for scroll tracking, not for drag gestures that must block scrolling.

### once and AbortSignal

`{ once: true }` removes the listener after its first call. `{ signal }` ties a listener to an `AbortController`: `controller.abort()` removes every listener registered with that signal — ideal for component cleanup.

```html
<button id="pay">Pay</button>
<script>
  const pay = document.getElementById("pay");
  const controller = new AbortController();

  pay.addEventListener("click", () => console.log("once"), { once: true });
  pay.addEventListener("click", () => console.log("until abort"), { signal: controller.signal });

  pay.click();
  pay.click();
  controller.abort();
  pay.click();
</script>
```

Output:

```text
console: once
console: until abort
console: until abort
```

Notice: the first click removes the `once` listener, the second runs only the abortable one, and the third logs nothing. One controller can clean up many listeners.

### Keyboard and mouse events

Keyboard events expose `key` (produced character, layout-aware), `code` (physical key) and modifiers (`ctrlKey`, `shiftKey`, `altKey`, `metaKey`). Mouse events expose `button` (0 left, 2 right) and `clientX/Y` (viewport), `pageX/Y` (document), `offsetX/Y` (target).

```html
<input id="search" placeholder="Search">
<script>
  const search = document.getElementById("search");

  search.addEventListener("keydown", (event) => {
    console.log(event.key, event.code, event.repeat, event.ctrlKey);
    if (event.key === "Enter") console.log("submit search");
  });

  search.addEventListener("click", (event) => {
    console.log("button:", event.button, "clientX:", event.clientX);
  });

  search.click();
  search.dispatchEvent(new KeyboardEvent("keydown", {
    key: "Enter",
    code: "Enter",
    bubbles: true,
  }));
</script>
```

Output:

```text
console: button: 0 clientX: 0
console: Enter Enter false false
console: submit search
```

Use `event.key` for behaviour and `event.code` for physical position when layout must not matter. Check `event.repeat` to ignore held keys.

---

# 5. Event Performance

### Event performance

Every listener costs memory and every dispatch walks the propagation path. Delegate, go passive, and remove listeners you no longer need.

```html
<ul id="team"></ul>
<script>
  const team = document.getElementById("team");

  team.addEventListener("click", (event) => {
    const item = event.target.closest("li");
    if (item) console.log("open", item.textContent);
  });

  for (let i = 1; i <= 1000; i++) {
    const li = document.createElement("li");
    li.textContent = `user-${i}`;
    team.append(li);
  }

  console.log("items:", team.children.length);
</script>
```

Output:

```text
console: items: 1000
```

Notice: 1000 rows, **one** listener; the alternative attaches and removes 1000 closures. Mark scroll/touch passive, throttle `scroll`/`resize`, and use one `AbortController` for teardown.

---

# Real-World Uses

- **Lists and tables** — one delegated listener survives filtering and new rows.
- **Modals** — `Escape`, backdrop click, focus trapping.
- **SPA cleanup** — `AbortController` on unmount.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **addEventListener** | Many listeners; duplicate function ignored; no `onclick` overwrite |
| **Event object** | `target` = origin; `currentTarget` = listening element |
| **Bubbling and capturing** | Capture down, target, bubble up |
| **stopPropagation** | Stops travel; not the default action |
| **preventDefault** | Cancels the default; event still bubbles |
| **Event delegation** | One ancestor listener + `closest` |
| **Common events** | `input` vs `change`; `submit`; `focusin` |
| **Custom events** | `detail`; `bubbles` false; synchronous dispatch |
| **Passive listeners** | No `preventDefault`; smooth scroll |
| **once and AbortSignal** | Auto-remove once; `abort()` removes the group |
| **Keyboard and mouse** | `key` vs `code`; `button`; coordinates |
| **Event performance** | Delegate, passive, throttle, clean up |

## The Mental Model

```text
                 EVENT
                   │
        ┌──────────┴──────────┐
        ↓                     ↓
    capture               target/bubble
 window → document       target → parent
        │                     │
        └──────────┬──────────┘
                   ↓
             your handler
             │          │
      preventDefault  stopPropagation
             │          │
        cancel action  stop travel
```

### The 3 things to remember

**1. `target` is where it happened; `currentTarget` is where you listen.**

```javascript
list.addEventListener("click", (e) => {
  console.log(e.target, e.currentTarget); // child, list
});
```

**2. Default action and propagation are separate.**

```javascript
event.preventDefault();     // no navigation / submit
event.stopPropagation();    // no parent handlers
```

**3. Delegate and clean up.**

```javascript
const controller = new AbortController();
el.addEventListener("click", handler, { signal: controller.signal });
controller.abort(); // removes it
```

> **Events travel down and back up: listen on the right element, delegate, and choose between stopping travel and cancelling the default action.**
