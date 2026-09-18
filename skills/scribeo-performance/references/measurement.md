# Measurement

Load when choosing a tool, capturing evidence, or reading the performance APIs.

## Match the tool to the question

| Question | Tool |
| --- | --- |
| Is the site fast for real users? | Field data (RUM). Nothing else answers this |
| Why is this specific page slow? | Lab: trace + network waterfall |
| Which resource blocks the LCP? | Network waterfall + LCP entry |
| Which code blocks the main thread? | Performance trace, long-task entries |
| Did my change help? | Identical lab re-measurement, same profile |
| Is it slow on a real phone? | Throttled lab run, or field data segmented to mobile |

## Playwright MCP: the lab loop

Available and verified in this session. Chromium 141, headless.

```
browser_navigate            { url }
browser_network_requests                         → waterfall, sizes, statuses
browser_console_messages    { level: "error" }    → errors that distort timing
browser_evaluate            { function }          → performance APIs
browser_start_tracing / browser_stop_tracing      → trace capture
```

Measure a clean load: navigate, wait for the condition that matters, then read. Measuring a warm page after several interactions produces numbers that describe nothing.

**Repeat every measurement at least three times.** Single runs vary by tens of percent from cache state, CPU contention, and network noise. Report the median and note the spread; a change smaller than the run-to-run spread is not a result.

## Performance APIs via evaluate

**Navigation timing** — TTFB and the load phases:

```js
() => {
  const n = performance.getEntriesByType("navigation")[0];
  return {
    ttfb: Math.round(n.responseStart - n.requestStart),
    dns: Math.round(n.domainLookupEnd - n.domainLookupStart),
    tls: Math.round(n.connectEnd - n.secureConnectionStart),
    response: Math.round(n.responseEnd - n.responseStart),
    domContentLoaded: Math.round(n.domContentLoadedEventEnd - n.startTime),
    load: Math.round(n.loadEventEnd - n.startTime),
  };
}
```

**LCP** — including which element it is, which is the fact that drives the fix:

```js
() => new Promise((resolve) => {
  new PerformanceObserver((list) => {
    const e = list.getEntries().at(-1);
    resolve({
      lcpMs: Math.round(e.startTime),
      element: e.element ? e.element.tagName + "." + e.element.className : null,
      url: e.url || null,
      size: e.size,
    });
  }).observe({ type: "largest-contentful-paint", buffered: true });
  setTimeout(() => resolve({ lcpMs: null, note: "no LCP entry observed" }), 5000);
})
```

**CLS** — accumulated, excluding user-initiated shifts:

```js
() => new Promise((resolve) => {
  let cls = 0; const sources = [];
  new PerformanceObserver((list) => {
    for (const e of list.getEntries()) {
      if (e.hadRecentInput) continue;
      cls += e.value;
      for (const s of e.sources || []) {
        if (s.node) sources.push(s.node.tagName + "." + s.node.className);
      }
    }
  }).observe({ type: "layout-shift", buffered: true });
  setTimeout(() => resolve({ cls: +cls.toFixed(4), sources: [...new Set(sources)] }), 4000);
})
```

`sources` names the shifting elements — that is the diagnosis, not the score.

**Long tasks** — the direct cause of poor INP:

```js
() => new Promise((resolve) => {
  const tasks = [];
  new PerformanceObserver((list) => {
    for (const e of list.getEntries()) tasks.push({ start: Math.round(e.startTime), dur: Math.round(e.duration) });
  }).observe({ type: "longtask", buffered: true });
  setTimeout(() => resolve({ count: tasks.length, totalMs: tasks.reduce((a, t) => a + t.dur, 0), tasks: tasks.slice(0, 20) }), 5000);
})
```

**Resource weight by type** — the fastest way to find a payload problem:

```js
() => {
  const byType = {};
  for (const r of performance.getEntriesByType("resource")) {
    const t = r.initiatorType || "other";
    byType[t] = byType[t] || { count: 0, kb: 0 };
    byType[t].count++;
    byType[t].kb += Math.round((r.transferSize || 0) / 1024);
  }
  return byType;
}
```

**INP cannot be measured without real interaction.** Observe `event` timing entries while driving actual clicks and key presses, or use TBT and long tasks as the lab proxy. A page that is never interacted with has no INP.

## What is not available here

Say so plainly rather than producing plausible output:

- **Lighthouse is not installed.** `lighthouse` is not on PATH. The npm package (13.4.1, Apache-2.0) could be added, but installing it is the user's dependency decision.
- **No field/RUM data.** That requires the `web-vitals` library (6.2.2, Apache-2.0) wired into the site, or a RUM provider. Without it, Core Web Vitals cannot be judged at the 75th percentile — only diagnosed in lab.
- **No interactive DevTools UI** in a headless container. Traces can still be captured and read.
- **Chromium only.** No cross-engine performance claims from this environment.

## Throttling

Unthrottled desktop measurement is the most common way to miss the actual problem. Always measure the constrained case explicitly — see `mobile.md`.

Record the profile with every number. `LCP 2.1s` is meaningless; `LCP 2.1s at 390×844, 4× CPU throttle, Slow 4G, median of 5` is a measurement.

## Discipline

- **Baseline before touching anything.** No baseline, no result.
- **One variable per cycle.**
- **Identical conditions** between baseline and verification, or the comparison is void.
- **Cold vs warm cache** stated explicitly; they are different measurements.
- **Record failures.** A change that did not help is a finding worth keeping.
