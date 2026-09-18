# JavaScript & Bundle Performance

Load when JavaScript execution or payload is the measured bottleneck.

**Measure before cutting.** Bundle size is the most-optimized and least-often-actually-binding constraint. A 300KB bundle on a page blocked 1.8s at TTFB is not the problem. Confirm that JS is the bottleneck — long tasks, poor INP, delayed hydration — before touching dependencies.

Two costs, frequently confused:

- **Transfer cost** — bytes over the network. Matters on slow connections.
- **Execution cost** — parse, compile, and run on the CPU. Matters on slow devices, and is usually the larger problem. 200KB of framework code costs far more CPU than 200KB of images.

Optimizing transfer when execution is the constraint produces no measurable change.

## Diagnose

1. **Long tasks** — how many, how long, when? (`measurement.md`)
2. **Where does the time go** — script evaluation, hydration, a specific handler? Read the trace.
3. **What is actually shipped** — bundle analysis, per-route.
4. **What is unused** — code loaded but never executed on this route.
5. **Third parties** — measure their share separately; it is often the majority.

## Bundle analysis

Analyse per-route, not in aggregate. A 1MB app where the landing page loads 90KB is fine; a 400KB app where every route loads all of it is not.

Look for, in order of typical payoff:

| Finding | Action |
| --- | --- |
| A single oversized dependency | Replace with a smaller one, or a platform API |
| Duplicate copies of one library | Deduplicate; usually a version mismatch in the tree |
| A whole library imported for one function | Import the function, or inline it |
| Moment/lodash-style full-package imports | Per-method imports, or native equivalents |
| Polyfills for browsers you do not support | Adjust the build target |
| Dev-only code in production | Fix the build |
| A heavy component on a route that rarely needs it | Dynamic import |

**Check the dependency's real cost before adding it** — installed size, whether it tree-shakes, whether it pulls transitive dependencies. A date formatter that adds 70KB to satisfy one call is a bad trade; `Intl.DateTimeFormat` is free.

## Code splitting

Split where the user's path actually diverges:

- **Route level** — the default and highest-value split.
- **Below the fold** — a component the user may never scroll to.
- **Interaction-gated** — modal, video player, map, chart, rich editor. Load on the interaction that needs it.
- **Conditional** — a feature only some users see.

```js
const Player = dynamic(() => import("./Player"), { ssr: false });
```

**Do not over-split.** Many tiny chunks add request overhead and can be slower than one well-sized chunk. Measure both.

**Tree shaking needs help:** ES module imports (not CommonJS), no side-effectful imports, `sideEffects: false` where accurate, and named imports rather than namespace imports.

## Next.js specifics

**Server vs Client Components are the single largest lever** in an App Router project.

- A component is a Server Component by default. `"use client"` opts it — **and its entire import subtree** — into the client bundle.
- **`"use client"` at the top of a layout or page pulls everything beneath it client-side.** This is the most common cause of an unexpectedly large bundle. Check where the boundaries actually sit before optimizing anything else.
- Push the boundary **down** to the smallest interactive leaf. A page with one interactive button should ship that button, not the page.
- Server Components can render Client Components; a Client Component cannot render a Server Component. Structure accordingly.

**Hydration cost** is real and shows up as input delay. Less client JS means less hydration. Static content in a Client Component is pure cost — it hydrates for nothing.

**Do not prescribe a rendering strategy universally.** Static generation, server rendering, streaming, and client rendering each suit different cases:

| Measured problem | Candidate |
| --- | --- |
| TTFB dominated by per-request work on stable content | Static generation, or caching |
| Large content payload delaying first paint | Streaming / progressive rendering |
| Interaction delayed by hydration of non-interactive content | Move it to a Server Component |
| Heavy component blocking initial load | Dynamic import, `ssr: false` if it needs no SSR |

The bottleneck decides. "SSG is faster" is not an argument.

## Third-party scripts

Usually the largest unowned cost, and the least examined.

- **Measure each one separately** — bytes, execution time, main-thread occupation, and requests initiated. Attribute honestly; a tag manager's cost includes everything it injects.
- **Load strategy:** defer anything not needed for first render. Analytics, chat widgets, and consent tooling rarely need to block.
- **Question necessity.** A 150KB chat widget on a page with negligible chat usage is a measured cost against an unmeasured benefit. Present the numbers; the business decision is the client's.
- **Self-host where licensing allows** to remove a connection and gain cache control.
- **Third-party iframes** are cheaper for the main thread than third-party scripts — they cannot block it.

## React execution cost

- **Large re-render trees** are the usual INP culprit. Find them in the trace, not by inspection.
- **Memoisation is not free** — it adds comparison cost and complexity. Apply where a trace shows repeated expensive renders, never pre-emptively.
- **Synchronous work in event handlers** delays the next paint directly. Break it up, or move it off the critical interaction path.
- **List virtualisation** for long lists is one of the few reliably large wins — but only once the list is genuinely long.

## Do not

- Do not cut dependencies without a measurement showing JS is the bottleneck.
- Do not micro-optimize application code when a single dependency or third party dominates.
- Do not remove a feature to save bytes — present the cost and route the decision.
- Do not report a bundle-size reduction as a performance improvement without a metric change to back it.
