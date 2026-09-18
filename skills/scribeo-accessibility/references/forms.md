# Forms

Load when auditing or fixing form accessibility. Forms are where accessibility failures cost conversions directly.

## Labels

**Every control needs a programmatically associated label.** Not a placeholder, not adjacent text.

```html
<label for="email">Email address</label>
<input id="email" type="email" name="email" autocomplete="email" required
       aria-describedby="email-hint">
<p id="email-hint">We reply within one working day.</p>
```

- `<label for>` must match the input's `id`. A wrapping `<label>` also works.
- **Placeholder is not a label** (3.3.2). It disappears on input, usually fails contrast, is hidden by autofill, and is not reliably announced. Use it for format examples only.
- **Visible labels serve everyone.** `aria-label` on a form field is a last resort — and it breaks 2.5.3 if visible text differs.
- **Never rely on proximity.** Text sitting next to an input is not associated with it.

```js
() => [...document.querySelectorAll("input:not([type=hidden]), select, textarea")]
  .filter(el => !el.labels?.length && !el.getAttribute("aria-label") && !el.getAttribute("aria-labelledby"))
  .map(el => `${el.tagName}[type=${el.type || "-"}][name=${el.name || "-"}]`)
```

Anything returned is unlabelled — a 3.3.2 and 4.1.2 failure.

## Required fields

- Use the native `required` attribute — it exposes the state programmatically.
- **Mark it in text, not by asterisk alone.** An asterisk with no legend, or conveyed only by colour, fails 1.4.1. `<span aria-hidden="true">*</span>` plus the word "required" in the label works.
- Where most fields are required, mark the **optional** ones instead — less noise, clearer.
- `aria-required="true"` only for custom controls that cannot use `required`.

## Instructions

**3.3.2 Labels or Instructions.** Requirements must be stated **before** the user encounters the error, not only after.

- Format and constraints belong in help text tied via `aria-describedby`.
- Never hide a requirement until submission fails.
- Keep instructions next to the field they govern.

## Input purpose and types

- **`autocomplete`** on every relevant field — `name`, `email`, `tel`, `street-address`, `postal-code`. Satisfies 1.3.5 Identify Input Purpose, and materially helps users with motor and cognitive differences. It is also a conversion lever.
- **Correct `type`** — `email`, `tel`, `url`, `number` — brings native validation and the right mobile keyboard.
- **`inputmode`** refines the mobile keyboard where `type` is too coarse.
- **Never block paste.** It obstructs password managers and anyone who cannot reliably type long strings. Related: **3.3.8 Accessible Authentication (Minimum) (AA)** — do not require a cognitive function test (like transcribing a code from memory) with no alternative.

## Grouping

```html
<fieldset>
  <legend>How should we contact you?</legend>
  <label><input type="radio" name="contact" value="email"> Email</label>
  <label><input type="radio" name="contact" value="phone"> Phone</label>
</fieldset>
```

Without `<legend>`, each option is announced with no group context — "Email, radio button, 1 of 2" tells the user nothing about the question.

## Errors

Three requirements: **identify** (3.3.1, A), **suggest a fix** (3.3.3, AA), and make it **perceivable in every mode**.

```html
<div class="field field--error">
  <label for="email">Email address</label>
  <input id="email" type="email" autocomplete="email"
         aria-invalid="true" aria-describedby="email-err">
  <p id="email-err" class="error">
    Enter an email address in the format name@example.com
  </p>
</div>
```

- **`aria-invalid="true"`** exposes the error state.
- **`aria-describedby`** associates the message, so it is announced with the field.
- **Never colour alone** (1.4.1) — a red border needs accompanying text.
- **Describe the fix**, not just the failure. "Invalid input" fails 3.3.3.
- **Validate on blur, not per keystroke.** Per-keystroke validation with a live region produces a stream of announcements while the user is still typing.

## Error summaries and focus

On failed submission:

1. Render a summary above the form listing every problem, each a link to its field.
2. **Move focus to the summary** (or to the first invalid field).
3. Give the summary an accessible name and make it focusable with `tabindex="-1"`.

```html
<div role="alert" tabindex="-1" id="err-summary">
  <h2>2 fields need attention</h2>
  <ul>
    <li><a href="#email">Email address — enter a valid address</a></li>
    <li><a href="#message">Message — this field is required</a></li>
  </ul>
</div>
```

**Leaving focus on the submit button after a failed submit is the most common form accessibility defect.** A keyboard or screen-reader user gets no indication anything happened.

`role="alert"` announces the summary on insertion. Do not also wrap it in a separate live region — one announcement mechanism, not two.

## Status and success

- **Success must be announced**, not only shown. A visual tick communicates nothing to a screen-reader user. Use a polite live region, or move focus to a confirmation heading. See `dynamic-content.md`.
- **Loading state** on submit: `aria-busy` on the region, and disable double submission without removing the button from the tab order.
- **Never clear the form on server error.** Losing typed input is a severe barrier for anyone who found entering it difficult.

## Redundant entry

**3.3.7 Redundant Entry (A, new in 2.2):** do not ask for the same information twice in one process unless essential. Auto-populate or offer selection instead — for example, a "billing address same as delivery" option rather than re-typing.

## Audit checklist

- [ ] Every control has an associated visible label
- [ ] No placeholder-as-label
- [ ] Required fields marked in text, not colour or asterisk alone
- [ ] Instructions present before errors occur
- [ ] `autocomplete` and correct `type` on all relevant fields
- [ ] Radio and checkbox groups in `<fieldset>` with `<legend>`
- [ ] Errors: `aria-invalid`, `aria-describedby`, text, and a suggested fix
- [ ] Error summary present; focus moves to it on failed submit
- [ ] Success announced, not just displayed
- [ ] Paste not blocked; no cognitive-function test without alternative
- [ ] Input preserved on error
- [ ] Whole form completable by keyboard alone
