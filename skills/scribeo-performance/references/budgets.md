# Performance Budgets

Load when setting, deriving, or enforcing budgets, or preventing regression.

A budget is a number that fails a build or a review. Without enforcement it is an aspiration, and pages get heavier by default — every feature adds, nothing subtracts.

## Derive, never copy

**There are no universal budget numbers**, and inventing authoritative-sounding ones is worse than having none. Derive from:

1. **Page type.** A marketing hero, a product listing, and a checkout have different legitimate weights.
2. **Device target.** The middle of the audience's device range, not the top.
3. **Network assumptions.** The realistic connection for the audience, not office broadband.
4. **Content.** A photography portfolio is legitimately heavier than a text page. The budget reflects the product.
5. **Business importance.** The primary conversion path gets the tightest budget.
6. **Current reality.** A budget nobody can meet is ignored. Start from measured current values.

**The one place authoritative numbers exist is the Core Web Vitals thresholds** (see `core-web-vitals.md`) — LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1, at the 75th percentile, segmented mobile and desktop. Anchor metric budgets to those; derive everything else.

## Two ways to set the first budget

**From current state (incremental).** Measure the page, set the budget at the current value, and forbid regression. This prevents drift immediately and is almost always the right start — it needs no negotiation and creates no failing build on day one.

**From a target (aspirational).** Work backwards from a metric: to hit LCP ≤ 2.5s on the target profile, the critical path can carry roughly *X* bytes. Harder, but it is how you fix a page that is already too heavy.

Use the first to hold the line and the second to plan improvement.

## Budget categories

Set these per route or per template — never site-wide, which hides the offender.

| Category | Measured as | Notes |
| --- | --- | --- |
| **JavaScript** | Transferred and uncompressed bytes per route | Track both: transfer matters on slow networks, uncompressed size predicts CPU cost |
| **Images** | Total bytes above the fold, and per-image maximum | Above-the-fold is the number that affects LCP |
| **Fonts** | Total bytes and file count | File count matters as much as size |
| **Video** | Bytes on the critical path before first paint | Not total file size — what loads before paint |
| **Requests** | Count on the critical path | A proxy for connection and latency overhead |
| **Page weight** | Total transferred, first load, cold cache | The headline number; least diagnostic |
| **Main-thread work** | Total long-task time, longest task | The best predictor of INP |
| **Metrics** | LCP, INP, CLS, plus TTFB as a diagnostic | Anchor to the official thresholds |

Track **uncompressed JavaScript size** alongside transferred size. Compression hides execution cost, and on mobile execution is usually the binding constraint.

## Enforcement

A budget is only real if something fails.

- **In CI** — the durable answer. Fail the build when a route exceeds its budget. **The CI and test architecture is `scribeo-testing`'s domain**; this skill defines the numbers and what they mean.
- **In review** — measure the changed route against the budget before merge.
- **On a schedule** — periodic lab runs catch drift from dependency updates and third-party changes, which no PR touches.
- **In the field** — the only enforcement that reflects reality. Requires RUM.

**Budgets need an owner and a written exception path.** A budget that blocks a release with no way to consciously accept an overage gets deleted within a month. Record the exception, the reason, and the date it is revisited.

## Regression prevention

Performance decays through changes nobody considered performance work:

- A dependency added for one helper.
- A third-party script added by marketing.
- An image committed at full camera resolution.
- A `"use client"` added to a layout, pulling the subtree client-side.
- A dependency upgrade that grew.
- A font weight added for one heading.

**Measure the changed route on every substantive change**, and compare to the recorded baseline. A 15KB increase per PR is invisible individually and doubles the bundle over a quarter.

Keep baselines **versioned with the code**, so a regression can be bisected to a commit rather than argued about.

## Reporting a budget state

```
Route:          /
Budget set:     2026-09-18, derived from measured baseline + CWV targets
Profile:        390 × 844, 4× CPU, Slow 4G, median of 5

Category            Budget    Current   Status
JavaScript (xfer)    180 KB    164 KB    ok
JavaScript (raw)     520 KB    611 KB    OVER  (+91 KB)
Images (ATF)         400 KB    312 KB    ok
Fonts                 90 KB     84 KB    ok
Video (critical)       0 KB      0 KB    ok   (poster-first)
Requests (critical)      25        22    ok
LCP                   2.5s      1.84s    ok
INP                  200ms      —        not measured (needs real interaction)
CLS                    0.1      0.02     ok
Long tasks (total)   300ms      480ms    OVER  (+180ms)

Finding: raw JS and long-task budgets breached together — consistent with a
hydration cost. Next: attribute the longest task. Owner: this skill to
diagnose; implementation likely scribeo-ux-engineering or app code.
```

State **"not measured"** where a value is not measured. Never fill a cell with an estimate — a fabricated number in a budget table becomes a false baseline everyone trusts.

## Do not

- Do not publish numbers as authoritative unless they come from verified official guidance.
- Do not set a budget nobody can meet — it will be ignored, and the discipline dies with it.
- Do not treat a synthetic score as a budget. Score a composite; budget the measurements.
- Do not enforce a budget by degrading usability, accessibility, or intentional design. Present the trade-off and route the decision.
