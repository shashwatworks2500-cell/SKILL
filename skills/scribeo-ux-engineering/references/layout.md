# Layout & Composition

Load for containers, grids, spacing rhythm, section composition, asymmetry, and editorial layout.

## Containers

Three container widths, chosen by content type — not one global wrapper:

| Container | Width | Use |
| --- | --- | --- |
| Text | 60–75ch | Prose, article body, long-form |
| Content | 1100–1280px | Standard sections, grids, cards |
| Wide | 1440–1600px | Immersive media, galleries, full compositions |
| Bleed | 100vw | Photography, video, colour fields |

```css
.container { width: 100%; margin-inline: auto; padding-inline: var(--gutter); }
.container--text    { max-width: 68ch; }
.container--content { max-width: 75rem; }
.container--wide    { max-width: 96rem; }
```

Gutters scale with viewport and never reach zero: `--gutter: clamp(1rem, 5vw, 3rem)`. A 16px minimum side gutter on mobile is the floor — text touching the screen edge reads as broken.

**Never let prose inherit the wide container.** A 1440px line of body copy is unreadable regardless of how good the type is.

## Spacing

One scale, geometric, used everywhere. Arbitrary values are how coherence dies.

```css
--space-2xs: 0.25rem;  --space-xs: 0.5rem;   --space-sm: 0.75rem;
--space-md: 1rem;      --space-lg: 1.5rem;   --space-xl: 2.5rem;
--space-2xl: 4rem;     --space-3xl: 6rem;    --space-4xl: 9rem;
```

**Proximity encodes relationship.** Space between a heading and its own paragraph must be visibly smaller than space to the next block. When those are equal, the reader cannot tell what belongs to what — the single most common spacing defect.

```css
/* Wrong — heading floats equidistant, ownership unclear */
h2, p { margin-block: var(--space-lg); }

/* Right — heading binds to its content, groups separate clearly */
h2 { margin-block: var(--space-2xl) var(--space-sm); }
p + p { margin-block-start: var(--space-md); }
```

Section rhythm should be fluid, not fixed:
```css
.section { padding-block: clamp(var(--space-2xl), 10vh, var(--space-4xl)); }
```

## Grids

Use CSS Grid for page and section structure, Flexbox for one-dimensional runs of items.

Prefer intrinsic sizing over breakpoint-counting — it removes an entire class of responsive bug:

```css
/* Self-adjusting card grid, no media queries, no orphan-column bugs */
.grid { display: grid; gap: var(--space-lg);
        grid-template-columns: repeat(auto-fit, minmax(min(18rem, 100%), 1fr)); }
```

`min(18rem, 100%)` is what prevents overflow on narrow screens — `minmax(18rem, 1fr)` alone overflows below 18rem.

Use an explicit column count only when the composition is deliberate (editorial asymmetry, a 7/5 split). Then name the lines:

```css
.editorial { display: grid; grid-template-columns:
  [full-start] minmax(var(--gutter),1fr)
  [wide-start] minmax(0,2rem)
  [text-start] min(68ch, 100% - var(--gutter)*2) [text-end]
  minmax(0,2rem) [wide-end]
  minmax(var(--gutter),1fr) [full-end]; }
.editorial > * { grid-column: text; }
.editorial > .bleed { grid-column: full; }
```

That pattern lets prose stay measured while images break out — the core editorial move — without wrapper divs.

**Use container queries for components, media queries for page layout.** A card that adapts to its container works in a sidebar, a grid, and a modal without variants:

```css
.card-host { container-type: inline-size; }
@container (min-width: 28rem) { .card { grid-template-columns: 8rem 1fr; } }
```

## Composition

Every section needs a **focal point** — one element the eye lands on first. Achieve it with contrast in scale, weight, colour, or isolation. If everything is emphasised, nothing is.

Hierarchy is **subtraction**. To strengthen the primary element, demote everything else rather than enlarging it further.

Vary section structure by purpose. A page where every section is centred heading → subheading → three cards reads as a template no matter how well styled. Alternate: full-bleed media, asymmetric split, measured prose, dense grid, quiet single statement.

**Vertical rhythm:** alternate density deliberately. Dense, dense, dense exhausts; a quiet section between two dense ones makes both land harder.

## Asymmetry

Asymmetry creates energy, direction, and editorial sophistication — when it follows a system.

**Useful when:** it follows the grid (7/5, 8/4 — not arbitrary offsets) · it directs the eye toward the primary action · it reflects genuine content imbalance (a hero image that deserves dominance) · it breaks monotony across a long page · a stable counterweight anchors it.

**Noise when:** it fights the grid rather than using it · it is applied per-section with no repeating logic · it pushes the primary action off the natural reading path · it collapses to an incoherent stack on mobile · it exists to look designed.

Test: state the rule out loud. "Content sits on the left five columns, imagery bleeds right, reversed every third section" is a system. "This bit is offset a little" is not.

## Alignment

- Pick a dominant alignment per section and commit. Mixed centre and left within one section reads as indecision.
- Centred text is for short statements — two lines, three at most. Centred paragraphs are hard to read because every line start moves.
- Align to optical edges, not just box edges: large quote marks, bullets, and icons need negative offsets to look aligned.
- Everything should sit on a shared invisible edge. Elements that are *nearly* aligned look accidental — full alignment or deliberate offset, never within a few pixels.

## Common layout defects

| Defect | Fix |
| --- | --- |
| Horizontal overflow on mobile | `min-width: 0` on flex/grid children; `min(x, 100%)` in `minmax()`; audit fixed widths and wide tables |
| Text spanning full ultrawide | Apply the text container to prose |
| Equal spacing everywhere | Bind headings to their content; separate groups |
| Sections all the same rhythm | Vary structure and density by purpose |
| Card grid with one orphan | `auto-fit` + intrinsic min, or reconsider the count |
| "Almost aligned" elements | Snap to a shared edge |
| Mobile stack in illogical order | Set source order to mobile-correct; use grid placement for desktop, never `order` to fix reading sequence |
