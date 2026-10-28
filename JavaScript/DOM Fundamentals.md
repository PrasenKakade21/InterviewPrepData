# DOM Fundamentals

The DOM is the browser's **live object model of the page**: HTML is parsed into a tree of nodes, and JavaScript reads or changes that tree to change what the user sees. The chain is **HTML → DOM tree → select a node → change it → browser re-renders**. Each step matters: how you find a node, change it, and what that costs.

```text
HTML text
    ↓
parser builds a tree of nodes
    ↓
document ─ html ─ head / body
    ↓
select    querySelector / getElementById
    ↓
change    textContent / classList / append
    ↓
reflow & repaint → new pixels
```

All examples here are **browser** snippets and cannot run in Node, which has no `document`.

---

# 1. The Tree and Selection

### DOM tree

Every element, text and comment is a **node** in a tree rooted at `document`. `document.documentElement` is `<html>`; `document.body` is `<body>`.

```html
<body>
  <h1 id="title">Kakade</h1>
  <ul><li>Asha</li><li>Rohan</li></ul>
  <script>
    console.log(document.documentElement.tagName);
    console.log(document.body.children.length);
    console.log(document.querySelector("li").parentElement.tagName);
  </script>
</body>
```

Output:

```text
console: HTML
console: 2
console: UL
```

Think:

```text
document
   └── html
        ├── head
        └── body
             ├── h1#title
             └── ul
                  ├── li "Asha"
                  └── li "Rohan"
```

### Selecting elements

The older methods are direct: `getElementById` returns one element or `null`; `getElementsByClassName` and `getElementsByTagName` return **live** HTMLCollections.

```html
<p id="lead" class="note">Kakade</p>
<p class="note">Asha</p>
<script>
  const byId = document.getElementById("lead");
  const notes = document.getElementsByClassName("note");
  const paragraphs = document.getElementsByTagName("p");
  console.log(byId.textContent, notes.length, paragraphs.length);
  console.log(document.getElementById("missing"));
</script>
```

Output:

```text
console: Kakade 2 2
console: null
```

Notice: `getElementById` returns `null` when nothing matches; duplicate IDs resolve to the first match.

### querySelector and querySelectorAll

`querySelector` accepts **any CSS selector** and returns the first match or `null`. `querySelectorAll` returns a **static** NodeList of every match — it does not change when the DOM changes later.

```html
<ul id="names"><li>Kakade</li></ul>
<script>
  const live = document.getElementsByTagName("li");
  const snapshot = document.querySelectorAll("li");
  const extra = document.createElement("li");
  extra.textContent = "Asha";
  document.querySelector("#names").append(extra);
  console.log(live.length, snapshot.length);
</script>
```

Output:

```text
console: 2 1
```

`getElementById` returns one element or `null`; the class/tag methods return live HTMLCollections; `querySelector` returns the first match or `null`; `querySelectorAll` returns a static NodeList.

`querySelectorAll` also accepts `:first-child`, attribute selectors and comma lists, and its NodeList has `forEach`.

---

# 2. Reading and Writing Content

### textContent vs innerHTML

`textContent` reads or writes **plain text**. `innerHTML` reads or writes an **HTML string**, which the browser parses. Use `textContent` for user data; `innerHTML` with user data is an XSS hole.

```html
<div id="out">Kakade</div>
<script>
  const out = document.getElementById("out");
  out.textContent = "<b>Asha</b>";
  console.log(out.innerHTML);
  out.innerHTML = "<b>Rohan</b>";
  console.log(out.textContent);
</script>
```

Output:

```text
console: &lt;b&gt;Asha&lt;/b&gt;
console: Rohan
```

Bad:

```javascript
out.innerHTML = `<img src=x onerror="stealCookies()">`; // user input → XSS
```

Good:

```javascript
out.textContent = userInput; // rendered as text, never executed
```

| Property | Treats value as | Parses HTML? | Cost |
| --- | --- | --- | --- |
| `textContent` | text | no | cheap, safe |
| `innerHTML` | HTML source | yes | parse + rebuild, XSS risk |

### Creating and appending nodes

`document.createElement(tag)` builds a detached element. `append` inserts nodes or strings at the end; `prepend` at the start; `before`/`after` place siblings around an element.

