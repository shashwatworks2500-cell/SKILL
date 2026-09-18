# Dynamic Content & Announcements

Load for live regions, status messages, async updates, and client-side routing.

**4.1.3 Status Messages (AA)** is the criterion here: when content changes without a change of context, the user must be able to learn about it without moving focus.

## Live regions

```html
<!-- Polite: waits for a pause. Default choice. -->
<div aria-live="polite" aria-atomic="true" id="status"></div>

<!-- Assertive: interrupts immediately. Errors only. -->
<div aria-live="assertive" id="errors"></div>

<!-- Shorthands -->
<div role="status">…</div>   <!-- ≈ aria-live="polite" -->
<div role="alert">…</div>    <!-- ≈ aria-live="assertive" -->
```

**The container must exist in the DOM before the content is inserted.** A live region created and populated in the same tick is frequently not announced — the most common reason "the live region doesn't work".

```js
/* Wrong: region and content appear together; often silent */
document.body.insertAdjacentHTML("beforeend", '<div role="status">Saved</div>');

/* Right: region is already present; only text changes */
document.getElementById("status").textContent = "Saved";
```

| Attribute | Effect |
| --- | --- |
| `aria-live="polite"` | Announced at the next natural pause. Use for almost everything |
| `aria-live="assertive"` | Interrupts. Reserve for errors and time-critical information |
| `aria-atomic="true"` | Announce the whole region, not just the changed node |
| `aria-relevant` | Which mutation types announce. Rarely needed; defaults are sensible |
| `aria-busy="true"` | Suppress announcements while a region is mid-update |

## When a live region is the wrong tool

Over-using `aria-live` turns the page into a stream of interruptions — a real barrier, not an improvement.

**Do not use a live region for:**
- Anything the user will hear anyway because focus moved there.
- Per-keystroke validation feedback.
- Content that changes constantly — a scroll progress indicator, a live counter, an animation state.
- Purely decorative changes.
- Something better handled by moving focus (a dialog opening).

**Announce these:** form submission results · async load completion where the user is waiting · search result counts · cart or selection changes · errors arising without user action · a background process finishing.

**The test:** would a sighted user notice this change and need to act on it? If yes, announce it. If they would ignore it, so should the announcement.

## Toasts

Awkward by nature: transient, often off to one side, sometimes carrying an action.

- `role="status"` for informational, `role="alert"` for errors.
- **Do not auto-dismiss anything containing an action or important information.** A toast that vanishes in 3 seconds is unusable for anyone reading slowly, and if it is auto-moving content lasting over 5 seconds, 2.2.2 Pause, Stop, Hide applies.
- Never put the **only** copy of critical information in a toast.
- If it has a dismiss button, that button must be keyboard reachable — which conflicts with auto-dismissal. Usually the sign a toast is the wrong pattern.

## Async loading

- **Announce meaningfully:** "Loading results" then "12 results found" — not a bare spinner, which is invisible to AT.
- **Reserve space** so arrival does not shift layout (a `scribeo-ux-engineering` structural concern, with a CLS dimension for `scribeo-performance`).
- **`aria-busy="true"`** on the region while it updates, cleared when done.
- **Do not move focus on arrival** unless the user explicitly requested the content.

## Client-side routing

A single-page navigation changes everything visually and **nothing** for assistive technology unless handled — no page-load event, no new title announcement, focus left wherever it was.

Three things are required on every route change:

1. **Update the document title.** Many AT users rely on it for orientation.
2. **Move focus deliberately** — to the new main region or `<h1>`, with `tabindex="-1"` so it accepts focus.
3. **Announce the change** via a polite live region, for users who may miss the focus move.

```js
function onRouteChange(title) {
  document.title = `${title} — Scribeo`;
  const h = document.querySelector("main h1");
  if (h) { h.setAttribute("tabindex", "-1"); h.focus(); }
  document.getElementById("route-status").textContent = `${title} page loaded`;
}
```

Also reset scroll position, and refresh anything that measured the old layout.

**This is one of the most commonly missed accessibility requirements in modern framework sites** — nothing visibly breaks, so nobody notices.

## Content on hover or focus

**1.4.13 Content on Hover or Focus (AA)** — tooltips, popovers, hover cards. Additional content must be:

- **Dismissible** without moving the pointer or focus — `Escape` closes it.
- **Hoverable** — the pointer can move onto the content itself without it vanishing.
- **Persistent** until dismissed, focus moves away, or it is no longer valid.

A tooltip that disappears when you try to read it, or that cannot be dismissed, fails this. And hover-only content fails 2.1.1 for keyboard users — pair every hover trigger with focus.

## Audit checklist

- [ ] Live region containers exist in the DOM before use
- [ ] `polite` by default; `assertive` reserved for errors
- [ ] No live region on constantly-changing content
- [ ] Async completion announced, not just visually indicated
- [ ] Form submission results announced
- [ ] Route changes: title updated, focus moved, change announced
- [ ] Toasts do not auto-dismiss when they carry actions or critical information
- [ ] Hover/focus content is dismissible, hoverable and persistent
- [ ] Nothing announced so often it becomes noise
