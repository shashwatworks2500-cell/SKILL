# Viewport & Breakpoint QA

Load when choosing widths and planning coverage.

## Derive widths, do not memorise them

**The right viewports come from the project, not from a list.** Choose them from, in order:

1. **The project's actual CSS breakpoints.** Read them out of the stylesheet or config. These are where layout genuinely changes, so they are where it genuinely breaks.
2. **Content transition points** — where a grid drops a column, nav collapses, or a heading rewraps. These may not sit on a declared breakpoint.
3. **Known failure points** — anywhere a defect has been found before.
4. **The audience's real device classes**, from analytics when available. A B2B tool and a consumer site have different critical widths.
5. **Extremes** — the narrowest supported width and an ultrawide desktop.

## Default matrix

A reasonable starting set when project data is unavailable. **Replace these with real breakpoints as soon as you can read them.**

| Class | Width × Height | Represents |
| --- | --- | --- |
| Small mobile | 320 × 568 | Narrowest realistically supported; overflow surfaces here first |
| Mobile | 390 × 844 | Common modern phone |
| Large mobile | 430 × 932 | Large phone |
| Tablet portrait | 768 × 1024 | The most-neglected width |
| Tablet landscape | 1024 × 768 | Different problem from portrait — check both |
| Laptop | 1280 × 800 | Common working width |
| Desktop | 1440 × 900 | Design default |
| Large desktop | 1920 × 1080 | Max-width behaviour, whitespace growth |

320px is the highest-yield single width in the set: if horizontal overflow exists anywhere, it usually shows there first.

## Test the edges, not just the middle

Most responsive defects live **at** a breakpoint, not in the comfortable middle of a range. For each real breakpoint, check three widths:

| Position | Example (768px breakpoint) | Catches |
| --- | --- | --- |
| Just below | 767 | The mobile layout at its widest — stretched cards, over-long measure |
| At | 768 | Off-by-one boundary errors, `min-`/`max-width` overlap or gaps |
| Just above | 769 | The desktop layout at its narrowest — the most common break point |

"Just above the breakpoint" is where multi-column layouts are most cramped and where overflow appears. Check it every time.

## Height matters too

Width gets the attention; height causes real defects.

- **Short viewports** (landscape phone, ~430×390) break full-height heroes and can hide the primary CTA entirely.
- **`100vh` on mobile** includes space under the browser chrome, causing a jump as it hides. `dvh` is the fix — verify which is used.
- **Sticky headers** consume a fixed slice of a short viewport; check the usable remainder.
- **Anchor targets** hidden under a sticky header need `scroll-padding-top`.

## Orientation

Verify both tablet orientations explicitly — 768×1024 and 1024×768 hit different breakpoints and different layouts. Landscape phone is worth one check on any site with a full-height hero.

## Zoom

Text zoomed to 200% must remain usable — real users browse this way. It commonly surfaces overflow and clipping that no viewport width reveals. Report what you see; conformance judgement is `scribeo-accessibility`'s.

## Coverage discipline

- **Every route at every viewport is usually wasteful.** Full matrix on templates that differ structurally; primary widths on routes that reuse a verified template.
- **After any responsive change, re-check the adjacent breakpoints**, not only the one that was fixed. Fixes routinely push the defect one range over.
- **Record the width with every finding.** A defect without a viewport is not reproducible.
