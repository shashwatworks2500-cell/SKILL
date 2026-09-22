---
name: scribeo-motion
description: Use when designing, implementing, debugging, or refining motion and animation on a website or web interface — scroll-triggered reveals, scrubbed and pinned scroll sections, GSAP timelines and ScrollTrigger, Motion (Framer Motion) component animation, Lenis smooth scrolling, page and route transitions, hover/focus/press feedback, micro-interactions, stagger and sequencing, entrance and exit choreography, easing, duration and timing, transforms, clip and mask reveals, reduced-motion implementation, and animation cleanup and lifecycle. Triggers on requests like "add scroll animations", "make this section animate on scroll", "build a GSAP timeline", "add a page transition", "animate the cards", "create a reveal animation", "create a stagger", "make the hero scroll-controlled", "add hover motion", "add Lenis", "coordinate these animations", "make this interaction smoother", "make this transition feel smoother", "fix the broken animation", "the scroll animation is jittery", "the animation fires twice", "the animation breaks after resize", "Lenis and ScrollTrigger are fighting", "implement reduced motion", or "optimize this animation". Whether the interface should feel restrained or expressive — and every other aesthetic judgement — belongs to frontend-design; page and component structure, states and responsive UX belong to scribeo-ux-engineering. Does not own visual verification (scribeo-visual-qa), performance budgets and Core Web Vitals (scribeo-performance), WCAG auditing (scribeo-accessibility), test authoring (scribeo-testing), or SEO (scribeo-seo). Not triggered by a general "build a website" request unless motion is explicitly part of the ask.
---

# Scribeo Motion

Scribeo Studio's motion engineering standard. Governs how interface elements move through time — choreography, timing, sequencing, scroll linkage, and the implementation and lifecycle beneath them.

Apply this whenever you are deciding *what moves, when, how fast, in what order, driven by what* — or fixing motion that is broken, conflicting, or janky.

## The law

**Motion must have a purpose you can state in one sentence.**

Every animation that ships must communicate at least one of: **hierarchy · continuity · spatial relationship · state change · feedback · focus · progression · orientation · emphasis.**

If an animation communicates none of these, it is decoration with a frame cost. Delete it.

Two corollaries that decide most arguments:

- **Motion is the slowest way to communicate anything.** It buys comprehension with the user's time. Spend it where comprehension is actually at risk.
- **The interface must be fully understandable with all motion removed.** Motion clarifies meaning; it never carries it. If removing the animation makes something unusable or unintelligible, the structure is wrong — that is a `scribeo-ux-engineering` problem, not a motion one.

### Default-deny list

Off by default. Each needs a stated, brief-specific reason:

parallax · continuous floating or bobbing · animated counters · marquees · scroll-hijacking · fade-and-slide on every section · hover transitions on every card · motion on more than one focal element at a time · looping background animation · text that animates in per character · pinned sections longer than one viewport of content · anything that delays first interaction

Never permitted, on any brief:

- **Motion that blocks or delays interaction** — a control that cannot be used until an entrance finishes.
- **Motion that makes content unreadable** while it plays.
- **Scroll hijacking that removes the user's scroll control**, or breaks keyboard, trackpad, or momentum scrolling.
- **Motion that ignores `prefers-reduced-motion`.** Non-negotiable; see `references/accessibility-motion.md`.
- **Entrance animation gating content** that must be indexable or immediately visible.

## Boundaries

**This skill owns:** motion architecture · interaction choreography · timing, easing, duration, interpolation · sequencing and stagger · entrance and exit choreography · reveal systems · scroll-triggered, scroll-linked and pinned behaviour · page and route transitions · micro-interactions and press/hover/focus feedback · motion states · motion hierarchy and budgets · reduced-motion implementation · responsive and input-aware motion · animation library selection and integration (GSAP, Motion, Lenis, CSS) · animation lifecycle, cleanup and conflict resolution · motion-specific performance.

**This skill does not own:**

| Concern | Belongs to |
| --- | --- |
| Whether the interface should feel restrained or expressive; aesthetic direction, palette, typeface, visual composition | `frontend-design` (Anthropic) |
| Page and section structure, information hierarchy, component states, responsive UX, usability, conversion structure | `scribeo-ux-engineering` |
| Verifying the rendered result matches intent; screenshot and visual regression review | `scribeo-visual-qa` |
| Performance budgets, Core Web Vitals, bundle and runtime profiling | `scribeo-performance` |
| WCAG conformance auditing, assistive-technology verification | `scribeo-accessibility` |
| Test strategy and authoring | `scribeo-testing` |
| Metadata, structured data, crawlability | `scribeo-seo` |

