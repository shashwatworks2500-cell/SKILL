# Video & Hero Media Performance

Load when video is on the page and performance is in question. This is the highest-stakes asset class on a Scribeo build: hero video is often both the design centrepiece and the largest single cost.

**Start from this position: the video is intentional.** It was chosen by `frontend-design` for a reason. This skill measures its cost precisely and presents options with trade-offs. It does not delete the creative decision to win a metric.

## Diagnose first

```js
() => [...document.querySelectorAll("video")].map(v => ({
  src: (v.currentSrc || "").split("/").pop(),
  preload: v.preload, autoplay: v.autoplay, muted: v.muted, loop: v.loop,
  poster: !!v.getAttribute("poster"),
  dimensions: v.videoWidth + "×" + v.videoHeight,
  rendered: Math.round(v.getBoundingClientRect().width) + "×" + Math.round(v.getBoundingClientRect().height),
  readyState: v.readyState, duration: v.duration,
}))
```

Pair with `browser_network_requests` for transferred bytes and whether the request is on the critical path. Then answer three questions:

1. **Is the video the LCP element, or is it delaying a different LCP candidate?** A video competing for bandwidth can delay a poster or headline without being the candidate itself.
2. **How many bytes land before first paint?** This is the number that matters, not the file size.
3. **Does the user see the video at all before they scroll?** Frequently it is below the fold and loading eagerly for nothing.

## The controlling decision: what loads immediately

| Case | Strategy |
| --- | --- |
| Video **is** the hero and plays immediately | Poster paints first as the LCP candidate; video streams behind it |
| Video is the hero but decorative | `preload="none"` or `metadata`; poster carries the first view |
| Video below the fold | Never eager. Load on approach or interaction |
| Video is the content (showreel, case study) | Click to play, poster only until then |
| Mobile | Usually a separate smaller asset, or poster only |

**A poster image is almost always the right first paint.** It is a fraction of the bytes, paints far sooner, and is a legitimate LCP candidate. The visual result on first view can be identical.

```html
<video poster="hero-poster.avif" preload="none" muted playsinline
       width="1920" height="1080">
  <source src="hero-mobile.mp4" media="(max-width: 767px)" type="video/mp4">
  <source src="hero.mp4" type="video/mp4">
</video>
```

**The poster must match the video's first frame**, or there is a visible jump when playback starts. Extract it from the video rather than choosing a different frame — and route the check to `scribeo-visual-qa`.

## preload

| Value | Behaviour | When |
| --- | --- | --- |
| `none` | Nothing until play | Default for anything not playing immediately |
| `metadata` | Dimensions and duration only | Need duration up front; no immediate play |
| `auto` | Browser may fetch the whole file | Rare. Only a short, immediately-playing, essential video |

**`preload="auto"` on a large hero is the most expensive single default in media-heavy sites.** It can pull megabytes onto the critical path before anything paints.

## Encoding

Cost is driven by resolution, bitrate, duration, and codec — not by "video" as a category.

- **Resolution:** match the rendered size, not the source. A full-bleed hero on a 1440px layout does not need 4K; the bitrate cost is quadratic in resolution.
- **Bitrate:** the dominant factor. Re-encoding at an appropriate bitrate frequently cuts file size by most of its weight with no visible difference at the rendered size.
- **Duration:** a looping hero rarely needs more than a few seconds. Trimming is often the largest single win.
- **Codec:** H.264/AVC has the broadest compatibility; modern codecs (HEVC, AV1) compress better at higher encode cost and more variable support. **Verify current playback support for your audience before shipping a single modern codec** — and keep a compatible fallback via multiple `<source>` elements.
- **Audio:** strip the track entirely from a muted decorative video. It is pure waste.

Report measured before/after bytes. Never state a compression figure you did not measure.

## Autoplay

Autoplay requires `muted` (and `playsinline` on iOS) or browsers block it. Beyond mechanics:

- Autoplay means the bytes are always spent, for every visitor, whether or not they look at it.
- It costs battery and CPU continuously, which matters most on the devices least able to afford it.
- Under `prefers-reduced-motion: reduce` it should not autoplay — that is a `scribeo-motion` and `scribeo-accessibility` requirement, and this skill should flag it if absent rather than ignore it.

## Scroll-controlled video

The most expensive video pattern. `scribeo-motion` owns the implementation; this skill measures whether it is viable.

- Seeking demands the whole relevant range be buffered — effectively the entire file for a full-page scrub.
- **Seek performance is poor and throttled on iOS.** Verify on a real device; do not assume.
- Encoding for seeking needs a short keyframe interval, which *increases* file size — a direct conflict with load performance. State the trade-off explicitly.
- An image sequence on a canvas is often smoother to scrub, at the cost of many requests and higher total bytes. Measure both before recommending.
- Mobile frequently cannot do this acceptably. Report that as a measurement, and let `scribeo-motion` and `frontend-design` decide the fallback.

## Mobile

Assume a separate, smaller asset — or none.

- Lower resolution and bitrate; often a shorter loop.
- Consider poster-only on mobile: it can be visually acceptable and removes the cost entirely.
- Data cost is a real user cost on a metered connection, not just a metric.

Serve variants via `<source media="…">`, and verify with `currentSrc` which one is actually selected. A mobile variant that never gets chosen is a common and invisible failure.

## Frequent findings

| Finding | Fix |
| --- | --- |
| `preload="auto"` on a large hero | `none` or `metadata`; poster carries first paint |
| No poster | Extract the first frame; add it |
| Poster ≠ first frame | Re-extract; verify via `scribeo-visual-qa` |
| Desktop asset served to mobile | `<source media>` variant; verify `currentSrc` |
| 4K source in a 1440px layout | Re-encode at the rendered resolution |
| Audio track on a muted video | Strip it |
| Below-fold video loading eagerly | Load on approach or interaction |
| 30s loop where 4s would do | Trim |
| Single modern codec, no fallback | Add a compatible `<source>` |
| Autoplaying under reduced-motion | Flag; route to `scribeo-motion` |
