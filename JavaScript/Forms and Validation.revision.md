# Forms and Validation — Revision Notes (Beginner)

> One-line mental model: named controls → submit event → FormData / constraint validation; call `preventDefault` synchronously and validate again on the server.

## Core recall

- **Form element**: `action` (URL), `method` (`get`/`post`, default GET), `enctype` (urlencoded, multipart for files), `novalidate`, `elements` (live collection).
- **Successful controls**: need a `name`, not disabled, checked if checkbox/radio. Unnamed, disabled and unchecked controls are **not** submitted.
- **submit event**: fires on the form for a submit button, Enter, or `requestSubmit()`; `event.submitter` identifies the button. `form.submit()` bypasses validation and the event.
- **preventDefault**: stop navigation/reload; must run synchronously, before any `await`. Without JS the form still submits.
- **FormData**: `new FormData(form)`, `get`/`getAll`/`entries`; files become `File`; unchecked boxes absent (not `false`).
- **Input types**: `email`, `url`, `number`, `range`, `date`, `tel`, `password`, `file`, `checkbox`, `radio` add semantics, keyboards and validation; attributes add `required`, `min`, `max`, `pattern`, `step`.
- **Constraint validation API**: `validity` flags (`valueMissing`, `typeMismatch`, `patternMismatch`, `rangeUnderflow`, `customError`…), `valid`, `validationMessage`, `checkValidity()`, `reportValidity()`.
- **setCustomValidity**: set a message to mark invalid; clear with `""` or the control stays invalid forever.
- **Accessibility**: `<label for>`, unique IDs, `fieldset`/`legend`, `aria-describedby`, `aria-invalid`, `role="alert"`; focus the first invalid field; never rely on color alone.
- **Serialization**: `Object.fromEntries(new FormData(form))` → all strings, duplicates collapse; `URLSearchParams` for urlencoded; FormData for multipart; JSON needs explicit type conversion.
- **File inputs**: `input.files` FileList of `File` objects; `accept` is a hint, not validation; validate type/size server-side; `URL.createObjectURL` + revoke for previews.
- **Common traps**: button defaults to `submit`; disabled/unchecked omitted; number values are strings; native validation is not security.

## Must-remember code

```javascript
// intercept submit and read data
form.addEventListener("submit", (event) => {
  event.preventDefault(); // before await
  const data = new FormData(form);
  console.log(data.get("email"));
});
```

```javascript
// serialize with duplicate support
const out = {};
for (const [key, value] of new FormData(form)) {
  out[key] = key in out ? [].concat(out[key], value) : value;
}
```

```javascript
// custom validity
if (confirm.value !== password.value) {
  confirm.setCustomValidity("Passwords do not match");
} else {
  confirm.setCustomValidity("");
}
```

```javascript
// native validation before submit
if (!form.checkValidity()) {
  event.preventDefault();
  form.reportValidity();
}
```

## Interview traps

- `<button>` inside a form defaults to `type="submit"`; a Cancel button submits unless you set `type="button"`.
- `preventDefault()` after `await` is too late — the browser already navigated.
- Disabled inputs and unchecked checkboxes are omitted from FormData; absence is not `false`.
- Number inputs produce strings; checkbox `value` defaults to `"on"`.
- `form.submit()` skips validation and the submit event; `requestSubmit()` does not.
- `setCustomValidity` is sticky — always clear it with `""` before re-checking.
- `pattern` is implicitly anchored; `minlength`/`tooShort` apply to user-edited values.
- `novalidate` disables native validation UI, not your submit handler.
- `accept="image/*"` is only a picker hint; validate MIME type and size on the server.
- Client validation is UX only — anyone can send a crafted request.

## 3 things to remember

1. **Only successful controls are submitted.**
   ```javascript
   new FormData(form).get("locked"); // null when disabled
   ```

2. **Prevent default synchronously.**
   ```javascript
   form.addEventListener("submit", (event) => {
     event.preventDefault();
     save(new FormData(form));
   });
   ```

3. **Native validation is UX, not security.**
   ```javascript
   form.checkValidity();  // client
   // server must validate again
   ```
