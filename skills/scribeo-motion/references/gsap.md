# GSAP

Load for timelines, cleanup, `matchMedia`, ScrollTrigger configuration, and animation conflicts.

`gsap` 3.15.0 ships under a **standard no-charge license** with plugins including ScrollTrigger included. `@gsap/react` 2.1.2 provides `useGSAP()`. Re-check the license terms before commercial delivery rather than assuming.

Register plugins once, at module scope, never inside a component body:

```js
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);
```

## Timelines over chained tweens

Use a timeline whenever more than one thing moves in sequence. Independent tweens with `delay` values drift, cannot be reversed as a unit, and cannot be interrupted coherently.

```js
const tl = gsap.timeline({ defaults: { ease: "power3.out", duration: 0.6 } });
tl.from(".hero__title", { y: 40, opacity: 0 })
  .from(".hero__sub",   { y: 24, opacity: 0 }, "-=0.35")
  .from(".hero__cta",   { y: 16, opacity: 0 }, "-=0.3");
```

- Put shared values in `defaults` — repeating `ease` and `duration` per tween is where inconsistency creeps in.
- Use **relative position parameters** (`"-=0.35"`, `"<"`, `">"`) so retiming one step does not require recalculating every delay.
- **`from()` sets the start state at init.** If the element must be invisible before the timeline runs, also set it in CSS, or a flash of unstyled content appears before JS executes. `fromTo()` is more explicit and usually safer.

## Cleanup — the thing that actually breaks

Every GSAP animation, ScrollTrigger, and listener must be destroyed with the component. This is the root cause of double-firing, resize breakage, and post-navigation leaks.

**In React, use `useGSAP()`.** It wraps `gsap.context()` and reverts everything created inside it on unmount:

```jsx
import { useRef } from "react";
import { useGSAP } from "@gsap/react";

function Hero() {
  const root = useRef(null);
  useGSAP(() => {
    // every animation and ScrollTrigger created here is auto-reverted
    gsap.from(".hero__title", { y: 40, opacity: 0, duration: 0.6 });
  }, { scope: root });
  return <section ref={root}>…</section>;
}
```

`scope` also scopes the selector strings to that subtree — which prevents a second instance of the component animating the first one's elements.

**Outside React**, do it by hand:

```js
const ctx = gsap.context(() => { /* animations, ScrollTriggers */ }, rootEl);
// teardown
ctx.revert();
```

```js
/* Wrong — leaks on every unmount, fires again on remount */
useEffect(() => { gsap.to(el, { x: 100 }); }, []);

/* Right */
useGSAP(() => { gsap.to(el, { x: 100 }); }, { scope: root });
```

## matchMedia for responsive motion

Do not read `window.innerWidth` and branch. `gsap.matchMedia()` creates and reverts per breakpoint automatically as the viewport crosses it:

```js
const mm = gsap.matchMedia();

mm.add("(min-width: 768px)", () => {
  const tl = gsap.timeline({ scrollTrigger: { trigger: ".panel", scrub: true, pin: true } });
  tl.to(".panel__inner", { xPercent: -60 });
  // returned cleanup is optional; matchMedia reverts what it created
});

mm.add("(max-width: 767px)", () => {
  gsap.from(".panel__inner", { opacity: 0, y: 16, scrollTrigger: { trigger: ".panel", start: "top 80%" } });
});

mm.add("(prefers-reduced-motion: reduce)", () => {
  gsap.set(".panel__inner", { opacity: 1, clearProps: "transform" });
});
```

This is also the correct place to drop pinning and scrubbing on touch — see `responsive-motion.md`.

## ScrollTrigger essentials

```js
ScrollTrigger.create({
  trigger: ".section",
  start: "top 80%",        // trigger top hits 80% down the viewport
  end: "bottom 20%",
  toggleActions: "play none none reverse",
  scrub: 0.5,              // number = smoothing lag in seconds
  pin: false,
  invalidateOnRefresh: true,
  markers: false,          // never commit markers: true
});
```

- **`toggleActions`** is `onEnter onLeave onEnterBack onLeaveBack`. `"play none none none"` for a once-only reveal; `"play none none reverse"` to undo on scroll-back.
- **`once: true`** for entrance reveals that should never replay. Cheaper than keeping the trigger alive.
- **`scrub: true`** locks progress to scroll exactly; a number (0.3–1) adds smoothing lag and usually feels better.
- **`invalidateOnRefresh: true`** recalculates recorded start values on refresh — essential when sizes depend on layout or fonts.
- **Never ship `markers: true`.**

**Refresh after layout changes.** ScrollTrigger measures at creation. Fonts loading, images arriving, accordions opening, or route content changing all invalidate those measurements:

```js
await document.fonts.ready;
ScrollTrigger.refresh();
```

Resize is handled automatically; content changes are not.

## Pinning

Pinning is the most expensive ScrollTrigger feature and the easiest to misuse.

- Pin only with a genuine narrative reason — a sequence the user must move through in order.
- **Never pin more than about one viewport of content.** Longer pins strand the user in a section where scrolling appears to do nothing.
- Pin a wrapper, not the animated element. GSAP wraps pinned elements; animating the same node fights its transforms.
- `pinSpacing` defaults sensibly; changing it usually means the layout is wrong.
- **Never pin on touch** unless verified on a real device. See `responsive-motion.md`.

## Conflicts

**One system owns each property per element.** The usual collisions:

- A CSS `transition` on `transform` while GSAP animates `transform` — the transition fights every frame GSAP sets. Remove the transition.
- Two timelines both animating `opacity` on the same node — the later one wins unpredictably. Merge them.
- Motion and GSAP on the same element — pick one.

```css
/* Wrong: CSS transition on a property GSAP drives */
.card { transition: transform 300ms ease; }
```

Use `gsap.killTweensOf(el)` before creating a replacement tween on the same target, or reuse one timeline and `.restart()` / `.reverse()` it. Creating a fresh tween on every event is what makes interactions accumulate and stutter.

## Too many triggers

Each ScrollTrigger costs measurement and per-scroll work. Fifty reveal triggers on one page is a performance problem.

- Use **one trigger per section** driving a staggered timeline, rather than one trigger per card.
- For simple "reveal when visible" behaviour with many elements, `IntersectionObserver` plus a CSS class is cheaper than ScrollTrigger.
- Batch with `ScrollTrigger.batch()` when many elements genuinely need individual triggers.