```html
<ul id="team"></ul>
<script>
  const team = document.getElementById("team");
  const asha = document.createElement("li");
  asha.textContent = "Asha";
  team.append(asha);
  console.log(team.textContent);
</script>
```

Output:

```text
console: Asha
```

Notice: `append` accepts several nodes and strings, returns `undefined`, and moves an existing node rather than copying it. `prepend`/`before`/`after` place nodes; `cloneNode(true)` copies descendants.

### Removing nodes

`element.remove()` detaches a node directly. `parent.removeChild(child)` does the same but needs the parent and returns the removed node. `replaceWith` swaps a node in one step.

```html
<ul id="team"><li>Kakade</li><li class="temp">Asha</li><li>Rohan</li></ul>
<script>
  document.querySelector(".temp").remove();
  const first = document.querySelector("#team li");
  const replacement = document.createElement("li");
  replacement.textContent = "New Kakade";
  first.replaceWith(replacement);
  console.log([...document.querySelectorAll("#team li")].map((li) => li.textContent).join(" | "));
</script>
```

Output:

```text
console: New Kakade | Rohan
```

Removing a node removes its listeners, but it stays in memory while referenced — a memory leak. `innerHTML = ""` destroys all children and their listeners, so avoid it for one.

---

# 3. Attributes, Classes and Traversal

### Classes and styles

`classList` is the safe API for classes: `add`, `remove`, `toggle`, `contains`, `replace`. `style` writes **inline** CSS only; `getComputedStyle` reads the final computed value after all stylesheets.

```html
<div id="card" class="card">Kakade</div>
<script>
  const card = document.getElementById("card");
  card.classList.add("active");
  card.classList.toggle("hidden");
  card.style.color = "tomato";
  console.log(card.className);
  console.log(getComputedStyle(card).color);
</script>
```

Output:

```text
console: card active hidden
console: rgb(255, 99, 71)
```

Notice: `classList.toggle` returns `true` when the class was added. `className` overwrites every class; `classList` edits one.

### Attributes and dataset

Attributes live in the HTML (`getAttribute`, `setAttribute`, `removeAttribute`); properties live on the DOM object (`element.id`, `element.disabled`). They usually mirror each other, but not always — `input.value` is a property; the `value` attribute is the default.

```html
<button id="save" data-user-id="42" data-role="admin" disabled>Save</button>
<script>
  const btn = document.getElementById("save");
  console.log(btn.dataset.userId, btn.dataset.role);
  btn.dataset.role = "editor";
  btn.removeAttribute("disabled");
  console.log(btn.getAttribute("data-role"), btn.disabled);
</script>
```

Output:

```text
console: 42 admin
console: editor false
```

Notice: `data-user-id` maps to `dataset.userId` (kebab → camelCase), and every dataset value is a **string**. For `disabled`, any value — even `"false"` — disables the control; remove the attribute.

### Traversing the DOM

From any element you can walk to its parent, children and siblings. The `Element` variants skip text and comment nodes, which is almost always what you want.

```html
<ul id="team">
  <li>Kakade</li>
  <li class="lead">Asha</li>
  <li>Rohan</li>
</ul>
<script>
  const lead = document.querySelector(".lead");
  console.log(lead.previousElementSibling.textContent);
  console.log(lead.nextElementSibling.textContent);
  console.log(lead.parentElement.id);
  console.log(lead.closest("ul").children.length);
</script>
```

Output:

```text
console: Kakade
console: Rohan
console: team
console: 3
```

Use `parentElement`, `children` (elements only), `childNodes` (all nodes), `firstElementChild`, `nextElementSibling` / `previousElementSibling`, and `closest(selector)` for the nearest matching ancestor.

`closest` is the backbone of event delegation: start from `event.target` and walk up to the component you care about.

---

# 4. Efficient DOM Work

### DocumentFragment

A `DocumentFragment` is a lightweight container **outside** the page. Build all nodes inside it, then append it once — the children move into the DOM in one insertion.

```html
<ul id="team"></ul>
<script>
  const fragment = document.createDocumentFragment();
  ["Kakade", "Asha", "Rohan"].forEach((name) => {
    const li = document.createElement("li");
    li.textContent = name;
    fragment.append(li);
  });
  document.getElementById("team").append(fragment);
  console.log(document.querySelectorAll("#team li").length);
</script>
```

