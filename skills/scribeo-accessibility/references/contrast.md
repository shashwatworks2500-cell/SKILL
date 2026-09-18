# Contrast

Load when auditing colour or non-text contrast.

All values verified against W3C Understanding documents (September 2026). **Re-verify before quoting in client-facing work.**

**"It looks readable to me" is not a measurement.** Contrast is a computed ratio. Measure it.

## 1.4.3 Contrast (Minimum) — Level AA

Text and images of text must have a contrast ratio of at least:

| Text | Ratio |
| --- | --- |
| Normal | **4.5:1** |
| Large-scale | **3:1** |

**Large-scale text** is defined as at least **18 point** (approximately 24px), or at least **14 point bold** (approximately 18.5px) — using W3C's stated conversion of 1pt ≈ 1.333px. Equivalent sizing applies for CJK.

**Exceptions** (exempt from the requirement):

- **Incidental text** — part of an inactive component, pure decoration, invisible, or text within a picture that carries significant other visual content.
- **Logotypes** — text that is part of a logo or brand name.
- **Disabled controls** — inactive user interface components not available for interaction.

The disabled exception is frequently over-applied. It covers genuinely inactive controls, not low-contrast text you would prefer to keep.

## 1.4.6 Contrast (Enhanced) — Level AAA

Raises the requirement to **7:1** for normal text (and 4.5:1 for large text). Relevant when a client targets AAA, or for body text on a content-heavy site where it is a quality decision rather than an obligation.

## 1.4.11 Non-text Contrast — Level AA

**3:1 minimum against adjacent colours** for:

1. **User interface components** — visual information required to identify components and their states, excluding inactive components.
2. **Graphical objects** — parts of graphics required to understand the content, unless a particular presentation is essential.

**Focus indicators fall under this criterion.** W3C is explicit: a focus indicator must have sufficient contrast against the adjacent background when the component is focused — whether it sits inside, outside, or straddles the component boundary. A subtle grey ring on a light background is a conformance failure, not a design preference.

**Exceptions:** inactive components · appearance determined by the user agent and not modified by the author · graphics where the presentation is essential (logos, flags, medical imagery, colour-coded data such as heatmaps) · purely decorative graphics, or ones whose information is also conveyed in text.

**What this catches in practice:** input borders that are nearly invisible · toggle switches whose on/off states differ only subtly · icon-only buttons as low-contrast line art · chart series distinguished only by similar tints · custom checkbox and radio outlines.

## Colour is never the only signal

**1.4.1 Use of Color (Level A)** — colour must not be the sole means of conveying information.

Common failures: a red border as the only error indicator · required fields marked only in colour · chart series distinguished only by colour · links in body text identified only by colour, with no underline or other non-colour distinction · an active nav item shown only by colour.

Pair colour with text, an icon, an underline, weight, or a shape change. This is independent of contrast: a perfectly-contrasting red that carries meaning alone still fails.

## Measuring

Read the actual computed values rather than the design file — implementations drift.

```js
() => {
  const lum = (rgb) => {
    const [r, g, b] = rgb.map(v => { v /= 255; return v <= 0.03928 ? v / 12.92 : Math.pow((v + 0.055) / 1.055, 2.4); });
    return 0.2126 * r + 0.7152 * g + 0.0722 * b;
  };
  const parse = (c) => (c.match(/\d+(\.\d+)?/g) || []).slice(0, 3).map(Number);
  const ratio = (a, b) => { const [l1, l2] = [lum(a), lum(b)].sort((x, y) => y - x); return (l1 + 0.05) / (l2 + 0.05); };
  const bgOf = (el) => { let n = el; while (n) { const bg = getComputedStyle(n).backgroundColor;
      if (bg && bg !== "rgba(0, 0, 0, 0)" && bg !== "transparent") return parse(bg); n = n.parentElement; }
    return [255, 255, 255]; };
  return [...document.querySelectorAll("p, li, a, span, h1, h2, h3, h4, button, label")]
    .filter(el => el.textContent.trim() && el.offsetParent)
    .map(el => { const s = getComputedStyle(el);
      const px = parseFloat(s.fontSize);
      const bold = parseInt(s.fontWeight, 10) >= 700;
      const large = px >= 24 || (bold && px >= 18.5);
      const r = ratio(parse(s.color), bgOf(el));
      return { text: el.textContent.trim().slice(0, 30), px: Math.round(px), large,
               ratio: +r.toFixed(2), required: large ? 3 : 4.5, pass: r >= (large ? 3 : 4.5) };
    })
    .filter(x => !x.pass);
}
```

Returns only failures. **Caveats to state when reporting from it:** it walks up for an opaque background, so it cannot resolve text over images, gradients, or semi-transparent overlays — those need visual judgement, and a screenshot routed to `scribeo-visual-qa`. It also samples a subset of selectors.

**Check every background a colour appears on.** A grey that passes on white commonly fails on a tinted section. One measurement is not coverage.

## Boundary with design

This skill owns the **requirement** — the ratio, which elements it applies to, and whether an implementation meets it.

`frontend-design` owns the **palette**. When a brand colour fails, do not pick a replacement here. Report the measured failure and the required ratio, and route the colour decision. Usually a small lightness adjustment preserves the brand and passes — but that is their call, not this skill's.

Where a fix changes rendering, route the visual check to `scribeo-visual-qa`.

## Audit checklist

- [ ] Body text ≥ 4.5:1; large text ≥ 3:1 (AA)
- [ ] Every background variant measured, not just the default
- [ ] Text over images and gradients assessed visually
- [ ] UI component boundaries and states ≥ 3:1
- [ ] Focus indicators ≥ 3:1 against adjacent colours
- [ ] Meaningful graphics and chart elements ≥ 3:1
- [ ] Nothing conveyed by colour alone
- [ ] Placeholder text measured (frequently fails)
- [ ] Disabled-state exemption applied only to genuinely inactive controls
- [ ] Palette changes routed to `frontend-design`
