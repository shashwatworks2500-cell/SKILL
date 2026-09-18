# Semantic HTML

Load when choosing elements, or auditing structure.

**Native semantics before ARIA, always.** A correct element brings role, state, keyboard behaviour, and focus management for free — and cannot fall out of sync with itself.

## button vs anchor vs div

| Element | Use | Keyboard |
| --- | --- | --- |
| `<button>` | Performs an action in the page | Enter and Space; focusable by default |
| `<a href>` | Navigates to a location | Enter; focusable by default |
| `<div>` / `<span>` | **Never** for an interactive control | Nothing |

```html
<!-- Wrong: no role, no focus, no keyboard, invisible to AT as a control -->
<div class="btn" onclick="submit()">Send</div>

<!-- Wrong: a link that acts, with no destination -->
<a href="#" onclick="submit()">Send</a>

<!-- Right -->
<button type="submit">Send enquiry</button>
<a href="/contact">Contact us</a>
```

The `<div onclick>` pattern fails multiple criteria at once — not keyboard operable (2.1.1), no name/role/value (4.1.2), no focus indicator (2.4.7). Adding `role="button"` and `tabindex="0"` *and* Enter/Space handlers reproduces by hand what `<button>` does natively, with more code and more ways to be wrong.

**`href="#"` with a click handler** is the second most common version of this defect: it announces as a link, navigates nowhere, and moves focus unexpectedly.

## Landmarks

```html
<header>…</header>
<nav aria-label="Primary">…</nav>
<main id="main">…</main>
<aside aria-label="Related">…</aside>
<footer>…</footer>
```

- **One `<main>` per page.** It is the skip-link target and how AT users jump to content.
- **Label repeated landmarks.** Two `<nav>` elements need distinguishing names (`aria-label="Primary"`, `aria-label="Footer"`), or they are indistinguishable in a landmark list.
- **`<section>` needs an accessible name to be a landmark.** An unnamed `<section>` is semantically no better than a `<div>`. Give it a heading referenced by `aria-labelledby`, or use a `<div>`.
- Landmarks satisfy 2.4.1 Bypass Blocks alongside a skip link — provide both.

## Headings

- **One `<h1>`**, describing the page.
- **No skipped levels** going down: `h2` then `h4` breaks the outline AT users navigate by.
- **Headings mark structure, not size.** Never choose a level for its appearance — style it. Never use a styled `<div>` where a heading belongs.
- **The outline must read as a coherent document** on its own. That is the fastest structural audit there is.

```js
() => [...document.querySelectorAll("h1,h2,h3,h4,h5,h6")]
  .map(h => `${h.tagName} ${h.textContent.trim().slice(0, 60)}`)
```

Read the output. If it does not describe the page, the structure is wrong — and that is a `scribeo-ux-engineering` finding as much as an accessibility one.

## Lists

Use `<ul>`, `<ol>`, `<dl>` for anything that is a list — navigation, card grids, feature sets. AT announces item counts and position, which is real orientation information a `<div>` stack loses.

Only `<li>` may be a direct child of `<ul>`/`<ol>`. A wrapper `<div>` between them breaks the list semantics.

## Forms

```html
<label for="email">Email address</label>
<input id="email" type="email" name="email" autocomplete="email" required>
```

- **Every control needs a programmatically associated label** — `<label for>` matching `id`, or a wrapping `<label>`.
- **`<fieldset>` + `<legend>`** for radio groups and checkbox groups. The legend is the group's name; without it each option is announced with no context.
- **Correct `type`** — `email`, `tel`, `url`, `number` — brings validation and the right mobile keyboard.
- **`autocomplete`** satisfies 1.3.5 and materially helps users with cognitive and motor differences.

Detail in `forms.md`.

## Tables

For tabular data only — never for layout.

```html
<table>
  <caption>Q3 enquiry volume by channel</caption>
  <thead><tr><th scope="col">Channel</th><th scope="col">Enquiries</th></tr></thead>
  <tbody><tr><th scope="row">Organic</th><td>412</td></tr></tbody>
</table>
```

`<caption>` names the table; `<th>` with `scope` lets AT announce which row and column a cell belongs to. Without `scope`, a data table is a grid of unlabelled numbers.

## Other elements worth using

- `<dialog>` — native modal semantics and focus handling. See `components.md`.
- `<details>` / `<summary>` — a disclosure with keyboard behaviour and state built in.
- `<time datetime="…">` — machine-readable dates.
- `<abbr title>` — expansions.
- `<fieldset disabled>` — disables a whole group.
- `lang` on `<html>`, and on any element that changes language (3.1.1, 3.1.2).

## The ARIA rule

> **Use ARIA to repair a real semantic gap, not to decorate ordinary HTML.**

Before adding any ARIA attribute, answer: *which native element would express this, and why can I not use it?* If there is no answer, remove the ARIA.

Detail on correct ARIA use in `names-roles-states.md`.
