# Form Controls — Revision Notes (Beginner)
> One-line mental model: match the control to the value — the browser then supplies the right keyboard, picker, validation, and semantics.

## Core recall
- `input type` drives keyboard, picker, and validation: `text`, `email`, `password`, `tel`, `url`, `search`.
- Numeric/date controls: `number`, `range`, `date`, `time`, `color`; constrain with `min`, `max`, `step`.
- `number` is wrong for phone numbers/PINs (leading zeros); use `tel` or `text` + `inputmode="numeric"`.
- Checkbox = independent booleans; radio = exactly one per shared `name`; only checked controls submit.
- `file` needs `accept` and a multipart form; `hidden` is state, never security.
- `select` + `optgroup` + `option` for fixed choices; `multiple` allows several.
- `textarea` value is its inner text, not a `value` attribute.
- `<button>` defaults to `submit`; always set `submit`, `button`, or `reset`.
- `datalist` suggests without restricting; `output` displays results; `progress` = completion, `meter` = measurement.
- `fieldset` + `legend` group controls and name the group (essential for radios/checkboxes).

## Must-remember code
```html
<label for="email">Email</label>
<input id="email" name="email" type="email" autocomplete="email" required />
```
```html
<fieldset>
  <legend>Experience</legend>
  <label><input type="radio" name="level" value="junior" /> Junior</label>
  <label><input type="radio" name="level" value="senior" checked /> Senior</label>
</fieldset>
```
```html
<label for="country">Country</label>
<select id="country" name="country">
  <optgroup label="Asia">
    <option value="in">India</option>
    <option value="jp">Japan</option>
  </optgroup>
</select>
```
```html
<button type="submit">Save</button>
<button type="button">Cancel</button>
```
```html
<label for="city">City</label>
<input id="city" name="city" list="cities" />
<datalist id="cities">
  <option value="Pune"></option>
  <option value="Mumbai"></option>
</datalist>
```

## Interview traps
- `type="number"` for phones, PINs, or currency.
- Radio buttons with different `name`s — they stop being a group.
- `<button>` without `type` submitting accidentally.
- Hidden inputs trusted for roles or prices.
- Missing `fieldset`/`legend` around radio groups.
- `textarea` given a `value` attribute instead of inner text.
- Placeholder treated as the label.

## 3 things to remember
1. Type matches value: email/date/file/range each get native behavior.
2. Radios share a name; checkboxes do not need to.
3. `fieldset` + `legend` names the group; buttons always declare type.
