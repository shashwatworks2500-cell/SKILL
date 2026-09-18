---
name: scribeo-accessibility
description: Use when auditing, fixing, or verifying web accessibility — WCAG conformance, semantic HTML, accessible names and roles, landmarks and heading structure, keyboard navigation and focus management, focus visibility, forms and error handling, dialogs, menus, disclosures, tabs and accordions, dynamic content and status announcements, ARIA, colour and non-text contrast, text resize, zoom and reflow, touch target size, reduced-motion requirements, accessible media, and screen-reader semantics. Triggers on requests like "make this accessible", "audit accessibility", "check WCAG", "fix keyboard navigation", "fix focus", "screen reader support", "add accessible labels", "fix form accessibility", "fix heading hierarchy", "fix contrast", "check tab navigation", "make this dialog accessible", "fix ARIA", "audit this website for accessibility", "test accessibility", "fix accessibility violations", "check accessibility before launch", "make the mobile UI accessible", "implement reduced motion for accessibility", and on diagnostic reports like "keyboard can't reach this", "focus disappears", "screen reader doesn't announce this", "button has no accessible name", "the form error isn't announced", "the contrast is too low", or "zoom breaks the layout". Visual design belongs to frontend-design, UX structure and usability to scribeo-ux-engineering, animation implementation to scribeo-motion, rendered visual verification to scribeo-visual-qa, performance measurement to scribeo-performance, general test architecture to scribeo-testing, and SEO to scribeo-seo. Not triggered by aesthetic requests, and not triggered by general UX or implementation work unless accessibility is specifically involved.
---

# Scribeo Accessibility

Scribeo Studio's accessibility engineering standard. One loop, in order:

**UNDERSTAND → AUDIT → FIX → VERIFY**

Apply this whenever the question is whether people with different access needs can actually **perceive, operate, understand,** and reliably interact with the interface.

## The law

**Accessibility is whether the interface works for someone using it differently from you.**

Not a score. Not a checklist. Four things this skill explicitly refuses to accept as accessibility work:

- **"Lighthouse/axe passed" is not accessible.** Automated tooling catches a minority of real barriers — typically things like missing alt attributes and computable contrast failures. It cannot tell you whether focus order is logical, whether an accessible name is *meaningful*, whether a custom widget's keyboard model makes sense, or whether an announcement arrives at a useful moment.
- **ARIA everywhere is not accessibility — it is usually damage.** The first rule of ARIA is not to use it when native HTML does the job. Incorrect ARIA is worse than none: it overrides real semantics with wrong ones.
- **Contrast alone is not accessibility.** It is one measurable criterion among many, and the easiest to fixate on.
- **"It looks fine to me" is not a measurement.** Perception, operation, and comprehension vary. Measure contrast, traverse with the keyboard, read the accessibility tree.

**Never invent a WCAG criterion to make a report look authoritative.** Cite a criterion only when you are confident it applies. When you are not, describe the barrier and say the exact criterion is uncertain — that is a stronger report than a wrong citation.

## POUR, practically

| Principle | The engineering question |
| --- | --- |
| **Perceivable** | Can it be received through more than one channel? Text alternatives, sufficient contrast, content that survives zoom, media alternatives, information not carried by colour alone |
| **Operable** | Can it be driven without a mouse? Keyboard reachable and operable, no traps, adequate targets, enough time, nothing that flashes dangerously |
| **Understandable** | Is behaviour predictable and are errors recoverable? Labels and instructions, consistent navigation, errors that explain the fix, no surprising context changes |
| **Robust** | Will assistive technology parse it correctly? Valid semantics, correct name/role/value, state changes exposed programmatically |

## Boundaries

**This skill owns:** accessibility requirements and WCAG-oriented audits · semantic HTML decisions for accessibility · accessible names, roles, states and descriptions · landmarks and heading structure · keyboard access and focus management · focus visibility · forms, labels, instructions, errors and error recovery · dialogs, menus, disclosures, tabs, accordions · dynamic content and status announcements · ARIA · colour and non-text contrast requirements · text resize, zoom and reflow · touch target accessibility · reduced-motion requirements from an accessibility standpoint · accessible media behaviour · accessibility testing and regression prevention · assistive-technology considerations.

**This skill does not own:**

| Concern | Belongs to |
| --- | --- |
| Visual design direction, aesthetics, palette and typeface selection, composition | `frontend-design` (Anthropic) |
| UX architecture, information hierarchy, usability, responsive UX structure, conversion UX | `scribeo-ux-engineering` |
| Animation implementation, timing, easing, scroll choreography | `scribeo-motion` |
| Visual inspection, screenshot QA, rendered visual defects, visual regression | `scribeo-visual-qa` |
| Performance measurement, Core Web Vitals, diagnosis, budgets | `scribeo-performance` |
| General test architecture, strategy, suites, CI | `scribeo-testing` |
| SEO architecture, metadata, structured data, indexing | `scribeo-seo` |

### The decision seam

Accessibility and UX overlap legitimately, and trying to eliminate the overlap produces worse skills. The seam is the **question being asked**:

