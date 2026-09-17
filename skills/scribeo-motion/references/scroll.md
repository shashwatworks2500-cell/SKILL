# Scroll-Driven Motion

Load for scroll reveals, scrubbing, pinning, horizontal sections, scroll-linked media, and Lenis.

## Triggered vs linked

Two different mechanisms with different costs. Choose deliberately.

| | **Scroll-triggered** | **Scroll-linked (scrubbed)** |
| --- | --- | --- |
| Behaviour | Fires once when element enters | Progress tied to scroll position |
| Clock | Its own duration | The user's scroll |
| Easing | Normal easing | `linear` — the input is already the curve |
| Cost | Low | Higher; runs work on every scroll frame |
| Use | Section reveals, entrances | Storytelling, progress, pinned sequences |

Most "add scroll animations" requests mean **triggered**. Reach for scrubbing only when progress itself is the message.

## Reveals

The default and usually correct pattern: one trigger per section, driving a staggered timeline.

```js
gsap.from(".card", {
  y: 24, opacity: 0, duration: 0.6, ease: "power3.out",
  stagger: 0.06,
  scrollTrigger: { trigger: ".cards", start: "top 80%", once: true },
});
```

- `start: "top 80%"` fires slightly before the section is fully in view, so content is present by the time it is read.
- `once: true` — reveals should not replay on scroll-back.
- **Never hide content that must be indexable or immediately available behind a reveal.** If JS fails, the content must still be visible; set the hidden state in a class JS adds, not in base CSS.

```css
/* Safe: only hidden once JS confirms it can animate */
.js-reveal .reveal-item { opacity: 0; }
```

## Scrubbing

```js
gsap.to(".progress", {
  scaleX: 1, ease: "none",
  scrollTrigger: { trigger: ".article", start: "top top", end: "bottom bottom", scrub: 0.3 },
});
```

- `ease: "none"` — anything else fights the scroll mapping.
- `scrub: 0.3`–`1` smooths jitter from trackpad and wheel input. `scrub: true` is exact but can feel twitchy.
- Keep scrubbed work to `transform`/`opacity`. A scrubbed `filter` or `box-shadow` repaints on every frame.

## Pinned sequences

Pinning plus scrubbing is the "scroll-controlled storytelling" pattern. It is powerful and frequently overused.

```js
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".story", start: "top top",
    end: () => "+=" + window.innerHeight,   // function = recalculated on refresh
    pin: true, scrub: 0.5, invalidateOnRefresh: true,
  },
});
tl.from(".story__a", { opacity: 0 }).from(".story__b", { opacity: 0 });
```

Requirements: a real narrative reason · roughly one viewport of pinned scroll, not five · a visible progress cue so the user knows the section ends · disabled or replaced on touch.

## Horizontal sections

```js
gsap.to(".track", {
  xPercent: -100 * (panels - 1), ease: "none",
  scrollTrigger: {
    trigger: ".h-scroll", pin: true, scrub: 1,
    end: () => "+=" + document.querySelector(".track").scrollWidth,
    invalidateOnRefresh: true,
  },
});
```

Compute `end` from measured width in a function so it survives resize. Horizontal scroll removes a familiar affordance — provide a visible progress indicator, keep it to a few panels, and consider a native horizontal scroll container with scroll-snap on touch instead.

## Scroll-linked video

```js
const v = document.querySelector("video");
v.pause();
await new Promise(r => v.readyState >= 2 ? r() : v.addEventListener("loadeddata", r, { once: true }));

gsap.to(v, {
  currentTime: v.duration, ease: "none",
  scrollTrigger: { trigger: ".hero", start: "top top", end: "+=150%", pin: true, scrub: 0.5 },
});
```

- Wait for metadata before reading `duration`; it is `NaN` until then.
- Seeking is expensive and **unreliable on iOS**, which throttles programmatic seeks. Verify on a real device; fall back to a poster image plus a triggered reveal on touch.
- Encode for seeking: short GOP length, modest resolution. Multi-megabyte 4K scrubbing drops frames on any laptop. Encoding strategy is `scribeo-performance`'s domain.
- An image sequence on a canvas is often smoother than video seeking, at the cost of many requests.

## Lenis

Smooth scrolling is a **product decision, not a default.** It overrides an OS-level behaviour users are calibrated to, and it must be justified by the brief. `lenis` is MIT; **`@studio-freight/lenis` is deprecated and renamed — never install it.**

Integrated with GSAP, the two must share one update loop or they will fight:

```js
import Lenis from "lenis";

const lenis = new Lenis({ autoRaf: false });

// 1. Lenis drives ScrollTrigger's notion of scroll position
lenis.on("scroll", ScrollTrigger.update);

// 2. GSAP's ticker drives Lenis — one loop, not two
gsap.ticker.add((time) => lenis.raf(time * 1000));

// 3. Stop GSAP compensating for frame lag, which desyncs the two
gsap.ticker.lagSmoothing(0);

// teardown
// gsap.ticker.remove(rafFn); lenis.destroy();
```

That three-step wiring is the fix for "Lenis and ScrollTrigger are fighting". The symptom is triggers firing at the wrong position, or a visible lag between scroll and animation.

Also required: disable or bypass Lenis under `prefers-reduced-motion` · verify anchor links and `scrollIntoView` still work (route them through `lenis.scrollTo`) · confirm keyboard `Page Up`/`Down`/`Home`/`End` still scroll · destroy on unmount, or a second instance will double-handle every wheel event.

**Never run two smooth-scroll systems.** Lenis plus CSS `scroll-behavior: smooth` plus a library equals unfixable behaviour. Pick one.

## Native CSS scroll-driven animation

CSS `animation-timeline: view()` / `scroll()` does simple reveals and progress bars with no JavaScript and no trigger lifecycle. Support is good in Chromium and uneven elsewhere at time of writing — **verify current support before shipping it as the only mechanism**, and treat it as a progressive enhancement over a visible baseline.

## Anti-patterns

| Anti-pattern | Why | Instead |
| --- | --- | --- |
| Hijacking scroll to move section-by-section | Removes control; breaks keyboard and momentum | Scroll-linked animation; native scroll |
| Pinning several viewports | User scrolls and nothing appears to happen | One viewport per pin, with a progress cue |
| One ScrollTrigger per card | Dozens of measurement and scroll callbacks | One per section, `stagger` inside |
| `end` as a fixed pixel value | Wrong after resize or font load | Function value + `invalidateOnRefresh` |
| Reveal state in base CSS | Content invisible if JS fails | Add the hiding class with JS |
| Scrubbing a `filter` or shadow | Repaint every frame | Scrub `transform`/`opacity` |
| Pinning on mobile untested | Frequently unusable on touch | `matchMedia`, alternative on touch |
| Smooth scroll "for polish" | Fights user expectation and accessibility | Only when the brief requires it |
