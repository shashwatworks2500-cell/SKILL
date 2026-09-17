# Motion Performance

Load for compositing, layout thrashing, frame budget, and diagnosing jank.

This covers **motion-specific** performance. Page-level budgets, Core Web Vitals, bundle size, and general profiling belong to `scribeo-performance`.

## The frame budget

At 60fps a frame is **16.7ms**, and the browser needs part of that for its own work. Treat **~10ms** as the usable budget for everything triggered by one frame. On a throttled mobile CPU, assume a third of that.

Miss the budget and a frame is dropped. Dropped frames during continuous motion are what users describe as "jittery", "juddering", or "not smooth".

## The property hierarchy

What you animate matters far more than how you animate it.

| Cost | Properties | Pipeline stages |
| --- | --- | --- |
| **Cheap** | `transform`, `opacity` | Composite only |
| **Moderate** | `color`, `background-color`, `border-color` | Paint + composite |
| **Expensive** | `filter`, `box-shadow`, `backdrop-filter`, `border-radius` | Paint (often large area) + composite |
| **Worst** | `width`, `height`, `top`, `left`, `margin`, `padding`, `font-size` | **Layout** + paint + composite |

`transform` and `opacity` can run on the compositor, off the main thread. Layout-triggering properties force the browser to recalculate geometry — for the animated element and potentially everything around it — every frame.

```css
/* Wrong: layout on every frame, for the element and its siblings */
.panel { transition: left 400ms ease, height 400ms ease; }

/* Right: composited */
.panel { transition: transform 400ms var(--ease-out); }
```

For genuine size and position change, use FLIP (animate a transform that *looks* like the layout change) or Motion's `layout` prop.

## Layout thrashing

Reading a geometry property after writing to the DOM forces a synchronous layout. Do it in a loop and you get one forced layout per element.

```js
/* Wrong: read → write → read → write, forcing layout each iteration */
items.forEach((el) => {
  const top = el.getBoundingClientRect().top;   // read (forces layout)
  el.style.transform = `translateY(${top * 0.1}px)`;  // write (invalidates)
});

/* Right: batch all reads, then all writes */
const tops = items.map((el) => el.getBoundingClientRect().top);
items.forEach((el, i) => { el.style.transform = `translateY(${tops[i] * 0.1}px)`; });
```

Geometry reads that force layout include `offsetTop`, `offsetHeight`, `scrollTop`, `getBoundingClientRect()`, and `getComputedStyle()`.

GSAP batches its own writes. The thrash usually comes from custom scroll handlers doing measurement per frame — cache measurements and recalculate on resize, not on scroll.

## Scroll handlers

- **Never attach a raw `scroll` listener that does layout work.** Scroll events fire faster than frames.
- Route per-frame work through one loop — `gsap.ticker` if GSAP is present — so there is a single `requestAnimationFrame` driver rather than several competing ones.
- Use `IntersectionObserver` for "is it visible" rather than measuring on scroll. It is designed for this and runs off the main thread.
- Mark passive listeners (`{ passive: true }`) where you do not call `preventDefault`, so scrolling is not blocked.

## Element count

Simultaneous animations, not animation complexity, is usually the limit.

- A handful of composited elements is free. Dozens of simultaneously animating elements will drop frames, especially on mobile.
- Animate a **container** instead of many children where the effect allows.
- For long lists, animate only what is visible. A 200-item staggered grid is a defect regardless of implementation.
- Each ScrollTrigger carries measurement and per-scroll cost — one per section, not one per card. See `gsap.md`.

## will-change and compositing

`will-change: transform` promotes an element to its own compositor layer before animation. Used carelessly it makes things worse.

- Apply to the few elements that genuinely animate, and **remove it when the animation finishes**.
- Never `will-change: transform` on a broad selector. Each layer consumes GPU memory; hundreds of layers is slower than none.
- GSAP applies its own optimisations; adding `will-change` on top is usually unnecessary.

## Expensive effects in motion

- **`filter: blur()`** — the most common cause of mobile motion jank. Blurring a large area every frame is very costly. Animate a pre-blurred element's opacity instead.
- **`backdrop-filter`** — worse than `filter`, because it samples what is behind it. Avoid animating entirely.
- **Large `box-shadow`** — animate a shadow pseudo-element's opacity rather than the shadow itself.
- **Canvas and WebGL** — a separate budget. Cap device pixel ratio, throttle when off-screen, and never run a full-resolution loop on mobile. Whether it belongs on the page at all is a `scribeo-performance` question.
- **High-resolution video seeking** — expensive and unreliable on iOS. See `scroll.md`.

## Diagnosing jank

**DevTools → Performance**, record while reproducing the motion:

1. **Check the frame track.** Red bars are dropped frames. Note whether they cluster at the start (init cost) or persist (per-frame cost).
2. **Look for purple Layout blocks during animation.** Any layout inside an animation loop means a layout-triggering property or a forced sync read.
3. **Look for long tasks** (>50ms). These block input as well as animation.
4. **Check the main thread vs compositor.** Composited animation shows little main-thread work; if the main thread is saturated, you are animating the wrong property.
5. **Throttle the CPU 4–6×** to surface what mobile users experience.

Also useful: **Rendering** panel → *Paint flashing* shows repaints (a composited animation should not repaint), and *Frame rendering stats* gives a live FPS read.

## Triage

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Judder throughout the animation | Layout-triggering property | Move to `transform`/`opacity` |
| Stutter only at start | Init cost, layer promotion, or asset decode | Warm up, pre-decode, stagger init |
| Scroll jank specifically | Work in a scroll handler | Move to ticker/IO; cache measurements |
| Fine on desktop, bad on mobile | Paint-heavy effect or element count | Remove blur/shadow motion; animate fewer nodes |
| Gets worse over time | Leaked animations or listeners accumulating | Audit teardown — see `react.md` |
| Input lags during animation | Long tasks on the main thread | Break up work; move off the hot path |

## Checklist

- [ ] Hot-path animation limited to `transform` and `opacity`
- [ ] No layout-triggering property animated
- [ ] DOM reads batched separately from writes
- [ ] No raw scroll listener doing measurement
- [ ] One `requestAnimationFrame` driver, not several
- [ ] Simultaneous animated element count deliberately bounded
- [ ] `will-change` narrow and removed after use
- [ ] No animated blur, backdrop-filter, or large shadow on mobile
- [ ] Profiled with CPU throttling, frame track clean
