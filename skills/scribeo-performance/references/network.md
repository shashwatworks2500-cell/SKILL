# Network Performance

Load when the delay is in delivery rather than execution.

## Read the waterfall first

`browser_network_requests` before anything else. The waterfall answers three questions faster than any other measurement:

1. **What is on the critical path** — requests that must finish before first paint.
2. **What is serialised that could be parallel** — a request that starts only when another finishes is a dependency chain.
3. **What is large, late, or duplicated.**

```js
() => performance.getEntriesByType("resource")
  .map(r => ({
    name: r.name.split("/").pop().slice(0, 40),
    type: r.initiatorType,
    startMs: Math.round(r.startTime),
    durMs: Math.round(r.duration),
    kb: Math.round((r.transferSize || 0) / 1024),
    cached: r.transferSize === 0 && r.decodedBodySize > 0,
  }))
  .sort((a, b) => b.kb - a.kb).slice(0, 25)
```

Look for: the largest transfers · anything starting late that should be early · requests to unexpected origins · the same URL fetched twice · `cached: false` on assets that should be cached.

## TTFB

Comprises redirect time, service worker startup, DNS lookup, connection and TLS negotiation, and request time up to the first response byte. **Guidance: ≤ 0.8s good, 0.8–1.8s needs improvement, > 1.8s poor — and it is explicitly not a Core Web Vital**, so treat it as a diagnostic rather than a target to satisfy.

Split it before fixing it (`measurement.md` gives the navigation-timing breakdown):

| Dominant phase | Cause | Fix |
| --- | --- | --- |
| Redirect | Chained redirects, http→https→www | Collapse to one hop |
| DNS | Cold lookup, many origins | Fewer origins; `preconnect` for critical ones |
| Connection / TLS | Distance to origin, no session reuse | CDN / edge |
| Request time | Slow server work, cold start, uncached render | Caching, static generation, warm paths |

**A poor TTFB caps everything.** No amount of asset optimization fixes a 2s wait for the first byte, and attempting it is the classic wasted audit.

## The critical path

Only three things block first render: HTML, render-blocking CSS, and synchronous scripts in `<head>`.

- **CSS is render-blocking by default.** Large stylesheets delay every paint. Inline the minimum needed for above-the-fold, load the rest without blocking.
- **Synchronous `<script>` in `<head>` blocks parsing.** Use `defer` for anything that does not need to run before render; `async` only where execution order genuinely does not matter.
- **`@import` in CSS serialises requests** — the browser must fetch and parse the first file before discovering the second. Never on a critical path.

## Priority hints — each needs a reason

**Every hint is a claim about relative importance, and the browser's defaults are usually right.** Hints that contradict them make things worse. Indiscriminate `preload` is a common anti-pattern: preloading ten resources means none is prioritised.

| Hint | Use | Cost of misuse |
| --- | --- | --- |
| `fetchpriority="high"` | The LCP image | Demotes something that mattered more |
| `preload` | A late-discovered critical resource — a font for above-the-fold text, a CSS background LCP image | Competes with the real LCP resource |
| `preconnect` | One or two critical third-party origins | Each costs a speculative connection |
| `dns-prefetch` | Origins used later, cheaper than preconnect | Minor |
| `prefetch` | A resource for the *next* likely navigation | Wasted bandwidth if the guess is wrong |

Rule: **add one hint, measure, keep it only if the metric moved.** A preload that does not improve a measured number is a regression in bandwidth.

## Caching

The highest-leverage and most-neglected lever. Check response headers; do not assume.

- **Immutable, content-hashed assets** (JS, CSS, images with a hash in the filename): long max-age, `immutable`. These never need revalidation.
- **HTML**: short or no-cache with revalidation, so deploys are picked up.
- **Fonts**: long-lived — they rarely change.
- A repeat visit re-downloading hashed assets means the headers are wrong. That is a one-line fix with a large effect.

## Compression

Text assets — HTML, CSS, JS, SVG, JSON — must be compressed in transit. Brotli generally beats gzip for these. **Already-compressed binaries (images, video, WOFF2) gain nothing and should not be re-compressed.**

Verify from the response, not the config: an uncompressed `content-length` on a large JS file is an immediate finding.

## Origins and third parties

Each additional origin costs DNS, connection, and TLS on first use.

- Consolidate where possible; self-host fonts and small third-party scripts where licensing allows.
- Measure each third-party origin's total contribution — bytes, time, and requests it triggers. Report them individually; aggregate numbers hide the offender.
- Third-party requests initiated by other third parties are the usual surprise. Follow the chain.

## Frequent findings

| Finding | Fix |
| --- | --- |
| Redirect chain before the document | Collapse to one hop |
| Large render-blocking CSS | Inline critical, defer the rest |
| Synchronous script in `<head>` | `defer` |
| `@import` on the critical path | Direct `<link>` |
| Everything preloaded | Keep only measured wins |
| Hashed assets re-downloaded | Long-lived cache headers |
| Uncompressed text assets | Enable Brotli/gzip |
| Same asset from two origins | Consolidate |
| Font from a third party on the critical path | Self-host, or `preconnect` |
| Many small serialised requests | Bundle, or parallelise |
