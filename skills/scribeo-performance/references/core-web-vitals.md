# Core Web Vitals

Load when diagnosing a specific metric. Thresholds verified against current official guidance (web.dev, September 2026); re-verify before quoting them in client-facing work.

All three Core Web Vitals are assessed at the **75th percentile of page loads, segmented across mobile and desktop.** A fast lab run does not mean a threshold is met.

## LCP — Largest Contentful Paint

**Measures** loading: when the largest content element in the viewport renders.

| Good | Needs improvement | Poor |
| --- | --- | --- |
| ≤ 2.5s | 2.5s – 4.0s | > 4.0s |

**Candidate elements:** `<img>`, `<image>` inside `<svg>`, `<video>`, elements with a `url()` background image, and block-level elements containing text.

**Diagnose in this order.** LCP includes unload time from the previous page, connection setup, redirects, and other TTFB delay — so the element is often not the problem.

1. **Identify the LCP element.** Everything downstream depends on this. Guessing here wastes the whole investigation.
2. **Split the time.** How much is TTFB (server/network), how much is waiting to discover and start the resource, how much is downloading it, how much is rendering after it arrives? Each has a different fix.
3. **Fix the dominant part only.**

| Dominant part | Typical cause | Appropriate fix |
| --- | --- | --- |
| TTFB | Slow server, redirect chain, no edge caching, cold function | Caching, CDN, reduce redirects, static generation where suitable |
| Resource discovery delay | Element in CSS background, JS-inserted, or late in the document | Make it discoverable in HTML; `fetchpriority="high"`; a justified `preload` |
| Download duration | Oversized image or video, wrong format, no responsive variant | See `images.md`, `video.md` |
| Render delay after arrival | Render-blocking CSS/JS, font blocking text paint | See `network.md`, `fonts.md` |

**Frequent own-goals:** `loading="lazy"` on the LCP image (delays the one image that must be immediate) · a hero video where a poster would paint sooner · a `srcset` whose chosen candidate is far larger than the rendered box · client-side rendering of above-the-fold content.

**Do not remove hero media reflexively.** Measure whether it is the bottleneck, then present options — smaller variant, poster-first, deferred load — with the trade-offs. The design decision is `frontend-design`'s.

## INP — Interaction to Next Paint

**Measures** interactivity: the latency of all click, tap, and keyboard interactions across the visit, from input to the next rendered frame.

| Good | Needs improvement | Poor |
| --- | --- | --- |
| ≤ 200ms | 200ms – 500ms | > 500ms |

**INP replaced FID**, which is retired. FID measured only input delay on the first interaction; INP observes every interaction and the full path to the next paint.

**Three phases — measure which one dominates:**

1. **Input delay** — time before handlers run, because the main thread is busy. Cause: long tasks from hydration, third-party scripts, or heavy initialisation.
2. **Processing duration** — the handler callbacks themselves. Cause: expensive work synchronously in the handler, large state updates, heavy DOM reads/writes.
3. **Presentation delay** — from callbacks finishing to the next frame. Cause: large re-render, expensive style recalculation or layout, oversized DOM.

The chain is **interaction → event handler → main-thread work → rendering.** A fix aimed at the wrong link does nothing.

**Common causes:** hydration still running when the user clicks · a handler doing synchronous layout reads · a React re-render of a large tree · third-party scripts occupying the main thread · animation started by the interaction competing for the same frame.

**In lab, INP needs real interaction.** Drive actual clicks and keys; otherwise use TBT and long tasks as the proxy and say that is what you measured.

## CLS — Cumulative Layout Shift

**Measures** visual stability: unexpected layout movement.

| Good | Needs improvement | Poor |
| --- | --- | --- |
| ≤ 0.1 | 0.1 – 0.25 | > 0.25 |

**Score:** each shift is `impact fraction × distance fraction` — the viewport area affected by unstable elements across two frames, times the greatest distance moved relative to the viewport's larger dimension. CLS reports the **worst session window**: shifts within 1s of each other, capped at a 5s window.

Shifts following user input within a short window are excluded (`hadRecentInput`), which is why the score must be read alongside its `sources`.

**Transform animations do not cause layout shift.** Official guidance is explicit: animating with `transform` — including `translate()` and `scale()` — moves elements without triggering layout. Animating `top`, `left`, `width`, `height`, or `margin` changes layout geometry and can shift surrounding content.

This is the single most useful CLS fact: it converts "the animation causes CLS" into a concrete, provable property choice. The implementation change belongs to `scribeo-motion`.

**Causes, in rough order of frequency:**

| Cause | Fix owner |
| --- | --- |
| Images/videos without `width`/`height` or `aspect-ratio` | `scribeo-ux-engineering` (structural) |
| Web font swap changing text metrics | `fonts.md` |
| Late-injected banners, consent bars, ads | Implementation |
| Content rendered after hydration with no reserved space | `scribeo-ux-engineering` |
| Layout-animated elements (`top`/`height`) | `scribeo-motion` |
| Dynamically loaded components above existing content | `scribeo-ux-engineering` |

**Diagnose with `sources`,** not the score. The score says there is a problem; the shifting element names it.

## Reading the three together

- LCP poor + TTFB poor → a server and delivery problem. Asset optimization will not save it.
- LCP poor + TTFB good → a resource discovery, size, or render-blocking problem.
- INP poor + many long tasks → a JavaScript execution problem. See `javascript.md`.
- CLS poor + fonts loading late → a font strategy problem, not a layout one.
- All three fine in lab but field data poor → a device or network reality your lab profile does not represent. Throttle harder.
