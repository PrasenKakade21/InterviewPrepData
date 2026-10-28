# Forms and Validation

A form **collects input, validates it and submits it**. The chain is **form element → named controls → submit event → FormData → constraint validation → your handler**. Interviews focus on what gets submitted, when the browser blocks it, and how to take over accessibly.

```text
<form>
   ↓
named controls (input, select, textarea)
   ↓
user edits → constraint validation state
   ↓
submit event
   ├── invalid → first invalid control focused
   └── valid   → default navigation / request
                 or preventDefault → your JS
   ↓
FormData / JSON → fetch
```

All examples are **browser** snippets; they need an HTML page, not Node.

---

# 1. Form Basics

### Form element

`<form>` groups controls and defines where data goes: `action` (URL), `method` (`get`/`post`) and `enctype` (urlencoded by default, multipart for files). `form.elements` is a live control collection.

```html
<form id="signup" action="/subscribe" method="post">
  <label for="email">Email</label>
  <input id="email" name="email" type="email" required>
  <button>Join</button>
</form>
<script>
  const form = document.getElementById("signup");
  console.log(form.method, form.elements.length);
  console.log(form.elements.email.type);
</script>
```

Output:

```text
console: post 2
console: email
```

Notice: only named controls are submitted, and `elements` includes the button. `method` defaults to **GET** — wrong for passwords.

### submit event

The `submit` event fires on the form, not the button. It fires for a submit button, Enter in a text field, or `form.requestSubmit()`. `event.submitter` identifies the button — useful with "Save" and "Delete".

```html
<form id="signup">
  <input name="name" value="Kakade">
  <button id="send">Send</button>
</form>
<script>
  const form = document.getElementById("signup");
  form.addEventListener("submit", (event) => {
    event.preventDefault();
    console.log("submitter:", event.submitter.id);
    console.log("name:", new FormData(form).get("name"));
  });
  form.requestSubmit();
</script>
```

Output:

```text
console: submitter: send
console: name: Kakade
```

Think:

```text
click / Enter / requestSubmit()
            ↓
   constraint validation
      ├── invalid → invalid events, focus first bad field
      └── valid   → submit event
                       ↓
              preventDefault → FormData → fetch
```

`requestSubmit()` validates then fires `submit`; `form.submit()` bypasses both — avoid it. A `click` listener misses Enter submissions.

### preventDefault

`event.preventDefault()` stops the browser's navigation/reload and hands control to your code. It must run **synchronously**, before any `await`; after a Promise resolves, the browser has already submitted.

```html
<form id="search" action="/search" method="get">
  <input name="q" value="Kakade">
  <button>Go</button>
</form>
<script>
  document.getElementById("search").addEventListener("submit", (event) => {
    event.preventDefault();
    const q = new FormData(event.currentTarget).get("q");
    console.log("would navigate to /search?q=" + q);
  });
</script>
```

Output:

```text
console: would navigate to /search?q=Kakade
```

Bad:

```javascript
form.addEventListener("submit", async (event) => {
  await validate();          // too late — the page already navigated
  event.preventDefault();
});
```

Good: call `preventDefault()` first, then `await`; without JS the form still submits.

---

# 2. Reading and Sending Data

### FormData

`new FormData(form)` collects **successful controls**: named, not disabled, checked if checkbox/radio. Files become `File` objects.

```html
<form id="profile">
  <input name="name" value="Asha">
  <input name="hobby" value="chess">
  <input type="checkbox" name="hobby" value="code" checked>
</form>
<script>
  const data = new FormData(document.getElementById("profile"));
  console.log(data.get("name"));
  console.log(data.getAll("hobby"));
  console.log([...data.keys()]);
</script>
```

Output:

```text
console: Asha
console: [ 'chess', 'code' ]
console: [ 'name', 'hobby', 'hobby' ]
```

Notice: an unchecked checkbox contributes **nothing** (`get` returns `null`); disabled inputs are omitted too. Build FormData manually with `append`, `set` and `delete`.

### Serialization

`Object.fromEntries(new FormData(form))` is the quickest plain object, but values are **strings** and duplicate names collapse. `new URLSearchParams(formData).toString()` gives `a=1&b=2`.

