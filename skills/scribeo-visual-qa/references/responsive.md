# Responsive QA

Load when verifying rendering across widths.

**Verify, do not redesign.** If responsive behaviour looks fundamentally wrong rather than broken, that is a design decision — report it and route to `scribeo-ux-engineering`. This skill establishes *what actually happens* at each width.

## Checklist

Run at every viewport in the matrix (`viewport.md`).

**Overflow — always first**
- [ ] No horizontal scrollbar. Verify with `scrollWidth` vs `clientWidth`, not by eye
- [ ] No element extending past the viewport edge
- [ ] Tables, code blocks, and long unbroken strings contained or intentionally scrollable
- [ ] Nothing clipped by a container that should not clip

**Navigation**
- [ ] Correct pattern for the width — full, compressed, or disclosure
- [ ] Mobile menu opens, closes, and is fully visible
- [ ] Menu does not scroll under a sticky header or exceed the viewport
- [ ] Primary CTA and any critical contact route still reachable
- [ ] Active state visible

**Typography**
- [ ] Sizes scale sensibly; nothing below ~16px for body on mobile
- [ ] Measure stays readable — not full-bleed on desktop, not squeezed on mobile
- [ ] Headings wrap acceptably; no single orphaned word where it matters
- [ ] No clipped descenders or cut-off lines
- [ ] Correct typeface loaded, not a fallback

**Images and media**
- [ ] All load; none broken
- [ ] Aspect ratio preserved; no stretch or squash
- [ ] Art direction or cropping keeps the subject in frame at narrow widths
- [ ] Space reserved before load — no shift
- [ ] Video controls usable and not overflowing

**Grids and cards**
- [ ] Column count sensible at each width
- [ ] No orphan item in a broken final row
- [ ] Equal heights where intended; no clipped content
- [ ] Gaps consistent and from the spacing scale

**Containers and spacing**
- [ ] Side gutters present at every width; text never touches the edge
- [ ] Section rhythm proportional — not desktop padding on a phone
- [ ] Max-width respected on large screens
- [ ] Spacing values consistent between comparable sections

**Buttons and forms**
- [ ] Labels fit without wrapping; if wrapping is intended, it is not pushing layout
- [ ] Targets look adequately sized and separated for touch
- [ ] Inputs full-width where intended; labels visible and associated
- [ ] Error messages visible and not overlapping
- [ ] Keyboard on mobile does not obscure the active field

**Fixed and sticky**
- [ ] Sticky header does not consume an unreasonable share of a short viewport
- [ ] Nothing important hidden behind a fixed bar
- [ ] Anchor targets land correctly beneath a sticky header
- [ ] Bottom bars respect safe-area insets; no overlap with system UI

**Viewport height**
- [ ] Full-height sections behave when mobile browser chrome hides and shows
- [ ] `dvh` used rather than `vh` where it matters
- [ ] Content reachable on a short landscape viewport

## Method

1. **Set the width, then reload.** Some defects only appear on a fresh load at that size, because measurement happened once at the old one.
2. **Also test by resizing without reload** — that catches the opposite class, where JS never recalculates. Report which path reproduces it.
3. **Scroll the full page** at each width. Sticky, scroll-linked, and footer defects live below the fold.
4. **Check breakpoint edges** — just below, at, just above.
5. **Verify both tablet orientations.**

## Frequently missed

| Area | Why it slips through |
| --- | --- |
| Tablet portrait (768) | Neither desktop nor mobile; rarely designed, frequently broken |
| Just above a breakpoint | The desktop layout at its most cramped |
| 320px | Below most designers' testing floor; overflow appears here first |
| Landscape phone | Short viewport breaks full-height heroes |
| Long content | Real copy wraps differently than placeholder |
| Empty content | Grids and sections collapse in ways placeholders hide |
| Footer | Rarely scrolled to during review |