> **UX asks:** is this interaction understandable and usable?
> **Accessibility asks:** can people with different access needs actually perceive, operate, understand, and interact with it?

Worked examples:

| Question | Owner |
| --- | --- |
| Does this navigation structure make sense? | `scribeo-ux-engineering` |
| Can keyboard users reach and operate this navigation? | **here** |
| Should this section animate into view? | `scribeo-motion` / `scribeo-ux-engineering` |
| Can a reduced-motion user still understand and operate it? | **here** (with `scribeo-motion`) |
| Does the animation drop frames? | `scribeo-performance` |
| Is the animation visibly broken? | `scribeo-visual-qa` |
| Does the animated state remain accessible? | **here** |

A shared concept is not a conflict. When a barrier's *fix* is structural — reordering content, changing a component's interaction model — report the accessibility requirement and route the structural decision to `scribeo-ux-engineering`.

### Reciprocal handoffs

- **`scribeo-motion` → here.** Motion implementation must respect accessibility constraints. Motion carries authoring-time reduced-motion guidance; this skill sets the requirement and audits conformance.
- **here → `scribeo-motion`.** When motion creates a barrier, state the requirement and the evidence; the animation change is Motion's to implement. Do not rewrite timelines here.
- **`scribeo-performance` → here.** Performance must never silently trade away accessibility. Any optimization touching focus, semantics, labels, or motion routes here first.
- **here → `scribeo-performance`.** Accessibility fixes should be performance-aware, but are **never compromised merely to save bytes or milliseconds.** Ask Performance for a cheaper implementation of the same accessible behaviour, not for a less accessible one.
- **`scribeo-visual-qa` → here.** QA reports visible symptoms — clipped focus rings, unreadable overlay text, obviously small targets — with evidence and no conformance judgement. Judging and fixing is this skill's.
- **here → `scribeo-visual-qa`.** Any accessibility change that alters rendering — focus styles, contrast adjustments, visible labels, skip links — must be visually verified. Route it.
- **`scribeo-ux-engineering` → here.** UX structure must account for accessibility: semantics, logical order, visible focus, adequate targets, labelled controls.
- **here → `scribeo-ux-engineering`.** When a UX pattern is the barrier, the structural decision is theirs; supply the requirement, not a redesign.

## Workflow

**Phase 1 — Understand (before auditing)**

1. **Establish scope.** Which routes, components, and states. Which conformance target (usually WCAG 2.2 Level AA) and whose requirement it is.
2. **Understand the intended behaviour.** You cannot judge whether a control behaves correctly without knowing what it is for.
3. **Identify the critical paths.** The tasks a user must complete. A barrier on the primary conversion path outranks a technically-identical one in the footer.

**Phase 2 — Audit**

4. **Automated pass first**, for breadth — and treat the result as a floor, never a verdict.
5. **Keyboard traversal.** Tab the whole page. This finds more real barriers than any tool.
6. **Accessibility tree inspection.** Read what is actually exposed: roles, names, states.
7. **Contrast and zoom measurement.** Measured values, not impressions.
8. **State and dynamic behaviour.** Open, closed, error, loading, empty. Announcements at the right moment.
9. **Record each barrier with evidence** in the report format below.

**Phase 3 — Fix**

10. **Prefer native semantics.** Reach for ARIA only to repair a genuine gap.
11. **Fix the cause, not the symptom.** An `aria-label` papering over a `<div>` that should be a `<button>` leaves the keyboard barrier in place.
12. **Route what you do not own** — structural changes to UX, animation changes to Motion.

**Phase 4 — Verify**

13. **Re-test the exact failing scenario**, in the same mode of interaction.
14. **Re-test the surrounding flow.** Focus and semantics changes have non-local effects.
15. **Verify rendering** via `scribeo-visual-qa` where the fix changed visuals.
16. **Record what was verified and what was not** — especially anything needing real assistive technology.

## Tooling in this environment

Verified in this session, not assumed:

| Capability | Status | Use |
| --- | --- | --- |
| **Accessibility tree** | ✅ `browser_snapshot` (Playwright MCP) | Roles, names, structure — the single most useful available probe |
| **Keyboard traversal** | ✅ `browser_press_key` | Tab / Shift+Tab / Enter / Space / Escape / arrows |
| **Computed styles, DOM probes** | ✅ `browser_evaluate` | Contrast inputs, focus styles, attribute audits |
| **Visual capture** | ✅ `browser_take_screenshot` | Focus-ring evidence, zoom and reflow evidence |
| **Chromium** | ✅ 141.0.7390.37, headless | The engine under test |
| **axe-core** | ❌ **Not installed** (`axe` not on PATH; `axe-core` 4.13.0 and `@axe-core/playwright` 4.13.0, MPL-2.0, exist on npm) | Adding it is a project dependency decision for the user, not an assumption |
| **Lighthouse / pa11y** | ❌ **Not installed** | Same |
| **Real screen readers** | ❌ **Unavailable here** — no NVDA, JAWS, VoiceOver or TalkBack | Screen-reader *behaviour* cannot be verified in this environment |
| **Real assistive tech, voice control, switch access** | ❌ Unavailable | State as unverified rather than implying coverage |

