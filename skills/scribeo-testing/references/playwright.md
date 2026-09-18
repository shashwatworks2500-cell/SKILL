# Playwright Test

Load when authoring or debugging Playwright Test suites.

API names and option values below were verified against the official Playwright documentation (September 2026). **Verify against the version a project actually pins** before relying on any of it — and never invent an API name.

## Playwright Test is not Playwright MCP

| | **Playwright Test** | **Playwright MCP** |
| --- | --- | --- |
| What | A test runner (`@playwright/test`) | Browser control as agent tools |
| Produces | Pass/fail, reports, CI gating | Observations, screenshots, snapshots |
| Owner | **this skill** | `scribeo-visual-qa`, `scribeo-performance` |

They share a name and nothing else. **Never present MCP output as a test result.**

**In this environment:** the global `playwright` CLI is **1.56.1**; `@playwright/test` is **not resolvable from a project here**, so a project must add its own dependency (latest on npm is **1.63.0**, Apache-2.0).

## Locators

Playwright's guidance is explicit: **prefer user-facing attributes to XPath or CSS selectors**, because the DOM changes easily and tests coupled to structure break on refactors.

Preference order:

| Locator | Use |
| --- | --- |
| `getByRole()` | Interactive elements — the default choice |
| `getByLabel()` | Form inputs |
| `getByText()` | Text content |
| `getByTestId()` | When nothing semantic identifies the element |

```ts
// Good: role and accessible name — resilient and meaningful
await page.getByRole("button", { name: "Send enquiry" }).click();
await page.getByLabel("Email address").fill("someone@example.com");

// Bad: coupled to structure and styling
await page.locator(".btn-primary.submit-btn > span").click();
```

**`getByRole` is doubly valuable:** it is resilient, and it only matches if the element exposes the right role and accessible name — so it quietly enforces part of the accessibility contract. If `getByRole("button", { name: "Send enquiry" })` cannot find your control, that is often an accessibility defect, not a locator problem. Route it to `scribeo-accessibility`.

Use `getByTestId` deliberately, not as a default escape hatch. A `data-testid` is a real contract: it must not be deleted in a refactor.

## Web-first assertions

Playwright's assertions on locators are **auto-retrying**: the element is re-tested until the condition holds or the timeout expires. This is what removes most timing races.

```ts
// Good: auto-retries until visible
await expect(page.getByText("Enquiry sent")).toBeVisible();

// Bad: resolves once, immediately — a classic flake
expect(await page.getByText("Enquiry sent").isVisible()).toBe(true);
```

**Auto-retrying assertions** include `toBeAttached()`, `toBeChecked()`, `toBeDisabled()`, `toBeEditable()`, `toBeEmpty()`, `toBeEnabled()`, `toBeFocused()`, `toBeHidden()`, `toBeInViewport()`, `toBeVisible()`, `toContainText()`, `toContainClass()`, `toHaveAccessibleDescription()`, `toHaveAccessibleName()`, `toHaveAttribute()`, `toHaveClass()`, `toHaveCount()`, `toHaveCSS()`, `toHaveId()`, `toHaveJSProperty()`, `toHaveRole()`, `toHaveScreenshot()`, `toHaveText()`, `toHaveValue()`, `toHaveValues()`, and `toMatchAriaSnapshot()`. Page-level ones include `toHaveScreenshot()`, `toHaveTitle()`, `toHaveURL()` and `toMatchAriaSnapshot()`.

**Non-retrying assertions** — `toBe()`, `toEqual()`, `toContain()`, `toThrow()` — execute immediately. Playwright's docs warn directly that using them against asynchronous web content leads to flaky tests. They are correct for plain values, wrong for page state.

The default assertion timeout is **5 seconds**, configured via the `expect` field in the test config.

**`toBeFocused()` and `toHaveAccessibleName()` are the useful pair** for encoding accessibility requirements as regression tests — for example, that focus moves to the first invalid field on a failed submit.

## Auto-waiting

Locator actions perform actionability checks — that the element is visible, enabled, stable — before acting. Combined with auto-retrying assertions, this removes the need for most explicit waits.

**Never use a fixed sleep** where a condition can be awaited. `waitForTimeout` in a test is nearly always a flake waiting to happen; see `flakiness.md`.

## Isolation

Playwright's model: **each test is fully isolated**, with its own storage, cookies and context. The docs are explicit that a test should run independently.

On failure, the worker process and browser instance are discarded and a fresh one starts — so a failing test cannot contaminate a healthy one.

**Do not defeat this** by sharing state between tests through module-level variables or a shared account. If tests genuinely must run in order, `test.describe.serial()` exists — but needing it is usually a design smell, and if one test in a serial block fails the rest are skipped.

## Fixtures

Fixtures provide isolated per-test context — each test receives its own `page` by default. Custom fixtures are the right place for setup that several tests share: a logged-in context, seeded data, a configured page object.

Prefer a fixture over `beforeEach` when the setup produces a value the test uses, and over a helper function when it needs teardown.

## Config essentials

Verified option values:

```ts
export default defineConfig({
  retries: 2,
  workers: 4,
  expect: { timeout: 5000 },
  use: {
    baseURL: "http://localhost:3000",
    viewport: { width: 1280, height: 720 },
    headless: true,
    trace: "on-first-retry",
    screenshot: "only-on-failure",
    video: "retain-on-failure",
  },
  projects: [
    { name: "desktop", use: { ...devices["Desktop Chrome"] } },
    { name: "mobile",  use: { ...devices["iPhone 15"] } },
  ],
});
```

- **`trace`** and **`video`** accept `'off'`, `'on'`, `'retain-on-failure'`, `'retain-on-first-failure'`, `'retain-on-failure-and-retries'`, `'on-first-retry'`, `'on-all-retries'`.
- **`screenshot`** accepts `'off'`, `'on'`, `'only-on-failure'`.
- `projects` is how you run the same tests across viewports or browsers; `devices[…]` supplies device profiles.
- Other `use` options include `storageState` (authenticated context), `locale`, `timezoneId`, `colorScheme`, and `browserName` (`chromium` | `firefox` | `webkit`).

**`trace: "on-first-retry"` is the highest-value single setting** — you get a full trace exactly when something failed and was retried, with no cost on green runs.

## Retries

`retries` is a **diagnostic safety net, not a flake cure.**

Playwright reports a test that **failed first and passed on retry as "flaky"** — a distinct category from passed. That category is a defect list, not a success list.

- Use retries to keep CI usable while you diagnose, and to capture a trace of the failure.
- **Never raise retries to make a red suite green.** See `flakiness.md`.
- `testInfo.retry` is available if a test genuinely needs to know it is on a retry.

## Timeouts

- Set an explicit test timeout rather than relying on a default that may change.
- Set assertion timeouts via `expect.timeout`.
- **Raising a timeout is a legitimate fix only when evidence shows the operation genuinely needs the time** — a large upload, a slow third-party sandbox. Raising it to hide a race is masking a defect.

## Evidence

Configure trace, screenshot and video so a CI failure is diagnosable without reproducing it locally. A failure with no artefact costs a full debugging cycle.

Attach the artefact to the test report (see the report format in `SKILL.md`), and remember the repo `.gitignore` already excludes `test-results/`, `playwright-report/` and `.playwright/`.
