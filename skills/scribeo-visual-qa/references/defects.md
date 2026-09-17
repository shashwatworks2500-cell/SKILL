# Defect Taxonomy & Evidence

Load when classifying a defect and deciding what evidence to capture.

Classification is not bureaucracy: the category determines the evidence, and the evidence determines whether the fix can happen without another QA round.

## Taxonomy

| Category | Typical symptoms | Evidence to capture | Likely owner |
| --- | --- | --- | --- |
| **Layout** | Misplaced, collapsed, or wrongly stacked elements | Screenshot + viewport; `boxes` snapshot of the container | `scribeo-ux-engineering` |
| **Spacing** | Inconsistent gaps, cramped or excessive rhythm | Measured values vs system spec; element screenshot | `scribeo-ux-engineering` |
| **Typography** | Wrong face, size, weight, leading; bad wrapping; orphans | Computed `fontFamily`/`fontSize`/`lineHeight`; `document.fonts.status`; font network entry | `scribeo-ux-engineering`, or `frontend-design` if the face itself is wrong |
| **Overflow** | Horizontal scroll; content escaping its container | `scrollWidth` vs `clientWidth`; offending element list; screenshot at width | `scribeo-ux-engineering` |
| **Clipping** | Text or focus ring cut off; ellipsis where none intended | Element screenshot; computed `overflow`/dimensions | `scribeo-ux-engineering` |
| **Alignment** | Elements nearly-but-not aligned; inconsistent edges | `boxes` snapshot with coordinates; annotated screenshot | `scribeo-ux-engineering` |
| **Sizing** | Wrong dimensions, aspect ratio, or scale | Measured box vs intended; screenshot | `scribeo-ux-engineering` |
| **Responsive** | Correct at one width, broken at another | Screenshots at each width incl. breakpoint edges | `scribeo-ux-engineering` |
| **Asset / rendering** | Missing or broken image, wrong crop, failed font or video | Network entry (status, URL); `document.images` check; screenshot | Implementation / `scribeo-performance` if size-related |
| **Interaction state** | Hover, focus, active, open, selected missing or wrong | Screenshot per state; note keyboard vs pointer | `scribeo-ux-engineering` |
| **Motion state** | Stuck mid-animation, missing entrance, wrong end state | Screenshots at sampled points; console errors | `scribeo-motion` |
| **Layering / z-index** | Wrong element on top; content behind an overlay | Screenshot; computed `z-index` and stacking context | `scribeo-ux-engineering` |
| **Sticky / fixed** | Header covering content or anchors; sticky not sticking | Screenshot while scrolled; computed `position`/offsets | `scribeo-ux-engineering` |
| **Viewport-specific** | Only at one size, orientation, or zoom | Exact viewport; screenshots either side | `scribeo-ux-engineering` |
| **Loading / state** | Layout shift on load; state never resolves; wrong empty state | Before/after-load screenshots; console; network | Implementation / `scribeo-performance` for shift |
| **Browser-specific** | Renders correctly in one engine only | Screenshots per engine; exact versions | Implementation |

## Evidence rules

**Every defect needs:** a screenshot or a measurement, the route, the viewport, and the state. Missing any one makes it non-reproducible.

- **Prefer a measurement to an impression.** "32px where the system says 48px" is actionable; "spacing looks tight" is not.
- **Capture the smallest useful frame.** An element screenshot of the broken component beats a full-page capture the reader must hunt through.
- **Include console and network output** for anything involving assets, fonts, media, or hydration. It is often the entire diagnosis.
- **State reproducibility** — every load, or N of M. Intermittent almost always means timing or asset loading.
- **Note when a defect only appears on resize** rather than on a fresh load, or vice versa. That distinction points straight at measurement-on-mount code.

## Assigning the likely owner

Give a best-guess owner, marked as a guess. It routes the work; it does not bind anyone.

- Rendering differs from a stated spec → the implementing skill for that surface.
- Structure, hierarchy, spacing, responsive behaviour → `scribeo-ux-engineering`.
- Animation wrong, stuck, or missing → `scribeo-motion`.
- The spec itself is unclear or absent → **not a defect**. Raise it as a question, and route the decision to `frontend-design` if it is aesthetic.
- Visible jank or slow paint → report the symptom, route to `scribeo-performance`.
- Clipped focus ring or unreadable overlay text → report what is visible, route to `scribeo-accessibility`.

## Not defects

Do not file these. Filing them erodes trust in the whole report.

- **Aesthetic preference** with no stated intent behind it.
- **Deliberate design decisions** you would have made differently.
- **Known intentional differences** between breakpoints or states.
- **Your own environment's artefacts** — a headless scrollbar difference, a missing local font, a stale cache.
- **Anything you did not actually observe.** Never file a defect inferred from reading code.
