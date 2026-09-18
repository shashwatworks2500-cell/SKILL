---
name: scribeo-performance
description: Use when measuring, diagnosing, optimizing, or verifying web performance — Core Web Vitals (LCP, INP, CLS), loading and runtime performance, JavaScript and bundle cost, hydration and client-component cost, image, video and font delivery, network waterfalls and caching, layout shift, long tasks and main-thread contention, scroll and animation frame cost, mobile performance constraints, performance budgets, and performance regression. Triggers on requests like "optimize website performance", "why is this site slow?", "improve Lighthouse", "fix Core Web Vitals", "reduce LCP", "fix CLS", "improve INP", "reduce JavaScript", "reduce bundle size", "fix slow page load", "optimize images", "optimize video", "fix scroll jank", "animation is dropping frames", "find what's slowing this page down", "profile this website", "check performance before launch", "set performance budgets", "find long tasks", "reduce main-thread work", and on diagnostic reports like "the page feels slow", "the page takes too long to load", "scrolling is laggy", "mobile performance is bad", or "the hero is making the page slow". Visual design belongs to frontend-design, layout and UX structure to scribeo-ux-engineering, animation implementation to scribeo-motion, rendered visual verification to scribeo-visual-qa, WCAG auditing to scribeo-accessibility, test architecture to scribeo-testing, and SEO to scribeo-seo. Not triggered by aesthetic requests, and not triggered merely because a page contains animation — only when performance is explicitly asked about or clearly identified as the problem.
---

# Scribeo Performance

Scribeo Studio's performance engineering standard. One loop, in order:

**MEASURE → DIAGNOSE → OPTIMIZE → VERIFY**

Apply this whenever the question is *how fast is it actually, what is actually making it slow, and did the change actually help.*

## The law

**No optimization without a measurement that justifies it.**

"Best practice" is not a reason. Every change must trace to a measured bottleneck on a specific page, at a specific viewport, under a specific network and CPU condition. An optimization applied without measurement is a guess with a diff attached — it costs review time, risks regressions, and usually targets something that was never the constraint.

Four rules follow:

- **Measure first, always.** If you cannot name the bottleneck with evidence, you are not ready to change code.
- **Optimize the actual bottleneck.** Shaving 40KB off a bundle when the page is blocked for 1.8s on TTFB changes nothing a user can feel.
- **Re-measure after every meaningful change.** An unverified improvement is not an improvement.
- **Never fabricate a number.** Every figure in a report is measured, or it is not stated. No estimated metrics, no illustrative values presented as results.

### Never trade these for a score

- **Do not degrade usability or accessibility** to move a metric. A faster page nobody can use is a regression. Anything touching accessibility routes to `scribeo-accessibility`.
- **Do not strip intentional design** to win points. Hero media, custom typefaces, and motion are deliberate decisions owned by `frontend-design` and `scribeo-motion`. Prove the cost with measurement, then propose options with trade-offs — do not unilaterally delete the design.
- **Do not optimize for the synthetic score over the real user.** A 100 in a lab tool and a slow site on a mid-range phone on 4G is a failure.

## Lab, field, and synthetic are three different things

Conflating these is the most common analytical error in performance work.

| Kind | What it is | Good for | Cannot tell you |
| --- | --- | --- | --- |
| **Lab** | A controlled run on one machine, one network profile, one viewport | Reproducible diagnosis, A/B of a change, trace analysis | What real users experience |
| **Field (RUM)** | Real users, real devices, real networks, aggregated at the 75th percentile | Whether the site is actually fast for the audience | Which line of code is responsible |
| **Synthetic score** | A composite number (e.g. a Lighthouse performance score) | A rough smoke signal, trend over time | Anything causal; it is a weighted aggregate, not a measurement |

**A Lighthouse score is not real-world performance.** It is one lab run, on one throttling profile, producing a weighted composite. Report it, if at all, as one lab data point alongside the actual metrics — never as the headline, and never as evidence that users are having a good time.

**Core Web Vitals thresholds are field thresholds, assessed at the 75th percentile of page loads, segmented across mobile and desktop.** A single fast lab run does not mean the threshold is met. Use lab measurement to *diagnose* and field data to *judge*.

## Core Web Vitals

Verified against current official guidance (web.dev, September 2026). Distinguish the **definition**, the **threshold**, and the **diagnostic signals** you use to chase it.

| Metric | Measures | Good | Needs improvement | Poor |
| --- | --- | --- | --- | --- |
| **LCP** — Largest Contentful Paint | Loading: when the largest content element renders | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| **INP** — Interaction to Next Paint | Interactivity: latency across all interactions | ≤ 200ms | 200ms – 500ms | > 500ms |
| **CLS** — Cumulative Layout Shift | Visual stability: unexpected layout movement | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |

All three are assessed at the **75th percentile**, segmented by mobile and desktop.

**INP replaced First Input Delay.** FID is retired; INP became a stable Core Web Vital in 2024. Unlike FID, which measured only input delay on the first interaction, INP observes the latency of all click, tap, and keyboard interactions across the visit.

