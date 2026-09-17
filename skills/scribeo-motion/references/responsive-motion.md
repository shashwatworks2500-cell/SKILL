# Responsive & Input-Aware Motion

Load for breakpoint motion, touch versus pointer differences, and mobile adaptation.

**Motion does not scale down; it is re-decided.** A hero scrub that reads as cinematic at 1440px is unusable at 375px on a mid-range phone. The intent carries across breakpoints; the mechanism usually does not.

## Preserve intent, change mechanism

Ask what the motion *communicates*, then find the cheapest way to communicate the same thing at each width.

| Desktop | Intent | Mobile equivalent |
| --- | --- | --- |
| Pinned scrubbed sequence | Progression through steps | Triggered reveals per step, natural scroll |
| Horizontal scroll track | Lateral exploration | Native scroll-snap carousel |
| Parallax depth layers | Spatial depth | Static composition, or a single subtle layer |
| Hover reveal of detail | Detail is available | Always-visible, or tap to expand |
| Large 64px staggered entrance | Arrival, sequence | 16px, faster, shorter total stagger |
| Scroll-linked video | Immersion | Poster image plus triggered reveal |

Dropping motion on mobile is a legitimate answer. Reducing it is usually better than reproducing it badly.

## Implementation

Branch with `gsap.matchMedia()`, never with a `window.innerWidth` check — matchMedia creates and reverts per breakpoint as the viewport crosses it, which is also the resize fix.

```js
const mm = gsap.matchMedia();

mm.add("(min-width: 1024px)", () => {
  gsap.to(".panel", { xPercent: -60, ease: "none",
    scrollTrigger: { trigger: ".stage", pin: true, scrub: 0.5, end: "+=100%" } });
});

mm.add("(max-width: 1023px)", () => {
  gsap.from(".panel__item", { y: 16, opacity: 0, stagger: 0.05,
    scrollTrigger: { trigger: ".stage", start: "top 85%", once: true } });
});
```

For CSS-driven motion, plain media queries are enough and cost no JavaScript:

```css
.reveal { transition: transform var(--dur-slow) var(--ease-out); transform: translateY(var(--move-md)); }
@media (max-width: 767px) { .reveal { transform: translateY(var(--move-sm)); } }
```

## Distance and duration by viewport

- **Travel should be relative, not fixed.** 40px is a considered reveal on desktop and a large fraction of a small phone's viewport. Scale with viewport units, percentages, or a per-breakpoint token.
- **Durations shorten slightly on mobile.** Smaller travel needs less time, and mobile users are typically in a hurry.
- **Cap total stagger harder on mobile.** Fewer items are visible at once, so a long stagger means the last item animates off-screen.

## Touch versus pointer

Touch is not "mouse without hover" — it is a different interaction model.

- **Hover does not exist.** Never hide information or an affordance behind it. Sticky emulated hover on tap is worse than no hover.
- **Press feedback matters more.** With no hover to confirm targeting, `:active` response is the only pre-commit signal that the tap landed.
- **Scroll is momentum-based and user-owned.** Scroll-jacking, heavy scrubbing, and pinning interfere with a gesture the user physically controls. Pinning is the most common mobile motion defect.
- **Fingers occlude.** Motion near the point of contact is partly hidden; position feedback where it can be seen.
- **Rubber-banding** at scroll extremes can fire triggers unexpectedly. Test at the very top and bottom.

Detect capability, not device:

```css
@media (hover: hover) and (pointer: fine) {
  .card:hover { transform: translateY(-4px); }
}
```

```js
mm.add("(hover: hover) and (pointer: fine)", () => { /* pointer-only motion */ });
```

Device-width checks misclassify touch laptops and large tablets. `(hover: hover)` and `(pointer: fine)` ask the real question.

## Performance headroom

Mobile GPUs and thermally-throttled CPUs have a fraction of desktop capacity, and the same animation can cost several times more.

- Reduce the number of simultaneously animated elements — the first thing to cut.
- Avoid `filter`, `backdrop-filter`, and large shadows in motion on mobile; they are paint-expensive at any resolution and worse on high-DPI screens.
- Scrubbed animation is costlier on mobile. Prefer triggered.
- Test on a real mid-range device or with CPU throttling. A flagship phone will hide the problem your users will hit.

## Orientation and resize

- **Verify both tablet orientations.** 768×1024 and 1024×768 hit different breakpoints and different pin behaviour.
- Mobile browser chrome changes viewport height as it hides and shows, which retriggers resize. `matchMedia` plus function-based `end` values survive this; fixed pixel values do not.
- Use `dvh` rather than `vh` for full-height animated sections, so the mobile URL bar does not cause a jump.

## Checklist

- [ ] Every scroll-linked or pinned effect has an explicit touch behaviour
- [ ] Nothing essential lives behind hover
- [ ] Press feedback present on all interactive elements
- [ ] Travel distance and stagger scaled per breakpoint
- [ ] Capability queries used, not width-based device guesses
- [ ] Verified on a throttled or real mid-range device
- [ ] Both tablet orientations checked
- [ ] Survives resize and mobile chrome show/hide
