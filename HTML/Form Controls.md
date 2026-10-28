# Form Controls

Every form control exists to collect a specific kind of value. `input type` decides the keyboard, validation, and picker the browser offers; `select`, `textarea`, and `button` cover the rest. Choosing the right control is half of good form UX.

The core idea:

```text
what value?
    ├── short text        → input type=text/email/password
    ├── number/date       → input type=number/range/date
    ├── one of many       → select or radio
    ├── many of many      → checkbox
    ├── long text         → textarea
    └── action            → button
```

The browser gives you free validation, mobile keyboards, and accessibility when you pick correctly.

---

# 1. Input Types Overview: text, email and password

`type="text"` is the default. `email` validates format and opens an email keyboard; `password` masks characters. `search` adds a clear button on some platforms.

```html
<label for="name">Name</label>
<input id="name" name="name" type="text" autocomplete="name" />

<label for="email">Email</label>
<input id="email" name="email" type="email" autocomplete="email" required />

<label for="password">Password</label>
<input id="password" name="password" type="password" autocomplete="new-password" minlength="8" />
```

Output:

```text
Name:     Kakade
Email:    kakade@example.com   ← must contain @ and a domain
Password: ••••••••             ← masked, min 8 chars
```

Mobile keyboards change automatically: `email` shows `@`, `tel` shows a dial pad, `url` shows `/`. This is why `type="text"` for everything is a UX bug.

---

# 2. number, range, date, time and Color

Numeric and date types give spinners, sliders, and pickers. `min`, `max`, and `step` constrain values; `range` always submits a string, so parse it on the server.

```html
<label for="age">Age</label>
<input id="age" name="age" type="number" min="18" max="99" step="1" />

<label for="hours">Daily study hours</label>
<input id="hours" name="hours" type="range" min="0" max="12" step="1" value="4" />

<label for="date">Session date</label>
<input id="date" name="date" type="date" min="2026-01-01" />

<label for="time">Start time</label>
<input id="time" name="time" type="time" />

<label for="color">Theme</label>
<input id="color" name="color" type="color" value="#0f172a" />
```

Output:

```text
Age: 24 (spinner, 18–99)
Hours: 4 (slider)
Date: 2026-10-28 (calendar picker)
Time: 09:30
Theme: #0f172a (color picker)
```

Interview trap: `type="number"` is not a currency or phone field — leading zeros and formatting get lost. Use `text` with `inputmode="numeric"` for IDs and phone numbers.

---

# 3. Checkbox and radio

Checkboxes are independent booleans (zero or more selected). Radios are mutually exclusive within the same `name` (exactly one selected). Only checked controls submit.

```html
<fieldset>
  <legend>Skills</legend>
  <label><input type="checkbox" name="skills" value="html" checked /> HTML</label>
  <label><input type="checkbox" name="skills" value="css" /> CSS</label>
</fieldset>

<fieldset>
  <legend>Experience</legend>
  <label><input type="radio" name="level" value="junior" /> Junior</label>
  <label><input type="radio" name="level" value="senior" checked /> Senior</label>
</fieldset>
```

Output:

```text
Submitted: skills=html&level=senior
Checking CSS later: skills=html&skills=css&level=senior
```

Notice: radios share a `name` to form a group. Different names mean different groups, even if they look related.

---

# 4. file, color, search and hidden Inputs

`file` opens the OS picker; `accept` filters types. `hidden` stores data the user should not edit — never trust it, since users can change it.

```html
<label for="avatar">Avatar</label>
<input id="avatar" name="avatar" type="file" accept="image/png,image/jpeg" />

<input type="hidden" name="userId" value="42" />

<label for="q">Search notes</label>
<input id="q" name="q" type="search" placeholder="HTML, CSS, JS" />
```

Output:

```text
Avatar picker limited to PNG/JPEG.
userId=42 is submitted but invisible.
Search field may show a clear (×) button.
```

Interview trap: hidden inputs are for state, not security. Validate ownership and permissions on the server.

---

# 5. select, optgroup, option, textarea

`select` is a dropdown; `option` is a choice; `optgroup` groups choices. Add `multiple` for multi-select. `textarea` handles multi-line text and its value is its inner text, not a `value` attribute.

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