### Supporting diagnostics — not Core Web Vitals

| Signal | Use | Guidance |
| --- | --- | --- |
| **TTFB** — Time to First Byte | Is the delay before the server responds? Comprises redirect, service worker startup, DNS, connection and TLS, and request time | Good ≤ 0.8s; 0.8–1.8s needs improvement; > 1.8s poor. **Explicitly not a Core Web Vital** — meeting it is not mandatory, it is a diagnostic |
| **FCP** — First Contentful Paint | Did anything paint? Separates server/network delay from render delay | Diagnostic for LCP |
| **TBT** — Total Blocking Time | Lab proxy for main-thread blocking; the practical lab stand-in for INP, which needs real interaction | Lab-only diagnostic |
| **Long tasks** | Tasks blocking the main thread; the direct cause of poor INP | Diagnostic |

Treat TTFB, FCP, TBT, and long tasks as **diagnostic signals that explain a Core Web Vital**, never as goals in themselves.

## Boundaries

**This skill owns:** measuring performance · diagnosing bottlenecks · loading, runtime and rendering performance · Core Web Vitals · JavaScript execution and bundle cost · hydration and client-boundary cost · image, video and font delivery performance · network waterfall and caching · layout shift measurement · long tasks and main-thread contention · animation and scroll frame cost · observable memory and runtime symptoms · performance budgets · performance regression prevention.

**This skill does not own:**

| Concern | Belongs to |
| --- | --- |
| Visual design direction, aesthetics, palette, typeface choice, composition | `frontend-design` (Anthropic) |
| Layout structure, hierarchy, usability, responsive UX decisions | `scribeo-ux-engineering` |
| Animation design and implementation — timing, easing, choreography, scroll linkage | `scribeo-motion` |
| Rendered visual verification, screenshot review, visual defect reporting | `scribeo-visual-qa` |
| WCAG conformance, assistive-technology verification | `scribeo-accessibility` |
| Test architecture, strategy, suite authoring and CI test design | `scribeo-testing` |
| Metadata, structured data, crawlability | `scribeo-seo` |

### Reciprocal handoffs

These are the seams that matter. Each runs in one direction.

- **`scribeo-visual-qa` → here.** QA reports observable symptoms — visible stutter, a long blank paint, layout shift on load, a stalled asset — with evidence and no diagnosis. **Deciding whether that is genuinely a performance problem, and proving it, is this skill's job.** Visual QA never profiles.
- **`scribeo-motion` → here.** Motion implements the animation. When motion is suspected of costing frames, this skill measures and proves it.
- **here → `scribeo-motion`.** When an animation is correctly implemented but expensive, supply the evidence (which property, which frames, which trace), name the bottleneck, and hand the implementation change to `scribeo-motion`. Do not rewrite the animation here.
- **here → `scribeo-visual-qa`.** Every optimization that can change rendering — image format or quality, font strategy, lazy-loading, deferred media — must be visually verified before it is called done. Route it.
- **here → `scribeo-accessibility`.** If an optimization would reduce accessibility or usability, stop and route the decision. Performance never wins that trade by default.
- **here → `scribeo-ux-engineering`.** Reserved space, aspect ratios, and skeletons that prevent layout shift are structural. Report the CLS measurement; the structural fix is theirs.

`scribeo-motion` carries its own motion-specific performance guidance for authoring-time decisions. **This skill owns page-level measurement, budgets, and profiling** — including of animation.

## Workflow

**Phase 1 — Establish ground truth**

1. **Establish the target.** What must be fast, for whom, on what device and network? "Fast" without a target is unmeasurable.
2. **Establish the baseline.** Measure before touching anything. Record route, viewport, device profile, network profile, and the metric values. Without a baseline there is nothing to compare to and no way to prove an improvement.
3. **Reproduce the problem.** If you cannot reproduce it, you cannot diagnose it. A symptom that appears once is not yet a finding.

**Phase 2 — Diagnose**

4. **Measure** with the appropriate tool for the question — see `references/measurement.md`.
5. **Identify the bottleneck.** One bottleneck, named, with evidence. Not a list of everything that could theoretically be faster.
6. **Form a hypothesis.** "LCP is 4.1s because the hero video is 6MB on the critical path and delays the LCP candidate" is a hypothesis. "The page has too much JavaScript" is not.

**Phase 3 — Optimize**

7. **Make the smallest meaningful change** that tests the hypothesis. One variable. Bundling five optimizations means you learn nothing about which one worked.

**Phase 4 — Verify**

8. **Measure again**, identically — same route, viewport, device and network profile.
9. **Compare against the baseline.** State the delta. If it did not move, the hypothesis was wrong: revert and go back to step 5.
10. **Check for visual and UX regressions.** Route to `scribeo-visual-qa`.
11. **Check mobile explicitly.** Desktop improvement is not mobile improvement.
12. **Record the result** in the report format below — including changes that did not help. A negative result is a finding and stops someone repeating it.

## Measurement tooling in this environment

Verified in this session, not assumed:

