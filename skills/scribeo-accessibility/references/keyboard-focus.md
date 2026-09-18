# Keyboard & Focus

Load for traversal, focus order, traps, focus visibility, and focus restoration.

**Keyboard traversal finds more real barriers than any automated tool.** It is the single highest-value manual audit. Do it first.

## The keys

| Key | Expected |
| --- | --- |
| `Tab` | Next focusable element, in document order |
| `Shift+Tab` | Previous |
| `Enter` | Activate a link or button; submit a form |
| `Space` | Activate a button; toggle a checkbox; scroll the page when nothing is focused |
| `Escape` | Close a dialog, menu, or popover; cancel |
| Arrows | Move within a composite widget — tabs, menus, radio groups, listboxes |

**Enter and Space differ, and native elements get it right for free.** `<button>` responds to both; `<a href>` responds to Enter only. Any custom control claiming `role="button"` must handle both — and must not swallow Space when the user meant to scroll.

## Traversal procedure

1. Click the very top of the page (or reload), then press `Tab` only.
2. At every stop record: what is focused, whether the indicator is visible, and whether the order makes sense.
3. Continue to the end of the page. Do not skip the footer.
4. `Shift+Tab` back up — order must be the exact reverse.
5. Operate every control with the keyboard alone. Open every menu, submit every form, close every dialog.
6. Verify you can always leave. Anywhere you cannot is a trap.

```
browser_press_key { key: "Tab" }
browser_snapshot                     → what is focused, its role and name
browser_take_screenshot              → is the focus indicator visible
```

```js
() => { const a = document.activeElement;
  return { tag: a.tagName, role: a.getAttribute("role") || "",
           name: (a.getAttribute("aria-label") || a.textContent || "").trim().slice(0, 50),
           outline: getComputedStyle(a).outlineStyle, outlineWidth: getComputedStyle(a).outlineWidth }; }
```

**Check for unreachable interactive elements:**

```js
() => [...document.querySelectorAll("[onclick], [role=button], [role=link], [role=tab], [role=menuitem]")]
  .filter(el => !el.matches("a[href], button, input, select, textarea") && el.tabIndex < 0)
  .map(el => `${el.tagName}.${el.className} role=${el.getAttribute("role") || "none"}`)
```

Anything returned is operable by mouse and not by keyboard — a 2.1.1 failure.

## Focus order

**2.4.3 Focus Order** requires an order that preserves meaning and operability. In practice: make DOM order the correct reading order, and let focus follow it.

- **Never use CSS to reorder content** in a way that desynchronises visual and focus order. `flex-direction: row-reverse` and `order` change what the eye sees, not what `Tab` does. This is a frequent defect in responsive layouts — and a `scribeo-ux-engineering` fix.
- **Never `tabindex` above `0`.** It jumps ahead of everything in natural order and becomes unmaintainable. `tabindex="0"` (in natural order) and `tabindex="-1"` (focusable only by script) are the only values to use.
- **Newly revealed content** should sit in the DOM where it belongs, so focus reaches it naturally.

## Keyboard traps

**2.1.2 No Keyboard Trap (Level A).** A user must always be able to Tab away — except in a modal dialog, where trapping is correct and required.

Common accidental traps: a custom widget capturing Tab without providing an exit · an embedded iframe or third-party player that swallows focus · a focus-restoring loop that fights the user · an "infinite scroll" footer that can never be reached.

**Focus trapping is only legitimate inside a modal dialog.** Anywhere else it is a Critical defect.

## Focus visibility

**2.4.7 Focus Visible (AA).** The focused element must be visibly indicated.

```css
/* Wrong: removes the only keyboard affordance */
:focus { outline: none; }

/* Right: never shows on mouse click, always on keyboard */
:focus-visible {
  outline: 3px solid var(--focus);
  outline-offset: 2px;
}
```

- **`outline: none` without an equal-or-better replacement is a conformance failure**, not a style choice. It is the most damaging single line in front-end CSS.
- `:focus-visible` is the right selector — keyboard users get the ring, mouse users do not.
- The indicator needs **3:1 contrast against adjacent colours** (1.4.11 covers focus indicators). See `contrast.md`.
- `outline-offset` keeps the ring clear of the element's own border.
- **Never rely on colour change alone** — a border-colour shift can be invisible to some users. Use an outline, ring, or shape change.

**Check for clipped rings.** `overflow: hidden` on an ancestor cuts an outline off. This is invisible in code review and obvious the moment you Tab and screenshot.

**2.4.11 Focus Not Obscured (Minimum) (AA, new in 2.2):** the focused element must not be entirely hidden by author-created content — most often a sticky header or a cookie bar. Tab through a page with a sticky header and watch for focus disappearing beneath it. `scroll-padding-top` on the root is the usual fix.

## Skip links

**2.4.1 Bypass Blocks.** The first focusable element on every page should skip to the main content.

```html
<a class="skip-link" href="#main">Skip to main content</a>
…
<main id="main" tabindex="-1">…</main>
```

```css
.skip-link { position: absolute; left: -9999px; }
.skip-link:focus { left: 1rem; top: 1rem; /* visible on focus */ }
```

It must become **visible when focused** — a permanently hidden skip link helps nobody, and a `display: none` one is not focusable at all. `tabindex="-1"` on the target makes focus land reliably.

## Focus management

Moving focus is powerful and easily misused.

**Move focus when** opening a dialog (to the dialog) · closing one (back to the trigger) · submitting a form with errors (to the first error or the summary) · revealing content the user asked for.

**Do not move focus when** the page loads (leave it at the start) · content loads asynchronously without being requested · a user is typing · an animation finishes.

**Never steal focus.** Focus moving without a user action is disorienting and can lose typed input.

**Restoration is not optional.** Closing a dialog and dropping focus to `<body>` strands the user at the top of the document, having lost their place.

```js
const opener = document.activeElement;   // before opening
// … open, trap, close …
opener?.focus();                          // after closing
```

**Route changes in client-side apps** need explicit handling: focus should move to the new page's heading or main region, and the change should be announced. See `dynamic-content.md`.
