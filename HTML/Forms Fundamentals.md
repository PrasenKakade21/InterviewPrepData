# Forms Fundamentals

Forms are the browser's original input system: user data → `form` → HTTP request → server. Interviewers test whether you know GET vs POST, why `name` matters, how `label` connects to an input, and why native validation exists.

The core idea:

```text
form
    ↓
named controls
    ↓
serialized data (key=value)
    ↓
GET → query string   |   POST → request body
```

If a control has no `name`, it is never submitted. That single fact explains most "my form is empty" bugs.

---

# 1. The form Element and action

`form` groups controls and defines where data goes. `action` is the URL that receives the submission; without it, the browser submits to the current page.

```html
<form action="/subscribe" method="post">
  <label for="email">Email</label>
  <input id="email" name="email" type="email" required />
  <button type="submit">Subscribe</button>
</form>
```

Output:

```text
User types kakade@example.com → clicks Subscribe
Browser POSTs to /subscribe with body: email=kakade%40example.com
```

Think:

```text
<form action="/subscribe" method="post">
     │                │
     │                └── where the data goes
     └── groups the controls that submit together
```

Multiple forms on one page are fine — each submits only its own controls. A submit button outside the form can target it with the `form="id"` attribute.

---

# 2. method GET vs POST

`GET` appends data to the URL query string; `POST` sends it in the request body. GET is for retrieval and is bookmarkable; POST is for changes and avoids exposing data in the URL.

```html
<!-- Search: safe, repeatable, shareable -->
<form action="/search" method="get">
  <input name="q" type="search" />
  <button type="submit">Search</button>
</form>

<!-- Signup: changes state -->
<form action="/signup" method="post">
  <input name="email" type="email" />
  <button type="submit">Create account</button>
</form>
```

Output:

```text
GET  → /search?q=html
POST → /signup   body: email=kakade%40example.com
```

| Aspect | GET | POST |
| ------ | --- | ---- |
| Data location | URL query string | Request body |
| Visible in history | Yes | No |
| Bookmarkable | Yes | No (re-submit warning) |
| Idempotent | Yes | No |
| Sensitive data | Never | Still needs HTTPS |
| Typical use | Search, filters | Create/update/delete |

Interview trap: POST is not "secure" by itself — it only keeps data out of the URL. Passwords and tokens still require HTTPS.

---

# 3. name, value and the input Element

The browser serializes successful controls as `name=value`. `input` is the workhorse control; its `type` changes behavior and validation. `value` sets the initial value.

```html
<form action="/profile" method="post">
  <input type="text" name="firstName" value="Kakade" />
  <input type="hidden" name="userId" value="42" />
  <input type="email" name="email" placeholder="kakade@example.com" />
</form>
```

Output:

```text
Submitted body:
firstName=Kakade&userId=42&email=...
```

Rules that matter:

- No `name` → control is not submitted.
- Disabled controls are not submitted; read-only controls are.
- Unchecked checkboxes/radios are not submitted.
- Duplicate names produce repeated keys (`tag=a&tag=b`).

`placeholder` is a hint, not a label — it disappears on typing and is not reliably announced.

---

# 4. The label Element

`label` gives a control its accessible name and expands the click target. Connect it with `for="id"` or by wrapping the input. Every input needs a label.

```html
<!-- Explicit: for + id -->
<label for="email">Email address</label>
<input id="email" name="email" type="email" />

<!-- Implicit: wrapping -->
<label>
  Subscribe to Kakade's newsletter
  <input type="checkbox" name="newsletter" />
</label>
```

Output:

```text
Clicking the text "Email address" focuses the email input.
Screen reader: "Email address, edit text"
```

Bad:

```html
<input type="email" placeholder="Email" />
```

Good:

```html
<label for="email">Email</label>
<input id="email" name="email" type="email" />
```

Interview trap: `for` must match the input's `id`, not its `name`. A label pointing at a nonexistent id silently does nothing.

---

