---
name: scribeo-visual-qa
description: Use when verifying how a website actually renders and behaves in a real browser — visual QA passes, screenshot review, checking a page across viewports and breakpoints, responsive rendering checks, spacing and typography rendering verification, component visual consistency, visual state and animation-state inspection, console and asset error checks, visual regression comparison, and confirming a visual fix actually worked. Triggers on requests like "visually inspect this website", "do a visual QA pass", "check the website on mobile", "compare desktop and mobile", "check for visual bugs", "take screenshots and review them", "find layout issues", "check spacing", "check responsive rendering", "check whether this matches the design", "verify the implementation visually", "find UI inconsistencies", "check the animation states visually", "do a screenshot regression check", "review this page at different breakpoints", "check the site before launch", and on diagnostic reports like "something looks off", "the mobile version looks broken", "this section doesn't look right", or "the layout breaks at this width". This skill inspects and reports; it does not design or implement. Visual design direction belongs to frontend-design, layout and UX structure fixes to scribeo-ux-engineering, animation implementation to scribeo-motion, performance diagnosis to scribeo-performance, WCAG auditing to scribeo-accessibility, test architecture to scribeo-testing, and SEO to scribeo-seo. Not triggered by aesthetic requests or by ordinary build and implementation work unless verification is explicitly asked for.
---

# Scribeo Visual QA

Scribeo Studio's rendered-output verification standard. Answers one question: **does the implemented site actually render and behave as intended?**

Apply this whenever the task is to *look at the real thing* — in a browser, at real viewports, with real content — and report what is actually there.

## The law

**Inspect the rendered result. Never infer it.**

Code that reads correctly renders incorrectly all the time: a font falls back, an image 404s, a container overflows at one width, a sticky header covers an anchor target, hydration produces different markup than the server. None of that is visible in the source.

Three rules follow:

- **"The code looks right" is not a QA result.** If you did not open it in a browser at that viewport in that state, you did not verify it.
- **Report what is there, not what should be there.** A QA pass that describes intent rather than observation is worthless.
- **Never guess the intended design.** If expected behaviour is ambiguous, say so, cite the design source or system, and ask. Inventing a target turns QA into unrequested redesign.

## Boundaries

**This skill owns:** visual inspection of the rendered page · screenshot capture and review · viewport and breakpoint verification · responsive rendering checks · spacing, typography and asset rendering verification · component visual consistency · visual, interaction and animation *state* verification · console and network error observation · visual regression comparison · defect identification, evidence capture and reproducible reporting · verification that a fix actually worked.

**This skill does not own:**

| Concern | Belongs to |
| --- | --- |
| Visual design direction, aesthetics, palette, typeface, composition | `frontend-design` (Anthropic) |
| Layout structure, hierarchy, usability, responsive UX decisions, component states | `scribeo-ux-engineering` |
| Animation design and implementation — timing, easing, choreography, scroll linkage | `scribeo-motion` |
| Performance diagnosis, Core Web Vitals, budgets, profiling | `scribeo-performance` |
| WCAG auditing, assistive-technology verification, conformance | `scribeo-accessibility` |
| Test architecture, strategy, and suite authoring | `scribeo-testing` |
| Metadata, structured data, crawlability | `scribeo-seo` |

**Finding is not fixing.** This skill finds, diagnoses to the point of a reproducible report, and re-verifies. The fix itself is handed to the owning skill. Report the likely owner in every defect; do not implement across the boundary.

**The design boundary, precisely:** a defect is a gap between intended and actual rendering. If there is no stated intent, there is no defect — only a question. "This looks unbalanced" is an aesthetic opinion and belongs to `frontend-design`; "this is 8px from the edge where every other section is 24px" is a defect, because the system states the intent.

**On performance:** report observable symptoms — visible jank, a long blank paint, layout shift on load, a stalled asset — with evidence. Do not profile, diagnose a cause, or set budgets. Route to `scribeo-performance`.

**On accessibility:** report what is *visible* — a clipped or missing focus ring, text over an image at unreadable contrast, a touch target obviously too small. Do not run a conformance audit or test with assistive technology. Route to `scribeo-accessibility`.

**On testing:** Playwright MCP is used here for interactive inspection. Writing durable test suites, choosing a runner, or designing test architecture is `scribeo-testing`. See the distinction in `references/playwright.md`.

## Workflow

**Phase 1 — Establish the target (before opening a browser)**

1. **Establish intent.** The design file, the design system, the spec, the previous release, or the stated requirement. Write down what "correct" means. If it cannot be established, that is finding #1.
2. **Identify routes and states.** Which pages, and which states of them — logged out/in, empty/populated, error, loading.

**Phase 2 — Baseline the environment**

3. **Start from a clean browser state.** This session's Playwright MCP runs `--isolated`, so the profile is in-memory and does not persist between sessions. Within a session, state does carry — reset deliberately.
4. **Verify the page loads at all.** A non-200, a redirect loop, or a blank render ends the pass; report it and stop.
5. **Check console and network before looking.** Errors here explain most visual defects and save a diagnosis cycle.

