# WCAG

Load when setting a conformance target or citing a criterion.

All facts below verified against W3C sources (September 2026). **Re-verify before quoting normative values in client-facing work** — standards move, and a wrong citation damages a report more than an uncited observation.

## Current status

- **WCAG 2.2 is a W3C Recommendation** (current revision dated December 2024). It is the version to target.
- **It is backwards compatible.** W3C states plainly: content conforming to WCAG 2.2 also conforms to WCAG 2.0 and WCAG 2.1. Where a policy requires 2.0 or 2.1, conforming to 2.2 satisfies it.
- **Conformance levels are A, AA, and AAA.** AA is the standard target for commercial work and what most legal and procurement requirements reference.
- **WCAG 3.0 is not a conformance standard.** The Accessibility Guidelines Working Group describes it as a major version under development — a multi-year effort. **Never cite WCAG 3 as a requirement, and never present its draft model as current obligation.** Target 2.2 AA.

## New in WCAG 2.2

Nine success criteria are new relative to 2.1:

| Criterion | Level |
| --- | --- |
| 2.4.11 Focus Not Obscured (Minimum) | AA |
| 2.4.12 Focus Not Obscured (Enhanced) | AAA |
| 2.4.13 Focus Appearance | AAA |
| 2.5.7 Dragging Movements | AA |
| 2.5.8 Target Size (Minimum) | AA |
| 3.2.6 Consistent Help | A |
| 3.3.7 Redundant Entry | A |
| 3.3.8 Accessible Authentication (Minimum) | AA |
| 3.3.9 Accessible Authentication (Enhanced) | AAA |

**One criterion was removed:** 4.1.1 Parsing. Do not cite it. Reports still citing 4.1.1 are working from outdated guidance — a useful tell when reviewing a third-party audit.

The three new AA criteria are the ones that change day-to-day work: **2.4.11** (a sticky header must not hide the focused element), **2.5.7** (a drag interaction needs a single-pointer alternative), and **2.5.8** (minimum target size).

## The criteria that carry most real-world weight

Not a substitute for the full spec — the subset that accounts for most findings on a marketing or product site.

**Perceivable**
- 1.1.1 Non-text Content (A) — text alternatives
- 1.3.1 Info and Relationships (A) — semantics convey structure
- 1.3.5 Identify Input Purpose (AA) — `autocomplete`
- 1.4.1 Use of Color (A) — colour never the only signal
- 1.4.3 Contrast (Minimum) (AA) — see `contrast.md`
- 1.4.4 Resize Text (AA) — text scalable to at least 200%
- 1.4.10 Reflow (AA) — see `zoom-reflow.md`
- 1.4.11 Non-text Contrast (AA) — see `contrast.md`
- 1.4.13 Content on Hover or Focus (AA) — dismissible, hoverable, persistent

**Operable**
- 2.1.1 Keyboard (A) — everything operable by keyboard
- 2.1.2 No Keyboard Trap (A)
- 2.2.2 Pause, Stop, Hide (A) — see `media-images.md`
- 2.3.1 Three Flashes or Below Threshold (A)
- 2.4.1 Bypass Blocks (A) — skip link or landmarks
- 2.4.3 Focus Order (A)
- 2.4.4 Link Purpose (In Context) (A)
- 2.4.6 Headings and Labels (AA)
- 2.4.7 Focus Visible (AA)
- 2.4.11 Focus Not Obscured (Minimum) (AA)
- 2.5.3 Label in Name (A)
- 2.5.7 Dragging Movements (AA)
- 2.5.8 Target Size (Minimum) (AA)

**Understandable**
- 3.1.1 Language of Page (A)
- 3.2.2 On Input (A) — no surprising context change
- 3.3.1 Error Identification (A)
- 3.3.2 Labels or Instructions (A)
- 3.3.3 Error Suggestion (AA)
- 3.3.7 Redundant Entry (A)
- 3.3.8 Accessible Authentication (Minimum) (AA)

**Robust**
- 4.1.2 Name, Role, Value (A) — the criterion most custom widgets fail
- 4.1.3 Status Messages (AA) — see `dynamic-content.md`

## Citing correctly

- **Cite only when confident.** A barrier described precisely with no criterion beats a barrier mislabelled with one.
- **State uncertainty as uncertainty.** "Possible 4.1.3, uncertain — depends on whether an announcement is expected here" is a professional finding.
- **Include the level.** A AAA issue is real but is not an AA conformance failure, and conflating them misrepresents the client's obligation.
- **One barrier can breach several criteria.** A `<div>` used as a button can fail 2.1.1, 4.1.2, and 2.4.7 at once. Cite the ones that genuinely apply.
- **Conformance is per-page, and non-interference matters.** Content that fails 2.2.2 or 2.1.2 can break the whole page for some users, regardless of the rest.

## What conformance is not

Meeting every AA criterion does not guarantee a usable experience. Criteria are testable minimums, not a definition of good. A page can be technically conformant and still be exhausting to operate — an accessible-name that is literal but meaningless, a technically-reachable focus order that jumps around the page, a form that passes every check and still confuses everyone.

Report conformance failures **and** usability barriers, labelled distinctly. The second category routes to `scribeo-ux-engineering`.
