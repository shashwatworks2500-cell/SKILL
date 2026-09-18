# CI & Local Verification

Load when deciding how and where tests run.

## The verification ladder

Climb only as far as the change warrants. Running the full suite for a copy edit wastes time; shipping a checkout change on a single unit test is negligent.

| Rung | Run | When |
| --- | --- | --- |
| **1. Targeted test** | The single test for the thing you changed | Every change, immediately |
| **2. Relevant group** | The file, module, or route's tests | Before considering the change done |
| **3. Broader suite** | Everything affected, or all fast tests | Before merge |
| **4. Build verification** | A production build — type-check, compile, bundle | Before merge on anything that could break the build |
| **5. Browser verification** | E2E against a real running build | Before release, and on changes to critical journeys |

**Rung 4 catches a class nothing else does.** Type errors, build-time failures, and server/client boundary mistakes frequently pass every test and fail the build. `tsc` passing is not a test result, but a build failure is a real defect.

**Do not skip rung 1 and go straight to the suite.** A targeted run gives a fast, unambiguous signal; a full-suite failure buries it.

## CI strategy scales with the project

**Do not prescribe unnecessary complexity for a small Scribeo marketing site.** Sharding, matrix builds and multi-stage pipelines have real maintenance cost, and on a suite that runs in ninety seconds they buy nothing.

| Project | Proportionate CI |
| --- | --- |
| **Marketing site, small suite** | One job: install, build, run everything. Artefacts on failure |
| **Larger site, suite in minutes** | Two jobs: fast checks (types, lint, unit) then E2E. Fail fast on the first |
| **Application, long suite** | Parallel jobs by layer; shard E2E; cache dependencies and browsers |

The deciding factor is **wall-clock runtime and how often it blocks someone.** Optimise when waiting becomes the problem, not before.

## Determinism in CI

CI is where environment-dependent tests surface. Pin what you can:

- **Pin the runtime and dependency versions.** A lockfile, and an explicit Node version. Floating versions mean a green suite can turn red with no code change.
- **Pin the browser version.** Playwright browsers are versioned with the package — pin `@playwright/test` and install its matching browsers, don't rely on whatever is present.
- **Set timezone and locale explicitly.** The commonest "passes locally, fails in CI" cause.
- **Install dependencies from the lockfile**, not a resolving install.
- **Cache dependencies and browser binaries** — the single biggest CI time saving, with no correctness cost.

Note this environment: the global `playwright` CLI is **1.56.1**, while npm's latest `@playwright/test` is **1.63.0**. A project pinning its own dependency will not match the global CLI — which is exactly why a project must not rely on an ambient install.

## Artefacts and failure logs

**A CI failure with no artefact costs a full debugging cycle.** Configure evidence before you need it.

- **Traces** — `trace: "on-first-retry"` is the best default: a full trace exactly when something failed and was retried, and no cost on green runs.
- **Screenshots** — `screenshot: "only-on-failure"`.
- **Video** — `video: "retain-on-failure"` where the failure is hard to read from a trace.
- **Upload artefacts on failure** and retain them long enough to investigate.
- **Keep the failure log readable.** A test whose failure message is self-explanatory saves opening the trace at all.

The repo `.gitignore` already excludes `test-results/`, `playwright-report/`, `.playwright/` and `coverage/` — these are CI artefacts, never committed.

## Parallelism

- **`workers`** controls parallelism in Playwright. More workers is faster until the machine saturates — beyond that, tests slow down and timing-sensitive ones start failing.
- **Parallelism exposes isolation defects.** A suite that only passes with one worker has shared state; that is a defect to fix, not a setting to keep. See `fixtures-data.md`.
- **Give workers separate data** — separate accounts, separate records — where the system allows.
- On a constrained CI runner, fewer workers is sometimes both faster and more stable. Measure rather than maximising.

## Retries in CI

- A small retry count keeps CI usable while flakes are diagnosed, and captures a trace.
- **Playwright reports retry-passes as "flaky"** — treat that list as a defect backlog, not a success column.
- **Never raise retries to turn a red suite green.** See `flakiness.md`.
- Retries on a **smoke** suite are especially suspect: a smoke test that needs retries is not fit for its job.

## Sharding

Only when E2E runtime genuinely blocks delivery. Splitting the suite across machines adds orchestration and result-merging complexity, and each shard must be independently green — which requires real isolation.

If sharding is needed, shard the slow layer (E2E) and leave fast layers in one job.

## Exit codes and gating

- **The suite's exit code is the gate.** Non-zero must fail the pipeline.
- **Never `|| true` a test command.** It converts the gate into decoration, and it is usually added under deadline pressure and never removed.
- **Fail fast on cheap checks.** Types and lint before E2E: if the build cannot compile, spending ten minutes on browser tests is waste.
- **Distinguish "failed" from "errored".** A suite that could not start (missing browser, failed install) is an infrastructure problem, not a test result — and reporting it as a test failure sends everyone to the wrong place.

## What CI does not verify

A green pipeline verifies **defined behaviour**. It does not verify appearance, accessibility conformance, performance, SEO, UX quality, or motion safety. Those route to their owning skills — see the boundary table in `SKILL.md`.

## Checklist

- [ ] Verification climbed to the rung the change warrants
- [ ] Build verification run where the change could break the build
- [ ] Runtime, dependency and browser versions pinned
- [ ] Timezone and locale set explicitly
- [ ] Dependencies installed from a lockfile; caches configured
- [ ] Trace, screenshot and video configured for failures
- [ ] Artefacts uploaded and retained
- [ ] Suite passes in parallel, not just single-worker
- [ ] Exit code gates the pipeline; no `|| true`
- [ ] Flaky list treated as a defect backlog
