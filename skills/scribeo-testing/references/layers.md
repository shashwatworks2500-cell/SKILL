# Test Layers

Load when writing a test at a specific layer — what to assert, and what to mock.

The layer decision itself is in `strategy.md`. This file covers execution.

## Unit tests

**For isolated, deterministic logic** — a function, a reducer, a validator, a formatter, a calculation.

**Assert:** the output for a given input; the error for invalid input; boundary behaviour.

**Mock:** almost nothing. If a unit test needs heavy mocking, the unit has too many dependencies — that is a design finding, not a testing problem.

```ts
// Good: behaviour, boundaries, and the error path
expect(formatPrice(1234.5, "GBP")).toBe("£1,234.50");
expect(formatPrice(0, "GBP")).toBe("£0.00");
expect(() => formatPrice(NaN, "GBP")).toThrow();
```

**Avoid:** asserting that an internal helper was called · restating the implementation · testing a trivial passthrough.

Unit tests are the cheapest to run and the easiest to over-produce. Their value is concentrated in logic that is genuinely non-obvious.

## Component tests

**For a component's behaviour and state transitions**, rendered but isolated from the full app.

**Assert** what a user could observe:

- Rendered output for given props
- The result of an interaction (click → panel opens)
- Conditional rendering (empty state vs populated)
- Accessible output — role and name — where the component's contract includes it
- Callbacks fired with the expected arguments

```ts
// Good: observable behaviour, semantic query
await user.click(screen.getByRole("button", { name: "Show details" }));
expect(screen.getByRole("region", { name: "Details" })).toBeVisible();
```

**Mock:** the network, and genuinely external modules. **Not** child components — replacing children with stubs usually removes the very integration the test was for.

**Avoid:** asserting internal state, hook call counts, or class names · snapshotting whole component trees as a substitute for assertions (a change-detector that everyone eventually updates without reading).

Framework-specific guidance is in `next-react.md`.

## Integration tests

**For two or more real modules or services collaborating** — a route handler with its validation and data layer, a service with a real database, a form submission through to persistence.

**Assert:** the observable outcome of the collaboration — the response status and body, the persisted record, the emitted event.

**Mock:** only what is genuinely external and outside your control — a third-party API, an email provider, a payment gateway. Keep your own modules real; that is the entire point of the layer.

```ts
// Good: real route + real validation, outcome asserted
const res = await app.request("/api/enquiry", {
  method: "POST",
  body: JSON.stringify({ email: "", message: "hello" }),
});
expect(res.status).toBe(422);
expect(await res.json()).toMatchObject({ errors: { email: expect.any(String) } });
```

**Avoid:** mocking your own modules until the test verifies nothing real · asserting a third party's behaviour instead of your handling of it · sharing a database state between tests (see `fixtures-data.md`).

## End-to-end / browser tests

**For critical user journeys through the real, running application.**

**Assert:** what the user sees and can do — the confirmation appears, the URL changed, the record exists, the error is shown.

**Mock:** as little as possible, but **do stub third parties you do not control.** Playwright's own guidance is explicit: only test what you control, and use route mocking to guarantee the response you need. A test that fails because someone else's API was slow is not testing your app.

```ts
// Good: journey, semantic locators, web-first assertion
await page.goto("/contact");
await page.getByLabel("Email address").fill("someone@example.com");
await page.getByLabel("Message").fill("Interested in a project.");
await page.getByRole("button", { name: "Send enquiry" }).click();
await expect(page.getByText("Enquiry sent")).toBeVisible();
```

**Keep E2E scarce and load-bearing.** Each one is slow, has the widest failure surface, and produces the least diagnosable failure. Ten well-chosen E2E tests beat a hundred that nobody trusts.

Runner specifics are in `playwright.md`.

## Visual testing — the boundary

Playwright provides `toHaveScreenshot()`, an auto-retrying assertion that compares against a stored baseline.

**What it is:** a **change detector**. It tells you pixels differ from a baseline. It cannot tell you whether the new pixels are correct.

**What it is not:** visual acceptance. Deciding whether a rendered result is right is `scribeo-visual-qa`'s judgement, and that skill defines what a valid baseline requires (fonts loaded, animations settled, deterministic content, `scale: "css"`).

Use screenshot assertions to catch **unintended** visual change on stable, deterministic surfaces. Do not use them as a substitute for functional assertions — a screenshot test cannot tell you the form submitted. And never blanket-update baselines; that is how a real regression is silently accepted.

## Accessibility automation — the boundary

Playwright exposes accessibility-relevant auto-retrying assertions including `toHaveAccessibleName()`, `toHaveAccessibleDescription()`, `toHaveRole()` and `toMatchAriaSnapshot()`. These are genuinely useful regression guards — an accessible name disappearing in a refactor is exactly the mechanical failure automation catches well.

**But automation is not conformance.** `scribeo-accessibility` states plainly that automated checks catch a minority of real barriers, and cannot judge whether a name is *meaningful*, whether focus order is logical, or whether an announcement arrives usefully.

So: encode the requirements that skill gives you, run them cheaply and often, and **never report a passing accessibility test as an accessibility result.** axe-core integration is available on npm but is not installed here.

## Performance tests — the boundary

A test can **enforce a budget that `scribeo-performance` has already defined** — a bundle size ceiling, a request count, a metric threshold in a controlled run.

It cannot **diagnose** performance, and **test timings are not performance measurements.** Instrumentation, parallel workers, and CI machine variance all contaminate them. A test asserting "the page loaded in under 2 seconds" will be flaky, will be papered over with a longer timeout, and will teach the team nothing.

Enforce known budgets; route measurement and diagnosis to `scribeo-performance`.

## API and route tests

Where a project has routes or an API, they are usually the highest-value integration layer: fast, deterministic, and covering real contracts.

**Assert:** status codes · response shape · validation rejections · authorisation behaviour where auth exists · error responses.

These tests often replace a large number of E2E tests at a fraction of the cost — the journey through the UI needs one E2E test, while every validation branch can be an API test.
