# Projects — Revision Notes (Beginner)
> One-line mental model: projects are where HTML becomes instinct — build structure first, then verify like a keyboard-only user.

## Core recall
- Every project starts with doctype, `lang`, charset, viewport, title, and description.
- Profile page: header/nav/main/article/section/footer, image alt + dimensions, `address`, footer.
- Signup form: labels tied by `for`/`id`, fieldset + legend, autocomplete tokens, validation attributes, `aria-describedby`, explicit button type.
- Pricing table: `caption`, thead/tbody/tfoot, `th scope`, `colspan`, scroll wrapper for mobile.
- Media gallery: `figure`/`figcaption`, `srcset` + `sizes`, dimensions, lazy loading, video `track` captions.
- Blog article: `article`, `time datetime`, blockquote, escaped code, author footer, SEO/OG metadata.
- Dashboard: landmarks, `dl` for stats, aside for tips.
- FAQ: native `details`/`summary`; modal: `dialog` (modal) or `popover` (non-modal).
- Verification: W3C validator, keyboard tab pass, CSS-disabled pass, screen-reader spot check.

## Must-remember code
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Kakade — Frontend Learner</title>
    <meta name="description" content="Kakade's profile and HTML notes." />
  </head>
  <body>
    <header><h1>Kakade</h1></header>
    <main><article><h2>About</h2></article></main>
    <footer><p>© 2026 Kakade</p></footer>
  </body>
</html>
```
```html
<figure>
  <img
    src="/img/desk-800.jpg"
    srcset="/img/desk-400.jpg 400w, /img/desk-800.jpg 800w"
    sizes="(max-width: 600px) 100vw, 50vw"
    alt="Kakade's desk with two monitors"
    width="800"
    height="450"
    loading="lazy"
  />
  <figcaption>The 2026 interview prep desk.</figcaption>
</figure>
```
```html
<details>
  <summary>Is HTML a programming language?</summary>
  <p>No — it is a markup language.</p>
</details>

<dialog id="contact">
  <form method="dialog">
    <p>Email kakade@example.com</p>
    <button>Close</button>
  </form>
</dialog>
```

## Interview traps
- Shipping a project without labels, alt, or dimensions.
- Using divs as buttons/links and failing keyboard tests.
- Hiding focusable elements with `aria-hidden`.
- Tables used for layout in the pricing build.
- Skipping the no-CSS and validator passes.
- Copying a framework component instead of writing semantic HTML first.

## 3 things to remember
1. Structure first: shell → landmarks → content → media → forms.
2. Label/alt/dimension everything.
3. Verify with validator + keyboard + no CSS.
