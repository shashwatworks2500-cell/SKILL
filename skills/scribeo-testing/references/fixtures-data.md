# Fixtures, Test Data & Isolation

Load when setting up test data, deciding what to mock, or diagnosing cross-test contamination.

## Explicit data beats shared data

**A test should be readable on its own.** If understanding a failure requires opening three other files to discover what the data was, the test is badly built.

```ts
// Bad: what is in the fixture? Why does this expect 3?
const enquiries = await loadFixture("enquiries");
expect(summarise(enquiries).urgent).toBe(3);

// Good: the input is visible, so the expectation is checkable
const enquiries = [
  { id: "1", priority: "urgent" },
  { id: "2", priority: "urgent" },
  { id: "3", priority: "normal" },
];
expect(summarise(enquiries).urgent).toBe(2);
```

**Use a factory for the noise, and state the relevant fields inline:**

```ts
const enquiry = (over: Partial<Enquiry> = {}): Enquiry => ({
  id: "test-1", email: "someone@example.com", priority: "normal",
  createdAt: new Date("2026-01-01T00:00:00Z"), ...over,
});

// The test states only what matters to it
expect(isUrgent(enquiry({ priority: "urgent" }))).toBe(true);
```

This keeps the test's *intent* explicit while the irrelevant fields stay out of the way — and a schema change means updating one factory, not fifty literals.

## Isolation

**Every test owns its state.** No test may depend on another having run, and no test may leave anything behind.

| Leak | Consequence |
| --- | --- |
| Shared mutable module state | Order-dependent failures; passes alone, fails in suite |
| Database rows not cleaned up | Counts drift; tests fail as the suite grows |
| Unrestored mocks | A stub leaks into an unrelated test |
| Attached listeners or timers | Memory growth and phantom behaviour |
| Shared login or account | Parallel tests fight each other |
| Cached module singletons | State survives between tests |

**The diagnostic:** a test that passes in isolation and fails in the suite is almost always an isolation failure, not a logic failure. Run it alone to confirm, then find what it inherited.

Playwright provides isolation per test by default — own storage, cookies and context — and discards the worker on failure. **Do not defeat it** with module-level shared variables.

## Order independence

Tests must pass in any order. If they do not, one of them depends on a side effect.

- Do not rely on execution order for setup.
- Do not accumulate state across tests in a file.
- **Randomising test order** where the runner supports it is the cheapest way to find order dependence — if the suite only passes in declaration order, it has a latent defect.

Where ordering is genuinely required, Playwright's `test.describe.serial()` exists — but needing it is usually a design smell, and a failure in a serial block skips the rest.

## Cleanup

Set up and tear down in the same place, so they cannot drift apart.

```ts
beforeEach(async () => { db = await createTestDb(); });
afterEach(async () => { await db.destroy(); vi.restoreAllMocks(); });
```

- **Prefer `beforeEach` to `beforeAll`** for anything mutable.
- **Tear down even when the test failed** — `afterEach` runs regardless; cleanup inside the test body does not.
- **Prefer creating fresh state to resetting shared state.** A reset that misses one field is a slow-burning source of confusion.
- For browser tests, prefer a **fresh context** over clearing cookies by hand.

## Mocking boundaries

**Mock at real system boundaries. Not at arbitrary internal functions.**

| Mock | Because |
| --- | --- |
| Network / HTTP | Non-deterministic, slow, outside your control |
| Third-party SDKs and services | Outside your control; may cost money or rate-limit |
| The clock, where time *is* the behaviour | Otherwise untestable |
| Filesystem, where incidental | Slow and environment-dependent |

| Do not mock | Because |
| --- | --- |
| Your own pure functions | Real is faster and more truthful |
| The module under test's own collaborators | The test then verifies nothing real |
| Child components, usually | Removes the integration you were testing |
| A database, in an integration test | Its real behaviour is the point |

**The smell:** if a test needs five mocks, it is probably verifying its mocks. That is a coupling finding about the code — report it; do not bury it under more mocks.

**Mock the third party's response, not your handling of it.** Playwright's guidance is to only test what you control, and to use route mocking to guarantee the response you need — so your handling runs for real against a controlled input.

## Fixtures in Playwright

Fixtures give each test isolated context — its own `page` by default — and are the right home for shared setup that produces a value.

Prefer a fixture over `beforeEach` when the setup yields something the test uses, and over a plain helper when it needs teardown. Keep a fixture's purpose narrow: one that sets up "everything" makes every test slow and every failure ambiguous.

## Authentication state

Only relevant where a project actually has auth.

- **Establish state directly** — a stored authenticated context — rather than driving the login UI in every test. Playwright's `storageState` exists for this.
- **Keep one real test of the login journey itself.** Bypassing it everywhere means never testing it.
- **Give parallel workers separate accounts** where the system allows, or tests will contend.

## Time and randomness

- **Fix dates explicitly** in fixtures. `new Date()` in a fixture makes the test's behaviour depend on when it runs — and it will eventually fail at a month boundary, in a different timezone, or on a leap day.
- **Seed any randomness**, or replace it with fixed values.
- **Set an explicit timezone and locale** where behaviour depends on them. A test that passes in one timezone and fails in CI is usually this.

## Checklist

- [ ] Relevant data is explicit in the test; factories supply the rest
- [ ] Each test creates and destroys its own state
- [ ] Suite passes in randomised order
- [ ] Mocks restored after every test
- [ ] Mocking only at real system boundaries
- [ ] No shared mutable module state
- [ ] Dates, randomness, timezone and locale pinned
- [ ] Auth state established directly, with one real login test retained
