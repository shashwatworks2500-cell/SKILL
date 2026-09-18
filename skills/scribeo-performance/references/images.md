# Image Performance

Load when images are the measured bottleneck.

Images are usually the largest share of page weight and the most common LCP element — and the cheapest thing to fix well.

## Diagnose

```js
() => [...document.images].map(i => ({
  src: (i.currentSrc || i.src).split("/").pop(),
  natural: i.naturalWidth + "×" + i.naturalHeight,
  rendered: Math.round(i.getBoundingClientRect().width) + "×" + Math.round(i.getBoundingClientRect().height),
  ratio: +(i.naturalWidth / Math.max(1, i.getBoundingClientRect().width)).toFixed(2),
  loading: i.loading, fetchPriority: i.fetchPriority,
}))
```

**`ratio` is the finding.** Anything much above the device pixel ratio is wasted bytes — a 3200px image in a 400px box on a 2× screen is ~4× the data needed. Pair with `browser_network_requests` for transferred size.

## Sizing

**Serve close to the rendered size × device pixel ratio.** Oversized source assets are the most common image defect and the easiest win.

```html
<img src="hero-800.jpg"
     srcset="hero-400.jpg 400w, hero-800.jpg 800w, hero-1600.jpg 1600w"
     sizes="(max-width: 768px) 100vw, 50vw"
     width="1600" height="900" alt="…">
```

- **`sizes` describes the layout, not the image.** Getting it wrong is silent: the browser picks a candidate for a box size that does not exist, usually a larger one. Verify with `currentSrc` at each breakpoint.
- **Always set `width` and `height`** (or `aspect-ratio`). This reserves space and prevents CLS. Non-negotiable.
- Cap density sensibly — beyond ~2× the returns are invisible and the bytes are not.

## Formats

Modern formats are substantially smaller than JPEG at equal quality; AVIF typically beats WebP, at higher encode cost. **Verify current browser support before relying on one alone**, and serve with fallbacks:

```html
<picture>
  <source type="image/avif" srcset="hero.avif">
  <source type="image/webp" srcset="hero.webp">
  <img src="hero.jpg" width="1600" height="900" alt="…">
</picture>
```

- **Photographs:** AVIF or WebP, lossy.
- **Flat graphics, logos, icons:** SVG, optimized. Almost always smaller and resolution-independent.
- **Screenshots, sharp edges, text:** PNG or lossless WebP; lossy formats produce visible artefacts on text.
- **Never a video codec for a still**, and never a still sequence where one image suffices.

## Quality

**Do not optimize to a byte target at the cost of visible quality.** On a premium build, visible compression artefacts are a worse defect than the bytes they save.

Find the threshold per asset class: compress progressively until a difference is visible at the rendered size on a good screen, then step back one notch. Photographs tolerate more compression than flat colour or gradients, which band. **Route the quality judgement to `scribeo-visual-qa`** for verification — this skill measures bytes, not acceptability.

## Loading priority

| Case | Attribute |
| --- | --- |
| The LCP image / above the fold | `fetchpriority="high"`, **no** `loading="lazy"` |
| Below the fold | `loading="lazy"` |
| Decorative, non-blocking | `loading="lazy"` + `decoding="async"` |

**`loading="lazy"` on the LCP image is a self-inflicted LCP failure** and one of the most common findings in a first audit. It delays discovery of the one image that must load immediately.

Lazy-load everything below the fold, but check the threshold: images just below the fold that load only after a scroll produce a visible pop-in. That is a `scribeo-visual-qa` observation.

## CSS background images

Background images are **discovered late** — the browser must fetch and parse CSS, build the tree, and resolve the rule before it starts the request. For an LCP candidate this is often the entire problem.

If the LCP element is a CSS background image, the fix is usually to make it a real `<img>` so it is discoverable in the HTML, or to add a justified `preload`. See `network.md`.

## Frequent findings

| Finding | Fix |
| --- | --- |
| Source far larger than rendered box | Resize; add `srcset` |
| `sizes` not matching layout | Correct it; verify `currentSrc` per breakpoint |
| LCP image lazy-loaded | Remove `loading="lazy"`, add `fetchpriority="high"` |
| No `width`/`height` | Add them — fixes CLS |
| JPEG/PNG where AVIF/WebP would serve | Convert with `<picture>` fallbacks |
| Flat graphic as PNG | Convert to optimized SVG |
| Background image as LCP | Promote to `<img>`, or preload |
| Full-size image behind a thumbnail | Serve a thumbnail |
| Same image re-requested per route | Caching headers — see `network.md` |
