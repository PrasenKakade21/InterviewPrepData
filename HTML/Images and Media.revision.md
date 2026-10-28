# Images and Media — Revision Notes (Beginner)
> One-line mental model: describe every image, reserve its space, and give every embed a title and fallback.

## Core recall
- `img` is void: `src` + `alt` required; `alt` describes purpose, not filename or pixels.
- Decorative images use `alt=""`; functional images inside links/buttons describe the action.
- Always set `width`/`height` (or CSS `aspect-ratio`) to reserve space and prevent CLS.
- `loading="lazy"` defers offscreen images; `decoding="async"` avoids blocking decode.
- `srcset` + `sizes` = resolution switching by viewport and DPR; `picture` + `source` = format/art-direction fallback, ending in `img`.
- `figure` + `figcaption` associates a caption with media; caption is first or last child.
- `audio`/`video` need `controls`, fallback text, poster for video, and `track` captions for speech.
- Autoplay with sound is an accessibility failure; browsers usually require `muted` anyway.
- `iframe` needs a descriptive `title`, plus `loading="lazy"` for offscreen embeds.
- `object` is preferred over `embed` for legacy embeds and can contain fallback content.

## Must-remember code
```html
<img src="/img/kakade.png" alt="Kakade presenting HTML notes" width="320" height="200" />
<img src="/img/divider.svg" alt="" width="600" height="8" />
```
```html
<img
  src="/img/setup-800.jpg"
  srcset="/img/setup-400.jpg 400w, /img/setup-800.jpg 800w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="Kakade's desk with two monitors"
  width="800"
  height="450"
  loading="lazy"
/>
```
```html
<picture>
  <source srcset="/img/kakade.avif" type="image/avif" />
  <img src="/img/kakade.jpg" alt="Kakade at a conference" width="800" height="450" />
</picture>
```
```html
<video controls width="640" height="360" poster="/img/poster.jpg">
  <source src="/media/talk.mp4" type="video/mp4" />
  <track kind="captions" src="/media/talk.vtt" srclang="en" label="English" default />
  Your browser does not support video.
</video>
```
```html
<iframe src="/map" title="Kakade's office location" loading="lazy"></iframe>
```

## Interview traps
- `alt="image"` or a filename — noise instead of meaning.
- Missing width/height — layout shift when the image loads.
- Using `picture` without a fallback `img`.
- Autoplaying audio or video with sound.
- `iframe` without `title` — announced as an unlabeled frame.
- Lazy-loading the hero image — delays LCP; keep above-the-fold images eager.
- `download` on cross-origin media — ignored by the browser.

## 3 things to remember
1. Alt = purpose; empty alt for decoration.
2. Dimensions and lazy loading protect layout and speed.
3. Captions, controls, and iframe titles make media usable.
