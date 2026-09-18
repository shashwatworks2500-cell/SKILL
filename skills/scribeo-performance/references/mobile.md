# Mobile Performance

Load whenever mobile is a target — which is always.

**Desktop performance is not evidence of mobile performance.** A page that is fast on a developer laptop on office broadband tells you nothing about a mid-range phone on a congested network. Measuring only the former is the most common way an audit misses the real problem.

## The constraints are different in kind

| Constraint | Reality |
| --- | --- |
| **CPU** | Mid-range phones are several times slower than a developer machine. JavaScript execution — parse, compile, run — scales directly with this |
| **Thermal** | Sustained load throttles the CPU further. Performance *degrades over a session*, which a single cold measurement never shows |
| **Memory** | Less headroom; background tabs are evicted. Large DOM trees, big images and canvases are far more expensive |
| **GPU** | Weaker, with high-DPI screens to drive. Paint-heavy effects cost disproportionately more |
| **Network** | Variable latency and throughput; latency often matters more than bandwidth |
| **Data** | Metered connections make bytes a direct user cost, not just a metric |
| **Battery** | Continuous animation, video, and rAF loops drain it. A real cost users feel |

**JavaScript is the constraint that scales worst.** Images get slower linearly with bandwidth; script execution gets slower with CPU *and* is on the critical interaction path. On mobile, cutting JS usually beats cutting bytes.

## Measure the constrained case

Always measure with throttling, and always record the profile with the number.

- **CPU throttling** — 4× as a routine default; harder to represent low-end devices.
- **Network throttling** — a slow-4G-class profile is the realistic baseline for many audiences.
- **Real device** where the finding is marginal, or where the behaviour is device-specific (iOS video seeking, Safari-specific rendering).

`LCP 2.1s` is not a measurement. `LCP 2.1s at 390×844, 4× CPU, Slow 4G, median of 5 runs` is.

**A flagship phone hides the problem your users have.** Test the middle of the market, not the top.

## Mobile-specific findings

| Area | What to check |
| --- | --- |
| **Media** | Is a desktop-sized image or video being served? Verify `currentSrc`, not the markup |
| **JS** | Is the same bundle shipped regardless of device? Execution cost dominates here |
| **Hydration** | Input delay from hydration is far worse on slow CPUs — the usual INP cause on mobile |
| **Paint** | Blur, backdrop-filter, large shadows: disproportionately expensive at high DPI |
| **Animation** | Element count is the limiting factor; scrubbed scroll animation is costlier |
| **Video** | Seek performance is poor and throttled on iOS — verify on a device |
| **Viewport** | `100vh` vs `dvh`; browser chrome show/hide triggers resize work |
| **Touch** | Non-passive touch listeners block scrolling |
| **Fonts** | A metrics mismatch shifts more text on a narrow viewport |
| **Memory** | Long pages with many images; canvases at full DPR |

## Serving less to mobile

Legitimate, and usually the largest win — provided it is a deliberate decision, not a degraded experience.

- **Smaller media variants** via `srcset`/`sizes` and `<source media>`. Verify the intended candidate is actually selected.
- **Poster instead of video**, where visually acceptable. Route the acceptability judgement to `frontend-design` and `scribeo-visual-qa`.
- **Reduced motion or fewer animated elements** on mobile — an implementation change for `scribeo-motion`, informed by your measurement.
- **Deferred non-essential components** until interaction.

**Do not silently ship a worse experience to justify a metric.** Present the trade-off; the design decision is not this skill's.

## Battery and sustained cost

Rarely measured, genuinely felt:

- Autoplaying video decodes continuously for every visitor.
- rAF loops that never pause — including behind scrolled-past sections.
- Scroll-linked animation runs for the whole scroll, not once.
- Polling and unthrottled listeners.

Pausing off-screen work is close to free to implement and removes a cost most audits never look for.

## Frequent findings

| Finding | Fix |
| --- | --- |
| Desktop media served to mobile | Variants; verify `currentSrc` |
| Identical JS bundle for all devices | Split; defer; move work server-side |
| INP fine on desktop, poor on mobile | Hydration and long tasks — see `javascript.md` |
| Jank only on mobile | Paint cost or element count — see `rendering.md`, `animation.md` |
| Autoplay video on mobile | Poster, or load on interaction |
| Animation running off-screen | Pause when not visible |
| `100vh` full-height sections | `dvh` — route to `scribeo-ux-engineering` |
| Only desktop measured | Re-measure throttled; treat prior conclusions as unverified |
