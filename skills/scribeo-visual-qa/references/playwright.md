# Playwright for Visual QA

Load when driving the browser — navigation, resizing, screenshots, console, and measurement.

## MCP is not the test runner

Two different tools with the same name. Do not conflate them.

| | **Playwright MCP** | **Playwright Test** |
| --- | --- | --- |
| What | Browser control exposed as agent tools | A test runner (`@playwright/test`) |
| Used for | Interactive inspection during a QA pass | Durable, repeatable automated suites |
| Output | Observations, screenshots, snapshots | Pass/fail, reports, CI gating |
| Owner | This skill | `scribeo-testing` |

Use **MCP** to look at the thing and find defects. If a check should run forever in CI, that is a **Test** artifact and `scribeo-testing` owns its architecture. Writing a throwaway script to reproduce a defect is fine; designing the suite is not this skill's call.

## This session's configuration

The configured MCP server runs headless Chromium with `--isolated` (in-memory profile), a default viewport of 1280×720, and screenshots written to a session output directory. Consequences for QA:

- **Profile does not persist across sessions.** Cookies, storage, and cache start empty — which is a good QA default, and also means a logged-in state must be established each session.
- **State does persist within a session.** Reset deliberately between cases rather than assuming a clean slate.
- **Headless rendering can differ subtly from headed** — scrollbar presence, some font rasterisation, and video behaviour. If a defect appears only in headless, verify before reporting it.

## Core loop

```
browser_navigate  →  browser_console_messages  →  browser_resize
                  →  browser_snapshot / browser_take_screenshot
```

**Navigate, then check errors before looking.** Console and network failures explain most visual defects and save a wasted diagnosis.

```
browser_navigate      { url: "https://example.com/pricing" }
browser_console_messages { level: "error" }
browser_network_requests
```

`browser_console_messages` takes `level` (`error` | `warning` | `info` | `debug`, each including more severe levels) and `all` to cover the whole session rather than since last navigation.

## Viewport

```
browser_resize { width: 390, height: 844 }
```

Resize **then reload** when verifying responsive behaviour. Many layout bugs only appear on a fresh load at that width, because JS measured once at the previous size. A defect that appears on resize but not on reload — or the reverse — is diagnostically important; state which in the report.

## Screenshots

```
browser_take_screenshot { scale: "css", filename: "pricing-390-hero.png" }
browser_take_screenshot { scale: "css", fullPage: true, filename: "pricing-390-full.png" }
browser_take_screenshot { scale: "css", target: "<ref or selector>", filename: "cta.png" }
```

- **`scale: "css"` for anything compared across runs or devices.** It produces CSS-pixel sizing, so images stay consistent. `scale: "device"` multiplies by device pixel ratio — higher detail, but the dimensions shift with the environment and break comparisons.
- `fullPage: true` captures the whole scrollable page; it cannot be combined with an element capture.
- Name files `<route>-<width>-<subject>.png`. A directory of `page-<timestamp>.png` is unreviewable an hour later.

Full detail in `screenshots.md`.

## Snapshot before screenshot

`browser_snapshot` returns the accessibility tree and is often the better first look: it is textual, cheap, and tells you what the page actually exposes.

```
browser_snapshot { boxes: true }
```

`boxes: true` adds each element's bounding box as `[box=x,y,width,height]` — viewport-relative CSS pixels from `getBoundingClientRect`. This measures spacing, alignment, and element size **without eyeballing a screenshot**, which is how you turn "spacing looks off" into "32px where the system says 48px".

Use `depth` to keep large pages manageable, and `target` to snapshot one subtree.

## Measuring with evaluate

`browser_evaluate` takes a `function` as a string. These are the high-value QA probes.

**Horizontal overflow** — the single most common responsive defect:

```js
() => ({
  scrollW: document.documentElement.scrollWidth,
  clientW: document.documentElement.clientWidth,
  overflowing: document.documentElement.scrollWidth > document.documentElement.clientWidth,
})
```

**Find the culprits** when it overflows:

```js
() => [...document.querySelectorAll("*")]
  .filter(el => el.getBoundingClientRect().right > document.documentElement.clientWidth + 1)
  .slice(0, 10)
  .map(el => `${el.tagName}.${el.className}` + " → " + Math.round(el.getBoundingClientRect().right))
```

**Broken or unloaded images:**

```js
() => [...document.images]
  .filter(img => !img.complete || img.naturalWidth === 0)
  .map(img => img.currentSrc || img.src)
```

**Font loading state** — catches fallback-font rendering reported as "the type looks wrong":

```js
() => ({ status: document.fonts.status, loaded: document.fonts.size })
```

**Computed style of a specific element**, to confirm a spacing or type claim:

```js
(el) => {
  const s = getComputedStyle(el);
  return { fontFamily: s.fontFamily, fontSize: s.fontSize, lineHeight: s.lineHeight,
           marginTop: s.marginTop, paddingTop: s.paddingTop };
}
```

Measure before reporting. "Spacing looks inconsistent" is an impression; "48px above section 1, 32px above section 2, system spec is 48px" is a defect.

## Interaction states

```
browser_hover      { element: "primary CTA", target: "<ref>" }
browser_press_key  { key: "Tab" }
browser_click      { element: "menu toggle", target: "<ref>" }
```

- **Hover:** hover, then screenshot. Confirm the state actually changed and nothing shifts layout.
- **Focus:** `Tab` through and screenshot each stop. Focus rings clipped by `overflow: hidden` or hidden behind a sticky header are common and invisible to code review.
- **Active/press:** hard to capture reliably; verify visually and note if not screenshot-confirmed.
- **Open/closed:** click the toggle, screenshot both states, verify focus moved correctly.

## Waiting

```
browser_wait_for { text: "…" }     // or time / textGone, per the tool's schema
```

Never screenshot immediately after navigation. Wait for the condition that matters — a text node, a network idle, fonts ready — then capture. A screenshot of a half-loaded page is a false defect, and chasing one wastes a cycle. See `screenshots.md`.

## Console and network as evidence

Attach these to reports; they frequently contain the whole diagnosis.

- A 404 on a font file explains "the typography is wrong".
- A 404 on an image explains a collapsed card grid.
- A hydration mismatch warning explains "it flashes then changes".
- A CORS error explains a blank media element.

```
browser_console_messages { level: "error", all: true }
browser_network_requests
```

## Do not

- **Do not fix code from here.** Observe, report, route. Fixing belongs to the owning skill.
- **Do not use `browser_run_code_unsafe`** for routine QA. Evaluate targeted expressions instead.
- **Do not build a test suite** in a QA pass. Reproduction scripts are fine; suite architecture is `scribeo-testing`.
- **Do not report a defect you saw only in a snapshot** when it is a visual claim — confirm visually.