| Tool | Status | Use |
| --- | --- | --- |
| **Playwright MCP** | ✅ Configured and connected | Browser-driven lab measurement: navigation, network requests, console, `performance` API via evaluate, tracing |
| **Chromium 141.0.7390.37** | ✅ Installed at `/opt/pw-browsers/chromium` | The rendering and trace engine |
| MCP tracing | ✅ `browser_start_tracing` / `browser_stop_tracing` | Capturing a performance trace |
| MCP network | ✅ `browser_network_requests`, `browser_network_request` | Waterfall, payload sizes, status |
| Browser performance APIs | ✅ via `browser_evaluate` | `PerformanceObserver`, `performance.getEntriesByType`, Navigation and Resource Timing |
| **Lighthouse** | ❌ **Not installed** (`lighthouse` not on PATH; npm `lighthouse` 13.4.1, Apache-2.0, is available to add) | Composite lab audit — say it is unavailable rather than inventing output |
| **Field / RUM data** | ❌ Not available in this environment | Requires the `web-vitals` library (npm, 6.2.2, Apache-2.0) wired into the site, or a provider |
| DevTools UI | ❌ Headless container, no interactive UI | Trace files can still be captured and analysed |

**Never invent tool output.** If Lighthouse is not installed, the honest answer is "not measured, and here is what I can measure instead" — not a fabricated score. Adding tooling is a dependency decision for the user, not something to assume.

## Performance report format

Every finding gets these fields. Measured values only.

```
Route:          /
Environment:    Lab — Playwright MCP, Chromium 141, headless
Device/viewport: 390 × 844, CPU throttle 4×
Network:        Simulated Slow 4G
Metric:         LCP
Baseline:       4.12s (LCP element: <video> hero poster)
Observed:       Hero video 6.2MB begins downloading before the poster paints;
                LCP candidate blocked on network
Bottleneck:     Hero video on the critical path
Evidence:       network-waterfall.json, trace.json, LCP entry via PerformanceObserver
Change:         preload="none" on the video; poster promoted with fetchpriority="high"
After:          LCP 1.84s (−2.28s)
Regression:     Visual verified by scribeo-visual-qa — poster frame matches
                first video frame; no CLS change (0.02 → 0.02)
Owner:          Implementation (media delivery); motion behaviour unchanged
```

```
Bad:   "Performance is bad."
Good:  "At 390px under the Slow 4G profile, the hero video contributes
        6.2MB to the critical load path and delays the LCP candidate;
        deferring it to the existing poster reduced measured LCP from
        4.12s to 1.84s."
```

State the environment with every number. A metric without its device, network, and viewport is not reproducible and not comparable.

## Reference routing

`SKILL.md` is the decision core. Load a reference when the investigation reaches that surface.

| Load | When |
| --- | --- |
| `references/measurement.md` | Choosing a tool, capturing traces, reading the performance APIs |
| `references/core-web-vitals.md` | Diagnosing a specific metric — LCP, INP, CLS phases and causes |
| `references/javascript.md` | Bundle size, dependency cost, hydration, client boundaries, code splitting |
| `references/images.md` | Format, sizing, responsive delivery, priority, lazy-loading |
| `references/video.md` | Hero video, codecs, poster strategy, mobile variants, scroll-linked video |
| `references/fonts.md` | Font payload, subsetting, `font-display`, fallback metrics, preload |
| `references/network.md` | Waterfalls, critical path, priority hints, caching, compression, CDN |
| `references/rendering.md` | Long tasks, forced layout, style recalculation, paint cost, DOM size |
| `references/animation.md` | Frame rate, compositing, scroll cost, and the Motion handoff |
| `references/mobile.md` | Mobile CPU, memory, network and GPU constraints; throttled testing |
| `references/budgets.md` | Deriving, setting and enforcing budgets; regression prevention |

## Quality gate

Do not report performance work complete until every line holds.

**Evidence**
- [ ] A baseline was measured and recorded before any change
- [ ] Every claim traces to a measured value, with environment stated
- [ ] The bottleneck is named, singular, and evidenced
- [ ] No fabricated, estimated, or illustrative numbers presented as results
- [ ] Unavailable tooling stated as unavailable, not simulated

**Method**
- [ ] One variable changed per measurement cycle
- [ ] Re-measured under identical conditions after each change
- [ ] Delta stated against baseline, including changes that did not help
- [ ] Hypotheses that failed are recorded, not silently dropped

**Safety**
- [ ] No usability or accessibility degraded for a metric
- [ ] No intentional design removed unilaterally — trade-offs presented, decision routed
- [ ] Visual regression verified via `scribeo-visual-qa`
- [ ] Mobile measured explicitly, not inferred from desktop

**Framing**
- [ ] Lab, field and synthetic results labelled as such and never conflated
- [ ] No synthetic score presented as real-user performance
- [ ] Core Web Vitals judged against field data at the 75th percentile where available; lab used for diagnosis

**Measured, not assumed** — if a line could not be verified, say so plainly rather than implying it passed.