```html
<form id="profile">
  <input name="name" value="Kakade">
  <input type="number" name="age" value="30">
  <input type="checkbox" name="active" checked>
</form>
<script>
  const form = document.getElementById("profile");
  const data = Object.fromEntries(new FormData(form));
  console.log(JSON.stringify(data));
  console.log(new URLSearchParams(new FormData(form)).toString());
</script>
```

Output:

```text
console: {"name":"Kakade","age":"30","active":"on"}
console: name=Kakade&age=30&active=on
```

Notice: `"30"` is a string and unchecked boxes are missing. Convert types, use `getAll` for repeated fields, and for files pass FormData to `fetch` without setting `Content-Type`.

### File inputs

A file input exposes a `FileList` through `input.files`; each `File` has `name`, `size` and `type`.

```html
<input id="avatar" type="file" accept="image/*" multiple>
<script>
  const input = document.getElementById("avatar");
  input.addEventListener("change", () => {
    const files = [...input.files];
    console.log(files.map((file) => file.name + ":" + file.type).join(", "));
    console.log("count:", files.length);
  });
</script>
```

Output (after choosing two images):

```text
console: kakade.png:image/png, asha.jpg:image/jpeg
console: count: 2
```

Notice: `accept="image/*"` is a picker hint, not validation — check `file.type` and `file.size` on the server. Preview with `URL.createObjectURL(file)` and revoke it. FormData with files is multipart, so do not set `Content-Type`.

---

# 3. Validation

### Input types

The input `type` gives free semantics, keyboards and validation: `email`, `url`, `number`, `range`, `date`, `tel`, `password`, `file`, `checkbox`, `radio`. Attributes add `required`, `min`, `max`, `pattern`, `step`.

```html
<form id="f">
  <input id="email" type="email" value="kakade@" required>
  <input id="age" type="number" min="18" max="60" value="15">
  <button>Send</button>
</form>
<script>
  const email = document.getElementById("email");
  const age = document.getElementById("age");
  console.log(email.validity.typeMismatch, email.checkValidity());
  console.log(age.validity.rangeUnderflow, age.validationMessage);
</script>
```

Output (Chrome, English):

```text
console: true false
console: true Value must be greater than or equal to 18.
```

Notice: `type="email"` rejects `kakade@`, so `typeMismatch` is `true`. Native validation is UX, not security.

### Constraint validation API

Every control exposes `validity`, `validationMessage`, `checkValidity()` and `reportValidity()`. `checkValidity()` returns a boolean and fires `invalid`; `reportValidity()` also shows the bubble and focuses the invalid control.

```html
<form id="f">
  <input id="user" name="user" required pattern="[A-Za-z]+">
  <button>Send</button>
</form>
<script>
  const user = document.getElementById("user");
  console.log(user.validity.valueMissing, user.validity.valid);
  user.value = "Kakade1";
  console.log(user.validity.patternMismatch, user.checkValidity());
  console.log(user.validationMessage);
</script>
```

Output (Chrome, English):

```text
console: true false
console: true false
console: Please match the requested format.
```

| Flag | True when |
| --- | --- |
| `valueMissing` | `required` and empty |
| `typeMismatch` | email/url value is malformed |
| `patternMismatch` | value fails `pattern` (implicitly anchored) |
| `tooShort` / `tooLong` | length limits (user-edited values) |
| `rangeUnderflow` / `rangeOverflow` | below `min` / above `max` |
| `customError` | `setCustomValidity` message is set |
| `valid` | every other flag is false |

### setCustomValidity

`setCustomValidity(message)` marks the control invalid with your message; `""` clears it. The error is **sticky**: until cleared, the control stays invalid even after the value changes.

```html
<form id="f">
  <input id="user" name="user" value="admin">
  <button>Send</button>
</form>
<script>
  const user = document.getElementById("user");
  user.setCustomValidity("That name is taken");
  console.log(user.validity.customError, user.validationMessage);
  user.setCustomValidity("");
  console.log(user.validity.customError, user.checkValidity());
</script>
```

Output:

```text
console: true That name is taken
console: false true
```

