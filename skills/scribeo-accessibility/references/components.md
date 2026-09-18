# Component Patterns

Load when building or auditing dialogs, menus, disclosures, tabs, accordions, or tooltips.

**Use an established pattern.** The WAI-ARIA Authoring Practices define expected keyboard behaviour for each of these, and users have learned those conventions. **Do not invent custom keyboard behaviour casually** — a widget that announces as a tablist and then does not respond to arrow keys is worse than a plain list of links.

**Prefer native elements where they exist.** `<dialog>` and `<details>`/`<summary>` bring semantics, keyboard handling, and state for free.

## Modal dialog

The only place focus trapping is correct.

```html
<dialog id="enquiry" aria-labelledby="enquiry-title">
  <h2 id="enquiry-title">Send an enquiry</h2>
  …
  <button type="button" data-close>Close</button>
</dialog>
```

Requirements:

- **Accessible name** from `aria-labelledby` pointing at the heading.
- **Focus moves in on open** — to the dialog, its heading, or the first control. Not left outside.
- **Focus is trapped** while open; Tab cycles within.
- **`Escape` closes it.**
- **Focus returns to the trigger** on close. Dropping focus to `<body>` strands the user.
- **Background content is inert** — `<dialog>` with `showModal()` handles this natively; otherwise `inert` on the rest of the page.
- **Background scroll locked**, without losing scroll position.

`showModal()` gives trapping, `Escape`, and inertness natively. Hand-rolled modals reimplement all of it, usually incompletely — most commonly missing focus restoration.

**Never trap focus in a non-modal** — that is a 2.1.2 keyboard trap and Critical.

## Disclosure (show/hide)

The simplest pattern, and often the right answer where a menu is proposed.

```html
<button type="button" aria-expanded="false" aria-controls="panel-1">
  Opening hours
</button>
<div id="panel-1" hidden>…</div>
```

- `aria-expanded` on the **trigger**, and it must track reality.
- `aria-controls` points at the panel.
- `hidden` (or `display: none`) when collapsed, so content inside is not focusable.
- Enter and Space both activate — free with `<button>`.
- **Focus stays on the trigger.** Do not move it into the panel; the user will Tab there if they want.

`<details>`/`<summary>` gives all of this natively, including state.

## Accordion

A set of disclosures. Same rules per item, plus:

- Each trigger is a `<button>` inside a heading at the correct level: `<h3><button …>`. The heading carries structure; the button carries interaction.
- Do not hijack arrow keys — Tab moves between accordion headers. Arrow-key navigation is for composite widgets like tablists, not accordions.
- If only one panel may be open, announce which. Do not silently collapse another without indication.

## Navigation menu

**Most site navigation should be links in a list, not a menu widget.** `role="menu"` is for application menus (like a desktop app's menu bar) and brings an arrow-key model users will expect. Applying it to site navigation makes links behave unexpectedly.

```html
<nav aria-label="Primary">
  <ul>
    <li><a href="/work" aria-current="page">Work</a></li>
    <li>
      <button type="button" aria-expanded="false" aria-controls="studio-sub">Studio</button>
      <ul id="studio-sub" hidden>…</ul>
    </li>
  </ul>
</nav>
```

A dropdown inside navigation is a **disclosure** containing links — not a menu. `aria-current="page"` marks the active item, and that state must also be visible without relying on colour alone.

**Mobile navigation** is a disclosure or a modal dialog. If it covers the screen and traps focus, treat it as a dialog and meet every dialog requirement. Keep the primary conversion route reachable outside it.

## Tabs

A genuine composite widget with a required keyboard model.

```html
<div role="tablist" aria-label="Project details">
  <button role="tab" id="t1" aria-selected="true"  aria-controls="p1">Overview</button>
  <button role="tab" id="t2" aria-selected="false" aria-controls="p2" tabindex="-1">Specification</button>
</div>
<div role="tabpanel" id="p1" aria-labelledby="t1" tabindex="0">…</div>
<div role="tabpanel" id="p2" aria-labelledby="t2" tabindex="0" hidden>…</div>
```

- **One tab stop for the whole tablist** — the selected tab has `tabindex="0"`, the rest `tabindex="-1"`. Tab enters and leaves the group; it does not step through every tab.
- **Arrow keys move between tabs**; `Home`/`End` jump to first/last.
- `aria-selected` on the active tab; inactive panels `hidden`.
- Each panel is labelled by its tab.

**If you are not going to implement the keyboard model, do not use tab roles.** Styled links or a disclosure set are honest alternatives.

## Tooltips

- Trigger must be focusable, and the tooltip must appear on **focus as well as hover** — hover-only fails 2.1.1.
- Associate via `aria-describedby` so it is announced with the control.
- Must satisfy 1.4.13: dismissible with `Escape`, hoverable, persistent.
- **Never put essential information or interactive content in a tooltip.** If it contains a link or button, it needs to be a popover with proper focus management.
- An icon-only control needs a real accessible name — a tooltip is not a substitute.

## Carousels

Frequently inaccessible, and rarely worth the cost.

- Real `<button>` controls for previous/next, with accessible names.
- Auto-advance is auto-moving content: **2.2.2 Pause, Stop, Hide (A)** requires a pause mechanism when it runs over 5 seconds. Better: do not auto-advance.
- Pause on hover **and on focus**.
- Off-screen slides must not be focusable — `hidden` or `inert`.
- Announce position ("Slide 2 of 5") via a status region, not by moving focus.

## Custom controls generally

Before building one, answer:

1. Which native element expresses this, and why can I not use it?
2. What is the established pattern's keyboard model?
3. Will I implement **all** of it — roles, states, keys, focus?

If the answer to 3 is no, use a simpler native pattern. **A well-built disclosure beats a broken combobox every time.**

## Audit checklist

- [ ] Native element used where one exists
- [ ] Dialogs: name, focus in, trapped, `Escape`, focus restored, background inert
- [ ] Focus trapping only inside modals
- [ ] Disclosures: `aria-expanded` on the trigger, tracking reality
- [ ] Accordion triggers inside correct-level headings
- [ ] Site navigation uses links, not `role="menu"`
- [ ] Tabs implement the full arrow-key model and single tab stop
- [ ] Tooltips on focus as well as hover; dismissible, hoverable, persistent
- [ ] Auto-advancing content has a pause mechanism
- [ ] Hidden content is not focusable
