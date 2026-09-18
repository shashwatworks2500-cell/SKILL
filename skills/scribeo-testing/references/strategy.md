# Test Strategy

Load when deciding what deserves a test and which layer it belongs in.

## What deserves a test

Not everything. A test with no plausible failure mode is maintenance cost with no return.

**Write a test when:**

- The behaviour is on a **critical path** — the journeys whose failure costs money, trust, or data.
- The logic is **non-obvious** — a calculation, a state machine, a parser, a permission rule.
- A **defect was confirmed** and the behaviour is stable enough to encode (see `regression.md`).
- The behaviour is a **contract** someone else depends on — an API shape, a serialised format, a public component prop.
- It is **easy to break silently** — something with no visible symptom when it regresses.

**Do not write a test when:**

- It only restates the implementation (`expect(sum(2,2)).toBe(4)` where `sum` is `a+b`).
- It exists to move a coverage number.
- The "behaviour" is visual, and the real question is appearance → `scribeo-visual-qa`.
- The behaviour is still being designed. Tests written against an unsettled decision get deleted.
- It asserts a third-party service's behaviour rather than your integration with it.

## The portfolio, not the pyramid

The classic pyramid — many unit, fewer integration, fewest E2E — is a useful default because cost and speed genuinely scale that way. But it is a heuristic, not a rule, and applying it dogmatically produces the wrong suite for a Scribeo marketing site.

**Reason about the portfolio instead.** For each behaviour ask: *what is the cheapest layer that can genuinely verify this?* Then weigh three things:

| Factor | Unit | Component | Integration | E2E |
| --- | --- | --- | --- | --- |
| Speed | Fastest | Fast | Moderate | Slowest |
| Confidence it reflects reality | Lowest | Moderate | Good | Highest |
| Maintenance cost | Lowest | Low | Moderate | Highest |
| Failure diagnosability | Best | Good | Moderate | Worst |

**Shape the portfolio to the project:**

- A **content-driven marketing site** with little logic has almost nothing worth unit testing. Its value is in a handful of E2E smoke tests over critical journeys — the form submits, the nav works, the pages render — plus tests for any genuine logic (pricing, filtering, form validation).
- An **application** with real domain logic inverts this: most value is in unit and integration tests, with E2E reserved for the journeys that cross every layer.

Writing forty unit tests for a brochure site's presentational components is a common and expensive mistake. So is relying solely on E2E for an app with complex rules.

## Choosing the layer

Work down this list and stop at the first layer that can genuinely verify the behaviour:

1. **Is it pure logic with inputs and outputs?** → Unit.
2. **Is it a component's behaviour or state transition?** → Component.
3. **Does it depend on two or more real modules or services collaborating?** → Integration.
4. **Does it only mean anything as a user journey through the real app?** → E2E.

**The trap in both directions.** Pushing a behaviour too low means mocking so much that the test verifies only the mocks. Pushing it too high means a slow, flaky test whose failure tells you nothing about where the problem is.

Detail on what to assert and mock at each layer is in `layers.md`.

## Coverage of behaviour, not of code

Plan test cases from the behaviour, not from the source. For any behaviour worth testing:

| Case | Ask |
| --- | --- |
| **Happy path** | Does the normal case work? |
| **Negative path** | Does it reject what it should reject, with the right result? |
| **Boundaries** | Empty, one, many, maximum, zero, negative, first, last |
| **Error path** | What happens when a dependency fails? |
| **State transitions** | Loading → success. Loading → error. Empty → populated. Populated → empty |

The negative and error paths are where defects actually live, and where suites are usually thinnest. A test file with only happy paths is an incomplete test file.

## Smoke, critical-path, and full coverage

Three different jobs — do not conflate them:

| Kind | Purpose | Scale |
| --- | --- | --- |
| **Smoke** | Is the deployment fundamentally alive? | A handful. Seconds |
| **Critical path** | Do the journeys that matter work? | Small, focused set |
| **Full suite** | Does defined behaviour hold everywhere? | Everything |

Smoke tests run on every deploy and must be fast and rock-solid. If a smoke test is flaky it is worse than useless — it trains everyone to ignore a red deploy.

## Authentication and other conditionals

**Only test what the project actually has.** Do not write authentication tests for a site without authentication, or data-boundary tests for a site with no data layer. Scope the suite to the real system.

Where auth does exist, prefer establishing state directly (a stored session) over driving the login UI in every test — and keep one real test of the login journey itself.

## Test maintenance

A suite is a liability as well as an asset. Review it:

- **Delete tests for deleted behaviour.** A test for a removed feature is noise.
- **Fix or delete a permanently-skipped test.** A skip with no expiry is a lie about coverage.
- **Rewrite a test that breaks on every refactor** — it is coupled to implementation, not behaviour.
- **Investigate a test that has never failed.** Either the behaviour cannot break, or the test cannot detect it.

When a test fails during unrelated work, that is information: either a real coupling exists, or the test is over-specified.
