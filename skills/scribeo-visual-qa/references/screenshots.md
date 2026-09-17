# Screenshot QA

Load for deterministic capture, waiting, and choosing what to capture.

## Determinism first

**A screenshot of an unsettled page is a false defect.** Chasing one costs a full cycle. Before every capture, control:

| Variable | Control |
| --- | --- |
| Fonts | Wait for `document.fonts.ready`; fallback rendering is the classic false positive |
| Images and media | Wait for load; confirm none are broken |
| Animations | Freeze, complete, or deliberately sample — see `animation-qa.md` |
| Viewport | Set explicitly; never rely on the default |
| Scale | `scale: "css"` for anything compared |
| Scroll position | Reset to a known position; scroll-linked effects depend on it |
| Dynamic content | Dates, counters, randomised or personalised content — stub or accept as noise |
| Browser state | Know whether you are on a fresh load or a resized page |

```js
// wait for fonts before capture
() => document.fonts.ready.then(() => document.fonts.status)
```

## Capture types

| Type | Use | Note |
| --- | --- | --- |
| **Viewport** | What the user actually sees; above-the-fold verification | Default choice |
| **Full page** (`fullPage: true`) | Whole-page structure, section rhythm, footer | Long pages produce tall unreviewable images; prefer sectional |
| **Element** (`target`) | One component, one defect | Best evidence for a report — unambiguous |
| **Before/after pair** | Fix verification, regression | Same viewport, same state, same scale, or the comparison is meaningless |

For a before/after pair, **change exactly one variable.** A different width or scroll position invalidates the comparison.

## Naming

```
<route>-<width>-<state>-<subject>.png

pricing-390-default-cta-wrap.png
pricing-390-after-fix-cta.png
home-1440-hover-nav.png
```

Default filenames (`page-<timestamp>.png`) are unreviewable within the hour and useless in a report. Name every capture at the moment you take it.

## When screenshots are the wrong tool

Screenshots are for **visual** claims. For anything measurable, measure instead — it is faster, exact, and produces a better report.

| Question | Better tool |
| --- | --- |
| "Is there horizontal overflow?" | `browser_evaluate` comparing `scrollWidth`/`clientWidth` |
| "Is the spacing right?" | `browser_snapshot { boxes: true }`, or computed style |
| "Is the font correct?" | Computed `fontFamily` plus `document.fonts.status` |
| "Did any image fail?" | `document.images` filter, plus network requests |
| "Is the heading order right?" | `browser_snapshot` — the accessibility tree |
| "Are there errors?" | `browser_console_messages` |

A screenshot showing text in the wrong typeface proves something looks off. The computed `fontFamily` plus a font 404 proves *what is wrong and why*. Capture the screenshot as evidence; lead the report with the measurement.

**Manual inspection beats screenshot comparison** when intent is subjective, when the page is intentionally dynamic, or when the question is "does this feel right" — which is `frontend-design`'s question, not a QA one.

## Capturing states

Loading, empty, and error states are where implementations are thinnest, and they are frequently never looked at.

- **Loading:** throttle or intercept to hold the state long enough to capture. Check that space is reserved and nothing jumps.
- **Empty:** clear the data source or navigate to a genuinely empty case. A search with no results is the easiest.
- **Error:** block the request or navigate to a failing route.
- **Interaction:** hover, focus, active, open, selected — capture each that exists.
- **Reduced motion:** re-run key captures under `prefers-reduced-motion: reduce`; it is a distinct rendering path.

The existence and content of these states is `scribeo-ux-engineering`'s standard. Verifying they actually render is this skill's.

## Reviewing

- **Compare against stated intent**, not memory of how it looked.
- **Check edges and corners.** Defects cluster where containers meet the viewport.
- **Look at what is absent** — a missing image, an empty slot, a section that did not render — not only at what looks wrong.
- **Zoom in on the defect** before reporting size or alignment. Do not estimate pixel values from a scaled-down full-page capture; measure.
