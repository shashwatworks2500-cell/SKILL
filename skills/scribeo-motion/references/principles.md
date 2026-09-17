# Motion Principles

Load when deciding whether and what to animate, or diagnosing motion that feels excessive.

## The nine purposes

An animation ships only if it communicates one of these. Name it before building.

| Purpose | What it does | Typical form |
| --- | --- | --- |
| **Hierarchy** | Shows what matters most | Focal element leads; supporting detail follows |
| **Continuity** | Connects two states as one object | Shared-element transition, FLIP |
| **Spatial relationship** | Shows where something came from or went | Directional slide, origin-anchored scale |
| **State change** | Confirms something is now different | Toggle, expand, check |
| **Feedback** | Confirms input was received | Press response, ripple, nudge |
| **Focus** | Directs attention to one place | Isolated reveal, dimming of surroundings |
| **Progression** | Shows movement through a sequence | Step indicator, scroll progress |
| **Orientation** | Shows where the user is | Route transition direction, nav state |
| **Emphasis** | Marks something as significant | The single bold moment on a page |

If two animations claim the same purpose in the same viewport, one is redundant.

## Motion hierarchy

Motion has a hierarchy exactly as type does, and flat motion is as weak as a flat type scale.

1. **Hero / focal motion** — one per page. May be expressive, longer, orchestrated. This is the moment the brief is paying for.
2. **Structural motion** — section reveals, route transitions. Quiet, consistent, repeatable.
3. **Interaction motion** — hover, press, focus, toggle. Fast, near-invisible, never expressive.
4. **Ambient motion** — background, looping. Off by default; needs a strong reason.

**One focal element moves at a time.** Two things competing for attention means neither has it. When a hero animation plays, everything else holds still.

Interaction motion must never borrow hero timing. A 600ms button press does not read as premium — it reads as broken.

## Restraint

**Subtraction is the primary tool.** To strengthen a motion design, remove the second-most-important animation rather than enhancing the first.

- **Fewer, better moments.** One orchestrated reveal beats eight scattered fades. This also matches `frontend-design`'s guidance: scattered fade-and-slide entrances read as generated default.
- **Shorter than feels right.** Motion almost always wants to be faster than the instinct that designed it. Instinct is calibrated on watching it once; users see it every visit.
- **Quiet repetition.** Anything the user sees repeatedly (card hover, nav open) must be nearly invisible. Novelty becomes irritation at the fifth encounter.
- **Stillness is a choice.** A page where nothing moves until the user acts can be the most premium option. Do not treat an absence of animation as unfinished work.

## Anti-patterns

| Anti-pattern | Why it fails | Instead |
| --- | --- | --- |
| Fade-and-slide on every section | Generic; delays every read; no hierarchy | Animate one focal section; let the rest be present |
| Random stagger everywhere | Stagger without sequence meaning is noise | Stagger only where order carries meaning |
| Parallax by default | Costs frames, risks vestibular discomfort, rarely communicates | Use only to establish genuine depth, subtly |
| Continuous floating | Permanent low-grade distraction | Static, or motion on interaction only |
| Animated counters | Delays the number, communicates nothing | Show the number |
| Scroll hijacking | Removes user control; breaks keyboard and momentum | Scroll-linked animation that leaves scroll native |
| Per-character text animation | Slows reading; often unreadable mid-flight | Animate the block, not the letters |
| Entrance on interactive controls | Delays usability | Controls present and usable immediately |
| Hover transition on every card | Reads as template default | One clear affordance change |
| Competing timelines on one element | Unpredictable, unfixable | One owner per property |

## The interruption test

Premium motion is judged by how it behaves when interrupted, not how it looks when played once through.

Check each interactive animation:

1. Trigger it, then immediately trigger the reverse. Does it reverse smoothly from the current position, or jump, or queue?
2. Trigger it repeatedly and fast. Does it accumulate, or stay stable?
3. Trigger it, then navigate away mid-flight. Does anything leak or error?

A transition that only looks correct when played once and uninterrupted is not finished. See `gsap.md` and `interaction.md` for the mechanics.

## Deciding against motion

Say no when:

- The purpose cannot be named.
- It delays content the user came for.
- It repeats often enough to become irritating.
- It only works on a fast desktop.
- It exists because the page "feels static" — that is a hierarchy problem for `scribeo-ux-engineering`, not a motion problem.
- The brief asked for restraint and this would break it. Motion character is `frontend-design`'s to set.

"The client wants it to feel alive" is not a purpose. Translate it: what should the user understand or feel, and at which moment? Then animate that one thing well.
