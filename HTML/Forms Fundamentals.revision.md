# Forms Fundamentals — Revision Notes (Beginner)
> One-line mental model: a form submits named, labeled controls — method decides URL vs body, and HTML validation is UX, not security.

## Core recall
- `form` groups controls; `action` is the submit URL (defaults to the current page).
- `method="get"` → data in query string, bookmarkable, idempotent; `method="post"` → data in body, for changes.
- POST is not secure by itself — HTTPS is still required for credentials.
- Every successful control submits as `name=value`; no `name` means no submission.
- Disabled controls and unchecked boxes/radios are excluded; read-only controls are included.
- `label` connects via `for="id"` or wrapping; gives the accessible name and a bigger click target.
- `placeholder` is a hint, never a label.
- Native validation: `required`, `type`, `pattern`, `min`/`max`, `minlength`/`maxlength`, `step`; re-validate on the server.
- `autocomplete` tokens (`email`, `current-password`, `new-password`) make autofill and password managers work.
- `enctype="multipart/form-data"` is required for file uploads; `novalidate` disables native validation.

## Must-remember code
```html
<form action="/login" method="post">
  <label for="email">Email</label>
  <input id="email" name="email" type="email" autocomplete="email" required />

  <label for="password">Password</label>
  <input id="password" name="password" type="password" autocomplete="current-password" required />

  <button type="submit">Log in</button>
</form>
```
```html
<form action="/upload" method="post" enctype="multipart/form-data">
  <label for="resume">Resume (PDF)</label>
  <input id="resume" name="resume" type="file" accept=".pdf" required />
  <button type="submit">Upload</button>
</form>
```
```html
<fieldset>
  <legend>Create account</legend>
  ...
</fieldset>
```

## Interview traps
- Missing `name` → field silently absent from the request.
- `placeholder` used as the only label.
- `for` pointing to a `name` instead of an `id`.
- Assuming POST encrypts data.
- Forgetting `multipart/form-data` for file uploads.
- Relying on client-side validation alone.
- Positive `tabindex` breaking natural focus order.

## 3 things to remember
1. `action` + `method` decide where and how data travels.
2. `name` + `label for` decide whether data and meaning exist.
3. `multipart/form-data` for files; server-side validation always.
