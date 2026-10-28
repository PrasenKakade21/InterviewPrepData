# DOM Fundamentals — Revision Notes (Beginner)

> One-line mental model: HTML becomes a tree; select a node, change it safely, and batch writes so layout runs once.

## Core recall

- **DOM tree**: live tree of nodes rooted at `document`; `documentElement` is `<html>`, `body` is the visible root. Source HTML is not the DOM — runtime changes only exist in memory.
- **Selecting**: `getElementById` → element or `null`; `getElementsByClassName/TagName` → **live** HTMLCollection; `querySelector` → first match or `null`; `querySelectorAll` → **static** NodeList with `forEach`.
- **textContent vs innerHTML**: text is safe and cheap; `innerHTML` parses HTML, can execute injected handlers (XSS), and recreates children on every assignment. Use `innerHTML` only for trusted markup.
- **Creating/appending**: `createElement`, then `append`/`prepend`/`before`/`after`; `append` takes nodes and strings, `appendChild` takes one node and returns it; `cloneNode(true)` copies descendants.
- **Removing**: `el.remove()` (no parent needed), `parent.removeChild(el)` (returns node), `el.replaceWith(newNode)`. Removed nodes with references stay in memory.
- **Classes/styles**: `classList.add/remove/toggle/contains`; `style` writes inline CSS; `getComputedStyle` reads final values. Prefer classes for anything reusable.
- **Attributes/dataset**: attributes are strings; `data-user-id` ↔ `dataset.userId`; `disabled` is true if present even as `"false"`. Property vs attribute can diverge (`input.value`).
- **Traversal**: `parentElement`, `children`, `childNodes`, `firstElementChild`, `nextElementSibling`, `closest`. Element variants skip whitespace text nodes.
- **DocumentFragment**: off-DOM container; append all rows, then insert once. The fragment itself never enters the DOM.
- **Reflow/repaint**: reflow = geometry, repaint = pixels. Reading `offsetWidth`/`getBoundingClientRect()` forces layout; write → read loops thrash. Batch reads then writes.
- **Templates**: `<template>` is inert; clone `template.content` with `cloneNode(true)` for each instance, fill with `textContent`.

## Must-remember code

```javascript
// select, then write text safely
const title = document.querySelector("#title");
title.textContent = "Kakade";
```

```javascript
// build a list off-DOM, insert once
const fragment = document.createDocumentFragment();
for (const user of users) {
  const li = document.createElement("li");
  li.textContent = user.name;
  fragment.append(li);
}
document.getElementById("team").append(fragment);
```

```javascript
// classes, dataset and traversal
card.classList.toggle("active");
card.dataset.userId = user.id;
const row = event.target.closest("[data-user-id]");
```

```javascript
// template clone
const node = template.content.cloneNode(true);
node.querySelector(".name").textContent = user.name;
list.append(node);
```

## Interview traps

- `querySelectorAll` is a **snapshot**; nodes added later are not in it. `getElementsByTagName` is live.
- `childNodes` counts whitespace text nodes; `children` counts only elements.
- `innerHTML +=` re-parses the whole subtree and destroys listeners — append instead.
- `textContent` vs `innerText`: the first is fast and includes hidden text; the second is layout-aware and slower.
- Appending `template.content` directly moves its children — clone for every row.
- `style.width = 100` needs a unit (`"100px"`) or it is ignored.
- `dataset` values are strings; `data-user-id` becomes `userId`.
- Removing a node does not free it while a JS reference or listener closure holds it.
- Reading layout inside a write loop forces synchronous reflow — measure first, then write.
## 3 things to remember

1. **Text for data, HTML for trusted markup.**
   ```javascript
   el.textContent = userInput;     // safe
   el.innerHTML = trustedMarkup;   // parses HTML
   ```

2. **Build off-DOM, insert once.**
   ```javascript
   const fragment = document.createDocumentFragment();
   rows.forEach((row) => fragment.append(makeRow(row)));
   list.append(fragment);
   ```

3. **Batch layout reads and writes.**
   ```javascript
   const width = box.offsetWidth;  // read
   box.style.width = width + 10 + "px"; // write
   ```
