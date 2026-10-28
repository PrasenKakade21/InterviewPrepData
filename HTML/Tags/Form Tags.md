# Form Tags

Form tags collect and submit user input. `form` wraps controls, `input` covers most value types, `select`/`textarea` handle longer choices, and `fieldset`/`legend` group them. Every control needs a `name` to submit and a `label` to be accessible.

The core idea:

```text
form
  ├── fieldset + legend → group name
  ├── label + input/select/textarea → named value
  ├── button → submit/reset/action
  └── datalist/output/progress/meter → helpers
```

---

# `<form>`

Groups controls and defines how data is submitted.

| Attribute | Purpose | Example |
| --------- | ------- | ------- |
| `action` | Submit URL | `/signup` |
| `method` | `get` or `post` | `post` |
| `enctype` | Body encoding | `multipart/form-data` |
| `target` | Where to display response | `_blank` |
| `novalidate` | Disable native validation | `novalidate` |
| `autocomplete` | Form-level autofill | `on`/`off` |

```html
<form action="/signup" method="post">
  <label for="email">Email</label>
  <input id="email" name="email" type="email" required />
  <button type="submit">Sign up</button>
</form>
```

Output:

```text
POST /signup  body: email=kakade%40example.com
```

Gotchas: controls without `name` are not submitted; nested forms are invalid; use `form="id"` for buttons outside the form.

---

# `<input>`

The most versatile control. Void element; behavior depends on `type`.

## Common input types

| Type | Purpose | Extra attributes |
| ---- | ------- | ---------------- |
| `text` | Single-line text | `maxlength`, `pattern` |
| `email` | Email address | `multiple` |
| `password` | Masked secret | `minlength` |
| `search` | Search field | `list` |
| `tel` | Phone number | `pattern` |
| `url` | URL | `pattern` |
| `number` | Numeric value | `min`, `max`, `step` |
| `range` | Slider | `min`, `max`, `step` |
| `date` / `time` / `datetime-local` | Date/time pickers | `min`, `max` |
| `month` / `week` | Month/week picker | `min`, `max` |
| `color` | Color picker | — |
| `file` | File upload | `accept`, `multiple` |
| `checkbox` | Independent boolean | `checked`, `value` |
| `radio` | One per name group | `checked`, `value` |
| `hidden` | Non-visible state | `value` |
| `submit` / `reset` / `button` | Button controls | `formaction`, `formmethod` |
| `image` | Image submit button | `src`, `alt` |

```html
<label for="email">Email</label>
<input id="email" name="email" type="email" autocomplete="email" required />

<label for="age">Age</label>
<input id="age" name="age" type="number" min="18" max="99" step="1" />

<label><input type="checkbox" name="newsletter" checked /> Newsletter</label>

<label><input type="radio" name="level" value="senior" /> Senior</label>
```

Output:

```text
email=...&age=24&newsletter=on&level=senior
```

Gotchas: `number` strips leading zeros (avoid for phone/PIN); only checked boxes/radios submit; `hidden` is tamperable.

---

# `<label>`

The accessible name for a control and a larger click target.

```html
<label for="password">Password</label>
<input id="password" name="password" type="password" />

<label>
  Remember Kakade
  <input type="checkbox" name="remember" />
</label>
```

Output:

```text
Clicking the text focuses the input or toggles the checkbox.
```

Gotchas: `for` must match `id`; a label can reference only one control; don't use placeholder as a label.

---

# `<select>`, `<optgroup>`, `<option>`

A dropdown of options. `multiple` allows several; `size` shows a list box.

```html
<label for="track">Track</label>
<select id="track" name="track">
  <optgroup label="Frontend">
    <option value="html">HTML</option>
    <option value="css" selected>CSS</option>
  </optgroup>
  <optgroup label="Backend">
    <option value="node">Node.js</option>
  </optgroup>
</select>
```

Output:

```text
Dropdown with two groups; CSS selected by default.
```

Gotchas: an option without `value` submits its text; `selected` is the default; `multiple` changes submission to repeated keys.

---

# `<textarea>`

Multi-line text input. Its value is the inner text, not a `value` attribute.

```html
<label for="bio">Bio</label>
<textarea id="bio" name="bio" rows="4" cols="40" maxlength="500">
Kakade is revising forms.</textarea>
```

Output:

```text
Textarea pre-filled with "Kakade is revising forms."
```

Gotchas: whitespace/newlines inside the tag become the initial value; use `rows`/`cols` for size or CSS.

---

# `<button>`

A clickable button. Default type inside a form is `submit`.

| `type` | Behavior |
| ------ | -------- |
| `submit` | Submits the form |
| `reset` | Resets controls |
| `button` | No default action |

```html
<button type="submit">Save</button>
<button type="button">Cancel</button>
<button type="reset">Reset</button>
```

Output:

```text
Save submits; Cancel does nothing by default; Reset clears.
```

Gotchas: always set `type`; a button can contain HTML (icons), unlike `input type=submit`.

---

# `<fieldset>` and `<legend>`

Group related controls and name the group.

```html
<fieldset>
  <legend>Contact details</legend>
  <label for="phone">Phone</label>
  <input id="phone" name="phone" type="tel" />
</fieldset>
```

Output:

```text
Screen reader: "Contact details, group"
  "Phone, edit text"
```

Gotchas: essential for radio/checkbox groups; `legend` must be the first child; fieldsets can be nested for complex forms.

---

# `<datalist>`

Suggests options for an input while still allowing free text.

```html
<label for="city">City</label>
<input id="city" name="city" list="cities" />
<datalist id="cities">
  <option value="Pune"></option>
  <option value="Mumbai"></option>
</datalist>
```

Output:

```text
Typing "P" suggests Pune; any other value is still accepted.
```

Gotchas: the input uses `list="datalist-id"`; datalist is a suggestion, not validation.

---

# `<output>`

Displays the result of a calculation. It has `for` to reference contributing controls.

```html
<form oninput="total.value = Number(hours.value) * 2">
  <label for="hours">Hours</label>
  <input id="hours" name="hours" type="number" value="4" />
  <output name="total" for="hours">8</output>
</form>
```

Output:

```text
Changing hours updates the output (8 for 4 hours).
```

Gotchas: `output` is display-only; it is not submitted unless you sync it to a hidden input.

---

# `<progress>`

Shows task completion.

```html
<progress value="70" max="100">70%</progress>
```

Output:

```text
Progress bar at 70%; fallback text "70%" for unsupported browsers.
```

Gotchas: omit `value` for an indeterminate spinner; `progress` is completion, not a measurement.

---

# `<meter>`

Shows a measurement within a known range (disk usage, score).

```html
<meter value="0.8" min="0" max="1" low="0.3" high="0.8" optimum="1">80%</meter>
```

Output:

```text
Meter filled to 80%, colored by low/high/optimum.
```

Gotchas: use `progress` for "how far along", `meter` for "how much of a known range".

---

# Quick Reference

| Tag | Purpose |
| --- | ------- |
| `form` | Submission container |
| `input` | All value types via `type` |
| `label` | Accessible name + click target |
| `select` / `optgroup` / `option` | Dropdown with groups |
| `textarea` | Multi-line text |
| `button` | Submit/reset/action |
| `fieldset` / `legend` | Group + group name |
| `datalist` | Suggestions |
| `output` | Calculated display |
| `progress` / `meter` | Completion / measurement |

> **Forms submit named, labeled controls — choose the control by value type, group related fields, and always declare button types.**