Notice: clear with `setCustomValidity("")` before every re-check, then set a new message if still bad. Validate on `input`/`blur`; `submit` calls `reportValidity()`.

---

# 4. Users and Traps

### Accessibility

A form is only usable if screen readers can connect labels, help text and errors. Use `<label for="id">`, unique IDs, `<fieldset>` + `<legend>` for groups, and `aria-describedby` for hints. Announce errors with `role="alert"` and set `aria-invalid="true"`.

```html
<form id="f">
  <label for="email">Email</label>
  <input id="email" name="email" type="email" aria-describedby="email-help" required>
  <p id="email-help">We only use this for receipts.</p>
  <p id="email-error" role="alert" hidden></p>
</form>
<script>
  const email = document.getElementById("email");
  const error = document.getElementById("email-error");
  email.addEventListener("invalid", () => {
    error.hidden = false;
    error.textContent = "Enter a valid email for Rohan";
    email.setAttribute("aria-invalid", "true");
    console.log("announced:", error.textContent);
  });
  email.value = "kakade@";
  email.checkValidity();
</script>
```

Output:

```text
console: announced: Enter a valid email for Rohan
```

Notice: a placeholder is not a label. Say what to fix ("Enter an email like name@example.com"). Do not rely on red alone.

### Common traps

A `<button>` inside a form defaults to **`submit`**, so "Cancel" submits. Disabled and unchecked controls are omitted, and `preventDefault` after `await` is too late.

```html
<form id="f">
  <input name="name" value="Kakade">
  <input name="locked" value="secret" disabled>
  <input type="checkbox" name="news" value="yes">
  <button id="cancel">Cancel</button>
</form>
<script>
  const form = document.getElementById("f");
  const data = new FormData(form);
  console.log(data.get("name"), data.has("locked"), data.get("news"));

  form.addEventListener("submit", (event) => {
    event.preventDefault();
    console.log("submitted");
  });
  document.getElementById("cancel").click();
</script>
```

Output:

```text
console: Kakade false null
console: submitted
```

Bad and good:

```html
<!-- Bad: submits the form -->
<button id="cancel">Cancel</button>

<!-- Good: just a button -->
<button id="cancel" type="button">Cancel</button>
```

Also remember: checkbox `value` defaults to `"on"`, number values are strings, and HTML validation can be bypassed. Never trust the client for prices or permissions.

---

# Real-World Uses

- **Login** — intercept submit, then `fetch` JSON.
- **File uploads** — FormData + multipart.
- **Search** — GET forms produce shareable URLs.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **Form element** | `action`, `method`, `enctype`, `elements` |
| **submit event** | Form-level; `submitter`; `requestSubmit()` validates |
| **FormData** | Successful controls; `get`/`getAll`; files |
| **Input types** | Add semantics, keyboards, validation |
| **Constraint validation API** | `validity`, message, `checkValidity`, `reportValidity` |
| **setCustomValidity** | Custom message; clear with `""` |
| **preventDefault** | Stop navigation synchronously, before `await` |
| **Accessibility** | `label for`, `fieldset`, `aria-describedby`, `role="alert"` |
| **Serialization** | Strings; `URLSearchParams`; FormData for files |
| **File inputs** | `input.files`; `accept` is a hint; check type/size |
| **Common traps** | Buttons submit; omitted controls; client validation is not security |

## The Mental Model

```text
                  FORM
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     controls     submit      FormData
   name + type   validation   successful
        │           │         controls only
        ↓           ↓           │
   constraint    preventDefault │
    validity          │         ↓
        │             ↓      JSON / multipart
        └────────→ your handler → fetch
```

### The 3 things to remember

**1. Only named, enabled, checked controls are submitted.**

```javascript
new FormData(form).get("locked"); // null when disabled
```

**2. Prevent the default synchronously.**

```javascript
form.addEventListener("submit", (event) => {
  event.preventDefault();     // before await
  save(new FormData(form));
});
```

**3. Native validation is UX, not security.**

```javascript
if (!form.checkValidity()) form.reportValidity();
// server must validate again
```

> **A form is named controls plus validation: intercept `submit`, read FormData, validate on both sides.**
