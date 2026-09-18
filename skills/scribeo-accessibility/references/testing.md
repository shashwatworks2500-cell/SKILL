# Accessibility Testing

Load when planning an audit, or deciding what can actually be verified.

## What automation catches

**Automated tooling finds a minority of real accessibility barriers.** It reliably detects computable facts: a missing `alt` attribute, an unlabelled input, a computable contrast failure, an empty button, a duplicate `id`, an invalid ARIA attribute.

**It cannot detect** whether an accessible name is *meaningful* · whether focus order is logical · whether a custom widget's keyboard model matches user expectation · whether an announcement arrives at a useful moment · whether alt text describes the right thing · whether an error message explains the fix · whether reduced motion leaves content reachable · whether the experience is actually usable.

**Never report "axe passed" as "accessible."** The honest form is: "automated checks found no violations; manual keyboard, tree, contrast and zoom testing found N barriers." Automation is a floor that catches regressions cheaply — it is not a verdict.

## The boundary with scribeo-testing

| Owner | Owns |
| --- | --- |
| **here** | Accessibility audit procedure, what to test, how to interpret results, manual verification, accessibility regression checks |
| `scribeo-testing` | Test architecture, runner choice, CI integration, suite organisation, broader automation strategy |

Writing a throwaway script to reproduce a barrier is this skill's. Designing the durable suite that runs it in CI is `scribeo-testing`'s.

## Available in this environment

Verified this session:

| Capability | Status |
| --- | --- |
| Accessibility tree (`browser_snapshot`) | ✅ Roles, names, structure — the most useful available probe |
| Keyboard traversal (`browser_press_key`) | ✅ Tab, Shift+Tab, Enter, Space, Escape, arrows |
| DOM and computed-style probes (`browser_evaluate`) | ✅ Contrast, focus styles, attribute audits, target sizes |
| Screenshots | ✅ Focus-ring, zoom and reflow evidence |
| Chromium 141 headless | ✅ |
| **axe-core** | ❌ **Not installed.** `axe-core` 4.13.0 and `@axe-core/playwright` 4.13.0 (MPL-2.0) exist on npm — adding them is a project dependency decision for the user |
| **Lighthouse, pa11y** | ❌ Not installed |
| **Real screen readers** (NVDA, JAWS, VoiceOver, TalkBack) | ❌ **Unavailable** — screen-reader behaviour cannot be verified here |
| Voice control, switch access, magnification | ❌ Unavailable |

**Say what was not tested.** A report implying screen-reader verification that did not happen is worse than one stating the limit plainly.

## Manual audit procedure

Run in this order — cheapest and highest-yield first.

**1. Keyboard traversal.** Tab the whole page. Record every stop, whether focus is visible, whether order makes sense, whether anything is unreachable or traps you. **Highest-value single test.** See `keyboard-focus.md`.

**2. Accessibility tree.** `browser_snapshot`. Check every interactive element has a sensible role and a meaningful name; check headings form a coherent outline; check landmarks are present and labelled.

**3. Automated pass**, if tooling is available — for breadth and regression value.

**4. Contrast measurement.** Computed values, on every background variant. See `contrast.md`.

**5. Zoom and reflow.** 200% text, and 1280px at 400% zoom. See `zoom-reflow.md`.

**6. States and dynamics.** Open, closed, loading, error, empty, success. Are changes announced? Does focus move correctly? See `dynamic-content.md`.

**7. Reduced motion.** Emulate it, confirm nothing is unreachable. See `motion.md`.

**8. Forms.** Labels, errors, focus after failed submit, success announcement. See `forms.md`.

**9. Targets and pointer.** Sizes, spacing, drag alternatives, hover-only content.

## Screen-reader testing

Cannot be performed in this environment — but state honestly what it would add, because the accessibility tree is not a substitute for real behaviour.

- **No single browser/screen-reader pairing represents all users.** Behaviour differs meaningfully between NVDA + Firefox, JAWS + Chrome, VoiceOver + Safari, and TalkBack on Android.
- Announcement order, verbosity, and live-region handling all vary.
- A page can look correct in the accessibility tree and announce confusingly in practice.
- Where a project's risk warrants it, recommend testing with at least two pairings, and with users who actually use them. That recommendation is a deliverable in itself.

## Accessibility regression

Accessibility decays through changes nobody considered accessibility work. Establish a **baseline → change → re-test** loop.

| Regression type | Cheapest guard |
| --- | --- |
| **Keyboard** | Traverse the critical path after any interactive change |
| **Focus** | Check focus styles still render and are not clipped; check restoration after dialogs |
| **Semantic** | Re-read the accessibility tree; watch for `<div>`s replacing controls |
| **Accessible name** | Re-run the unnamed-control probe after refactors |
| **Contrast** | Re-measure after any palette or theme change |
| **Responsive** | Re-check reflow after layout work |
| **Reduced motion** | Re-check after any motion change |
| **Forms** | Re-test error and success paths after validation changes |

The changes that most often cause regressions: a refactor replacing a `<button>` with a styled `<div>` · a design tweak lightening text colour · a new component with no keyboard handling · a modal added without focus management · `outline: none` reintroduced in a CSS cleanup · a client-side routing change dropping focus management.

**Automated checks are genuinely valuable here** — cheap to run, and they catch the mechanical subset reliably. Suite and CI design routes to `scribeo-testing`.

## Reporting an audit

State scope, target, method, and limits up front:

```
Scope:        /, /work, /contact — desktop 1280×800 and mobile 390×844
Target:       WCAG 2.2 Level AA
Method:       Keyboard traversal, accessibility tree inspection, computed
              contrast measurement, 400% zoom reflow, reduced-motion emulation
Not tested:   Real screen readers (unavailable in this environment);
              automated axe pass (axe-core not installed); voice control;
              switch access
Findings:     3 Critical, 5 High, 4 Medium, 2 Low — detail below
```

Then each barrier in the report format from `SKILL.md`. **Cite WCAG criteria only where confident**, and mark uncertainty as uncertainty.

## Audit checklist

- [ ] Conformance target stated
- [ ] Keyboard traversal completed on every route
- [ ] Accessibility tree inspected
- [ ] Contrast measured on every background variant
- [ ] Zoom 200% and reflow at 400% verified
- [ ] Dynamic states and announcements checked
- [ ] Reduced-motion path verified
- [ ] Automated results labelled as a floor, not a verdict
- [ ] Untested areas listed explicitly, including screen readers
- [ ] No claimed result that was not observed