# 5. required, Validation Attributes and autocomplete

HTML5 validates natively before submit: `required`, `type`, `pattern`, `min`/`max`, `minlength`/`maxlength`, `step`. `autocomplete` helps browsers and password managers fill fields correctly.

```html
<form action="/signup" method="post">
  <label for="email">Email</label>
  <input
    id="email"
    name="email"
    type="email"
    required
    autocomplete="email"
    placeholder="kakade@example.com"
  />

  <label for="password">Password</label>
  <input
    id="password"
    name="password"
    type="password"
    required
    minlength="8"
    autocomplete="new-password"
  />

  <button type="submit">Create account</button>
</form>
```

Output:

```text
Empty submit → browser blocks: "Please fill out this field."
Short password → browser blocks: "Please lengthen this text to 8 characters or more."
Autofill fills email and suggests a strong password.
```

Native validation is a UX layer, **not** security — always validate again on the server. `novalidate` on the form disables it when you need custom validation.

---

# 6. enctype and Form Accessibility

`enctype` controls how the body is encoded. The default is `application/x-www-form-urlencoded`; file uploads require `multipart/form-data`. `text/plain` exists but is rarely useful.

```html
<form action="/upload" method="post" enctype="multipart/form-data">
  <label for="resume">Resume (PDF)</label>
  <input id="resume" name="resume" type="file" accept=".pdf" required />
  <button type="submit">Upload</button>
</form>
```

Output:

```text
Request body is multipart with file bytes + field metadata.
Without multipart/form-data, the file would submit only its filename.
```

Accessibility checklist:

- Every control has a `label`.
- Group related controls with `fieldset` + `legend`.
- Mark required fields in text, not only with color.
- Show errors next to the field and describe them in text.
- Keep tab order logical; don't use positive `tabindex`.

---

# Real-World Uses

A complete signup form combines action/method, labeled controls, autocomplete, native validation, and a submit button.

```html
<form action="/signup" method="post" novalidate>
  <fieldset>
    <legend>Create Kakade's account</legend>

    <label for="name">Full name</label>
    <input id="name" name="name" type="text" autocomplete="name" required />

    <label for="email">Email</label>
    <input id="email" name="email" type="email" autocomplete="email" required />

    <label for="password">Password</label>
    <input
      id="password"
      name="password"
      type="password"
      autocomplete="new-password"
      minlength="8"
      required
    />

    <button type="submit">Sign up</button>
  </fieldset>
</form>
```

Output:

```text
All fields labeled and grouped.
Browser autofill works per field.
Submission posts name, email, password to /signup.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **form** | Groups controls; `action` is the target URL |
| **action** | Where data is submitted; defaults to current page |
| **GET vs POST** | Query string vs body; retrieval vs change |
| **name and value** | Serialized as `name=value`; no name = not submitted |
| **input** | `type` drives behavior and validation |
| **label** | `for` matches `id`; always label controls |
| **Validation attributes** | `required`, `pattern`, `min`, `maxlength` |
| **autocomplete** | Correct tokens help autofill/password managers |
| **enctype** | `multipart/form-data` for file uploads |
| **Form accessibility** | Labels, fieldsets, text errors, logical tab order |

## The Mental Model

```text
form
  ├── action → where
  ├── method → GET (URL) or POST (body)
  ├── enctype → urlencoded or multipart
  └── controls
        ├── name → submitted key
        ├── value → submitted value
        ├── label → accessible name
        └── validation → browser check before submit
```

### The 3 things to remember

**1. Label every control:**

```html
<label for="email">Email</label>
<input id="email" name="email" type="email" />
```

**2. GET retrieves, POST changes:**

```html
<form action="/search" method="get">...</form>
<form action="/signup" method="post">...</form>
```

**3. Files need multipart:**

```html
<form method="post" enctype="multipart/form-data">...</form>
```

> **A form submits named, labeled controls — method decides where the data travels, and native validation is UX, never security.**
