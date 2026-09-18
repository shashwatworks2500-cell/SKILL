# Motion Accessibility

Load when motion may create an accessibility barrier.

## The boundary

| Owner | Owns |
| --- | --- |
| **here** | Whether motion creates a barrier · reduced-motion **requirements** · vestibular safety · essential vs non-essential classification · ensuring functionality survives reduced motion |
| `scribeo-motion` | Animation implementation, timelines, easing, scroll choreography, and the authoring-time reduced-motion pattern |
| `scribeo-performance` | The performance cost of motion — frames, jank, main-thread cost |
| `scribeo-visual-qa` | Visual verification that the rendered result is correct |

**This skill sets the requirement; Motion implements it.** When motion creates a barrier, state the requirement and the evidence, then route the animation change. Do not rewrite timelines, easing, or choreography here.

## Why it matters

Vestibular disorders are common, and large-scale or unexpected motion can cause genuine nausea, dizziness and migraine — not mild irritation. Parallax, large scrubbed scroll effects, and continuous background movement are the usual triggers. `prefers-reduced-motion: reduce` is a user telling you they are affected.

## The relevant criteria

- **2.3.3 Animation from Interactions (Level AAA)** — motion animation triggered by interaction can be disabled, unless the animation is essential to the functionality or information conveyed. W3C names **`prefers-reduced-motion` as a technique for meeting it**.
- **2.2.2 Pause, Stop, Hide (Level A)** — moving, blinking or scrolling content that starts automatically, lasts **more than five seconds**, and is presented in parallel with other content must have a mechanism to pause, stop or hide it. The same applies to auto-updating information (pause, stop, hide, or control frequency). Exception: where the movement is essential.
- **2.3.1 Three Flashes or Below Threshold (Level A)** — flashing must not exceed three flashes per second, subject to luminance thresholds. Seizure risk; there is no design justification that overrides it.

**Note the levels honestly.** 2.3.3 is AAA, so on a AA engagement reduced-motion support is a Scribeo standard and a strong recommendation rather than a AA conformance failure. 2.2.2 and 2.3.1 are **Level A** — the baseline, non-negotiable under any conformance target.

## Essential vs non-essential

The criteria hinge on this, so classify deliberately:

**Non-essential** (must be disableable): decorative parallax · entrance and reveal animations · hover flourishes · background loops · scroll-linked decoration · page-transition effects.

**Essential** (may remain): a loading indicator showing that work is in progress · animation that *is* the content (a demonstration video, a data animation where the movement conveys the information) · motion that is the only way to convey a state change, where no static alternative exists.

The bar for "essential" is high. "It's central to the brand feel" is not essential — that is an aesthetic argument, and it belongs to `frontend-design`. Present the accessibility requirement and let them decide how to express the brand within it.

## Reduced motion done properly

**Reduced does not mean removed.** The state change must still happen and still be legible; only the travel goes.

```css
/* Wrong: removes all feedback, including state changes */
@media (prefers-reduced-motion: reduce) {
  * { animation: none !important; transition: none !important; }
}
```

Prefer the opt-in structure, so the reduced case is the untouched baseline and content is visible even if JavaScript never runs:

```css
.reveal { opacity: 1; }                     /* baseline: visible */

@media (prefers-reduced-motion: no-preference) {
  .reveal { opacity: 0; transform: translateY(2rem);
            transition: opacity .5s, transform .5s; }
  .reveal.is-in { opacity: 1; transform: none; }
}
```

| Keep under reduced motion | Remove or reduce |
| --- | --- |
| The state change itself | Large positional travel |
| Short opacity cross-fades | Parallax and depth effects |
| Colour and border changes | Scrubbed and pinned scroll sequences |
| Focus indicators | Continuous and looping motion |
| Progress and loading indication | Scale and rotation flourishes |
| A static directional cue | Auto-playing video and carousels |

**The critical audit question:** is any content or control *unreachable* under reduced motion? A reveal that never fires because its trigger was disabled leaves content permanently invisible. That is **Critical** — content is inaccessible.

Also required: **smooth-scroll overrides must be disabled** under reduced motion; a library that overrides native scrolling for everyone is a barrier.

And the preference can change mid-session — implementations must respond to the media query changing, not read it once.

## Auditing

```js
() => matchMedia("(prefers-reduced-motion: reduce)").matches
```

Procedure:

1. **Emulate `prefers-reduced-motion: reduce`** and reload.
2. **Verify all content is present and readable** — nothing stuck invisible.
3. **Complete the primary task** with the preference on.
4. **Check state changes are still communicated** — not silently instant.
5. **Confirm no smooth-scroll override** remains active.
6. **Screenshot both paths** for evidence; route rendering questions to `scribeo-visual-qa`.

Also audit with motion **on**: is anything large-scale, continuous, or auto-starting beyond five seconds without a pause mechanism (2.2.2)? Does anything flash (2.3.1)?

## Reporting to Motion

```
Barrier:     Section reveals do not fire under prefers-reduced-motion: reduce,
             leaving three sections permanently at opacity 0
Requirement: Content must be fully visible and operable with the preference
             enabled. Reduced motion removes travel, never content
Criterion:   Not a 2.2.2 or 2.3.1 failure. 2.3.3 is AAA. Reported as a
             Scribeo standard failure and a content-inaccessibility barrier
Evidence:    Screenshots with the preference on and off; reveal elements
             retain opacity 0 in the tree
Severity:    Critical — content unreachable in a supported user preference
Owner:       scribeo-motion — recommend the opt-in no-preference structure so
             the baseline is visible
```

Requirement, evidence, severity, owner. **No prescribed easing, duration, or choreography** — those remain Motion's decisions.

## Audit checklist

- [ ] All content present and readable under reduced motion
- [ ] Primary task completable with the preference on
- [ ] State changes still communicated, not merely instantaneous
- [ ] Nothing depends on animation to become reachable
- [ ] Smooth-scroll overrides disabled under reduced motion
- [ ] Media query re-evaluated on change, not read once
- [ ] Auto-moving content beyond five seconds has pause/stop/hide (2.2.2, A)
- [ ] Nothing flashes more than three times per second (2.3.1, A)
- [ ] Essential-motion classifications justified, not assumed
- [ ] Implementation changes routed to `scribeo-motion`
