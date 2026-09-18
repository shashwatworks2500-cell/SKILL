# Zoom, Reflow, Target Size & Pointer

Load when auditing zoom, text resize, reflow, or pointer and touch accessibility.

Values verified against W3C Understanding documents (September 2026). **Re-verify before quoting.**

**Responsive is not the same as accessible.** A layout can adapt perfectly to a narrow viewport and still fail reflow, because reflow is tested by **zooming**, not by resizing the window. Zoom scales content within the same viewport, which stresses different code paths.

## 1.4.4 Resize Text — Level AA

Text must be resizable to at least **200%** without loss of content or functionality.

Practical implications:

- **Never size text in `vw` alone** — viewport units do not respond to the user's text-size preference, defeating the requirement.
- Use relative units (`rem`, `em`) so the root size scales.
- Containers must grow or scroll rather than clip.
- Watch for fixed heights on text containers — the classic failure, where the box stays and the text is cut off.

## 1.4.10 Reflow — Level AA

Content must be presentable without loss of information or functionality, and **without requiring scrolling in two dimensions**, at:

- **320 CSS pixels** wide for vertically-scrolling content
- **256 CSS pixels** tall for horizontally-scrolling content

W3C states the 320px width is **equivalent to a 1280px viewport at 400% zoom** — which is how to test it.

**Exceptions:** parts of content requiring two-dimensional layout for use or meaning — images needed for understanding (maps, diagrams), video, games, presentations, data tables (the table, not individual cells), and interfaces where keeping toolbars in view is necessary.

**How to test:** set the viewport to 1280px, apply 400% browser zoom, and confirm no horizontal scrolling and nothing lost. Testing only by narrowing the window is not equivalent and misses real failures.

Common reflow failures: fixed-width containers · large fixed-position elements consuming the viewport · sticky headers eating most of the height at high zoom · absolutely-positioned elements overlapping when scaled · wide tables (which have a data-table exception, but need a scroll container) · content clipped by `overflow: hidden` on a fixed-height ancestor.

```js
() => ({
  scrollW: document.documentElement.scrollWidth,
  clientW: document.documentElement.clientWidth,
  twoDimensional: document.documentElement.scrollWidth > document.documentElement.clientWidth + 1,
})
```

Run this **at 320px effective width** to check reflow, not at the default viewport.

## 1.4.12 Text Spacing — Level AA

Content must survive user-applied spacing overrides — increased line height, paragraph spacing, letter and word spacing — without loss of content or functionality.

The failure mode is identical to text resize: fixed-height containers clipping expanded text. Test by injecting the spacing overrides and checking for clipping.

## 2.5.8 Target Size (Minimum) — Level AA (new in 2.2)

Pointer targets must be **at least 24 × 24 CSS pixels**, with five exceptions:

1. **Spacing** — an undersized target passes if a 24px-diameter circle centred on its bounding box does not intersect another target's circle.
2. **Equivalent** — the same function is available via another control on the page that does meet the size.
3. **Inline** — targets within a sentence, or constrained by the line height of surrounding text.
4. **User agent control** — size determined by the user agent and not modified by the author.
5. **Essential** — where the size or spacing is fundamental to the information (map pins, data visualisations, legally required form replicas).

**24 × 24 is the AA floor, not a design target.** The spacing exception is what makes small icon buttons conformant when adequately separated — so measure both size *and* separation.

**2.5.5 Target Size (Enhanced)** is the stricter AAA criterion for important controls. **Verify its exact value from W3C before citing a number** — do not assume it.

```js
() => [...document.querySelectorAll("a[href], button, input, select, [role=button], [role=link]")]
  .filter(el => el.offsetParent)
  .map(el => { const r = el.getBoundingClientRect();
    return { el: `${el.tagName}.${el.className}`.slice(0, 40),
             w: Math.round(r.width), h: Math.round(r.height),
             under24: r.width < 24 || r.height < 24 };
  })
  .filter(x => x.under24)
```

Anything returned needs checking against the spacing and inline exceptions before being reported as a failure.

## 2.5.7 Dragging Movements — Level AA (new in 2.2)

Any function using a dragging movement must have a **single-pointer alternative** that does not require dragging — unless dragging is essential.

Affects: sliders (provide arrow-key and/or text input), drag-and-drop reordering (provide move up/down buttons or a menu), carousels driven by swipe (provide previous/next buttons), map panning (provide directional controls or zoom buttons), custom range inputs.

Native `<input type="range">` supports keyboard operation already — a strong argument for using it over a custom slider.

## 2.5.2 Pointer Cancellation — Level A

For single-pointer activation, at least one must hold: no down-event activation · the action is completed on up-event with the ability to abort by moving away · up-event reverses the down-event outcome · down-event activation is essential.

In practice: **activate on `click`/`pointerup`, not on `pointerdown`** or `mousedown`. Down-event activation means a user cannot abort a mis-touch by sliding their finger away — a real barrier for anyone with tremor or limited precision.

## 2.5.1 Pointer Gestures — Level A

Multipoint or path-based gestures (pinch, two-finger swipe, drawing a shape) must have a single-pointer alternative unless essential.

## Hover-only content

Content available only on hover fails for keyboard and touch users (2.1.1), and must also satisfy **1.4.13 Content on Hover or Focus (AA)** — dismissible, hoverable, persistent. See `dynamic-content.md`.

Detect capability rather than device:

```css
@media (hover: hover) and (pointer: fine) {
  .card:hover .detail { opacity: 1; }
}
```

The information must still be reachable without hover.

## Mobile accessibility

- **Never disable zoom.** `user-scalable=no` or `maximum-scale=1` in the viewport meta tag blocks pinch-zoom and fails 1.4.4 for many users. This is one of the most damaging single attributes in mobile web.
- Verify targets and spacing at real mobile sizes.
- Ensure the on-screen keyboard does not obscure the focused field.
- Confirm orientation is not locked unnecessarily (**1.3.4 Orientation**, AA).
- Touch and screen-reader gestures (TalkBack, VoiceOver) can conflict with custom gesture handlers — a reason to avoid custom gestures.

## Audit checklist

- [ ] Text scales to 200% with no loss
- [ ] Reflow verified at 1280px + 400% zoom — no two-dimensional scrolling
- [ ] No text sized in `vw` alone
- [ ] Text-spacing overrides survive without clipping
- [ ] Targets ≥ 24 × 24, or a documented exception applies
- [ ] Dragging functions have a single-pointer alternative
- [ ] Activation on up-event, not down-event
- [ ] Gestures have simple alternatives
- [ ] Nothing essential is hover-only
- [ ] Zoom not disabled in the viewport meta tag
- [ ] Orientation not locked without cause
