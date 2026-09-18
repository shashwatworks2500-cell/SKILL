# Coverage

Load when reading or configuring coverage.

## What coverage is

**Coverage measures what executed. Nothing more.**

It does not measure whether behaviour is correct, whether assertions are meaningful, or whether the important cases are tested. A line is "covered" by a test that runs it and asserts nothing at all.

```ts
// This produces 100% coverage of formatPrice and verifies nothing
test("formatPrice", () => {
  formatPrice(1234.5, "GBP");   // no assertion
});
```

That is the whole problem with coverage as a quality measure, in four lines.

## The four types

| Type | Measures | Blind spot |
| --- | --- | --- |
| **Line** | Which lines ran | A line with two branches counts as covered if either ran |
| **Statement** | Which statements ran | Similar to line; finer on multi-statement lines |
| **Branch** | Which branches of each conditional ran | The most informative. Uncovered branches are untested decisions |
| **Function** | Which functions were called | Says nothing about what happened inside |

**Branch coverage is the one worth reading.** An uncovered branch is a decision your suite never exercised — which is a genuine, actionable gap. High line coverage with low branch coverage means the happy path runs and the error paths do not.

## How to use it

**As a discovery tool, not a score.**

- **Read the uncovered report, not the percentage.** Is anything important in it? An untested error path in payment handling matters; an untested `default` in a logging switch does not.
- **Look for uncovered branches** specifically — they are the highest-signal entries.
- **Watch the direction of travel** on the surfaces that matter. A steady fall in coverage of a critical module is informative; a fluctuating global percentage is not.
- **Use it after writing tests**, to find what you missed — never before, to decide what to write. Planning from coverage produces tests that chase lines rather than behaviour.

## What not to do

- **Do not treat the percentage as a quality score.** It measures execution, not correctness.
- **Do not write tests to raise it.** Coverage-driven tests assert little, cost maintenance, and create false confidence — the worst combination available.
- **Do not require 100%.** The last stretch is almost entirely defensive branches, unreachable error handlers and framework glue. The cost is high and the return is near zero.
- **Do not adopt a universal Scribeo threshold.** A marketing site with almost no logic and an application with complex domain rules cannot share one meaningful number.
- **Do not delete an uncoverable line to improve the number.** That is optimising the metric against the product.

## Thresholds, if used

A threshold is **project-specific engineering policy**, justified by that project's risk — not a Scribeo standard and not a number to import from another project.

Where a project adopts one:

- **Derive it from current reality.** Set it at or just below the current measurement and forbid regression. A threshold nobody can meet is ignored, and the discipline dies with it.
- **Scope it to what matters.** A threshold on the domain logic directory is meaningful; one on the whole repository averages away the signal.
- **Prefer "must not fall" to "must reach".** Preventing decay is achievable and useful; chasing an aspirational number produces filler tests.
- **Write down the reasoning**, so the next person knows whether it is load-bearing or arbitrary.

## Tooling

`@vitest/coverage-v8` provides the V8 coverage provider for Vitest (**5.0.1**, MIT, on npm — not installed in this environment). Coverage is configured under Vitest's `coverage` config field.

Playwright is an end-to-end runner; **coverage of application source is not its purpose**, and E2E coverage figures are misleading — a single journey executes a large fraction of the codebase while asserting very little of it. Do not combine E2E coverage into a headline number.

The repo `.gitignore` already excludes `coverage/`.

## Reporting coverage honestly

When coverage appears in a report:

- State the **type** (branch, line) and the **scope** (which directories).
- State it as **information, not achievement**.
- Name what is actually uncovered and whether it matters.

```
Coverage:  Branch 74% over src/domain (line 88%)
Uncovered: Payment retry path, and the three error branches in
           reconcile(). The payment retry path is worth covering;
           the logging default branch is not.
```

```
Bad:   "Coverage is 88% — good."
Good:  "88% line, 74% branch over src/domain. The uncovered branches
        include the payment retry path, which is worth a test."
```

Never present a coverage percentage as evidence that behaviour is correct.

## Checklist

- [ ] Coverage used to find gaps, not to score quality
- [ ] Branch coverage read, not just line
- [ ] Uncovered report actually read, and gaps triaged by importance
- [ ] No tests written purely to raise the number
- [ ] No universal threshold imported from elsewhere
- [ ] Any threshold derived from current reality and documented
- [ ] E2E coverage not folded into a headline figure
- [ ] Reported with type and scope, as information
