# Flaky Tests

Load when a test passes sometimes and fails sometimes.

**A flaky test is a defect requiring diagnosis — not weather.** It is either a real intermittent bug in the product, or a real bug in the test. Both are defects. Neither is "just flaky".

Flakiness is corrosive beyond the test itself: once a suite has flakes, people stop believing red results, and a genuine regression sails through unnoticed. That is the actual cost.

## Prohibited: "just increase the timeout"

**Raising a timeout to make a flake go away is masking a defect.** It converts an intermittent failure into a slower intermittent failure, and hides whatever race is actually there.

A timeout increase is legitimate **only when evidence shows the operation genuinely needs the time** — a large file upload, a cold serverless start you have measured, a third-party sandbox with known latency. "Evidence" means a measurement, not a hunch. Record why.

The same applies to retries: they are a diagnostic safety net so CI stays usable and a trace gets captured. **Raising `retries` to turn a suite green is hiding a defect list.** Playwright reports retry-passes as **"flaky"** precisely so they stay visible.

## The diagnostic process

**REPRODUCE → CLASSIFY → ISOLATE → FIX ROOT CAUSE → RE-RUN → VERIFY STABILITY**

**1 — Reproduce.** Run the test repeatedly until it fails. Run it alone, and in the full suite. Note the failure rate — 1 in 50 and 1 in 3 point at different causes. If you cannot reproduce it, capture evidence on the next CI failure (trace, screenshot, log) rather than guessing.

**2 — Classify.** Use the table below. Classification decides the fix; skipping it produces speculative changes.

**3 — Isolate.** Change one variable. Run alone versus in suite. Run with one worker versus many. Run with the network stubbed. Run with reduced motion. Each answer eliminates a class of cause.

**4 — Fix the root cause.** Not the symptom. If the cause is a race, await the condition. If it is shared state, isolate it. If it is a real product bug, route it to the owning skill and keep the test failing until it is fixed.

**5 — Re-run.** Many times — 20 or more for a rare flake. One green run proves nothing about a 1-in-30 failure.

**6 — Verify stability.** Confirm it holds in CI, in parallel, and over several runs. Only then call it fixed.

## Classification

| Cause | Signature | Fix |
| --- | --- | --- |
| **Improper waiting** | Fails more on slow or loaded machines | Await the condition with an auto-retrying assertion. Delete fixed sleeps |
| **Timing race** | Fails at a consistent low rate, unrelated to load | Find the two things racing; await the real signal, not a proxy |
| **Animation timing** | Fails when clicking or asserting mid-transition | Await the end state, or emulate reduced motion |
| **Shared state** | Passes alone, fails in suite | Isolate. See `fixtures-data.md` |
| **Test-order dependence** | Passes in declaration order, fails randomised | Remove the inherited side effect |
| **Parallel execution** | Fails only with multiple workers | Separate data or accounts per worker |
| **Non-deterministic data** | Fails at month boundaries, in other timezones, occasionally | Pin dates, seed randomness, set timezone and locale |
| **Network instability** | Fails with timeouts on external calls | Stub the third party. Only test what you control |
| **Third-party services** | Fails when someone else's service is slow or down | Mock at the boundary |
| **Fonts and assets** | Layout- or screenshot-sensitive failures | Await fonts and assets before asserting |
| **Viewport differences** | Fails on one project or viewport only | Check the assertion is valid at that size |
| **Stale fixtures** | Fails after unrelated schema or seed changes | Rebuild fixtures; type them |
| **Environment differences** | Passes locally, fails in CI | Compare versions, timezone, locale, CPU, headless vs headed |

## The three commonest causes

**Improper waiting.** A non-retrying assertion against asynchronous content, or a fixed sleep chosen to be "long enough". Playwright's docs warn directly that non-retrying assertions lead to flaky tests.

```ts
// Flaky: resolves once, immediately
expect(await page.getByText("Saved").isVisible()).toBe(true);

// Flaky: "long enough" until the machine is loaded
await page.waitForTimeout(1000);

// Stable: retries until the condition holds
await expect(page.getByText("Saved")).toBeVisible();
```

**Shared state.** The clean diagnostic: run the test alone. Passes alone, fails in suite → it inherited something.

**Animation timing.** A click landing mid-transition, or an assertion reading an intermediate state. Auto-retrying assertions absorb most of it. Where they do not, **emulating `prefers-reduced-motion: reduce` is the cleanest determinism lever** — and it exercises a path `scribeo-accessibility` requires anyway. Never reach for fake timers to stabilise animation.

## Environment differences

"Passes locally, fails in CI" is a class, not a mystery. Check, in order: runner and browser versions · timezone and locale · CPU count and available memory · headless versus headed · worker count · whether the CI machine is slower under parallel load · whether a service reachable locally is unreachable there.

A test that fails only in CI is still a real failure. CI is an environment your product's verification depends on.

## When it is a real product bug

**Sometimes the flake is correct.** An intermittently failing test can be honestly reporting an intermittent defect — a genuine race in the application, a cleanup bug that only bites under load, a duplicate initialisation that depends on timing.

Before concluding the test is at fault, ask: *could a user hit this?* If yes, the test found something valuable. Route the defect to the owning skill and **keep the test failing** until it is fixed. Deleting or skipping it would discard the only detection you have.

## Never do these

- Raise a timeout without measured evidence.
- Raise retries to make a suite green.
- Add `test.skip` with no expiry and no issue.
- Delete a test because it is inconvenient.
- Wrap an assertion in a try/catch that swallows the failure.
- Loop an assertion manually instead of using an auto-retrying one.
- Mark a flake "fixed" after one green run.

## Checklist

- [ ] Reproduced, with a failure rate recorded
- [ ] Classified against the table, not guessed
- [ ] Isolated by changing one variable at a time
- [ ] Root cause identified and fixed — not the symptom
- [ ] Re-run enough times to be meaningful for that failure rate
- [ ] Verified stable in CI and in parallel
- [ ] No timeout or retry increase without measured evidence
- [ ] Product bugs routed to their owner, with the test left failing
