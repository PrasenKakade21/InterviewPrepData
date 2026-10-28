# Entities, Encoding and Validation — Revision Notes (Beginner)
> One-line mental model: encoding decides whether characters exist, entities keep markup safe, and the doctype decides which rendering rules apply.

## Core recall
- Encoding maps bytes to characters; UTF-8 covers all Unicode and is ASCII-compatible.
- `<meta charset="UTF-8" />` must appear early in `head`, and files must be saved as UTF-8.
- Mojibake (cafÃ©) = bytes decoded with the wrong charset; fix charset + file encoding.
- Named entities: `&lt;`, `&gt;`, `&amp;`, `&quot;`, `&copy;`, `&mdash;`, `&nbsp;`.
- Numeric entities: `&#169;` decimal, `&#xA9;` hex; useful for untypeable characters.
- Escape `<`, `>`, `&` whenever they are text, not markup.
- HTML collapses whitespace; `&nbsp;` prevents a line break but is not a layout tool.
- Emoji/symbols are Unicode; label functional symbols with text or `aria-label`.
- The W3C validator catches errors browsers silently repair: misnesting, duplicate ids, missing alt, bad attribute values.
- Doctype selects standards vs quirks mode; `document.compatMode` reports which is active.

## Must-remember code
```html
<meta charset="UTF-8" />
```
```html
<p>Use &lt;div&gt; for layout &amp; structure.</p>
<p>&copy; 2026 Kakade &mdash; &#8377;499</p>
```
```html
<pre><code>&lt;p&gt;Hello Kakade&lt;/p&gt;</code></pre>
```
```html
<!DOCTYPE html>
<html lang="en">...</html>
```

## Interview traps
- Missing/late charset → mojibake.
- Raw `<div>` inside text instead of `&lt;div&gt;`.
- Chaining `&nbsp;` for indentation.
- Emoji as the only label for an action.
- Duplicate `id`s and missing `alt` — validator catches, browser hides.
- Missing doctype → quirks mode and different box sizing.
- Patching encoding bugs with entities instead of fixing charset.

## 3 things to remember
1. UTF-8 declaration + UTF-8 file save.
2. Escape `<`, `>`, `&` in text.
3. Doctype first, entities for special characters.
