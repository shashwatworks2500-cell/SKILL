# Animation QA

Load when verifying that motion renders and resolves correctly.

**This skill verifies motion output. `scribeo-motion` designs and implements it.** Report what the animation actually does; never prescribe what it should look like, and never adjust timing or easing here. If motion is technically working but feels wrong, that is a motion or aesthetic judgement — route it, do not file it as a defect.

## What to verify

Motion QA is mostly about **states**, not the movement between them. The defects that matter are stuck, missing, and wrong-end-state.

| State | Verify |
| --- | --- |
| **Initial** | Correct pre-animation appearance. Content not permanently invisible if JS fails |
| **Entering** | Animation actually starts; nothing flashes unstyled first |
| **Active / mid** | Content readable during motion; nothing overlaps or clips |
| **Completed** | **The most important one.** Final state is exactly correct — no residual transform, opacity, or offset |
| **Hover / focus / press** | State changes visibly; reverses cleanly; focus ring intact |
| **Scroll-linked** | Progress tracks scroll at several points; correct at 0% and 100% |
| **Pinned** | Pin engages and releases at the right positions; nothing stranded |
| **Route transition** | Outgoing and incoming states both resolve; nothing persists across navigation |
| **Reduced motion** | Content fully visible and usable; state changes still communicated |

**The completed state is where real defects hide.** A stuck `opacity: 0.98` or an un-cleared `translateY` is invisible in motion and obvious in a static capture.

## Sampling animation states

Screenshots capture one instant, so the instant must be chosen deliberately.

**Wait for completion** — the default for verifying end state:

```
browser_navigate → browser_wait_for → browser_take_screenshot
```

**Freeze via reduced motion** — the most reliable determinism lever, and it verifies a required path at the same time. Emulate `prefers-reduced-motion: reduce` and capture; content should be fully visible with no travel.

**Sample scroll-linked progress** at fixed positions rather than by scrolling and hoping:

```js
() => { window.scrollTo(0, document.body.scrollHeight * 0.5); return window.scrollY; }
```

Capture at 0%, 25%, 50%, 75%, 100% and confirm progress is monotonic and the endpoints are exact.

**Check for residue** after everything settles — the highest-yield single probe in animation QA:

```js
() => [...document.querySelectorAll("[style*='transform'], [style*='opacity']")]
  .slice(0, 20)
  .map(el => `${el.tagName}.${el.className}: ${el.getAttribute("style")}`)
```

Leftover inline transforms or a non-`1` opacity after completion is a defect — usually a cleanup or interruption bug for `scribeo-motion`.

## Common motion defects

| Symptom | What to capture | Likely cause (route, don't fix) |
| --- | --- | --- |
| Element never appears | Screenshot + console errors | Trigger never fired, or init failed |
| Stuck mid-animation | Screenshot + inline style dump | Interrupted timeline, or missing cleanup |
| Content invisible with JS disabled or failing | Screenshot with JS blocked | Hidden state set in base CSS |
| Animates every scroll-by | Video or repeated captures | Missing `once` |
| Flash of unstyled or un-positioned content | Capture immediately after navigation | Initial state set too late |
| Wrong final position | Screenshot + computed transform | Residual transform not cleared |
| Fires twice / doubled motion | Console + repeated captures | Duplicate initialisation |
| Breaks after resize | Capture before and after resize | Values measured once |
| Scroll feels stuck | Scroll-position samples | Pinning, or a scroll system conflict |
| Janky or dropped frames | Note it; do not profile | Route to `scribeo-performance` |

## Reduced motion

Always verify this path — it is a required rendering mode, not an edge case.

- All content visible and readable
- No element left hidden because its reveal never ran
- State changes still communicated
- Smooth-scroll overrides disabled
- No horizontal or pinned scroll behaviour that depends on animation

A page that is fine normally and has invisible sections under reduced motion is a **Critical** defect: content is inaccessible.

## Recording motion

For defects that only exist in movement, a still cannot show it. `browser_start_video` / `browser_stop_video`, or tracing, capture the sequence. Use them only when a still genuinely cannot carry the evidence — video is heavier to review and harder to compare.

## Do not

- Do not adjust timing, easing, duration, or choreography — that is `scribeo-motion`.
- Do not judge whether motion is tasteful — that is `frontend-design`.
- Do not profile frame rate or diagnose jank causes — that is `scribeo-performance`; report the symptom.
- Do not file "the animation feels slow" as a defect. File "the entrance takes 1.4s, during which the CTA is not clickable" — that is impact.
