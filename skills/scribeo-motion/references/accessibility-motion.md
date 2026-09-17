# Accessible Motion

Load for `prefers-reduced-motion`, vestibular safety, focus during motion, and motion-triggered content.

This covers the **motion-specific** parts of accessibility. Conformance auditing, assistive-technology testing, and the broader WCAG picture belong to `scribeo-accessibility`. Build it right here; that skill proves it.

## Why this is not optional

Vestibular disorders are common, and large or unexpected motion can cause genuine nausea, dizziness, and migraine — not mild annoyance. Parallax, large-scale scrubbing, and continuous background movement are the usual triggers. `prefers-reduced-motion` is a user telling you they are affected.

## prefers-reduced-motion, done properly

**Reduced does not mean removed.** The state change must still happen and still be legible; only the *travel* goes. Zeroing every duration is the standard lazy failure: it destroys continuity cues, and can leave `AnimatePresence`-style exits mid-flight.

```css
/* Wrong: nukes everything, including state-change feedback */
@media (prefers-reduced-motion: reduce) {
  * { animation: none !important; transition: none !important; }
}
```

Prefer opt-in: animate only when motion is welcome, so the reduced case is the untouched baseline.

```css
.reveal { opacity: 1; }                      /* baseline: visible, no transform */

@media (prefers-reduced-motion: no-preference) {
  .reveal { opacity: 0; transform: translateY(var(--move-md));
            transition: opacity var(--dur-slow) var(--ease-out),
                        transform var(--dur-slow) var(--ease-out); }
  .reveal.is-in { opacity: 1; transform: none; }
}
```

This structure means a reduced-motion user gets fully visible content even if JavaScript never runs.

**What to keep, what to drop:**

| Keep | Drop or reduce |
| --- | --- |
| The state change itself | Large positional travel |
| Opacity cross-fades (short) | Parallax and depth movement |
| Colour and border changes | Scrubbed and pinned sequences |
| Focus indicators | Continuous or looping motion |
| Progress and loading indication | Scale and rotation flourishes |
| Directional hint via a static cue | Auto-playing video and carousels |

In JavaScript, branch with the same query, and respond to changes rather than reading once:

```js
const mq = window.matchMedia("(prefers-reduced-motion: reduce)");

const build = () => {
  if (mq.matches) {
    gsap.set(".reveal", { opacity: 1, clearProps: "transform" });
    return;
  }
  gsap.from(".reveal", { y: 40, opacity: 0, stagger: 0.06,
    scrollTrigger: { trigger: ".section", start: "top 80%", once: true } });
};

build();
mq.addEventListener("change", build);   // users toggle this mid-session
```

With GSAP, `gsap.matchMedia()` handles the same thing and reverts automatically:

```js
gsap.matchMedia().add("(prefers-reduced-motion: no-preference)", () => { /* motion */ });
```

**Reduced motion must also disable smooth scrolling.** Lenis overrides native scroll for everyone; under reduced motion, bypass or destroy it.

## Focus during motion

- **Focus must never be lost.** An element animating out while focused strands the user. Move focus deliberately before removal.
- **The focus ring must stay visible throughout.** Do not animate `outline`, and beware transforms that push a focused element under a sticky header.
- **Opening a panel moves focus in; closing returns it to the trigger.** The animation plays around that, never instead of it.
- **Never animate a focused element off-screen.** Scroll-linked motion plus keyboard traversal is the case to check: tab through a scrubbed section and confirm focused elements remain in view.
- Do not gate focusability behind an entrance animation — a control must be tabbable as soon as it exists.

## Motion-triggered content

Content revealed by scroll has real accessibility consequences.

- **Never make content depend on motion to exist.** Scroll-revealed text must be in the DOM and available to screen readers and crawlers regardless of animation state.
- `opacity: 0` leaves content in the accessibility tree — a screen reader announces text the sighted user cannot see. For genuinely hidden-then-revealed content use `visibility` or `hidden`, or accept that it is announced and ensure that is acceptable.
- Never set `aria-hidden` on something that will be revealed and then forget to clear it.
- Announce asynchronous changes with a live region; a visual transition alone communicates nothing to a screen reader.

## Auto-playing and continuous motion

WCAG is explicit: anything moving, blinking, or scrolling automatically for more than five seconds needs a mechanism to pause, stop, or hide it.

- Carousels: provide controls, do not auto-advance by default, and pause on hover and focus.
- Background video: `muted`, and a visible pause control. Respect reduced motion by showing the poster frame.
- Marquees and tickers: on the default-deny list. If required, make them pausable.
- Looping decorative animation: off under reduced motion, always.

## Checklist

- [ ] `prefers-reduced-motion: reduce` handled on every non-essential animation
- [ ] Reduced-motion path preserves state changes and meaning, not just `duration: 0`
- [ ] Content visible and readable if JavaScript never runs
- [ ] The media query is re-evaluated on change, not read once
- [ ] Smooth scrolling disabled or bypassed under reduced motion
- [ ] Focus never lost, obscured, or animated out of view
- [ ] Focus indicators unaffected by motion
- [ ] Anything auto-moving beyond five seconds is pausable
- [ ] No content depends on animation to be reachable