**The frontend-design boundary, precisely:** `frontend-design` decides *whether and how much* the interface should move — the motion character, the restraint, which single moment is allowed to be bold. This skill decides *how that moves in time and how it is built* — the timeline, the easing curve, the trigger, the cleanup. Take the motion character as a given and engineer it. Do not escalate a brief's quiet motion into something expressive because it is technically impressive, and do not decide the aesthetic here.

**The ux-engineering boundary, precisely:** `scribeo-ux-engineering` defines *that* a state exists and what it must communicate — loading, empty, error, success, open, selected. This skill defines *how the transition between those states plays*. The state must be legible statically before any animation is added. Never introduce a state here, and never rely on motion to make an unclear state clear.

**On performance and accessibility:** motion is a frequent cause of jank and a genuine accessibility risk, so this skill carries the motion-specific parts of both — compositor-friendly properties, frame budget, `prefers-reduced-motion`, vestibular safety. It does not own the broader domains. Page-level budgets and Core Web Vitals go to `scribeo-performance`; conformance audits go to `scribeo-accessibility`.

**If `frontend-design` is unavailable.** It is an Anthropic-provided skill, not part of the `scribeo-skills` marketplace, so it may not be installed. Never invent the aesthetic here to unblock yourself, and never stall work the aesthetic does not gate. Say plainly that the visual direction is unset, ask for it, and proceed with everything this skill owns that the direction does not affect: the purpose statement, trigger, sequencing, easing, reduced-motion behaviour, lifecycle and cleanup. Hold the motion *character* — restrained or expressive, one bold moment or none. On existing work, read the character from what already ships and match it; do not escalate it.

## Workflow

Never open an animation library first. Most motion defects are decisions made in the wrong order.

**1 — Establish purpose.** Which of the nine purposes does this serve? Name it. If you cannot, stop and propose removing it.

**2 — Confirm the static baseline exists.** The element must already be correct, legible, and usable with no motion. Animating an unfinished state hides structural problems.

**3 — Take the motion character from the brief.** Restrained or expressive, one bold moment or none — that is `frontend-design`'s call. Read it; do not set it.

**4 — Set the motion budget.** How many things may move at once (usually one focal element plus supporting detail), and how much of the page's attention motion may consume. Write it down; it is what you enforce later.

**5 — Choose the cheapest mechanism that works.** See the tool selection table below. Reach for JavaScript only when CSS genuinely cannot do it.

**6 — Choose the driver.** User input, element entering the viewport, scroll progress, route change, or data arrival. Scroll-*linked* (scrubbed) and scroll-*triggered* (fire once) are different decisions with different costs — see `references/scroll.md`.

**7 — Derive values, don't guess them.** Duration, easing, distance, and stagger come from interaction importance, distance travelled, element size, and viewport. See `references/system.md`.

**8 — Implement with lifecycle from the start.** Creation and teardown in the same block. Retrofitted cleanup is the single largest source of double-firing and resize bugs. See `references/react.md`.

**9 — Handle reduced motion in the same commit.** Not as a follow-up. See `references/accessibility-motion.md`.

**10 — Verify on real input.** Keyboard, touch, trackpad momentum, slow CPU, after resize, after route change and return. Not one smooth scroll on a fast desktop.

**11 — Cut.** Remove the weakest animation on the page and check whether anything was lost. Usually nothing is.

## Tool selection

Pick the lowest row that satisfies the requirement.

| Need | Use |
| --- | --- |
| Hover, focus, press, simple open/close, single-property state change | **CSS** `transition` |
| Looping or keyframed decoration, self-contained and non-interactive | **CSS** `@keyframes` |
| Enter/exit of React components, layout shifts between positions, gesture-driven or spring interaction | **Motion** (`motion`, MIT) |
| Multi-element timelines, precise sequencing, scroll-scrubbing, pinning, complex orchestration | **GSAP** + ScrollTrigger |
| Normalised smooth scrolling across input devices, as a deliberate product decision | **Lenis** (`lenis`, MIT) |

**CSS first, genuinely.** A hover state written in GSAP is a defect: it ships a library, an event listener, and a cleanup obligation to do what one `transition` declaration does on the compositor.

**Do not run two systems over the same property on the same element.** That is the root cause of most "the animation is fighting itself" reports. One owner per property per element.

Verified package facts (re-check before pinning): `gsap` 3.15.0 — standard *no-charge* license, plugins including ScrollTrigger included. `motion` 13.4.0 — MIT; this is the current package name, and `framer-motion` is the legacy name at the same version. `lenis` 1.3.26 — MIT; **`@studio-freight/lenis` is deprecated and renamed**, so never install that. `@gsap/react` 2.1.2 provides `useGSAP()`.

## Non-negotiable engineering defaults

**Properties**
- Animate `transform` and `opacity` by default — they composite without layout or paint.
- Never animate `width`, `height`, `top`, `left`, `margin`, or `padding` to move or resize something. Use `transform`. For genuine layout change, use a FLIP technique or Motion's layout animation.
- `filter`, `box-shadow`, and `backdrop-filter` are paint-expensive. Animating them across a large area is a reliable way to drop frames.