**Phase 3 — Inspect**

6. **Primary viewport first.** Usually desktop, or whatever the audience actually uses.
7. **Work the viewport matrix.** See `references/viewport.md` — including just above, at, and just below each real breakpoint.
8. **Capture evidence as you go.** Screenshots at the moment of the defect, not reconstructed later.
9. **Inspect systematically** — layout, spacing, typography, assets, states, motion states. Not by scrolling and forming an impression.
10. **Record each defect with evidence** in the report format below.

**Phase 4 — Verify the fix**

11. Hand each defect to the owning skill. *(Not this skill's step — noted so the loop is complete.)*
12. **Re-run the exact failing scenario** — same route, same viewport, same state. Not a similar one.
13. **Re-check adjacent breakpoints.** Responsive fixes routinely break the neighbour.
14. **Regression pass** over what was previously correct. See `references/regression.md`.
15. **Report what remains**, including anything deliberately accepted.

Keep the four activities distinct in your reporting: **finding** a defect, **diagnosing** it, **fixing** it, and **verifying** the fix. Conflating them is how an unverified fix gets reported as done.

## Severity

Severity describes **impact on the user**, never how much it offends the eye.

| Severity | Definition |
| --- | --- |
| **Critical** | Blocks core interaction, makes important content inaccessible, or is a major layout failure. The page cannot do its job. |
| **High** | Major visual break, important responsive failure, broken interaction state, or significant content obstruction. |
| **Medium** | Noticeable inconsistency, localised layout or spacing defect, non-critical rendering issue. |
| **Low** | Minor inconsistency, small alignment or spacing deviation, cosmetic with minimal impact. |

A defect that is ugly but harmless is **Low**. A defect that is invisible to most people but hides the primary CTA on the most common phone width is **Critical**. If you cannot state the user impact, you are reporting taste — reclassify it as a question for `frontend-design`.

## Defect report format

Every defect gets all ten fields. Vague reports cost a round trip.

```
Route:        /pricing
Viewport:     390 × 844 (mobile)
State:        Default, after full load, reduced-motion off
Defect:       Primary CTA wraps to a second line
Expected:     Single-line CTA, per the design system button spec
Actual:       "Book a consultation" wraps; button grows to 88px tall and
              pushes hero body copy below the fold
Evidence:     pricing-390-cta-wrap.png
Severity:     High — primary conversion action pushed out of first view
Likely owner: scribeo-ux-engineering (layout/measure)
Repro:        1. Navigate to /pricing  2. Resize to 390×844
              3. Reload  4. Observe hero CTA. Reproduces every load.
```

```
Bad:   "The mobile version looks broken."
Good:  "At 390px the CTA wraps onto a second line and pushes the hero copy
        below the fold; reproduced after reload."
```

State **"reproduces every load"** or **"intermittent, N of M attempts"**. Intermittent visual defects are usually timing or asset-loading related, and that distinction is the diagnosis.

## Reference routing

`SKILL.md` is the decision core. Load a reference when the pass reaches that surface.

| Load | When |
| --- | --- |
| `references/playwright.md` | Driving the browser — navigation, resizing, screenshots, console, measuring, MCP vs Test |
| `references/viewport.md` | Choosing widths, breakpoint-edge testing, the matrix |
| `references/screenshots.md` | Deterministic capture, waiting, full-page vs viewport, when not to screenshot |
| `references/defects.md` | Defect taxonomy and what evidence each type requires |
| `references/responsive.md` | The responsive verification checklist |
| `references/regression.md` | Baselines, diff review, intentional change, snapshot discipline |
| `references/animation-qa.md` | Verifying motion output and sampling animation states |
| `references/common-bugs.md` | Diagnosing a specific known failure mode |

## Quality gate

Do not report a visual QA pass complete until every line holds.

**Coverage**
- [ ] Intent established and written down, or its absence reported
- [ ] Every target route visited in the browser
- [ ] Full viewport matrix covered, including breakpoint edges
- [ ] Relevant states checked — loading, empty, error, interaction, motion
- [ ] Real content used, not placeholder boxes

**Rigour**
- [ ] Console checked for errors; network checked for failed assets
- [ ] Every defect has evidence attached
- [ ] Every defect has expected, actual, severity, owner, and repro steps
- [ ] Severity reflects user impact, not aesthetic preference
- [ ] Ambiguous intent reported as a question, never resolved by guessing

**Verification**
- [ ] Each fix re-verified in the exact failing scenario
- [ ] Adjacent breakpoints re-checked after any responsive fix
- [ ] Regression pass run over previously-correct areas
- [ ] Remaining and accepted defects listed explicitly

**Observed, not assumed** — every claim traces to something seen in a browser. If a line could not be verified, say so plainly rather than implying it passed.
