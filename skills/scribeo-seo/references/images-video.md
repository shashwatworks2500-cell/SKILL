# Images & Video

Load when working on media discoverability.

**Scope here is narrow.** Accessibility owns alt text as a requirement; performance owns file size and delivery. This file covers only the discoverability implementation.

## The alt-text boundary — read this first

**`scribeo-accessibility` owns the alt-text requirement.** Alt text exists so that people who cannot see an image receive its meaning. That is its purpose, and it is not negotiable for SEO reasons.

**SEO benefits from meaningful image context as a side effect** — Google uses alt text among the signals for understanding an image. But:

**Never turn alt text into a keyword slot.** Keyword-stuffed alt text is worse for the people it exists to serve, contrary to Google's own guidance on keyword stuffing, and it does not buy what folklore claims.

```html
<!-- Right: describes the image, serves users, incidentally informative -->
<img src="/products/rose-gold-solitaire.jpg"
     alt="Rose gold solitaire ring with a round brilliant diamond">

<!-- Wrong: keyword stuffing; hostile to the people alt text is for -->
<img src="/products/rose-gold-solitaire.jpg"
     alt="engagement rings bristol rose gold ring diamond rings uk buy rings">
```

Write the alt text an accessibility audit would approve. That is also the version that serves search.

Decorative images take `alt=""` — an accessibility rule this skill does not override for discoverability reasons.

## Image discoverability

**Requirements:**

- **Real `<img>` elements** for meaningful images. CSS background images are not discoverable as images and carry no alt text.
- **Stable, crawlable URLs.** Images behind authentication, blocked in robots.txt, or served from an expiring signed URL are not discoverable.
- **Not blocked in robots.txt** — this also affects rendering; see `javascript-rendering.md`.
- **Descriptive filenames** where practical. `rose-gold-solitaire-ring.jpg` carries more context than `IMG_4471.jpg`. A **Scribeo recommendation**, not a documented requirement — and a filename is a weak signal compared with surrounding content and alt text.
- **Surrounding context matters.** An image near a relevant heading and caption is better understood than one in isolation.
- **Lazy-loading is fine** for below-the-fold images, but the `src`/`srcset` must still resolve to real URLs. A placeholder that only becomes real after a scroll event is less reliably discoverable.

**Image sitemaps** can list images associated with a page where image discovery genuinely matters — a jewellery catalogue is a legitimate case. Next.js supports an `images` property per sitemap entry; see `nextjs.md`. **Verify Google's current image sitemap guidance before implementing.**

**Performance belongs to `scribeo-performance`** — format, compression, sizing, `srcset` strategy and priority. Do not set image performance rules here.

## Video

- **Real, crawlable page URLs** for video content, rather than videos reachable only inside a modal or carousel.
- **Meaningful surrounding content** — title, description, transcript. A transcript serves accessibility, users and understanding simultaneously.
- **`VideoObject` structured data** where the page genuinely features a video. **Verify current required and recommended properties from Google's documentation before implementing** — this feature's requirements change.
- **Thumbnails** must be at stable, reachable URLs.
- **Video sitemaps** where video is a significant part of the catalogue; Next.js supports a `videos` property per sitemap entry.

**Captions and transcripts are `scribeo-accessibility`'s requirement.** They also help search understand the content — but implement them because they are required, not as an SEO tactic.

**Autoplay, preload strategy and encoding are `scribeo-performance`'s** — see that skill's video guidance. Do not duplicate thresholds here.

## Checklist

- [ ] Meaningful images use real `<img>` elements
- [ ] Image URLs stable, reachable, not robots-blocked
- [ ] Alt text written for users, never keyword-stuffed
- [ ] Decorative images carry `alt=""`
- [ ] Filenames descriptive where practical
- [ ] Images sit in relevant surrounding content
- [ ] Image sitemap only where image discovery genuinely matters
- [ ] Video pages have real URLs and supporting text
- [ ] `VideoObject` only where a video genuinely features, with properties verified
- [ ] Performance and accessibility requirements routed to their owners
