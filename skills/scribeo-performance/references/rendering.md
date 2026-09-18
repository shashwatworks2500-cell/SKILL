# Rendering & Main-Thread Performance

Load when the main thread is the bottleneck — long tasks, poor INP, jank.

## The pipeline

**JavaScript → Style → Layout → Paint → Composite.**

Where a change enters the pipeline decides its cost. `transform` and `opacity` can be handled at composite; changing geometry re-runs layout and everything after it, for the element and potentially its surroundings.

## Long tasks

A task over 50ms blocks the main thread: input is not handled, frames are not produced. Long tasks are the direct cause of poor INP.

Measure them (`measurement.md`), then attribute each one in the trace. **Attribution is the whole job** — "there are long tasks" is not a diagnosis; "a 340ms task during hydration of the testimonials Client Component" is.

Common sources, in rough order: hydration of large client trees · third-party script execution · a single oversized `for` loop or data transform · synchronous layout reads in a loop · large re-renders · JSON parsing of an oversized payload.

**Fixes** are structural, not cosmetic: do less work, do it later, do it in smaller pieces, or do it off the main thread. Yield between chunks so input can be handled; move genuinely heavy pure computation to a Worker.

## Forced synchronous layout

Reading a geometry property after a write forces the browser to lay out immediately, before the frame is due. In a loop it happens once per iteration — the classic layout thrash.

```js
/* Wrong: read → write → read → write, one forced layout each pass */
els.forEach(el => { el.style.width = el.offsetWidth + 10 + "px"; });

/* Right: batch reads, then batch writes */
const widths = els.map(el => el.offsetWidth);
els.forEach((el, i) => { el.style.width = widths[i] + 10 + "px"; });
```

Properties that force layout when read include `offsetTop/Left/Width/Height`, `scrollTop`, `clientWidth/Height`, `getBoundingClientRect()`, and `getComputedStyle()`.

In a trace, look for purple layout blocks *inside* a JavaScript task — that is the signature.

## Style recalculation

Cost scales with the number of affected elements × selector complexity.

- **Changing a class on a high ancestor** can invalidate the whole subtree. Scope changes to the smallest node that needs them.
- **Deeply descendant-heavy selectors** cost more to match. Flat, class-based selectors are cheaper.
- **CSS custom properties on `:root`** that many elements consume will invalidate broadly when changed — a real cost in themed or animated systems.
- **`:has()` and complex sibling selectors** are powerful and can be expensive at scale. Measure before using them in a hot path.

## DOM size

A large DOM raises the cost of style, layout, and memory across the board.

- Thousands of nodes is a symptom worth investigating; tens of thousands is a problem regardless of what else is optimized.
- **Virtualise long lists** — one of the few reliably large wins, once the list is genuinely long.
- Watch for accidental duplication: a component rendering its subtree twice, or dev-only wrappers shipped to production.

```js
() => ({
  nodes: document.getElementsByTagName("*").length,
  maxDepth: (function d(n, l = 0) { return n.children.length ? Math.max(...[...n.children].map(c => d(c, l + 1))) : l; })(document.body),
})
```

## Paint cost

Paint is proportional to area × complexity.

- **`filter: blur()`** over a large area is the most expensive common effect, and the usual cause of mobile jank.
- **`backdrop-filter`** is worse — it samples what is behind it every frame.
- **Large or multiple `box-shadow`s**, especially animated, repaint a wide area.
- **`border-radius` with overflow clipping** on large or animated elements adds real cost.
- **Large composited layers** consume GPU memory; hundreds of promoted layers is slower than none.

In the Rendering tools, paint flashing shows what actually repaints. A correctly composited animation should not repaint at all.

## Canvas and WebGL

A separate budget from the DOM.

- Cap device pixel ratio — rendering at 3× on a phone is 9× the fragment work.
- Stop the loop when off-screen or hidden. A `requestAnimationFrame` loop running behind a scrolled-past section is pure waste.
- Measure GPU memory and frame time separately from main-thread time; a page can be main-thread idle and still drop frames.
- Whether the effect belongs on the page at all is a design decision — supply the cost, route the decision.

## Diagnostic procedure

1. **Reproduce** the slow interaction or scroll reliably.
2. **Record** a trace while reproducing it.
3. **Identify** the longest task or the dropped-frame cluster. Work on the largest one only.
4. **Locate** the responsible code or resource from the trace, not by inspection.
5. **Change one variable.**
6. **Re-measure** identically.
7. **Verify** no regression elsewhere — including visually, via `scribeo-visual-qa`.

Repeat from step 3 with the next-largest cost. Stop when the remaining costs are below the threshold of perception or of the budget.

## Frequent findings

| Finding | Fix |
| --- | --- |
| Long task during load | Attribute it; defer, split, or remove the work |
| Layout inside a JS task | Batch reads and writes |
| Class toggled on `<body>` for a local change | Scope it down |
| Animated `filter`/`backdrop-filter` | Animate opacity of a pre-rendered layer |
| Tens of thousands of nodes | Virtualise; remove duplication |
| rAF loop running off-screen | Pause when not visible |
| Jank only on mobile | Paint cost or element count — see `mobile.md` |
| Animation janky but correctly built | Evidence to `scribeo-motion`; see `animation.md` |
