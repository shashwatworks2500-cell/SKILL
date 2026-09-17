# Typography

Load for type selection, scale, hierarchy, measure, line-height, tracking, and responsive scaling.

Typography carries most of the perceived quality of a premium site. It is also where "generic" is decided — a flat scale in a default sans reads as unconsidered no matter what surrounds it.

## Selection

Choose from brand evidence, not preference. In order: existing brand typeface → a typeface matching the brand's character → a well-engineered neutral.

Match **character** to positioning:

| Direction | Type character | Notes |
| --- | --- | --- |
| Editorial, heritage, authority | Transitional/old-style serif display | Pair with a humanist sans for body |
| Luxury, fashion, restraint | High-contrast didone or refined serif | Needs large sizes and generous space; fragile when small |
| Modern, confident, direct | Grotesque sans, tight tracking at display | Weight contrast does the hierarchy work |
| Technical, precise, systematic | Neo-grotesque or mono accents | Mono for metadata only, never body |
| Warm, human, approachable | Humanist sans or soft serif | Avoid rounded novelty faces |

**Two families maximum** — one display, one text. A third is a decision requiring a reason. A single well-engineered family with a real weight range often beats a pairing.

Pairing rule: **contrast or clearly harmonise — never nearly match.** Two similar sans faces look like a mistake. A serif display with a sans body is a decision.

## Scale

Build a scale with genuine contrast. The most common quality failure is a scale that is too flat: 16 / 18 / 20 / 24 asks the reader to measure differences rather than perceive them.

Use a ratio of ~1.25 for text steps and a deliberate jump to display:

```css
--text-xs: 0.75rem;   /* metadata, legal */
--text-sm: 0.875rem;  /* labels, captions */
--text-base: 1rem;    /* body — never below this on mobile */
--text-lg: 1.125rem;  /* lead paragraph */
--text-xl: 1.375rem;  /* h4 / large supporting */
--text-2xl: 1.75rem;  /* h3 */
--text-3xl: 2.25rem;  /* h2 */
--text-4xl: 3rem;     /* h1 */
--text-display: clamp(2.75rem, 7vw, 6rem);  /* hero only */
```

Hierarchy needs **at least two levels of visual separation** between adjacent roles — size, or weight, or colour, or letter-spacing. One dimension alone is usually too subtle.

## Responsive scaling

```css
h1 { font-size: clamp(2rem, 1.25rem + 3.5vw, 4rem); }
```

`clamp()` with a `rem` floor and ceiling, and `vw` only in the middle term. Three rules:

1. **Never `font-size: 5vw` alone** — it prevents users scaling text and fails WCAG 1.4.4.
2. **Floor at readable, not minimum** — 16px body on mobile is the floor. Below 16px in a form input, iOS Safari zooms the viewport on focus.
3. **Display type shrinks faster than body.** Body barely changes across breakpoints; a 6rem hero must become ~2.5rem. Scale them independently, never with one global multiplier.

## Line-height

Inversely proportional to size — large type needs tighter leading:

| Role | Line-height |
| --- | --- |
| Display (48px+) | 1.0–1.1 |
| Headings (24–48px) | 1.15–1.25 |
| Body | 1.5–1.65 |
| Long-form reading | 1.6–1.75 |
| UI labels, buttons | 1.2–1.4 |

Body at 1.2 is cramped and reads as low quality. Display at 1.5 disintegrates into unrelated lines.

## Tracking

| Context | Tracking |
| --- | --- |
| Large display | `-0.02em` to `-0.04em` |
| Headings | `-0.01em` to `-0.02em` |
| Body | `0` (trust the designer) |
| Small caps, eyebrow labels | `0.06em` to `0.12em` |
| All-caps at any size | `0.04em` minimum |

All-caps without added tracking is a reliable amateur tell. Negative tracking on body text damages readability — it is a display-only tool.

## Measure

**60–75 characters** for body copy. Below ~45 the eye returns too often; above ~85 it loses the line on return.

```css
.prose { max-width: 68ch; }
```

Use `ch` — it tracks the actual font, unlike a fixed `px`. Shorter measure (45–60ch) suits large lead paragraphs and multi-column layouts.

## Roles

Assign every text role explicitly before styling:

| Role | Treatment |
| --- | --- |
| Display | Largest, tightest leading, negative tracking, restrained weight |
| Heading | Clear step down; consistent weight across a level |
| Lead | 1.125–1.25rem, sometimes lighter colour, shorter measure |
| Body | 1rem, 1.5–1.65 leading, 60–75ch |
| Supporting | Smaller or lower contrast — never both, or it becomes unreadable |
| Label | Small, often uppercase with tracking, medium weight |
| Button | Same size as body or one step down; never lighter than medium |
| Navigation | Body size or smaller; weight/colour for active state, not size |
| Metadata | Smallest step; must still meet 4.5:1 contrast |

## Contrast and colour

- Body text 4.5:1 minimum. Large text (≥24px, or ≥18.66px bold) 3:1 minimum.
- Never use pure `#000` on pure `#fff` for long-form — harsh and fatiguing. Near-black on warm off-white reads as more considered and more comfortable.
- "Muted" text still needs 4.5:1. A grey that passes on white fails on a tinted section — check per background, not once.
- Weight is not a substitute for contrast. Light weight at low contrast is the most common readability failure in "premium minimal" work.

## Type defects

| Defect | Fix |
| --- | --- |
| Scale too flat | Widen the ratio; add weight/colour contrast |
| Hero shrinks poorly on mobile | Separate `clamp()` per role, not one global scale |
| Body under 16px on mobile | Raise to 16px; inputs zoom on iOS below it |
| Full-width prose | Apply `max-width` in `ch` |
| All-caps untracked | Add ≥0.04em |
| Two near-identical families | Increase contrast or drop to one |
| Muted text unreadable | Verify contrast on every background it appears on |
| Body tracking tightened | Reset to 0 |
