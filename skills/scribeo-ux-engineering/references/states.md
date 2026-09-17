# States

Load when defining interaction, validation, loading, empty, or error behaviour.

**Rule: define every state before building the happy path.** An interface that only handles success is unfinished, and retrofitting states produces the gaps users actually hit.

## Interaction states

Every interactive element declares the states that apply to it:

| State | Purpose | Requirement |
| --- | --- | --- |
| Default | Communicates affordance | Must look actionable without hover |
| `:hover` | Confirms targeting | Pointer devices only — never the sole path to meaning |
| `:focus-visible` | Keyboard position | **Mandatory.** Clearly visible, ≥3:1 against adjacent colours |
| `:active` | Press confirmation | Immediate, no delay |
| `[aria-pressed]` / `[aria-selected]` | Persistent selection | Not colour alone |
| `:disabled` | Unavailable | Must explain why, or be avoided |
| Loading | Work in progress | Prevent double submission; hold dimensions |
| Success | Confirm completion | Persist long enough to read |
| Error | Explain and recover | Text, next to the cause |

```css
/* Wrong — removes the only keyboard affordance */
button:focus { outline: none; }

/* Right — never shows on mouse click, always on keyboard */
button:focus-visible {
  outline: 2px solid var(--focus);
  outline-offset: 2px;
}
```

If a custom focus style is used, it must be at least as visible as the default. Removing the outline without replacement makes the site unusable by keyboard — the single most damaging accessibility defect in practice.

**Hover rules:** never the only way to reach content · no layout shift on hover (causes flicker at boundaries) · always paired with `:focus-visible` · assume it does not exist on touch.

**Disabled buttons are usually the wrong answer.** A disabled submit tells the user nothing about what is missing. Prefer: keep it enabled, validate on submit, move focus to the first problem and explain it. Where disabled is genuinely correct (an unavailable date), state the reason in text — and note that `disabled` removes the element from tab order, so it cannot announce its own reason. Use `aria-disabled="true"` with a real explanation when the control must stay focusable.

## Loading

Match the treatment to the wait:

| Duration | Treatment |
| --- | --- |
| <100ms | Nothing — a flash of spinner is worse than no feedback |
| 100ms–1s | In-place indicator on the triggering control |
| 1s–3s | Skeleton matching the final layout |
| >3s | Progress plus a statement of what is happening |

**Reserve the space.** A skeleton must occupy the final dimensions, or content arrival shifts the layout — a UX failure and a CLS failure (measurement belongs to `scribeo-performance`).

**Button loading** must preserve width, or the layout jumps on every submit:
```html
<button type="submit" data-loading="true" aria-busy="true">
  <span class="btn__label">Send enquiry</span>
  <span class="btn__spinner" aria-hidden="true"></span>
</button>
```
```css
[data-loading="true"] .btn__label { visibility: hidden; }
[data-loading="true"] { pointer-events: none; }
```

Announce async changes to assistive tech with a polite live region. Respect `prefers-reduced-motion` — replace spinners with a static or fading indicator rather than continuous rotation.

## Empty

Empty states are design work, not an edge case. Three distinct kinds:

| Kind | Content |
| --- | --- |
| **Nothing yet** (new user, no data) | Explain what will appear here and the action that creates it |
| **No results** (filter/search) | State what was searched, offer a way back — clear filters, broaden, or suggest alternatives |
| **Nothing to show** (legitimately empty) | Plain statement. No apology, no illustration |

Never render a bare container, and never say only "No results found". A property search returning nothing should show the criteria, a clear-filters action, and nearby alternatives — that is a recoverable dead end instead of an exit.

Also define the **partially empty** case: one testimonial where the design assumes three, a listing with no photograph, a card with no excerpt. These are the states real content actually produces.

## Errors

**Field level** — beside the field, in text, describing the fix:

```html
<div class="field field--error">
  <label for="email">Email address</label>
  <input id="email" type="email" autocomplete="email"
         aria-invalid="true" aria-describedby="email-error">
  <p id="email-error" class="error" role="alert">
    Enter an email address in the format name@example.com
  </p>
</div>
```

- `aria-invalid="true"` and `aria-describedby` pointing at the message.
- Never colour alone — red border plus text, always.
- Validate on blur, not keystroke. Re-validate on submit.
- On failed submit: move focus to the first error, and summarise ("2 fields need attention") for screen readers.

**Form level** — a summary above the form, focusable, listing each problem as a link to its field.

**System level** — say what failed, whether the data survived, and what to do next.

```
Wrong:  "An error occurred."
Right:  "We couldn't send your enquiry — the connection dropped.
         Your message is saved. Try again, or call 01632 960123."
```

Never blame the user, never expose stack traces or status codes alone, and never lose their input on failure. A form that clears itself after a server error loses the conversion permanently.

**Success** — confirm specifically and durably: what happened, what happens next, when. "Enquiry sent — we'll reply within one working day." A toast that vanishes in two seconds is not confirmation for a slow reader. For significant actions, change the page state rather than showing a transient message.

## Motion and states

State changes may be animated, but meaning must never depend on animation.

- Every state must be understandable in a static screenshot.
- Full functionality under `prefers-reduced-motion: reduce`.
- Transitions on state change should be short enough not to delay feedback.

Choreography, easing, and timing craft belong to `scribeo-motion`. This skill's requirement is only that state is **communicated** — clearly, statically, and accessibly.
