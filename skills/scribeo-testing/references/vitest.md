# Vitest

Load when authoring unit, component or integration tests with Vitest.

Verified against the official Vitest documentation (September 2026): Vitest is described as a next-generation testing framework powered by Vite; current published version is **5.0.1** (MIT), with `@vitest/coverage-v8` at the same version. **Not installed in this environment** — adding it is a project decision.

**Verify config fields against the version a project pins.** Where the official reference did not confirm a detail, this file says so rather than guessing.

## When Vitest is the right tool

**Use it for:** pure logic · reducers, validators, formatters, calculations · component behaviour in a DOM environment · integration between your own modules · anything that does not need a real browser.

**Do not use it for:** real cross-browser behaviour · genuine user journeys · anything depending on real layout, real scrolling, or real browser APIs at fidelity. Those are Playwright's (`playwright.md`).

**Do not force Vitest into a project that does not need it.** A content-driven marketing site with no logic layer gains nothing from a unit-test runner. Its value is in E2E smoke coverage. Adding a test runner and its config to such a project is cost without return.

Vitest's natural fit is a project already using Vite — it reads `vite.config.*` by default, so existing plugins and aliases work without duplication.

## Config

Vitest accepts `vite.config.*` or a dedicated `vitest.config.*`.

Confirmed fields from the config reference:

| Field | Purpose |
| --- | --- |
| `environment` | Selects the test environment. **Confirm the accepted value list for your version** — `node` and a DOM environment such as `jsdom` are the usual choices; do not assume a value without checking |
| `include` | Test file patterns |
| `setupFiles` | Files run before the suite |
| `globals` | Whether test APIs are available without import |
| `coverage` | Coverage configuration, including the provider |
| `isolate` | Test isolation behaviour |

**Prefer explicit imports over `globals: true`.** Explicit imports keep TypeScript honest about what is in scope and make a test file readable on its own.

## Core API

```ts
import { describe, test, expect, beforeEach, afterEach, vi } from "vitest";

describe("formatPrice", () => {
  test("formats GBP to two decimal places", () => {
    expect(formatPrice(1234.5, "GBP")).toBe("£1,234.50");
  });

  test("throws on a non-numeric amount", () => {
    expect(() => formatPrice(NaN, "GBP")).toThrow();
  });
});
```

`test` and `it` are interchangeable — pick one per project and stay consistent.

**Write the assertion so the failure is readable.** `toEqual` on a whole object produces a useful diff; a chain of booleans produces "expected true, received false", which tells you nothing.

## Mocking boundaries

Vitest provides `vi.fn()` (mock function), `vi.spyOn()` (wrap an existing method), and `vi.mock()` (replace a module).

**The rule that matters more than the API:** mock at **real system boundaries**, not arbitrary internal functions.

| Mock this | Do not mock this |
| --- | --- |
| Network calls (`fetch`, an HTTP client) | Your own pure functions |
| A third-party SDK | Your own modules under test |
| The clock, where time is the behaviour | Child components, usually |
| The filesystem | Anything whose real behaviour is the point |

```ts
// Good: the network boundary is mocked, your logic is real
vi.mock("./api-client", () => ({ fetchEnquiries: vi.fn().mockResolvedValue([]) }));

// Bad: mocking the thing under test's collaborator until nothing real runs
vi.mock("./validate-enquiry");
```

**A heavily mocked test usually verifies its mocks.** If a test needs five mocks to run, that is a design finding about coupling — report it; do not bury it under more mocks.

**Restore between tests.** `vi.restoreAllMocks()` / `vi.resetAllMocks()` in teardown, or the equivalent config option, prevents a mock from leaking into the next test — a common cause of order-dependent failures.

## Setup and teardown

```ts
let db: TestDb;
beforeEach(async () => { db = await createTestDb(); });
afterEach(async () => { await db.destroy(); });
```

- **Prefer `beforeEach` over `beforeAll`** for anything mutable. `beforeAll` state shared across tests creates order dependence.
- **Always tear down** what you set up — timers, listeners, DOM nodes, database rows, mocks.
- Keep setup minimal and visible. Setup hidden three files deep makes failures hard to read; see `fixtures-data.md`.

## Determinism

- **No real network.** Mock at the boundary.
- **No real time dependence.** If behaviour depends on the clock, control the clock — but only then. **Do not introduce fake timers unless genuinely required**; they change the execution model and cause confusing interactions with promises and framework scheduling.
- **No random data** without a fixed seed.
- **No shared mutable module state** between test files.
- **`isolate`** controls isolation behaviour; understand what a project has set before diagnosing a cross-test leak.

## Coverage

`@vitest/coverage-v8` provides the V8 provider. Enable it deliberately and read `coverage.md` before setting any threshold — coverage measures what executed, not whether behaviour is correct.

## Component testing

Vitest with a DOM environment plus a component testing library covers component behaviour. `@testing-library/react` is at **16.3.3** (MIT) on npm — not installed here.

Assert through semantic queries — role, label, text — for the same reason Playwright prefers them: they reflect what a user perceives, and they break when the accessible output breaks. Framework specifics are in `next-react.md`.
