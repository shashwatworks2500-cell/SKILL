# Visual Regression

Load when comparing against a baseline or running a regression pass.

## Baselines

A baseline is only meaningful if it was captured under controlled conditions. Record them alongside it: URL and commit, viewport, scale, state, date.

**Every baseline requires:**

- Fixed viewport and `scale: "css"` — device scale shifts dimensions with the environment and produces a full-image diff
- Fonts loaded before capture
- Animations settled or frozen
- Known scroll position
- Stable content — dynamic dates, counters, randomised or personalised data neutralised
- Clean, known browser state

A baseline captured from an unsettled page bakes a defect in as "correct", and every later comparison inherits it.

## Sources of false positives

These produce diffs with no real change. Eliminate them before trusting any comparison.

| Source | Control |
| --- | --- |
| Font not yet loaded | Await `document.fonts.ready` |
| Animation mid-flight | Freeze, complete, or sample deterministically |
| Timestamps, "N days ago" | Stub, or mask the region |
| Randomised or rotating content | Seed, stub, or exclude |
| Personalised or A/B content | Pin the variant |
| Image lazy-loading | Scroll and wait, or disable |
| Scrollbar presence | Consistent viewport and browser mode |
| Device pixel ratio | `scale: "css"` |
| Video frame | Pause at a fixed time, or poster only |
| Third-party embeds | Stub or exclude the region |

## Diffs need interpretation

**A diff is a signal, not a verdict.** Every difference falls into one of three buckets, and only a human or agent reading it can tell which:

1. **Intended change** — the work did this deliberately. Update the baseline.
2. **Unintended regression** — a real defect. File it.
3. **Noise** — a false positive from the table above. Fix the capture conditions, not the code.

Report them separately. A diff summary that does not distinguish these is unusable.

**Never blanket-update snapshots after a change.** "Update all" is how a genuine regression is silently accepted, and it destroys the baseline's value permanently. Review each diff, classify it, and update only the intended ones — stating what changed and why.

## When regression comparison earns its cost

**Worth it:** stable design systems and component libraries · pre-launch checks on finished pages · verifying a refactor changed nothing visually · high-traffic templates where a silent break is expensive.

**Not worth it:** pages under active design iteration, where everything differs every run · intentionally dynamic content · early-stage builds · anything where "correct" is still being decided.

In those cases, targeted manual inspection of what actually changed is faster and produces better reports. Automated comparison that always shows differences trains everyone to ignore it.

Building and maintaining an automated regression *suite* — the runner, CI integration, storage, review workflow — is `scribeo-testing`'s architecture. This skill defines what a valid baseline is, and interprets the diffs.

## The regression pass

After any fix, before reporting complete:

1. **Re-run the exact failing scenario** — same route, viewport, state. Not a similar one.
2. **Check adjacent breakpoints.** Responsive fixes routinely move the defect one range over.
3. **Re-check what previously passed** on that route, especially anything sharing a component with the fix.
4. **Check the states**, not just the default view — a fix to a hover style can break the resting state.
5. **State scope explicitly:** what was re-verified, and what was not.

A fix verified only in the scenario that was broken is half-verified. Shared components mean a fix in one place surfaces somewhere unrelated.