**Lifecycle**
- Every animation, listener, observer, and ticker callback is destroyed where it was created. No exceptions.
- Nothing initialises twice. React StrictMode double-invokes effects in development — if your setup cannot survive that, it is broken, not the mode.
- Recalculate on resize and on content reflow. Scroll positions measured once at mount are wrong the moment anything changes.

**Interaction**
- Feedback for a direct user action starts within ~100ms. Anything slower reads as an unresponsive interface, not as elegance.
- Animations must be interruptible and reversible. A user who re-hovers mid-transition should see it reverse, not queue.
- Motion must never be the only signal of a state change; pair it with a static difference.

**Reduced motion**
- Honour `prefers-reduced-motion: reduce` on every non-essential animation.
- Reduced does not mean deleted: keep the state change, keep the meaning, drop the travel. Setting every duration to `0` is a lazy failure mode that breaks continuity cues.

## Symptom triage

Start here when motion is broken rather than absent. Each row names the usual cause and where the fix lives.

| Symptom | Usual cause | Reference |
| --- | --- | --- |
| Animation fires twice | Duplicate init — StrictMode, re-render, or missing teardown | `references/react.md` |
| Breaks after resize | Values measured once; no refresh or `invalidateOnRefresh` | `references/scroll.md` |
| Lenis and ScrollTrigger fight | Two scroll systems, unsynchronised update loops | `references/scroll.md` |
| Jittery or juddering scroll | Layout-triggering property, or scroll work off the ticker | `references/performance-motion.md` |
| Stutters only on mobile | Paint-heavy effect or too many simultaneous animations | `references/performance-motion.md` |
| Feels sluggish, not smooth | Duration too long for the interaction tier | `references/system.md` |
| Two animations conflict | Two systems owning one property | `references/gsap.md` |
| Works on load, breaks after navigation | Route-change teardown missing | `references/react.md` |
| Nothing animates in production | SSR/hydration boundary, or element measured before layout | `references/react.md` |
| Fine on desktop, unusable on touch | Hover-dependent or pin-dependent motion on touch | `references/responsive-motion.md` |

## Reference routing

`SKILL.md` is the decision core. Load a reference when the work reaches that surface; do not load all of them.

| Load | When |
| --- | --- |
| `references/principles.md` | Deciding whether and what to animate; motion hierarchy; anti-patterns |
| `references/system.md` | Duration, easing, distance and stagger tiers, and deriving values |
| `references/gsap.md` | Timelines, `useGSAP`/`gsap.context`, `matchMedia`, ScrollTrigger config, cleanup |
| `references/scroll.md` | Scroll reveals, scrub, pin, horizontal sections, scroll-linked video, Lenis |
| `references/interaction.md` | Hover/focus/press, micro-interactions, page transitions, Motion, CSS patterns |
| `references/react.md` | Client boundaries, hydration, StrictMode, route transitions, teardown |
| `references/responsive-motion.md` | Breakpoint motion, touch vs pointer, mobile adaptation |
| `references/accessibility-motion.md` | `prefers-reduced-motion`, vestibular safety, focus, motion-triggered content |
| `references/performance-motion.md` | Compositing, layout thrash, frame budget, DevTools inspection |

## Quality gate

Do not report motion work complete until every line holds.

**Purpose**
- [ ] Every animation serves a nameable purpose from the nine
- [ ] The interface is fully usable and understandable with all motion removed
- [ ] One focal element moves at a time; the motion budget is respected
- [ ] Nothing from the default-deny list appears without a stated reason
- [ ] The weakest animation was removed and nothing was lost

**Behaviour**
- [ ] Direct-action feedback begins within ~100ms
- [ ] Animations interrupt and reverse cleanly; nothing queues
- [ ] No animation blocks or delays interaction
- [ ] Motion is never the sole indicator of a state change

**Implementation**
- [ ] CSS used wherever JavaScript was unnecessary
- [ ] One system owns each animated property per element
- [ ] Every animation, listener, observer and ticker callback is torn down where created
- [ ] Survives StrictMode double-invocation, resize, and route change and return
- [ ] Only `transform`/`opacity` on the hot path; no layout-triggering properties

**Accessibility and reach**
- [ ] `prefers-reduced-motion: reduce` handled deliberately — meaning preserved, travel removed
- [ ] No continuous or large-area motion that risks vestibular discomfort
- [ ] Keyboard focus never lost or obscured mid-animation
- [ ] Touch behaviour verified; no hover-only or pin-dependent motion on touch

**Verified, not assumed** — checked in a browser, on real input, after resize and navigation, with reduced motion on and off. If a line could not be verified, say so explicitly rather than implying it passed.
