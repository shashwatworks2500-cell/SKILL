# Motion System

Load for duration, easing, distance and stagger values, and for deriving them.

A motion system means values chosen by rule, not by feel-per-instance. Arbitrary durations scattered through a codebase are why motion stops feeling coherent.

**The tiers below are a defensible default, not a universal law.** A brand with a deliberately languid character will scale durations up; a utility interface will scale them down. Derive, then stay consistent.

## Duration tiers

| Tier | Range | Use |
| --- | --- | --- |
| **Instant** | 80–120ms | Press feedback, focus ring, toggle, colour change |
| **Fast** | 150–250ms | Hover, small reveals, tooltip, icon state |
| **Standard** | 250–400ms | Open/close, dropdown, accordion, card expand |
| **Slow** | 400–700ms | Modal, drawer, route transition, section reveal |
| **Expressive** | 700–1200ms | The single hero moment, orchestrated sequence |

Anything above ~1200ms that is not scroll-linked is almost certainly wrong. Scroll-linked animation has no duration — the user's scroll is the clock.

**Derive duration from four inputs:**

1. **Interaction importance** — direct input demands the Instant/Fast tiers. Feedback slower than ~100ms reads as lag.
2. **Distance travelled** — longer travel needs more time, but sub-linearly. Doubling distance warrants roughly 1.3–1.5× duration, not 2×.
3. **Element size** — large surfaces need longer to read as physical; small elements look sluggish at the same duration.
4. **Perceived responsiveness** — exits may be faster than entrances. The user has already decided; do not make them wait to leave.

## Easing tiers

| Tier | Curve | Use |
| --- | --- | --- |
| **Standard out** | `cubic-bezier(0.2, 0, 0, 1)` | Entrances, expansions — fast start, settled finish |
| **Standard in** | `cubic-bezier(0.4, 0, 1, 1)` | Exits — accelerate away |
| **Smooth in-out** | `cubic-bezier(0.4, 0, 0.2, 1)` | Moves where both ends matter |
| **Linear** | `linear` | Scroll-linked and continuous only |
| **Spring** | Motion's physics defaults | Gesture-driven and interruptible interaction |

Rules that matter more than the exact curve:

- **Entrances ease out; exits ease in.** Things arrive settling and leave accelerating. Reversing this is the commonest easing mistake.
- **Never `linear` for discrete motion.** It reads mechanical. Its place is scroll-scrubbing, where the input is already linear.
- **Never `ease-in` for an entrance.** A slow start reads as latency.
- **Springs for anything the user drags or may interrupt** — they handle mid-flight redirection naturally. Duration-based tweens fight it.
- **Avoid overshoot on functional UI.** Bounce on a dropdown is noise; it belongs to the one expressive moment, if anywhere.

## Distance tiers

| Tier | Travel | Use |
| --- | --- | --- |
| Subtle | 4–8px | Hover lift, press depth, nudge |
| Small | 12–24px | Reveals, tooltips, small entrances |
| Medium | 32–64px | Section reveals, card entrances |
| Large | 25–100% of element or viewport | Drawers, panels, route transitions |

Travel scales with element size and viewport — not fixed pixels across breakpoints. A 64px reveal that reads as considered on desktop is a third of a small phone's viewport height. Prefer relative units or reduce travel at narrow widths (see `responsive-motion.md`).

**Reveals need less distance than instinct suggests.** 16–32px with the right easing reads better than 80px, and costs less attention.

## Stagger rules

Stagger implies sequence. Use it only where order carries meaning — reading order, process steps, importance ranking.

| Item count | Stagger per item | Notes |
| --- | --- | --- |
| 2–4 | 60–100ms | Clearly sequential |
| 5–8 | 40–60ms | Keep total under ~500ms |
| 9–20 | 20–35ms | Consider a grid-aware pattern |
| 20+ | Do not stagger individually | Animate the container, or the visible rows only |

- **Cap the total.** Items × stagger should stay under ~600ms. A 12-item list at 100ms makes the last item arrive 1.2s late — the user has already scrolled past.
- **Stagger along the reading direction.** Top-to-bottom, or leading edge inward. Random or centre-out stagger communicates nothing.
- **Never stagger unbounded lists.** A 200-item grid must animate as a group or per visible row.
- **Do not stagger interactive controls.** The third button being unusable for 200ms longer than the first is a usability defect.

## Hierarchy of tiers by role

| Role | Duration | Easing | Distance |
| --- | --- | --- | --- |
| Hero / focal | Expressive | Standard out, or custom | Large |
| Section reveal | Slow | Standard out | Small–Medium |
| Route transition | Slow | In for exit, out for entrance | Large |
| Component open/close | Standard | Smooth in-out | Small–Medium |
| Hover | Fast | Standard out | Subtle |
| Press | Instant | Standard out | Subtle |
| Focus ring | Instant | Standard out | None |

## Encode it, don't repeat it

Put the system in tokens so drift is impossible:

```css
:root {
  --dur-instant: 100ms;  --dur-fast: 200ms;    --dur-standard: 320ms;
  --dur-slow: 520ms;     --dur-expressive: 900ms;
  --ease-out: cubic-bezier(0.2, 0, 0, 1);
  --ease-in: cubic-bezier(0.4, 0, 1, 1);
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
  --move-subtle: 6px;  --move-sm: 16px;  --move-md: 40px;
}
```

Mirror the same values in the JS animation config so CSS and GSAP agree. Two sources of truth for timing is how a page ends up with four different "fast".
