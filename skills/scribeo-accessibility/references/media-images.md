# Images & Media

Load when auditing alt text, captions, or media behaviour.

## Alt text is contextual

**1.1.1 Non-text Content (Level A).** Alt text is not a description of pixels — it is **the function the image performs in this context.** The same photograph needs different alt text on a portfolio page, in a news article, and inside a link.

Ask: *if this image were removed, what would the reader lose?* Write that.

| Image type | Alt approach |
| --- | --- |
| **Decorative** | `alt=""` — empty, never omitted. Removes it from the accessibility tree |
| **Informative** | Describe the information it conveys, not its appearance |
| **Functional** (inside a link or button) | Describe the **destination or action**, not the picture |
| **Text in an image** | Reproduce the text (and prefer real text in the first place) |
| **Complex** (chart, diagram, map) | Short alt plus a longer description nearby or via `aria-describedby` |

```html
<!-- Decorative: empty alt, not a missing attribute -->
<img src="texture.jpg" alt="">

<!-- Informative -->
<img src="workshop.jpg" alt="Three makers assembling frames at a long bench">

<!-- Functional: describe the destination -->
<a href="/"><img src="logo.svg" alt="Scribeo home"></a>

<!-- Wrong: describes the picture inside a link -->
<a href="/"><img src="logo.svg" alt="Scribeo logo"></a>
```

**`alt=""` and a missing `alt` are different.** Empty alt marks the image decorative and hides it. A missing attribute means some AT falls back to announcing the filename — `hero-final-v3-compressed.jpg`.

**Avoid redundancy:**

- No "image of", "photo of", "graphic of" — the role is already announced.
- Do not repeat adjacent caption or heading text. An image whose caption already describes it is decorative: `alt=""`.
- Do not duplicate the link text in a functional image's alt; that produces a doubled announcement.

```js
() => [...document.images].map(i => ({
  src: (i.currentSrc || i.src).split("/").pop().slice(0, 30),
  alt: i.getAttribute("alt"),
  missing: !i.hasAttribute("alt"),
  suspicious: /^(image|photo|graphic|picture|img|icon)\b/i.test(i.getAttribute("alt") || "")
             || /\.(jpg|jpeg|png|webp|avif|svg)$/i.test(i.getAttribute("alt") || ""),
  inLink: !!i.closest("a[href]"),
})).filter(x => x.missing || x.suspicious)
```

Missing attributes are failures; "suspicious" entries need human judgement.

**SVG:** decorative inline SVG needs `aria-hidden="true"` and `focusable="false"`. Meaningful SVG needs `role="img"` and an accessible name via `<title>` or `aria-label`.

**CSS background images** are invisible to AT entirely. Anything informative must not be a background image — or must have a text equivalent elsewhere.

## Video and audio

Requirements depend on whether media is prerecorded or live, and whether it has audio, video, or both.

| Content | Requirement | Criterion |
| --- | --- | --- |
| Prerecorded video with audio | Captions | 1.2.2 Captions (Prerecorded), A |
| Prerecorded video with audio | Audio description, or a full text alternative | 1.2.3 (A) / 1.2.5 Audio Description (AA) |
| Prerecorded audio only | Transcript | 1.2.1, A |
| Prerecorded video only, no audio | Audio track or text alternative | 1.2.1, A |
| Live video with audio | Captions | 1.2.4 Captions (Live), AA |

**Verify the exact obligation against W3C for the specific media type** before stating it in a report — the 1.2.x family is the most commonly mis-cited group in the standard, because the requirement depends precisely on what the media contains and its level differs per case.

**Practical points:**

- **Captions are not the same as subtitles.** Captions include speaker identification and relevant non-speech audio.
- **Auto-generated captions are a starting point, not compliance.** Accuracy matters; names, technical terms and punctuation are usually wrong.
- **A transcript benefits more people than captions alone** — it is searchable, skimmable, and usable without playing the media.
- **A silent decorative background video** carries no audio information, so captions do not apply — but 2.2.2 does if it auto-plays and runs over five seconds in parallel with other content.

## Media controls

- **Native `<video controls>` gives keyboard-accessible controls.** Custom players must reimplement all of it — focusable controls with accessible names, keyboard operation, state exposure. Most custom players fail 2.1.1 and 4.1.2.
- Controls must be reachable and operable by keyboard, with visible focus.
- Do not hide controls behind hover only.

## Autoplay

- **Audio that plays automatically for more than three seconds** needs a mechanism to pause or stop it, or to control volume independently (**1.4.2 Audio Control, Level A**). Autoplaying sound is also simply hostile; `muted` is the norm for a reason.
- **Auto-playing video** is auto-moving content: if it starts automatically, lasts over five seconds, and sits alongside other content, **2.2.2 Pause, Stop, Hide (A)** requires a pause, stop or hide mechanism.
- **Under `prefers-reduced-motion: reduce`, do not autoplay** — show the poster. See `motion.md`.

## Flashing

**2.3.1 Three Flashes or Below Threshold (Level A).** Nothing may flash more than three times per second, subject to luminance thresholds. This is a seizure-safety requirement with no aesthetic exemption.

Watch for: rapid-cut video · strobe effects · fast-toggling animations · flashing notification indicators. If in doubt, slow it down or remove it — the risk is not worth the effect.

## Audit checklist

- [ ] Every `<img>` has an `alt` attribute (empty for decorative)
- [ ] Alt text describes function in context, not appearance
- [ ] Functional images describe destination or action
- [ ] No "image of" prefixes or filename alt text
- [ ] No informative content in CSS background images
- [ ] Decorative SVG hidden; meaningful SVG named
- [ ] Complex images have an extended description
- [ ] Video with audio has accurate captions
- [ ] Transcripts provided where required
- [ ] Media controls keyboard operable with visible focus
- [ ] No autoplaying audio without a control (1.4.2, A)
- [ ] Auto-playing video over five seconds has pause/stop/hide (2.2.2, A)
- [ ] Nothing flashes more than three times per second (2.3.1, A)
- [ ] Autoplay suppressed under reduced motion
