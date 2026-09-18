# Font Performance

Load when fonts affect LCP, CLS, or payload.

Fonts are small relative to media but sit on the critical rendering path — they can block text paint and shift layout after it. Typeface *selection* is `frontend-design`'s; delivery is this skill's.

## Diagnose

```js
() => ({
  status: document.fonts.status,
  count: document.fonts.size,
  families: [...new Set([...document.fonts].map(f => `${f.family} ${f.weight} ${f.style}`))],
})
```

Pair with `browser_network_requests` filtered to font files: how many, how large, how early, and from which origin.

## The four levers

**1. Number of files.** Each weight, style, and subset is a separate request. A four-weight family plus italics is eight files.

Audit what is actually used:

```js
() => [...new Set([...document.querySelectorAll("*")].map(el => {
  const s = getComputedStyle(el);
  return `${s.fontFamily.split(",")[0]} ${s.fontWeight} ${s.fontStyle}`;
}))]
```

Loaded weights that never appear are pure cost. Two or three weights is usually enough; a variable font can replace a range in one file — worth it when several weights are genuinely used, wasteful for one.

**2. File size.** WOFF2 for everything; it is the smallest broadly-supported format and there is no reason to ship WOFF or TTF alongside it for modern targets.

**3. Subsetting.** Full Unicode fonts carry glyphs no Latin site uses. Subsetting to the needed ranges commonly removes most of the file. Use `unicode-range` so the browser only fetches what a page needs.

```css
@font-face {
  font-family: "Scribeo Sans";
  src: url("/fonts/scribeo-sans.woff2") format("woff2");
  font-weight: 400;
  font-display: swap;
  unicode-range: U+0000-00FF, U+0131, U+2000-206F, U+2212;
}
```

Be careful subsetting a site with user-generated content or multiple languages — a missing glyph is a visible defect.

**4. Discovery timing.** A font referenced only in CSS is discovered late: fetch CSS, parse it, match the rule, then request the font. For a font that renders the LCP text, that chain is often the render delay.

## font-display

| Value | Behaviour | Performance effect |
| --- | --- | --- |
| `swap` | Fallback immediately, swap when ready | Text paints fast; **shift risk when metrics differ** |
| `optional` | Very short block; may skip the font entirely this load | Best for metrics; may not use the font |
| `fallback` | Short block, short swap window | Compromise |
| `block` | Invisible text up to ~3s | Worst — blocks text paint, can wreck LCP |
| `auto` | Browser default, usually block-like | Avoid; be explicit |

**`swap` is the usual default** and converts a blocking problem into a shift problem. Fix the shift with metric matching rather than reverting to `block`.

## Eliminating the swap shift

The swap shift happens because the fallback and the web font have different metrics, so text reflows on arrival. Control it by matching the fallback:

```css
@font-face {
  font-family: "Fallback Adjusted";
  src: local("Arial");
  size-adjust: 96%;
  ascent-override: 90%;
  descent-override: 22%;
  line-gap-override: 0%;
}
body { font-family: "Scribeo Sans", "Fallback Adjusted", sans-serif; }
```

Tune the overrides until the fallback occupies the same space, then verify visually — the shift measurement is this skill's, the acceptability of the fallback rendering is `scribeo-visual-qa`'s.

## preload — only with a reason

```html
<link rel="preload" href="/fonts/display.woff2" as="font" type="font/woff2" crossorigin>
```

**Preload at most the one or two fonts that render above-the-fold text.** Every preload competes for bandwidth with the LCP resource; preloading four fonts will make LCP worse, not better.

`crossorigin` is required for fonts even same-origin, or the file is fetched twice. That is a common silent waste.

## Self-hosting vs third-party

| | Self-hosted | Third-party |
| --- | --- | --- |
| Connection | None extra | Extra DNS + TLS on the critical path |
| Priority control | Full — preload, headers, subsets | Limited |
| Caching | Your headers, long-lived | Their policy |
| Privacy | No third-party request | A third-party request per visitor |

**Self-hosting is usually faster** — it removes a connection setup and gives control over subsetting, priority, and cache lifetime. If a third-party host must stay, `preconnect` to it, and only it.

## Frequent findings

| Finding | Fix |
| --- | --- |
| Six weights loaded, two used | Drop the unused; or one variable font |
| WOFF/TTF alongside WOFF2 | WOFF2 only |
| Full Unicode range | Subset with `unicode-range` |
| `font-display: block` or unset | `swap`, plus metric-matched fallback |
| Text shifts on font load | `size-adjust` / metric overrides |
| Preloading every font | Preload only above-the-fold faces |
| Missing `crossorigin` on font preload | Add it — otherwise double fetch |
| Third-party font on the critical path | Self-host, or `preconnect` |
| Variable font for a single weight | Use a static instance |
