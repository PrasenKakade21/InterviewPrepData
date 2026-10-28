# Obsolete and Deprecated Tags

Browsers still render many old tags for backward compatibility, but they are obsolete: invalid in HTML5, harmful for accessibility, and replaced by CSS or modern elements. Interviewers sometimes ask about them to check whether you know the modern replacement.

The core idea:

```text
old tag
    ↓
still renders (legacy support)
    ↓
but invalid + inaccessible
    ↓
use CSS / semantic replacement
```

Never use these in new code.

---

# Presentational Tags Replaced by CSS

## `<center>`

Centered content. Replaced by `text-align: center` or flex/grid centering.

```html
<!-- Obsolete -->
<center>Kakade</center>

<!-- Modern -->
<p class="center">Kakade</p>
```

Output:

```text
Kakade   (centered either way; modern version is valid)
```

## `<font>`

Sets color, face, and size. Replaced by CSS `color`, `font-family`, `font-size`.

```html
<!-- Obsolete -->
<font color="red" size="5">Kakade</font>

<!-- Modern -->
<p class="highlight">Kakade</p>
```

Output:

```text
Kakade (red, large via CSS class)
```

## `<big>` and `<tt>`

`big` enlarged text; `tt` teletype/monospace. Replaced by CSS and `<code>`.

```html
<!-- Obsolete -->
<big>Kakade</big>
<tt>kakade.dev</tt>

<!-- Modern -->
<span class="big">Kakade</span>
<code>kakade.dev</code>
```

Output:

```text
Kakade (larger)
kakade.dev (monospace)
```

## `<strike>` and `<s>`

`strike` is obsolete; `<s>` is valid for content no longer accurate. For edits, use `<del>`/`<ins>`.

```html
<!-- Obsolete -->
<strike>₹599</strike>

<!-- Modern -->
<s>₹599</s>
<del datetime="2026-10-28">Old price</del>
```

Output:

```text
₹599 (struck through)
Old price (deleted)
```

## `<blink>`

Blinking text. Never standardized, removed, and harmful (motion, seizure risk). Use CSS animation only with `prefers-reduced-motion` support.

```html
<!-- Obsolete -->
<blink>Kakade</blink>

<!-- Modern -->
<span class="fade">Kakade</span>
```

Output:

```text
Kakade (no blink; animate subtly or not at all)
```

## `<marquee>`

Scrolling text. Obsolete and distracting; replaced by CSS animations with reduced-motion handling.

```html
<!-- Obsolete -->
<marquee>Kakade's news</marquee>

<!-- Modern -->
<div class="ticker">Kakade's news</div>
```

Output:

```text
Static ticker text, animated by CSS if appropriate.
```

---

# Acronym and Applet Tags

## `<acronym>`

Obsolete; use `<abbr>`.

```html
<!-- Obsolete -->
<acronym title="HyperText Markup Language">HTML</acronym>

<!-- Modern -->
<abbr title="HyperText Markup Language">HTML</abbr>
```

Output:

```text
HTML with expansion available.
```

## `<applet>`

Embedded Java applets. Dead technology; use modern web APIs, `<canvas>`, or `<object>` for documents.

```html
<!-- Obsolete -->
<applet code="Kakade.class"></applet>

<!-- Modern -->
<canvas id="app" width="320" height="200"></canvas>
```

Output:

```text
Applet does not run; canvas is the replacement surface.
```

## `<basefont>`

Set a base font for the page. Replaced by CSS on `html`/`body`.

```html
<!-- Modern -->
<style>
  body { font-family: Inter, sans-serif; }
</style>
```

Output:

```text
Consistent base font via CSS.
```

---

# Frames and Legacy Layout

## `<frame>`, `<frameset>`, `<noframes>`

Old multi-document layouts. Removed from HTML5; use `<iframe>` only for embedding, and CSS grid/flexbox for layout.

```html
<!-- Obsolete -->
<frameset cols="50%,50%">
  <frame src="/nav.html" />
  <frame src="/main.html" />
</frameset>

<!-- Modern -->
<div class="layout">
  <nav>...</nav>
  <main>...</main>
</div>
```

Output:

```text
Modern layout with landmarks and responsive CSS.
```

Gotchas: frames break bookmarks, accessibility, printing, and mobile; never revive them.

## `<isindex>`

Ancient single-field search prompt. Replaced by a real `<form>` with an `<input type="search">`.

```html
<form action="/search" method="get">
  <label for="q">Search</label>
  <input id="q" name="q" type="search" />
</form>
```

Output:

```text
Accessible search field with a label and a submit action.
```

## `<spacer>`

Inserted blank space (Netscape). Replaced by CSS margins/padding.

---

# Other Obsolete Elements

## `<keygen>`

Generated a key pair in forms. Removed; use the Web Crypto API and proper authentication flows.

## `<menuitem>`

Part of the abandoned context-menu spec. Use `<menu>` with buttons or a custom menu pattern with ARIA.

## `<param>`

Passed parameters to `<object>`. Obsolete; pass data via `data-*`/query parameters or configure the embedded document directly.

## `<rb>` and `<rtc>`

Ruby base/container elements. Poorly supported; use `<ruby>`, `<rt>`, and `<rp>`.

## `<xmp>`, `<listing>`, `<plaintext>`

Preformatted/raw text legacy tags. Replaced by `<pre>` and `<code>`; `<plaintext>` also breaks parsing for everything after it.

```html
<!-- Modern -->
<pre><code>&lt;div&gt;Kakade&lt;/div&gt;</code></pre>
```

Output:

```text
<div>Kakade</div>   (shown as code, not rendered)
```

## `<image>`

Legacy alias for `<img>` in some parsers. Always write `<img>`.

## `<nextid>`, `<shadow>`

Abandoned experiments; never use. Shadow DOM is implemented via JavaScript APIs, not a `<shadow>` tag.

---

# Why They Still Render

Browsers keep legacy parsing rules for backward compatibility with old pages. That does not make the tags valid or accessible.

```text
old tag in HTML
    ↓
browser maps it to an unknown/presentational element
    ↓
looks okay visually
    ↓
but validator errors + no semantics + inconsistent behavior
```

Interview answer: "It renders because of legacy support, but it is obsolete — the modern replacement is CSS or a semantic element."

---

# Quick Reference

| Obsolete | Replacement |
| -------- | ----------- |
| `center` | CSS centering |
| `font` | CSS color/font properties |
| `big`, `tt` | CSS / `code` |
| `strike` | `s` / `del` |
| `blink`, `marquee` | CSS animation (reduced motion) |
| `acronym` | `abbr` |
| `applet` | canvas / modern APIs |
| `basefont` | CSS on body |
| `frame`, `frameset`, `noframes` | iframe + CSS layout |
| `isindex` | form + search input |
| `spacer` | CSS spacing |
| `keygen` | Web Crypto API |
| `menuitem` | menu + buttons |
| `param` | object data/config |
| `rb`, `rtc` | ruby/rt/rp |
| `xmp`, `listing`, `plaintext` | pre + code |
| `image` | img |
| `nextid`, `shadow` | never use |

> **Obsolete tags still render because browsers forgive old pages — replace them with CSS or semantic HTML and never ship them in new code.**
