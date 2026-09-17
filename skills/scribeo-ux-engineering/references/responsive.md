# Responsive Engineering

Load for breakpoint strategy, per-device composition, navigation adaptation, and content density.

## The governing rule

**Mobile is not a narrowed desktop.** Each width is a composition with its own priorities. Ask per breakpoint: what does the user need *here*, in this context, on this device?

Decide the responsive strategy during structuring — before CSS. Retrofitting mobile onto a finished desktop layout produces the compromises users feel.

## Breakpoints

Derive breakpoints from where **content breaks**, not from device names. Inspect the layout, find the width where it fails, place a breakpoint there.

Practical baseline, adjusted per project:

| Range | Target | Composition |
| --- | --- | --- |
| 320–479 | Small phones | Single column, maximum density reduction |
| 480–767 | Phones | Single column, some pairing |
| 768–1023 | Tablet portrait | 2-column, restructured nav |
| 1024–1279 | Tablet landscape, small laptop | Near-desktop, tighter gutters |
| 1280–1535 | Desktop | Full composition |
| 1536+ | Large desktop | Max-width holds; whitespace grows |

Write mobile-first (`min-width` queries). Base styles are the mobile case; enhancements add upward. This keeps the cascade additive and source order mobile-correct by default.

**Prefer intrinsic responsiveness over breakpoints** — `clamp()`, `auto-fit`, `min()`, container queries. The best responsive layout has the fewest media queries, because each one is a place to forget something.

## Desktop

- **Max-width strategy:** layouts stop growing; whitespace absorbs the excess. Never let a 2560px monitor stretch content — but never centre a narrow column in a sea of empty space either. Grow whitespace *and* increase compositional ambition (bleed imagery, asymmetric splits).
- **Grid:** full composition available. Asymmetry, overlap, and editorial breakouts belong here.
- **Whitespace:** the primary premium signal at this width. Generous section padding, wide gutters, and space around focal elements.
- **Typography:** display type at full scale; largest contrast between display and body.
- **Navigation:** horizontal, fully visible, no disclosure needed. Mega-menus only with genuine breadth.
- **Hover:** enhancement only. Reveal supporting detail, never essential content. Must have a keyboard equivalent, and must not shift layout on hover (causes flicker loops at boundaries).

## Tablet

The most-neglected width, and where template-built sites visibly break.

- **Do not simply inherit either neighbour.** Desktop layouts are cramped at 768px; mobile layouts waste it.
- **Grid restructuring:** 3-column → 2-column, not straight to 1. 4-column → 2×2. Reconsider the grid rather than dropping columns.
- **Navigation:** the real decision point. Horizontal nav usually survives portrait tablet if the item count is low; compress spacing and reduce type before collapsing to a menu. Landscape tablet should keep full nav.
- **Typography:** intermediate display sizes — `clamp()` handles this, but verify actual rendering at 768 and 1024. Interpolated sizes sometimes land awkwardly.
- **Touch, not hover:** tablets are touch devices at desktop-ish widths. Never gate content behind hover here. Target sizes follow mobile rules, not desktop.
- **Verify both orientations** — 768×1024 and 1024×768 are different problems.

## Mobile

- **Thumb reach:** the bottom half of the screen is comfortable; the top corners are not. Primary actions belong within reach or in a persistent bar. Destructive actions do not.
- **Touch targets:** ≥44×44px for primary controls; absolute floor 24×24px with spacing (WCAG 2.2 SC 2.5.8). Adjacent targets need separation — size alone does not prevent mis-taps.
- **Content prioritisation:** re-rank, do not just stack. What was a right-hand sidebar may belong immediately after the hero — or be dropped. State what mobile hides, and why.
- **Source order:** set the DOM to the mobile-correct reading order and use grid placement for desktop. Never use `order` or `flex-direction: row-reverse` to fix reading sequence — it desynchronises visual and keyboard/screen-reader order.
- **Vertical rhythm:** reduce section padding, but keep rhythm proportional. Mobile sections that use desktop padding waste the viewport; sections with no breathing room read as cramped.
- **Typography:** 16px minimum body. Display type must actually shrink — a 6rem hero at 375px is unusable. Measure is naturally constrained; check that padding does not squeeze it below ~40ch.
- **No horizontal overflow, ever.** Verify at 320px. Usual causes: fixed widths, `min-width` on flex/grid children, wide tables, unconstrained images, long unbroken strings, `100vw` with a visible scrollbar.
- **CTA accessibility:** the primary action must be reachable without hunting. Either repeated at natural decision points or persistent. Persistent bars must not obscure content — pad the page bottom by the bar height, and respect `env(safe-area-inset-bottom)`.
- **Media:** images need explicit `width`/`height` or `aspect-ratio` to prevent layout shift. Serve appropriately sized files via `srcset`/`sizes`. Autoplaying background video on mobile is a bandwidth and battery cost — use a poster image unless the video *is* the content. (Byte-level optimisation belongs to `scribeo-performance`.)
- **Density:** reduce deliberately — fewer simultaneous elements, shorter copy blocks, collapsed secondary detail. Reducing density is not reducing content; it is sequencing it.

## Navigation adaptation

| Width | Pattern |
| --- | --- |
| Desktop | Full horizontal, all primary items visible |
| Tablet landscape | Full horizontal, compressed spacing |
| Tablet portrait | Horizontal if ≤5 items, else disclosure |
| Mobile | Disclosure (drawer/sheet) with critical action kept visible outside it |

Mobile menu requirements: a real `<button>` with `aria-expanded` · focus moves in on open and returns to the trigger on close · `Escape` closes · focus is trapped while open · background scroll locked · the primary CTA and phone number remain visible outside the menu when they are the conversion path.

## Verification

Not optional, and not a screenshot at one width:

- 320, 375, 414, 768, 1024, 1280, 1440, 1920
- Both tablet orientations
- Real device or accurate emulation for touch behaviour
- Keyboard-only traversal at each major width
- Long content, empty content, missing images
- Text zoomed to 200% (WCAG 1.4.4) — layout must survive
- `prefers-reduced-motion: reduce`

## Responsive defects

| Defect | Cause / fix |
| --- | --- |
| Horizontal scroll on mobile | Fixed widths, unconstrained children (`min-width: 0`), wide tables, `100vw` |
| Tablet looks broken | No intermediate composition — design 768–1024 explicitly |
| Hero unreadable on mobile | Display type not scaled independently of body |
| Mobile reading order wrong | Fixed with `order` instead of correct source order |
| CTA unreachable | No repetition or persistent access on long mobile pages |
| Hover-only content on touch | Hover used as the only disclosure path |
| Layout shift as images load | Missing `width`/`height` or `aspect-ratio` |
| Sticky bar covers content | No bottom padding, no safe-area inset |
