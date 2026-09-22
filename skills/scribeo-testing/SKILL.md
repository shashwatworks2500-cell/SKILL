---
name: scribeo-testing
description: Use when writing, structuring, debugging, or reasoning about automated tests and test architecture — unit tests, component tests, integration tests, end-to-end and browser tests with Playwright Test, Vitest suites, API and route tests, regression and smoke coverage, fixtures and test data, mocking boundaries, test isolation and determinism, flaky-test diagnosis, failure triage, CI test strategy, and coverage interpretation. Triggers on requests like "write tests", "add tests", "create tests", "test this feature", "test this component", "test this page", "test this flow", "add regression coverage", "add a test so this doesn't regress", "fix failing tests", "debug test failure", "flaky test", "why is this Playwright test flaky?", "tests are failing", "this keeps breaking", "E2E testing", "end-to-end test", "unit testing", "component testing", "integration testing", "test architecture", "test strategy", "test suite", "test coverage", "CI tests", "regression testing", "smoke tests", "browser tests", "Playwright tests", or "verify with tests". UX structure and usability belong to scribeo-ux-engineering, animation implementation to scribeo-motion, rendered visual verification to scribeo-visual-qa, performance measurement and budgets to scribeo-performance, accessibility conformance to scribeo-accessibility, SEO to scribeo-seo, and visual design to frontend-design. Not triggered by general implementation, design, UI or animation work — only when automated testing is specifically involved.
---

# Scribeo Testing

Scribeo Studio's testing and test-architecture standard. One loop, in order:

**UNDERSTAND → PLAN → TEST → DIAGNOSE → VERIFY**

Apply this whenever the question is *what observable behaviour must remain correct, how do we automate verifying it, and why is a test failing.*

## The law

**Test the behaviour and contract, not the implementation detail.**

A test exists to answer one question: *if someone changes the code, will this behaviour still hold?* A test coupled to internal structure — function names, class names, component internals, call counts — fails when the code is refactored and passes when the behaviour breaks. That is worse than no test: it costs maintenance and provides false confidence.

**And the corollary that keeps this skill honest:**

> **Passing tests do not prove that the interface is good.**

Testing verifies **defined behaviour** and prevents **regressions**. That is all it does. A green suite does **not** mean the product is:

| Not proven by a green suite | Proven by |
| --- | --- |
| Visually correct | `scribeo-visual-qa` |
| Accessible or conformant | `scribeo-accessibility` |
| Fast, or within budget | `scribeo-performance` |
| Good UX | `scribeo-ux-engineering` |
| Motion-safe or well-choreographed | `scribeo-motion` |
| Discoverable and indexable | `scribeo-seo` |
| Well-designed | `frontend-design` |

Testing is **one verification layer** in the Scribeo system, not the system's verdict. Never report "tests pass" as though it answered any of the questions above.

## Boundaries

**The seam is the question each skill asks:**

| Skill | Asks |
| --- | --- |
| `frontend-design` | What should this look and feel like? |
| `scribeo-ux-engineering` | What should the experience and interaction structure be? |
| `scribeo-motion` | How should movement and animation be implemented? |
| `scribeo-visual-qa` | Does the rendered interface actually look and behave visually as intended? |
| `scribeo-performance` | How fast and efficiently does it perform under measurement? |
| `scribeo-accessibility` | Can people with different access needs perceive, operate, understand, and interact with it? |
| **this skill** | **What observable behaviour or contract must remain correct — and can we reliably automate verifying it?** |
| `scribeo-seo` | Can search engines discover, understand, and appropriately index the content? |

**This skill owns:** test strategy and portfolio reasoning · test layer selection · unit, component, integration, end-to-end and browser test authoring · API and route tests · regression and smoke coverage · fixtures, test data and mocking boundaries · test isolation and determinism · flake diagnosis and failure triage · retries, timeouts and parallelism policy · CI test strategy and the local verification ladder · coverage interpretation · test maintenance.

**This skill does not own:** visual judgement · accessibility conformance judgement · performance budgets or diagnosis · UX decisions · animation choreography · SEO requirements · aesthetic direction. It **automates verification of contracts those skills define.**

### Detecting, deciding, implementing, verifying

Four distinct activities, and conflating them is how ownership leaks:

| Activity | Typically owned by |
| --- | --- |
| **Detecting** a problem | Whichever skill observed it — often `scribeo-visual-qa` |
| **Deciding** the requirement | The domain owner (UX, Accessibility, Performance, Motion, SEO) |
| **Implementing** the fix | The implementing skill for that surface |
| **Verifying** it stays fixed | **this skill**, where the behaviour is stable and automatable |

This skill's centre of gravity is the fourth column. It may also detect (a failing test is a detection) and it authors the automation — but it never decides the requirement.

### Reciprocal handoffs