Output:

```text
console: 3
```

Notice: the fragment never appears in the DOM; appending it moves its children and empties it, avoiding N separate insertions.

### Reflow and repaint

**Reflow** (layout) computes geometry — positions and sizes. **Repaint** draws pixels. Reading `offsetWidth`, `offsetHeight`, `scrollTop` or `getBoundingClientRect()` forces pending style and layout work immediately.

```html
<div id="box">Kakade</div>
<script>
  const box = document.getElementById("box");

  // Bad: write → read → write → read forces layout twice
  box.style.width = "100px";
  console.log(box.offsetWidth);
  box.style.width = "200px";
  console.log(box.offsetWidth);

  // Good: batch writes, then read once
  box.style.width = "150px";
  console.log(box.offsetWidth);
</script>
```

Output:

```text
console: 100
console: 200
console: 150
```

Bad: alternating reads and writes in a loop ("layout thrashing") forces many synchronous layouts per frame. Good: read first into variables, then write; change one class instead of many inline styles; animate `transform` and `opacity`, which skip layout.

### Templates

A `<template>` holds inert HTML that is not rendered and not executed. Its `.content` is a DocumentFragment; clone it for each item.

```html
<template id="row">
  <li class="row"><span class="name"></span></li>
</template>
<ul id="team"></ul>
<script>
  const template = document.getElementById("row");
  const team = document.getElementById("team");
  ["Kakade", "Asha"].forEach((name) => {
    const node = template.content.cloneNode(true);
    node.querySelector(".name").textContent = name;
    team.append(node);
  });
  console.log(team.children.length, team.firstElementChild.textContent);
</script>
```

Output:

```text
console: 2 Kakade
```

Notice: `cloneNode(true)` is required — appending `template.content` directly moves its children out and leaves nothing for the next clone.

---

# Real-World Uses

- **Rendering lists** — clone a template or build a fragment, then insert once.
- **Live search** — filter data and update `textContent` instead of re-parsing HTML.
- **Theming** — toggle a class on `<html>` rather than editing inline styles.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **DOM tree** | Nodes rooted at `document`; elements nest; `body` is the visible root |
| **Selecting elements** | `getElementById`, `getElementsByClassName/TagName`; null when missing |
| **querySelector / All** | CSS selectors; first match or `null`; `All` is a static NodeList |
| **textContent vs innerHTML** | Text and safe vs parsed HTML and XSS risk |
| **Creating and appending** | `createElement` + `append` / `prepend`; `cloneNode(true)` to copy |
| **Removing nodes** | `remove()`, `removeChild`, `replaceWith`; drop references |
| **Classes and styles** | `classList` for classes; `style` is inline; `getComputedStyle` reads |
| **Attributes and dataset** | Attributes are strings; `data-x-y` → `dataset.xY` |
| **Traversing the DOM** | `parentElement`, `children`, `*ElementSibling`, `closest` |
| **DocumentFragment** | Build off-DOM, insert children in one go |
| **Reflow and repaint** | Reads like `offsetWidth` force layout; batch writes |
| **Templates** | Inert markup; clone `content` for each instance |

## The Mental Model

```text
                    document
                       │
                    DOM tree
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       select        read         change
   getElementById  textContent   classList
   querySelector   attributes    append/remove
          │            │            │
          └────────────┼────────────┘
                       ↓
                 style + layout
                       ↓
                    repaint
                       ↓
                    pixels
```

### The 3 things to remember

**1. Select, then change one thing.**

```javascript
const title = document.querySelector("#title");
title.textContent = "Kakade"; // safe
```

**2. Text for data, HTML only for trusted markup.**

```javascript
el.textContent = userInput;        // safe
el.innerHTML = "<b>Kakade</b>";    // static, trusted markup only
```

**3. Batch DOM writes.**

```javascript
const fragment = document.createDocumentFragment();
items.forEach((item) => fragment.append(makeRow(item)));
list.append(fragment); // one insertion
```

> **The DOM is a live tree: select the right node, change it safely, and batch writes so layout runs once.**
