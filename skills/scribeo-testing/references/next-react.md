# Next.js, React & the Scribeo Stack

Load when testing a Next.js / React / TypeScript project, including the GSAP, Motion and Lenis layer.

Not a framework tutorial — the testing implications only.

## Test observable behaviour, not framework internals

The single rule that prevents most brittle React tests: **assert what a user could observe.**

| Do not assert | Assert instead |
| --- | --- |
| Internal state values | The rendered result of that state |
| Hook call counts or render counts | The output after an interaction |
| Whether a specific child component rendered | The content or role the user perceives |
| Class names from a styling system | Role, accessible name, visible text |
| Context provider internals | The behaviour the context enables |

```tsx
// Bad: coupled to internals; breaks on any refactor
expect(wrapper.find("PanelBody")).toHaveLength(1);

// Good: coupled to behaviour; survives refactors
await user.click(screen.getByRole("button", { name: "Show details" }));
expect(screen.getByRole("region", { name: "Details" })).toBeVisible();
```

**Tailwind classes are not a contract.** Asserting `class="px-4"` tests the styling implementation, which changes freely. If a visual property genuinely matters, that is `scribeo-visual-qa`'s surface.

## Server and client boundaries

Next.js App Router components are server components by default; `"use client"` opts a component and its import subtree into the client.

Testing implications:

- **A server component that only renders markup** is usually best covered by an E2E or route-level test — it has no client behaviour to unit test.
- **A client component with interaction** is the natural component-test target.
- **Do not import a server component into a DOM-environment unit test** and expect it to behave — the execution model differs. Test its rendered output through the running app instead.
- **Be deliberate about which side you are testing.** A test that passes in a DOM environment but fails in the real app usually crossed this boundary silently.

Route handlers and API routes are excellent integration-test targets: fast, deterministic, and testing real contracts. See `layers.md`.

## Hydration-sensitive behaviour

Hydration mismatches are a real defect class and are invisible to most component tests, because a component test never hydrates.

- **Anything that renders differently on server and client** — dates, random values, viewport or capability checks, `localStorage` reads — is a hydration risk. The correct pattern is to apply those in an effect, and the correct test is an E2E check on the running app.
- **A console error assertion is a legitimate E2E test here.** Failing a test on a hydration warning catches a class of defect nothing else catches cheaply.
- **Interaction before hydration completes** is a real user experience. An E2E test that clicks immediately after navigation exercises it; one that waits for a settled page does not.

## Testing the animation layer

The stack uses GSAP, Motion and Lenis. **`scribeo-motion` owns their implementation; this skill tests stable user-observable outcomes.**

**Test these:**

- The end state. After the interaction, is the panel open, the content visible, the control operable?
- Reachability. Is content that animates in actually present and operable?
- **The reduced-motion path.** With `prefers-reduced-motion: reduce`, is all content visible and the journey completable? `scribeo-accessibility` requires this, and it is genuinely automatable — high value.
- No duplicate initialisation. A console error assertion, or a check that an effect ran once, catches the classic double-init defect.

**Do not test these** unless a value is an explicit, stated contract:

- Durations, easing curves, intermediate transform values.
- Frame-by-frame progress of a scrubbed scroll animation.
- Internal GSAP timeline or Lenis state.

```ts
// Good: the observable outcome, regardless of how it animated
await page.getByRole("button", { name: "Opening hours" }).click();
await expect(page.getByRole("region", { name: "Opening hours" })).toBeVisible();

// Bad: asserts an animation internal that Motion may legitimately change
expect(await page.evaluate(() => gsap.getById("panel")?.duration())).toBe(0.32);
```

**Animation is a leading cause of E2E flakiness** — a test that clicks mid-transition behaves differently run to run. Playwright's auto-retrying assertions absorb most of it. Where they do not, emulating reduced motion is the cleanest determinism lever, and it exercises a required path at the same time. See `flakiness.md`.

Frame rate and jank are **not** testable here — `scribeo-performance` owns measurement, and test-environment timings are contaminated.

## Effects and cleanup

- **React StrictMode double-invokes effects in development.** A component whose test passes only without StrictMode has a cleanup defect, not a test problem.
- **Browser-side effects must be torn down** — listeners, observers, GSAP contexts, Lenis instances, tickers. A component test that leaves listeners attached leaks into the next test; an app that does it leaks into the next route.
- **Assert cleanup where it matters.** Mounting and unmounting a component and checking that no listeners or timers remain is a legitimate and valuable test for anything wrapping an animation library.

## Fake timers

**Do not introduce fake timers unless genuinely required.** They replace the scheduling model, and interact badly with promises, framework scheduling and animation libraries — producing tests that hang, or pass for the wrong reason.

Legitimate use: logic whose behaviour *is* time (a debounce interval, a token expiry calculation). Even then, prefer injecting a clock over patching the global one.

Never reach for fake timers to make an animation deterministic. Emulate reduced motion, or await the end state.

## TypeScript

- **Types are not tests.** They prevent a class of defect at compile time; they say nothing about runtime behaviour. `tsc` passing is not verification.
- **Do not weaken types to satisfy a test.** A test needing `as any` to construct a value is usually signalling that the value is not a legitimate input — which is itself the finding.
- Keep test fixtures typed. An untyped fixture drifts from the real shape silently, and the test keeps passing against a shape that no longer exists.

`tsc` 6.0.2 is available globally in this environment; a project should pin its own.

## Practical checklist

- [ ] Assertions are on observable behaviour, not internals
- [ ] No Tailwind class assertions standing in for visual checks
- [ ] Server/client boundary is deliberate, not accidental
- [ ] Hydration-sensitive behaviour covered by an E2E check, not a DOM unit test
- [ ] Animation tests assert end states, not internals
- [ ] The reduced-motion path is tested
- [ ] Cleanup verified for anything attaching listeners or animation contexts
- [ ] No fake timers without a stated reason
- [ ] Fixtures typed against the real shape