- **`scribeo-ux-engineering` → here.** Supplies the intended behaviour and the interaction states that matter.
- **here → `scribeo-ux-engineering`.** Tests the observable UX contract. **Never redesigns the experience** to make a test simpler; if a flow is untestable because it is ambiguous, that ambiguity is the finding.
- **`scribeo-motion` → here.** Identifies meaningful interaction states and animation-dependent behaviours that need regression coverage.
- **here → `scribeo-motion`.** Tests stable user-observable outcomes — the panel is open, the content is visible, the control is operable. **Does not assert arbitrary animation internals** (durations, easing, intermediate transforms) unless a value is an explicit, stated contract.
- **`scribeo-visual-qa` → here.** Supplies reproducible behavioural defects that deserve regression tests.
- **here → `scribeo-visual-qa`.** Automates the behavioural checks, then routes **rendered appearance** there. A screenshot assertion is a change-detector, not visual acceptance.
- **`scribeo-performance` → here.** Identifies measurable budgets or regressions that may warrant automated enforcement.
- **here → `scribeo-performance`.** Runs the enforcement where a budget is already defined. **Never draws performance conclusions from test timings** — test-environment timings are contaminated by instrumentation, parallelism and CI noise. Measurement belongs there.
- **`scribeo-accessibility` → here.** Supplies accessibility requirements that can be encoded as regression tests.
- **here → `scribeo-accessibility`.** Automates the repeatable subset. **Automation is never equated with conformance** — that skill states plainly that automated checks catch a minority of real barriers.
- **`scribeo-seo` → here.** Supplies crawlability and indexability contracts that can be regression-tested.
- **here → `scribeo-seo`.** Tests deterministic technical contracts — a title renders, a canonical tag is present and correct, a route returns the expected status. Does not decide SEO strategy.

**If `frontend-design` is unavailable.** It is an Anthropic-provided skill, not part of the `scribeo-skills` marketplace, so it may not be installed. Never invent the aesthetic here to unblock yourself, and never stall work the aesthetic does not gate. Say plainly that the visual direction is unset, ask for it, and proceed with the full test portfolio. Nothing this skill owns depends on the visual direction — tests assert behaviour and contracts, never appearance.

## Workflow

**Phase 1 — Understand**

1. **Establish the behaviour.** What must be true, from the user's or caller's point of view? If nobody can state it, there is nothing to test yet — and that is the finding.
2. **Identify the contract.** What is genuinely promised, versus what happens to be true today? Only the promise is worth encoding.
3. **Identify the critical paths.** The journeys whose failure actually costs something.

**Phase 2 — Plan**

4. **Choose the layer** — the cheapest one that can genuinely verify the behaviour. See `references/strategy.md`.
5. **Decide what deserves a test at all.** Not everything does. A test with no plausible failure mode is maintenance cost with no return.
6. **Plan the negative and edge cases**, not just the happy path. Loading, error, empty, boundary, and rejection paths are where defects live.

**Phase 3 — Test**

7. **Write for diagnosability.** A failure message should identify the problem without opening a debugger.
8. **One meaningful reason to fail** per test, where practical.
9. **Make it deterministic** — explicit data, isolated state, awaited conditions rather than sleeps.

**Phase 4 — Diagnose**

10. **Triage every failure into one of three:** a real defect · a wrong test · a flaky test. Each has a different response, and guessing wastes the cycle.
11. **Flakiness is a defect**, not weather. Diagnose it. See `references/flakiness.md`.

**Phase 5 — Verify**

12. **Confirm the test actually fails** for the reason you think — ideally before the fix exists. A test that cannot fail verifies nothing.
13. **Run the surrounding suite**, not just the new test.
14. **Report honestly** — including skipped, retried and unrun tests.

## Design principles

The rules that decide most reviews:

**Contract, not construction**
- Test what a user or caller can observe. Do not assert internal structure users cannot see, unless the structure *is* the contract (a public API shape, a serialised format).
- Prefer semantic, user-facing selectors. Avoid CSS and class selectors unless the selector itself is the contract.
- **Never weaken a product requirement to make a test pass.** If the test is right and the code is wrong, fix the code; if the requirement changed, change it deliberately and say so.

**Determinism**
- Await conditions; never sleep for a fixed duration when a condition can be awaited.
- Isolate every test — own data, own state, no order dependence.
- Make test data explicit in the test. Hidden shared fixtures are the commonest source of confusing failures.
- **Treat flakiness as a defect requiring diagnosis.** Never hide it behind retries.

**Proportion**
- Mock at real system boundaries — the network, the clock, a third-party service. Not arbitrary internal functions.
- Avoid unnecessary mocks: a heavily-mocked test often verifies only the mocks.
- **Do not write tests to raise a coverage number.** See `references/coverage.md`.
- Add a regression test for a confirmed defect **when the behaviour is stable enough to encode** — and do not overfit it to the implementation that happened to be broken.

## Tooling in this environment

Verified this session, not assumed:

| Tool | Status | Note |
| --- | --- | --- |
| **Playwright CLI** | ✅ `playwright` **1.56.1** installed globally | The `playwright` package. `playwright test --version` responds |
| **`@playwright/test`** | ⚠️ **Not resolvable from a project here** | A project needs its own dependency. Latest on npm is **1.63.0** (Apache-2.0) — the global CLI is behind it |
| **Playwright MCP** | ✅ Connected (`@playwright/mcp` 0.0.81) | **Agent browser inspection — not a test runner.** See below |
| **Chromium** | ✅ 141.0.7390.37, headless | |
| **Vitest** | ❌ Not installed | Latest on npm **5.0.1** (MIT), with `@vitest/coverage-v8` 5.0.1 |
| **Jest** | ❌ Not installed | |
| **TypeScript** | ✅ `tsc` 6.0.2 global | |
| ESLint / Prettier | ✅ 10.1.0 / 3.8.1 global | |
| **axe-core** | ❌ Not installed, and **not to be installed in this task** | `axe-core` / `@axe-core/playwright` 4.13.0 (MPL-2.0) exist on npm |
| `http-server`, `serve` | ✅ global | Serving a static build for browser tests |

**No dependencies were added to this repository, and it contains no `package.json`.** The repo's `.gitignore` already excludes `test-results/`, `playwright-report/`, `.playwright/` and `coverage/` — an existing convention to preserve when a project does add tooling.

**Playwright MCP is not Playwright Test.** MCP is browser control exposed as agent tools, used by `scribeo-visual-qa` and `scribeo-performance` for inspection. Playwright Test is a runner that produces pass/fail in CI, and its architecture is this skill's. They share a name and nothing else. Never present MCP output as a test result.

## Coverage, briefly

Coverage measures **what executed**, not whether behaviour is correct. A line can be covered by a test that asserts nothing.

**Do not treat a coverage percentage as a quality score, and do not adopt a universal Scribeo threshold.** Where a threshold is used it is project-specific engineering policy, justified by that project's risk. Detail and the four coverage types in `references/coverage.md`.

## Test report format

```
Scope:          Contact form submission — /contact
Environment:    Local, Chromium 141 headless, @playwright/test (project-local)
Executed:       14 tests (12 e2e, 2 unit)
Passed:         11
Failed:         2
Skipped:        1  (iOS Safari project — not available in this environment)
Flaky/retried:  1  (passed on retry — logged as a defect, not dismissed)
Defects found:  Empty-email submission leaves focus on the submit button and
                renders no associated error (see accessibility report)
Root cause:     Confident — error node is not linked to the input and focus is
                never moved. Reproduced 5/5 runs
Evidence:       trace.zip (on-first-retry), failure screenshots, run log
Regression:     Added 2 tests — empty-required-field path, and server-error
                path preserving user input
Remaining risk: No cross-browser run (WebKit/Firefox unavailable here).
                Visual appearance of the error not verified — routed to
                scribeo-visual-qa
Next action:    Route the focus/association fix to scribeo-accessibility,
                re-run the 2 failing tests after the fix
```

**Never claim a test ran if it did not run.** Never claim a browser, device or OS was tested if it was not — list it as skipped with the reason. A flaky test is reported as flaky, never folded into "passed".

## Reference routing

`SKILL.md` is the decision core. Load a reference when the work reaches that surface.

| Load | When |
| --- | --- |
| `references/strategy.md` | What deserves a test, portfolio reasoning, choosing the layer |
| `references/layers.md` | What to assert and what to mock at each layer |
| `references/playwright.md` | Playwright Test: locators, web-first assertions, fixtures, config |
| `references/vitest.md` | Vitest: config, assertions, mocking, setup and teardown |
| `references/next-react.md` | Next.js, React, TypeScript, and the GSAP/Motion/Lenis stack |
| `references/fixtures-data.md` | Test data, fixtures, isolation, cleanup, mocking boundaries |
| `references/flakiness.md` | Diagnosing and fixing a flaky test |
| `references/regression.md` | Turning a confirmed defect into durable coverage |
| `references/ci.md` | Verification ladder, CI strategy, artifacts, sharding |
| `references/coverage.md` | Coverage types and how to read them |

## Quality gate

Do not report testing work complete until every line holds.

**Design**
- [ ] Every test asserts observable behaviour or a stated contract
- [ ] No assertions on internals users cannot observe, unless the internal is the contract
- [ ] Semantic selectors used; CSS/class selectors only where the selector is the contract
- [ ] One meaningful reason to fail per test, where practical
- [ ] Failure messages identify the problem without a debugger
- [ ] Negative, edge, and error paths covered — not only happy paths
- [ ] Mocks sit at real system boundaries

**Determinism**
- [ ] No fixed sleeps where a condition can be awaited
- [ ] Tests isolated; no order dependence; no shared mutable state
- [ ] Test data explicit
- [ ] No flakiness masked by retries; every flake diagnosed or logged as a defect

**Verification**
- [ ] Each new test observed to fail for the intended reason
- [ ] Surrounding suite run, not just the new test
- [ ] No product requirement weakened to make a test pass
- [ ] No coverage-driven tests added

**Honesty**
- [ ] Report distinguishes passed, failed, skipped, and flaky
- [ ] Nothing claimed as tested that was not executed
- [ ] Untested browsers, devices and platforms listed explicitly
- [ ] Remaining risk stated, and non-testing verification routed to its owner

**Executed, not assumed** — if a line could not be verified in this environment, say so plainly rather than implying it passed.
