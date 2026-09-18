# Animation Performance

Load when animation or scrolling is the suspected bottleneck.

**The boundary, precisely.** `scribeo-motion` designs and implements animation. This skill **measures and proves** its performance cost. When an animation is correctly built but expensive, supply the evidence, name the bottleneck, and hand the implementation change back to `scribeo-motion`. Never rewrite the animation here, and never adjust its timing or easing.

Also note: `scribeo-visual-qa` may report *"scrolling visibly stutters."* That is a symptom, not a diagnosis. **Deciding whether it is genuinely a performance problem, and quantifying it, is this skill's job.**

## Frame budget

At 60fps a frame is **16.7ms**, and the browser needs part of that itself. Treat **~10ms** as the usable main-thread budget per frame. On a throttled mobile CPU assume a fraction of it.

Exceed it and a frame is dropped. Dropped frames during continuous motion are what users call jank.

Measure frame pacing rather than guessing:

```js
() => new Promise((resolve) => {
  const frames = []; let last = performance.now(); let n = 0;
  const tick = (t) => { frames.push(t - last); last = t;
    if (++n < 120) requestAnimationFrame(tick);
    else {
      const sorted = [...frames].sort((a, b) => a - b);
      resolve({
        frames: n,
        medianMs: +sorted[Math.floor(n / 2)].toFixed(2),
        worstMs: +sorted.at(-1).toFixed(2),
        dropped: frames.filter(f => f > 18).length,
      });
    }
  };
  requestAnimationFrame(tick);
})
```

Run this *while* the animation or scroll is happening, or it measures an idle page. `dropped` and `worstMs` are the findings.

## Property cost

| Cost | Properties | Pipeline |
| --- | --- | --- |
| Cheap | `transform`, `opacity` | Composite only |
| Moderate | `color`, `background-color` | Paint + composite |
| Expensive | `filter`, `box-shadow`, `backdrop-filter` | Large-area paint |
| Worst | `width`, `height`, `top`, `left`, `margin` | **Layout** + paint + composite |

**Animating a layout property is the most common animation performance defect** — and per official CLS guidance, `transform`-based animation avoids layout shift entirely while geometry changes can move surrounding content. That makes the property choice both a performance and a CLS finding.

Confirm what is actually animated rather than assuming:

```js
() => [...document.querySelectorAll("*")].flatMap(el => {
  const s = getComputedStyle(el);
  const props = (s.transitionProperty + "," + s.animationName).toLowerCase();
  return /width|height|top|left|margin|padding|filter|box-shadow/.test(props) && s.animationName !== "none" || /width|height|top|left|margin/.test(s.transitionProperty)
    ? [`${el.tagName}.${el.className}: ${s.transitionProperty} / ${s.animationName}`] : [];
}).slice(0, 20)
```

## Scroll performance

Scroll jank has a small set of causes. Check in this order:

1. **Work in a scroll handler.** Scroll events fire faster than frames. Any layout read or DOM write in a raw handler is a per-event forced layout. Should be on a single rAF-driven loop, or replaced by `IntersectionObserver`.
2. **Non-passive listeners.** A listener that could call `preventDefault` blocks scrolling until it runs. `{ passive: true }` where applicable.
3. **Too many scroll-linked animations.** Each scrubbed animation runs work per frame. Dozens of ScrollTriggers is a measurable cost.
4. **Expensive properties in scroll-linked animation.** A scrubbed `filter` repaints every frame.
5. **Pinned sections** with large subtrees — pinning forces layout and can promote large layers.
6. **Smooth-scroll libraries** (e.g. Lenis) add per-frame work and can conflict with other scroll systems. Measure with it disabled to isolate its share.

```js
() => ({ scrollListeners: "inspect via trace", passiveHint: "check listener options in DevTools",
         animatedNodes: document.querySelectorAll("[style*='transform']").length })
```

The count of simultaneously animated nodes is usually the single most predictive number for scroll jank.

## Element count

Simultaneous animations, not animation complexity, is generally the limit.

- A handful of composited elements is effectively free.
- Dozens animating together will drop frames, especially on mobile.
- Animating a container instead of many children is the standard fix — an implementation change for `scribeo-motion`.
- Long staggered lists animating every item are a reliable defect at scale.

## Isolating animation cost

To prove an animation is the problem rather than assuming it:

1. Measure frame pacing with the animation running. Record median, worst, dropped.
2. Disable **only** the animation — via `prefers-reduced-motion`, a feature flag, or removing the class.
3. Re-measure identically.
4. Compare. If frame pacing does not improve, the animation was not the bottleneck and you have saved everyone a pointless rewrite.

`prefers-reduced-motion: reduce` is the cleanest isolation lever, because a well-built site already has that path.

## Reporting to Motion

A useful handoff contains, at minimum:

```
Symptom:     Scroll stutters through the testimonials section, 390px, 4× CPU throttle
Measurement: Median frame 31ms, worst 88ms, 34/120 frames dropped while scrolling
             the section. With prefers-reduced-motion: median 16ms, 0 dropped.
Bottleneck:  18 simultaneously animated cards, each with an animated
             box-shadow (paint-bound, not composite)
Evidence:    trace.json; paint flashing shows full-section repaints per frame
Owner:       scribeo-motion — recommend animating the container, and opacity of a
             pre-rendered shadow layer rather than the shadow itself
```

Evidence, bottleneck, owner. **No prescribed timing, easing, or choreography** — those are Motion's decisions, informed by this measurement.

## Do not

- Do not change animation implementation here. Measure, prove, route.
- Do not report "the animation is slow" without frame numbers and an isolation test.
- Do not recommend deleting an animation. Present the cost and the options; the design decision belongs to `frontend-design` and `scribeo-motion`.
- Do not conflate a visual stutter reported by `scribeo-visual-qa` with a measured performance problem until you have measured it.