**Never claim a screen-reader result you did not observe.** The honest form is: "the accessibility tree exposes role `button`, name 'Close dialog'; actual announcement not verified — no screen reader available in this environment."

## Severity

Impact-based. Severity describes the barrier's consequence for a user, never a preference.

| Severity | Definition |
| --- | --- |
| **Critical** | Blocks an essential task for an input or assistive mode; a serious keyboard trap; critical content or controls unreachable |
| **High** | Major barrier; an important interaction cannot be operated or perceived in a given mode |
| **Medium** | Significant but localised barrier, or one with a viable workaround |
| **Low** | Minor issue with limited impact |

A keyboard trap in a newsletter modal is **Critical** — it strands the user on the whole page. A missing `alt` on a decorative flourish is **Low**.

## Accessibility defect report

```
Route:        /contact
Component:    Enquiry form — submit flow
State:        After submitting with an empty required email field
Mode:         Keyboard only; accessibility tree inspected
Viewport:     1280 × 800
Issue:        Validation error is rendered visually but not programmatically
              associated with the field, and focus stays on the submit button
Expected:     Focus moves to the first invalid field; the field is marked
              aria-invalid and describedby the error text; error is announced
Actual:       Error appears as a sibling <p> with no association; input has no
              aria-invalid; focus unmoved. Tree shows textbox "Email address"
              with no description
WCAG:         3.3.1 Error Identification (A) — confident.
              Possible 4.1.3 Status Messages (AA) — uncertain, depends on
              whether an announcement is expected here
Evidence:     a11y-tree-contact-error.txt; keyboard-traversal notes
Severity:     High — a keyboard or screen-reader user cannot discover why
              submission failed
Repro:        1. Tab to email  2. Leave empty  3. Tab to submit  4. Enter
              5. Observe focus and tree. Reproduces every attempt
Owner:        this skill (association + focus); scribeo-ux-engineering if the
              error-summary pattern changes
```

```
Bad:   "The form isn't accessible."
Good:  "Submitting with an empty required field renders an error that is not
        programmatically associated with the input and does not move focus, so
        a keyboard user cannot discover the cause (3.3.1, confident)."
```

Always state the **mode** — keyboard, pointer, tree inspection, zoom — and whether real assistive technology was used. A finding without a mode is not reproducible.

## Reference routing

`SKILL.md` is the decision core. Load a reference when the audit reaches that surface.

| Load | When |
| --- | --- |
| `references/wcag.md` | Conformance targets, versions, criteria that matter, citing correctly |
| `references/semantics.md` | Element choice, landmarks, headings, lists, tables; native before ARIA |
| `references/names-roles-states.md` | Accessible names, descriptions, roles, state exposure, ARIA rules |
| `references/keyboard-focus.md` | Traversal, order, traps, skip links, focus visibility and restoration |
| `references/forms.md` | Labels, instructions, required fields, errors, error summaries, grouping |
| `references/dynamic-content.md` | Live regions, status messages, async updates, client-side routing |
| `references/components.md` | Dialogs, menus, disclosures, tabs, accordions, tooltips |
| `references/contrast.md` | Colour and non-text contrast requirements and measurement |
| `references/zoom-reflow.md` | Text resize, zoom, reflow, target size, pointer and touch |
| `references/motion.md` | Reduced motion, vestibular safety, and the Motion handoff |
| `references/media-images.md` | Alt text decisions, captions, transcripts, autoplay, flashing |
| `references/testing.md` | Audit procedure, what automation catches, manual checks, regression |

## Quality gate

Do not report accessibility work complete until every line holds.

**Coverage**
- [ ] Conformance target stated (version and level)
- [ ] Full keyboard traversal performed on every audited route
- [ ] Accessibility tree inspected, not just the DOM
- [ ] Contrast measured, not eyeballed
- [ ] Zoom and reflow checked
- [ ] Dynamic states checked — open, closed, loading, error, empty
- [ ] Reduced-motion path checked

**Rigour**
- [ ] Automated results treated as a floor, never a verdict
- [ ] Every barrier has evidence and a stated interaction mode
- [ ] WCAG criteria cited only where confident; uncertainty stated as uncertainty
- [ ] Severity reflects user impact, not preference
- [ ] No screen-reader behaviour claimed that was not observed

**Fixes**
- [ ] Native semantics preferred; ARIA only where it repairs a real gap
- [ ] No `outline: none` without an equal-or-better replacement
- [ ] Causes fixed, not symptoms papered over
- [ ] Structural and motion changes routed to their owners, not implemented here
- [ ] No accessibility compromised for performance or aesthetics

**Verification**
- [ ] Each fix re-tested in the exact failing scenario and mode
- [ ] Surrounding flow re-tested for focus and semantic side effects
- [ ] Rendering changes verified via `scribeo-visual-qa`
- [ ] Remaining and unverifiable items listed explicitly

**Observed, not assumed** — if a line could not be verified in this environment, say so plainly rather than implying it passed.
