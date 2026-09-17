# Components

Load when building buttons, CTAs, cards, sections, navigation, or forms.

## Buttons and CTAs

**Semantics first.** `<button>` performs an action; `<a href>` navigates. A `<div onclick>` is never acceptable — it loses keyboard access, focus, role, and Enter/Space handling.

```html
<!-- Wrong: no keyboard access, no role, no focus -->
<div class="btn" onclick="submit()">Send enquiry</div>

<!-- Right -->
<button type="submit" class="btn btn--primary">Send enquiry</button>
<a class="btn btn--primary" href="/contact">Book a viewing</a>
```

**Hierarchy — exactly three levels, no more:**

| Level | Role | Rule |
| --- | --- | --- |
| Primary | The one action that matters | **One per viewport.** Highest contrast |
| Secondary | Meaningful alternative | Outlined or tonal; clearly subordinate |
| Tertiary | Low-commitment | Text/link style |

Two primary buttons side by side means the hierarchy decision was avoided. Decide which one matters.

**Labels** state the outcome, in the user's words: "Book a viewing", "Get a quote", "Send enquiry". Never "Submit", "Click here", or "Learn more" (learn what?). Avoid first person ("Get my free guide") unless the brand voice genuinely uses it — it reads as a template.

**Requirements:** ≥44px touch height for primary · adequate horizontal padding (cramped padding reads cheap) · visible `:focus-visible` · disabled state that explains itself, or is avoided entirely (better: keep enabled, validate on submit, explain what is missing) · loading state that prevents double submission while preserving width.

**Placement:** the primary CTA appears where the user has enough information to act — not only in the hero. On a long page, repeat it at natural decision points. Never bury the only CTA in the footer.

## Cards

Cards group related content into a scannable unit. They are not a default container.

**Use a card when** items are comparable and repeated, each is independently actionable, and the group benefits from visual separation. **Do not use one** for a single piece of content, to add visual interest, or to wrap every section (the fastest route to generic).

Structure:
```html
<article class="card">
  <img src="..." alt="" width="640" height="420">
  <div class="card__body">
    <h3 class="card__title"><a href="/property/12">Riverside apartment, Bath</a></h3>
    <p class="card__meta">2 bed · 2 bath · £485,000</p>
    <p class="card__excerpt">…</p>
  </div>
</article>
```

Rules: heading inside the card is a real heading at the correct level · **one primary link per card** — the title link, made large by a pseudo-element overlay rather than wrapping the whole card in `<a>` (which produces an unusable screen-reader label and swallows nested links) · decorative images get `alt=""`, meaningful ones get real alt text · equal heights via grid, never fixed heights (truncation is content loss) · define the state where an image, excerpt, or price is missing.

```css
/* Whole-card click target without breaking semantics */
.card { position: relative; }
.card__title a::after { content: ""; position: absolute; inset: 0; }
```

Avoid: shadow + border + radius + gradient stacked on one card · cards containing three competing CTAs · card grids where every item is visually identical and none is emphasised.

## Sections

A section earns its place by doing a job the page needs. Derive sections from content and goal — never assemble them from a remembered template.

Each section should answer: what does the user know now that they did not before, and what are they able to do next?

- One purpose per section. Two purposes means two sections.
- One focal point per section.
- Vary structure across the page: bleed media, asymmetric split, measured prose, dense grid, single statement. Uniform section structure is the clearest template tell.
- Vary density deliberately — a quiet section makes the next dense one land harder.
- Use `<section>` with an accessible name (a heading, or `aria-labelledby`). An unnamed `<section>` provides no more semantics than a `<div>`.

## Navigation

**Primary:**
```html
<nav aria-label="Primary">
  <ul>
    <li><a href="/work" aria-current="page">Work</a></li>
    <li><a href="/studio">Studio</a></li>
  </ul>
</nav>
```
A list of links in a labelled `<nav>`. `aria-current="page"` marks the active item — and the active state must be visible without relying on colour alone.

Keep primary items to 5–7. More means the IA needs work, not a mega-menu.

**Mobile:** real `<button>` with `aria-expanded` and `aria-controls` · focus moves into the panel on open, returns to the trigger on close · `Escape` closes · focus trapped while open · background scroll locked · keep the conversion path (CTA, phone) visible outside the menu.

**Sticky:** justified when navigation or the CTA is needed throughout a long page. Requirements: compact height (a sticky header eating 15% of a mobile viewport is a cost, not a feature) · `scroll-padding-top` on the root so anchor targets are not hidden beneath it · does not obscure focused elements during keyboard traversal · no layout shift at the stick point.

**Breadcrumbs** only with genuine hierarchical depth (property listings, categorised catalogues, documentation). On a five-page marketing site they are noise. Use an ordered list in `<nav aria-label="Breadcrumb">` with the current page marked `aria-current="page"`.

**Skip link** to `#main` as the first focusable element on every page. Visible on focus. Non-negotiable.

## Forms

Forms are where conversion is lost. Full state behaviour in `states.md`.

**Structure**
```html
<div class="field">
  <label for="email">Email address</label>
  <input id="email" name="email" type="email"
         autocomplete="email" inputmode="email"
         required aria-describedby="email-hint">
  <p id="email-hint" class="hint">We reply within one working day.</p>
</div>
```

**Non-negotiables**

- **Every input has a visible `<label>`** with `for`/`id`. Placeholder-only labelling fails: the label vanishes on input, contrast is usually too low, and autofill hides it.
- **Placeholders are examples, never labels.** Use them for format hints ("07700 900123") or omit them.
- **`autocomplete` on every relevant field** (`name`, `email`, `tel`, `street-address`, `postal-code`). This is a genuine conversion lever on mobile, not a nicety.
- **`inputmode`/`type` to summon the right mobile keyboard** — `type="email"`, `type="tel"`, `inputmode="numeric"`. Getting this wrong adds real friction to every mobile submission.
- **Font size ≥16px on inputs** — iOS Safari zooms the viewport below that.
- **Mark required fields in text**, not by asterisk alone. Better still, mark the optional ones when most are required.
- **Label groups** with `<fieldset>`/`<legend>` for radios and checkboxes.
- **Never disable paste**, especially on email and password fields.
- **Targets ≥44px tall**, with generous spacing between fields.

**Reduce friction.** Every field costs conversions. Ask only what is needed to take the next step — a lead-gen form needs name, contact, and message; it does not need company size, job title, and budget range. Split genuinely long forms into steps with visible progress.

**Errors** appear next to the field, in text, describing the fix — not "Invalid input". Validate on blur, not on keystroke; re-validate on submit. On failed submit, move focus to the first error and summarise the count. Never rely on colour alone.

## Content density

Density is a decision, not a side effect.

| Lower density when | Higher density when |
| --- | --- |
| Considered, high-value purchase | Comparison or browsing task |
| Luxury or editorial positioning | Utility or transactional intent |
| Emotional or narrative content | Returning, goal-directed users |
| Mobile, early in the journey | Data-rich listings or catalogues |

Reducing density means sequencing content, not deleting it: progressive disclosure, accordions for genuinely secondary detail (never for primary content), pagination or lazy reveal for long lists.
