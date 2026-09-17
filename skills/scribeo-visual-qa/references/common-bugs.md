# Common Visual Bugs

Load when diagnosing a specific failure mode. Each entry: how to confirm it, what to capture, where the fix belongs.

## Horizontal overflow

**Confirm:** `document.documentElement.scrollWidth > clientWidth`. **Find it:** list elements whose `getBoundingClientRect().right` exceeds `clientWidth` (see `playwright.md`).
**Usual causes:** fixed widths; a flex/grid child without `min-width: 0`; wide tables; long unbroken strings; `100vw` with a visible scrollbar; negative margins.
**Capture:** width, measured values, offending elements, screenshot. → `scribeo-ux-engineering`

## Unexpected scrollbar

**Confirm:** compare `scrollHeight`/`clientHeight` and the same for width. A vertical scrollbar on a short page usually means an element slightly exceeding `100vh` — often `height: 100vh` plus margin or padding.
**Note:** headless rendering may show scrollbars differently from headed. Verify before reporting. → `scribeo-ux-engineering`

## Content clipped by the viewport

**Confirm:** element bounding box extends beyond viewport bounds, or an ancestor has `overflow: hidden`.
**Usual causes:** fixed heights, `overflow: hidden` on a container that grows, absolute positioning at narrow widths. Focus rings clipped this way are frequently missed. → `scribeo-ux-engineering`

## Fixed element covering content

**Confirm:** scroll to an anchor target and screenshot — is the heading under the header? Check computed `position` and heights.
**Usual causes:** no `scroll-padding-top` for a sticky header; a bottom bar with no compensating page padding; a mobile bar ignoring `env(safe-area-inset-bottom)`. → `scribeo-ux-engineering`

## Broken sticky

**Confirm:** scroll and capture at several positions.
**Usual causes:** an ancestor with `overflow: hidden` or `overflow: auto` (kills `position: sticky`), a missing offset (`top`), or a parent too short to sticky within. → `scribeo-ux-engineering`

## Incorrect z-index / layering

**Confirm:** screenshot the overlap; read computed `z-index`, `position`, `transform`, `opacity`, `filter` on both elements and their ancestors.
**Key point:** a `transform`, `filter`, or non-`1` `opacity` creates a new stacking context, so a high `z-index` inside it cannot escape. This is the usual explanation for "the z-index is huge and it still goes behind". → `scribeo-ux-engineering`

## Font fallback

**Confirm:** computed `fontFamily` on the element, `document.fonts.status`, and the font's network entry.
**Symptoms:** wrong metrics, different wrapping, heavier or lighter colour than intended. Often reported as "the typography looks wrong".
**Usual causes:** 404, CORS on a cross-origin font, wrong `font-family` name, missing `preload`, blocked third-party host. → implementation; the face itself is `frontend-design`

## Images not loading

**Confirm:** `document.images` filter for `!complete || naturalWidth === 0`, plus network status.
**Usual causes:** wrong path, case-sensitive filename on a Linux host, broken `srcset` candidate, CORS, lazy-loading that never triggers, expired signed URL. → implementation

## Wrong aspect ratio

**Confirm:** measured box vs the image's `naturalWidth`/`naturalHeight`.
**Usual causes:** width and height set independently; missing `object-fit`; `aspect-ratio` fighting an explicit height. → `scribeo-ux-engineering`

## Layout shift on load

**Confirm:** capture immediately after navigation and again after settle; compare.
**Usual causes:** images without `width`/`height` or `aspect-ratio`; web font swap; late-injected banners; content rendered after hydration.
**Report the symptom and evidence; do not measure CLS or set budgets** → `scribeo-performance`

## Mobile breakpoint mismatch

**Confirm:** test just below, at, and just above the breakpoint.
**Usual causes:** overlapping or gapped `min-`/`max-width` ranges; a JS width check disagreeing with the CSS breakpoint; a device-width assumption instead of a capability query. → `scribeo-ux-engineering`

## Animation captured mid-transition

**Confirm:** re-capture after waiting for completion, or under reduced motion. If it resolves, the first capture was the bug — not the page.
**This is a QA error, not a defect.** Fix the capture method before filing anything. See `animation-qa.md`.

## Hydration-dependent mismatch

**Confirm:** capture immediately on load and after hydration; check console for hydration warnings.
**Symptoms:** flash then change; different content server vs client; interaction dead until hydration completes.
**Usual causes:** render-time branching on viewport, `localStorage`, dates, or randomness. → implementation

## Viewport-height bugs

**Confirm:** on mobile, scroll to trigger browser chrome hiding; capture before and after.
**Usual causes:** `100vh` including area under the chrome — `dvh` is the fix; full-height heroes on short landscape viewports. → `scribeo-ux-engineering`

## Safe-area issues

**Confirm:** capture on a device profile with notch or home indicator; check content at the very top and bottom.
**Usual causes:** no `env(safe-area-inset-*)` padding; a fixed bottom bar under the home indicator; no `viewport-fit=cover`. → `scribeo-ux-engineering`

## Stale CSS / cache-looking behaviour

**Confirm:** hard reload with cache disabled, and check the stylesheet's network entry and response headers. Confirm the served CSS actually contains the expected rule.
**Before reporting:** verify the build deployed and you are on the expected commit. A large share of "the fix didn't work" reports are this. → implementation / deployment

## Browser-specific rendering

**Confirm:** reproduce in a second engine. This session's MCP is Chromium-only, so cross-engine claims need another environment — say so rather than implying coverage you do not have.
**Usual causes:** unsupported CSS feature, vendor-prefix gaps, differing default styles. → implementation
