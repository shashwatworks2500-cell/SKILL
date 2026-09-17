# Interaction Motion

Load for hover, focus, press, micro-interactions, page transitions, Motion usage, and CSS patterns.

Interaction motion is the motion users meet most often, so it is judged on responsiveness and repeatability — never on expressiveness.

## The 100ms rule

Feedback for a direct user action must **begin** within ~100ms. Beyond that the interface reads as laggy no matter how refined the curve.

This is about when motion *starts*, not when it finishes. A 300ms transition that begins immediately feels instant; a 150ms transition with a 200ms delay feels broken.

**Never delay interaction feedback.** `transition-delay` on hover is almost always a mistake — the exception is a deliberate hover-intent guard on a menu, where the delay prevents accidental opening.

## CSS first

Hover, focus, press, and single-property state changes belong in CSS. They are cheaper, they composite, they need no cleanup, and they cannot leak.

```css
.card {
  transition: transform var(--dur-fast) var(--ease-out),
              box-shadow var(--dur-fast) var(--ease-out);
}
.card:hover { transform: translateY(-4px); }
.card:active { transform: translateY(-1px); }      /* press reads as depth */
.card:focus-visible { outline: 2px solid var(--focus); outline-offset: 2px; }
```

```js
/* Wrong: a library, a listener, and a cleanup obligation to do what CSS does */
el.addEventListener("mouseenter", () => gsap.to(el, { y: -4 }));
```

Three rules:

- **Transition named properties, never `all`.** `transition: all` animates properties you did not intend, including layout ones, and costs frames.
- **Declare the transition on the base element**, not inside `:hover`, or the return journey has no transition.
- **Never `transition` a property that GSAP or Motion also drives** — see `gsap.md`.

## Press and pointer

- **Press feedback is the highest-value micro-interaction** and the most often missing. `:active` with a 2–4px depth or subtle scale confirms the tap landed, which matters most on touch where there is no hover.
- Use `:focus-visible`, not `:focus`, so keyboard users get a ring and mouse users do not.
- Keep the same motion on keyboard activation as on click. A button that responds to the mouse but not to `Enter` is inconsistent.
- Hover must never be the only affordance — it does not exist on touch. See `responsive-motion.md`.

## Motion (the library)

`motion` 13.4.0, MIT. This is the current package name; `framer-motion` is the legacy name at the same version. Import from `motion/react` in React.

Use Motion — not GSAP — for:

- **Component enter/exit** tied to React state, where the element must survive long enough to animate out.
- **Layout animation** between positions, which is genuinely hard by hand.
- **Gesture-driven and interruptible** interaction, where springs handle mid-flight redirection.

```jsx
import { motion, AnimatePresence } from "motion/react";

<AnimatePresence mode="wait">
  {open && (
    <motion.div
      initial={{ opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: 8 }}
      transition={{ duration: 0.24, ease: [0.2, 0, 0, 1] }}
    >
      …
    </motion.div>
  )}
</AnimatePresence>
```

- `AnimatePresence` exists because React unmounts immediately; without it there is no exit animation.
- `mode="wait"` makes the outgoing element finish before the incoming one starts — the right choice when both occupy the same space.
- `layout` animates position and size changes via FLIP. Use it instead of animating `width`/`height`.
- **Do not put Motion and GSAP on the same element.** Choose per component: React-state-driven → Motion; timeline- or scroll-driven → GSAP.

## Micro-interactions

Small, fast, and repeatable. They confirm rather than perform.

| Interaction | Treatment |
| --- | --- |
| Button press | 2–4px depth or 0.98 scale, Instant tier |
| Toggle / switch | Knob translate + track colour, Fast tier |
| Checkbox | Draw or scale the mark, Fast tier |
| Input focus | Border or ring change, Instant tier — never move the label with layout |
| Accordion | Height via `grid-template-rows` or FLIP, Standard tier |
| Tooltip | Opacity + 4–8px translate, Fast tier, no bounce |
| Copy confirmation | Icon swap plus a brief state, no toast for a trivial action |

Anything a user encounters dozens of times must be nearly invisible. Interaction motion that draws attention to itself becomes irritating by the fifth use.

## Page and route transitions

A route transition must not delay the destination. The common failure is an exit animation that adds 400ms of nothing before navigation.

Constraints: total perceived cost under ~400ms · exit faster than entrance, since the decision is already made · content interactive as soon as it is visible · never block on a transition that could fail · never animate away a page that has not finished loading its replacement.

**View Transitions API** is the cleanest route-transition mechanism where supported — it handles the cross-fade and shared elements at the browser level, with no manual orchestration. Support is uneven at time of writing, so treat it as progressive enhancement: with support, a smooth transition; without, an immediate navigation, which is an acceptable baseline.

```css
@media (prefers-reduced-motion: no-preference) {
  ::view-transition-old(root) { animation-duration: 180ms; }
  ::view-transition-new(root) { animation-duration: 220ms; }
}
```

For framework-driven transitions, the lifecycle matters more than the animation — see `react.md`.

**Shared-element transitions** are the one genuinely premium route transition: a card that becomes the detail page header communicates continuity. Everything else is usually a cross-fade dressed up.

## Interruption

Every interactive animation must reverse cleanly from its current position.

```js
/* Wrong: a new tween per event — they accumulate and fight */
el.addEventListener("mouseenter", () => gsap.to(el, { y: -4 }));
el.addEventListener("mouseleave", () => gsap.to(el, { y: 0 }));

/* Right: one reversible timeline */
const hover = gsap.to(el, { y: -4, duration: 0.2, paused: true, ease: "power2.out" });
el.addEventListener("mouseenter", () => hover.play());
el.addEventListener("mouseleave", () => hover.reverse());
```

CSS transitions interrupt correctly for free — another reason to prefer them. Springs in Motion also handle redirection naturally. Duration-based JS tweens are the case that needs care.