<label for="notes">Notes</label>
<textarea id="notes" name="notes" rows="4" maxlength="500">
Kakade is revising forms.</textarea>
```

Output:

```text
Track: CSS (default selected)
Notes: Kakade is revising forms.
```

A placeholder-like empty option (`<option value="">Choose…</option>`) is common, but it must be `disabled` and `selected` if you want to force a real choice.

---

# 6. Button Types, datalist, output, progress and meter

`button` defaults to `type="submit"` inside a form — always set the type explicitly. `datalist` offers suggestions while still allowing free text. `output` shows calculated results; `progress` and `meter` visualize completion and measurements.

```html
<button type="submit">Save</button>
<button type="button">Cancel</button>
<button type="reset">Reset</button>

<label for="city">City</label>
<input id="city" name="city" list="cities" />
<datalist id="cities">
  <option value="Pune"></option>
  <option value="Mumbai"></option>
</datalist>

<output name="total" for="hours">4</output>
<progress value="70" max="100">70%</progress>
<meter value="0.8" min="0" max="1">80%</meter>
```

Output:

```text
Save submits; Cancel does nothing; Reset clears.
City input suggests Pune/Mumbai but accepts typed values.
Total output: 4
Progress bar: 70%
Meter: 80% (measurement, not task progress)
```

Interview trap: a `<button>` with no `type` inside a form submits it — a common cause of "Cancel accidentally saved the form".

---

# 7. fieldset and legend

`fieldset` groups related controls; `legend` names the group. They are essential for radio/checkbox groups, where the group itself needs an accessible name.

```html
<form action="/profile" method="post">
  <fieldset>
    <legend>Contact details</legend>

    <label for="phone">Phone</label>
    <input id="phone" name="phone" type="tel" autocomplete="tel" />

    <label for="country">Country</label>
    <select id="country" name="country">
      <option>India</option>
      <option>Japan</option>
    </select>
  </fieldset>

  <button type="submit">Save</button>
</form>
```

Output:

```text
Screen reader: "Contact details, group"
  "Phone, edit text"
  "Country, combo box"
"Save, button"
```

Grouping helps everyone: visually it separates sections, semantically it gives context, and for radios it supplies the missing group label.

---

# Real-World Uses

A profile form uses fieldsets, every major input type, a select, a textarea, and correctly typed buttons.

```html
<form action="/profile" method="post">
  <fieldset>
    <legend>Kakade's profile</legend>

    <label for="name">Name</label>
    <input id="name" name="name" type="text" autocomplete="name" required />

    <label for="email">Email</label>
    <input id="email" name="email" type="email" autocomplete="email" required />

    <label for="role">Role</label>
    <select id="role" name="role">
      <option value="fe">Frontend</option>
      <option value="be">Backend</option>
    </select>

    <label for="bio">Bio</label>
    <textarea id="bio" name="bio" rows="3"></textarea>

    <label><input type="checkbox" name="newsletter" /> Newsletter</label>
  </fieldset>

  <button type="submit">Save profile</button>
  <button type="button">Cancel</button>
</form>
```

Output:

```text
All controls grouped and labeled.
Submit sends name, email, role, bio, and newsletter only if checked.
Cancel does not submit.
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Input types** | `type` sets keyboard, picker, validation |
| **Text, email, password** | Basic text; email validates; password masks |
| **Number, range, date, time** | Constrain with `min`/`max`/`step` |
| **Checkbox and radio** | Many independent vs one per `name` group |
| **File, color, search** | Picker, color input, search field |
| **select/optgroup/option** | Dropdown with grouped choices |
| **textarea** | Multi-line; value is inner text |
| **Button types** | `submit`, `button`, `reset` — always explicit |
| **datalist and output** | Suggestions; calculated display |
| **progress and meter** | Task completion vs measurement |
| **fieldset and legend** | Group name for related controls |

## The Mental Model

```text
control decision
  │
  ├── one line of text → input type=text/email/password/search
  ├── constrained value → number/range/date/time/color
  ├── boolean → checkbox
  ├── one of many → radio (shared name) or select
  ├── long text → textarea
  ├── file → input type=file (+ multipart form)
  ├── suggestion → datalist
  └── action → button type=button/submit/reset
        │
        └── group with fieldset + legend
```

### The 3 things to remember

**1. Type for the value:**

```html
<input type="email" name="email" autocomplete="email" />
<input type="date" name="date" min="2026-01-01" />
```

**2. Radios share a name:**

```html
<label><input type="radio" name="level" value="senior" /> Senior</label>
```

**3. Buttons need a type:**

```html
<button type="submit">Save</button>
<button type="button">Cancel</button>
```

> **Pick the control that matches the value — the browser then gives you the right keyboard, validation, and accessible behavior for free.**
