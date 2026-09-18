# Names, Roles & States

Load when auditing accessible names, or deciding whether ARIA is warranted.

**4.1.2 Name, Role, Value (Level A)** is the criterion custom widgets most often fail. Every interactive control must expose: a **name** (what it is), a **role** (what kind of thing), and its **state/value** where applicable.

## Accessible names

Sources, in the order the browser generally resolves them — later entries override earlier ones:

1. `aria-labelledby` (strongest)
2. `aria-label`
3. Native label: `<label for>`, `<caption>`, `<legend>`, `alt`
4. Element content (a button's text)
5. `title` (weakest, and unreliable — never the primary source)

**Prefer a visible label.** A visible text label serves everyone, satisfies 2.5.3 Label in Name, and cannot drift out of sync with the visual.

```html
<!-- Best: visible text is the name -->
<button type="submit">Send enquiry</button>

<!-- Icon-only: aria-label is legitimate here -->
<button type="button" aria-label="Close dialog">
  <svg aria-hidden="true" focusable="false">…</svg>
</button>

<!-- Better still if space allows: visible text, visually hidden if needed -->
<button type="button">
  <svg aria-hidden="true" focusable="false">…</svg>
  <span class="visually-hidden">Close dialog</span>
</button>
```

**`aria-label` is not the default answer.** It is correct for icon-only controls and for distinguishing repeated controls. It is wrong when:

- A visible label exists — it then *overrides* the visible text, and a voice-control user saying what they see will fail (2.5.3).
- It is used to "improve" already-adequate text.
- It is applied to a non-interactive element, where it may simply be ignored.
- It duplicates nearby text, producing a doubled announcement.

**2.5.3 Label in Name** requires the accessible name to contain the visible text. A button reading "Submit" with `aria-label="Send your enquiry to our team"` breaches it.

**Repeated links** need distinguishing names. Five "Read more" links are useless out of context (2.4.4). Either extend the visible text, or use `aria-labelledby` to combine the link with its heading:

```html
<h3 id="post-12">Our approach to editorial layout</h3>
<a href="/blog/12" aria-labelledby="post-12 rm-12">
  <span id="rm-12">Read more</span>
</a>
```

`aria-labelledby` accepts multiple IDs and concatenates them in the order listed.

## Auditing names

```js
() => [...document.querySelectorAll("button, a[href], input, select, textarea, [role=button], [role=link]")]
  .map(el => {
    const name = el.getAttribute("aria-label")
      || (el.getAttribute("aria-labelledby") || "").split(" ").map(id => document.getElementById(id)?.textContent.trim()).filter(Boolean).join(" ")
      || (el.labels?.[0]?.textContent.trim())
      || el.textContent.trim()
      || el.getAttribute("title") || "";
    return { tag: el.tagName, role: el.getAttribute("role") || "", name: name.slice(0, 50), empty: !name };
  })
  .filter(r => r.empty)
```

Anything returned has **no accessible name** — a 4.1.2 failure. This is the highest-yield single probe in a name audit.

Then read `browser_snapshot`: it reports the computed accessibility tree, which is the authoritative view of what AT receives. The DOM and the accessibility tree are not the same thing.

## Descriptions

`aria-describedby` supplies supplementary information — a hint, a format requirement, an error message. It is announced after the name, and it does not replace it.

```html
<label for="pw">Password</label>
<input id="pw" type="password" aria-describedby="pw-hint pw-err" aria-invalid="true">
<p id="pw-hint">At least 12 characters.</p>
<p id="pw-err">Password is too short.</p>
```

Multiple IDs are concatenated. Use this for errors and hints rather than folding them into the label.

## Roles

**Do not add a role that duplicates the element.** `<button role="button">` and `<nav role="navigation">` are noise. Worse, a *wrong* role silently replaces real semantics: `<a href="/x" role="button">` announces as a button but behaves as a link — Space will not activate it, and the user's model is broken.

Add a role only when building something with no native equivalent (tablist, tree, combobox) — and then implement the **whole** pattern, including its keyboard model. See `components.md`.

## States

State must be exposed programmatically, not only visually.

| State | Attribute | Applies to |
| --- | --- | --- |
| Expanded / collapsed | `aria-expanded="true\|false"` | Disclosure triggers, menu buttons, accordions |
| Pressed (toggle) | `aria-pressed="true\|false"` | Toggle buttons |
| Selected | `aria-selected="true\|false"` | Tabs, options |
| Checked | `checked`, or `aria-checked` for custom | Checkboxes, radios, switches |
| Current | `aria-current="page\|step\|true"` | Active nav item, current step |
| Disabled | `disabled`, or `aria-disabled="true"` | Controls |
| Invalid | `aria-invalid="true"` | Form fields with errors |
| Busy | `aria-busy="true"` | Regions updating |
| Controls | `aria-controls="id"` | Trigger → controlled region |

**The attribute must track the real state.** A hardcoded `aria-expanded="false"` that never updates is worse than absent — it actively lies. Audit dynamically, after interacting, not from source.

**`disabled` vs `aria-disabled`:** `disabled` removes the element from tab order, so it cannot explain itself. `aria-disabled="true"` keeps it focusable and announceable while your code ignores activation — usually the better choice when the user needs to know *why* it is unavailable.

## Hiding things correctly

| Goal | Technique |
| --- | --- |
| Hidden from everyone | `display: none`, `hidden`, or `visibility: hidden` |
| Hidden visually, available to AT | A `.visually-hidden` clip pattern — **not** `display: none` |
| Visible, hidden from AT | `aria-hidden="true"` — decorative only |

**Never `aria-hidden="true"` on anything focusable or containing focusable content.** It creates a control that is reachable by keyboard but invisible to AT — one of the most confusing states possible.

Decorative inline SVG needs both `aria-hidden="true"` and `focusable="false"` (the latter for legacy IE-era focus behaviour that still appears in some engines).

## Common ARIA mistakes

| Mistake | Why it fails |
| --- | --- |
| `role="button"` on a `<div>` instead of `<button>` | Reimplements natively-free behaviour, usually incompletely |
| `aria-label` on an element with visible text | Overrides it; breaks voice control (2.5.3) |
| Redundant roles (`<nav role="navigation">`) | Noise |
| Wrong role (`<a role="button">`) | Announced behaviour ≠ actual behaviour |
| `aria-expanded` never updated | Lies about state |
| `aria-hidden` on focusable content | Reachable but unannounced |
| `role="presentation"` on something meaningful | Strips needed semantics |
| `aria-live` on everything | Constant noise; see `dynamic-content.md` |
| `tabindex` above 0 | Breaks natural order; see `keyboard-focus.md` |
| A role added without its keyboard model | Announces as a widget that cannot be operated |
