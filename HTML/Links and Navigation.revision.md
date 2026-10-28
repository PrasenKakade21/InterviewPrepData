# Links and Navigation — Revision Notes (Beginner)
> One-line mental model: a link is navigation with a real href — resolve the URL, secure new tabs, and write self-explanatory text.

## Core recall
- `<a href="...">` is a link only with `href`; without it the element is not focusable or announced as a link.
- Absolute URL = protocol + domain. Root-relative = starts with `/`, resolves from the domain root. Document-relative = resolves against the current page.
- Root-relative paths survive route changes; they break on subpath deployments without a base path.
- `target="_blank"` should ship with `rel="noopener noreferrer"` (reverse-tabnabbing, referrer privacy).
- `rel="nofollow"`, `sponsored`, `ugc` give search engines relationship hints.
- `download` saves same-origin files and can suggest a filename; it is ignored cross-origin.
- `mailto:` opens the mail client, `tel:` opens the dialer; both can carry query params.
- Fragment links `#id` jump in-page; `id` must be unique. Skip links are fragment links placed first in tab order.
- A link navigates; a button performs an action. Never use `<a>` with a click handler as a button.
- Descriptive link text and image-link `alt` matter for screen readers and SEO.

## Must-remember code
```html
<a href="/notes">Read Kakade's HTML notes</a>
<a href="https://kakade.dev" target="_blank" rel="noopener noreferrer">Kakade.dev</a>
<a href="/files/kakade-resume.pdf" download="Kakade-Resume.pdf">Download resume</a>
```
```html
<a href="#main" class="skip-link">Skip to content</a>
<main id="main">...</main>
```
```html
<a href="/">
  <img src="/logo.svg" alt="Kakade.dev home" width="120" height="32" />
</a>
```
```html
<a href="mailto:kakade@example.com?subject=Interview">Email Kakade</a>
<a href="tel:+911234567890">Call Kakade</a>
```

## Interview traps
- `<a>` without `href` — not a link, not keyboard focusable.
- `target="_blank"` without `rel="noopener noreferrer"` — legacy reverse-tabnabbing risk.
- "Click here" link text — meaningless in a screen-reader link list.
- `download` on cross-origin URLs — filename and behavior ignored.
- Using `name` instead of `id` for fragment targets — modern browsers scroll to `id`.
- Duplicate `id`s — fragment jumps to the first match.
- Using a link for an in-page action instead of a `button`.

## 3 things to remember
1. Real `href`, correct resolution: absolute vs root-relative vs relative.
2. New tab needs `rel="noopener noreferrer"`.
3. Skip link first, descriptive text everywhere.
