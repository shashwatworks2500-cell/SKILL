# Regression Coverage

Load when turning a confirmed defect into durable coverage.

## When a bug earns a test

**Write a regression test when:** the defect was confirmed and reproduced · the correct behaviour is stable and stateable · it reached (or could reach) users · it is the kind of thing that could return silently.

**Do not write one when:** the behaviour is still being decided · it was a one-off environment problem, not a code defect · the reproduction depends on conditions you cannot recreate deterministically · the "fix" was a design change, in which case the new behaviour gets normal coverage rather than a bug-shaped test.

**A defect with no test is a defect that will return.** But a regression test written against unsettled behaviour gets deleted within a month, so the stability check matters.

## The loop

1. **Reproduce the defect.** If you cannot reproduce it, you cannot verify a fix. This step is not optional.
2. **Encode the stable observable contract** — what *should* happen, from the user's or caller's point of view. Not the mechanism that was broken.
3. **Watch the test fail**, for the right reason. A regression test never observed failing may be asserting nothing. Where practical, write it before the fix.
4. **Fix the root cause** — or route it to the owning skill.
5. **Watch the test pass.**
6. **Run the surrounding suite.** Fixes have non-local effects.
7. **Check you have not overfitted** (below).

Step 3 is the one most often skipped and the most valuable. A test that passes both before and after the fix is testing something else.

## Encode the contract, not the bug

The most common regression-test mistake is encoding the *shape of the original bug* rather than the *behaviour that should hold*. Such a test passes forever and catches nothing, because the next occurrence arrives by a different route.

```ts
// Overfitted: asserts the specific internal that happened to be wrong
expect(form.getAttribute("data-validation-state")).toBe("invalid");

// Contract: asserts what must be true for the user, however it is implemented
await expect(page.getByRole("textbox", { name: "Email address" }))
  .toHaveAttribute("aria-invalid", "true");
await expect(page.getByText("Enter an email address")).toBeVisible();
await expect(page.getByRole("textbox", { name: "Email address" })).toBeFocused();
```

The second survives a rewrite of the validation layer. The first does not, and it would not have caught the defect arriving from a different direction.

**Ask: would this test still be correct if the feature were reimplemented from scratch?** If not, it is overfitted.

## Choosing the layer

Put the regression test at the **lowest layer that genuinely reproduces the defect** — cheaper, faster, and far more diagnosable.

| Defect | Layer |
| --- | --- |
| Wrong calculation or validation result | Unit |
| Component fails to change state on interaction | Component |
| Route accepts invalid input, or returns the wrong status | Integration / API |
| Journey breaks only when the whole system runs | E2E |
| Focus or accessible-name behaviour | Component or E2E, per `scribeo-accessibility`'s requirement |

If a defect reproduces at two layers, take the lower one — and consider whether the higher-layer gap is itself worth covering.

## Four kinds of test, four jobs

| Kind | Purpose | Trigger to run |
| --- | --- | --- |
| **Unit regression** | A specific logic defect cannot return | Every run — it is cheap |
| **Regression test** | A specific confirmed defect cannot return | Every run, or per relevant group |
| **Smoke test** | The deployment is fundamentally alive | Every deploy. Must be fast and rock-solid |
| **Full E2E journey** | A critical path works end to end | Pre-merge or pre-release, depending on runtime |

**A smoke test is not a regression test.** Smoke answers "is this deployment broken?" in seconds. Adding bug-specific checks to the smoke set makes it slow and brittle, and a flaky smoke test trains everyone to ignore a red deploy.

## Regressions from other skills

Defects arrive here from the other Scribeo skills, and each carries a different contract:

| From | What to encode | What not to encode |
| --- | --- | --- |
| `scribeo-visual-qa` | The behavioural part of the defect — content present, control operable, state correct | Appearance. Route that back for visual verification |
| `scribeo-accessibility` | The stated requirement — accessible name, focus movement, `aria-invalid`, reduced-motion reachability | Conformance as a whole; automation is not conformance |
| `scribeo-motion` | The end state and reachability after animation | Durations, easing, intermediate transforms |
| `scribeo-performance` | An already-defined budget, where enforcement is wanted | A timing threshold invented here |
| `scribeo-ux-engineering` | The intended behaviour and state transitions they specify | A redesign of the flow |
| `scribeo-seo` | Deterministic contracts — a title renders, a canonical is correct, a route's status | SEO strategy |

In every case: **they decide the requirement, this skill automates the verification.**

## Verifying the fix

- **Re-run the exact failing test** — not a similar one.
- **Run the surrounding group**, since a fix at one layer can break another.
- **Verify in the same mode** the defect occurred in — same viewport, same project, same state.
- **Route non-testable verification** to its owner: appearance to `scribeo-visual-qa`, conformance to `scribeo-accessibility`, measurement to `scribeo-performance`.
- **Report what was verified and what was not.** A fix verified only by a unit test is verified at one layer; say so.

## Checklist

- [ ] Defect reproduced before writing the test
- [ ] Test observed failing for the right reason
- [ ] Contract encoded, not the bug's internal shape
- [ ] Test would still be correct after a reimplementation
- [ ] Placed at the lowest layer that reproduces it
- [ ] Root cause fixed, not the symptom
- [ ] Surrounding suite run after the fix
- [ ] Non-testable verification routed to its owner
