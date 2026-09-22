---
name: scribeo-ux-engineering
description: Use when structuring, restructuring, or reviewing the UX and front-end architecture of a website or web interface — information hierarchy, page and section structure, navigation, buttons and CTAs, forms, cards, layout structure, responsive behaviour across mobile/tablet/desktop, type scale and measure, spacing rhythm, content density, and interaction/loading/empty/error states. Covers information architecture, usability, semantic HTML structure, conversion-oriented structure, touch and keyboard interaction, and accessible interaction design. Triggers on requests like "improve the layout", "make it responsive", "fix the mobile layout", "restructure this page", "improve the CTA", "build a contact form", "handle the empty and error states", "is this usable on mobile", "the hierarchy is unclear", or any work turning content and business goals into production interface structure. Visual design direction — aesthetic exploration, palette, typeface choice, visual composition and distinctive styling — belongs to frontend-design; this skill consumes that direction and engineers the structure, behaviour and states beneath it. Also does not own animation implementation (scribeo-motion), visual regression review (scribeo-visual-qa), performance profiling (scribeo-performance), dedicated accessibility auditing (scribeo-accessibility), SEO (scribeo-seo), or test authoring (scribeo-testing).
---

# Scribeo UX Engineering

Scribeo Studio's senior front-end UX engineering standard. Governs how a brief becomes a premium, modern, conversion-aware interface — the structure, hierarchy, responsive behaviour, and component semantics beneath the visual design.

Apply this whenever you are deciding *what goes where, at what size, in what order, and how it behaves*.

## The law

**Every design decision must have a reason you can state in one sentence.**

Before any visual element ships, it must improve at least one of: **communication, usability, hierarchy, brand expression, conversion, or emotional impact.** An element that improves none of these is deleted, not refined.

When you cannot articulate the reason, that is the signal — not a prompt to add more polish.

### Default-deny list

These are **off by default**. Each requires a stated, brand-specific reason to appear:

gradients · glassmorphism · blanket rounded cards · stacked shadows · oversized type without hierarchy · decorative animation · floating shapes and blobs · dashboard chrome on a marketing site · pill tags used as decoration · animated counters · asymmetry for its own sake · full-width sections with no compositional logic

These are **never** permitted, under any brief:

**fabricated testimonials, statistics, client logos, trust badges, review counts, user counts, urgency ("3 spots left"), or scarcity.** If the client has not supplied it, build the slot and leave it empty or omit the section. Never invent social proof. This is not a style rule — inventing proof is a lie in the client's voice, and it is the fastest way to destroy the trust the interface exists to build.

## Boundaries

**This skill owns:** information architecture · UX architecture · layout structure · responsive UX strategy · type scale, measure and information hierarchy · spacing and rhythm · navigation structure · component behaviour and state design · forms UX · usability · content density · touch and keyboard interaction · semantic HTML structure · accessible interaction design · conversion structure.

**This skill does not own:**

| Concern | Belongs to |
| --- | --- |
| Visual design direction, aesthetic exploration, palette, typeface choice, visual composition, distinctive styling | `frontend-design` (Anthropic) |
| Animation implementation, timing, easing, choreography | `scribeo-motion` |
| Visual regression, cross-browser fidelity review | `scribeo-visual-qa` |
| Core Web Vitals, bundle size, render profiling | `scribeo-performance` |
| WCAG audit, remediation, assistive-tech verification | `scribeo-accessibility` |
| Metadata, structured data, crawlability | `scribeo-seo` |
| Unit, integration, e2e test authoring | `scribeo-testing` |

**The frontend-design boundary, precisely:** `frontend-design` decides *how it looks* — aesthetic direction, palette, typeface selection, visual composition, the distinctive point of view, and the polish of the rendered surface. This skill decides *how it works* — what goes where and in what order, how it behaves, how it responds, what every state does. On a new build, take the visual direction from `frontend-design` and engineer beneath it; do not re-derive the aesthetic here, and do not defer structure, states, or responsive behaviour to it. When a request is purely aesthetic ("make this look premium", "this feels templated", "choose a typeface"), that is `frontend-design`'s work, not this skill's.

**The accessibility boundary, precisely:** this skill *designs accessible interaction structures* — correct semantics, logical order, visible focus, adequate targets, labelled controls, motion-independent meaning. `scribeo-accessibility` *audits and remediates* against WCAG. Build it right here; prove it there. Do not run a conformance audit from this skill, and do not defer basic semantics to a later audit.

**If `frontend-design` is unavailable.** It is an Anthropic-provided skill, not part of the `scribeo-skills` marketplace, so it may not be installed. Never invent the aesthetic here to unblock yourself, and never stall work the aesthetic does not gate. Say plainly that the visual direction is unset, ask for it, and proceed with everything this skill owns that the direction does not affect: semantic structure, information hierarchy, responsive strategy, form behaviour, and every interaction, loading, empty and error state. Hand back the palette, typeface and composition questions when the direction arrives.

## Workflow

Never start with styling. Styling a structure you have not established produces decoration, then rework.

**Phase 1 — Understand (before any markup)**

1. **Project** — what is this, what does it replace, what does success look like?
2. **Users** — who arrives, from where, on what device, with what intent and what anxiety?
3. **Business goal** — the one outcome the client is paying for.
4. **Primary action** — the single thing a user must do. Name it. If you cannot, ask.
5. **Constraints** — brand assets, tone, references, existing design system, content inventory, tech stack.

If the brief does not answer 3 and 4, ask before building. Everything downstream inherits these two answers, and a wrong guess invalidates the whole page.

**Phase 2 — Structure (before any CSS)**

6. **Information hierarchy** — order the content by what the user needs, not by what the client wants to say first. Write the page as a bare heading outline and confirm it reads correctly with no styling.
7. **Responsive strategy** — decide what mobile shows, hides, reorders, and collapses. Decide this now, not after the desktop layout exists.

**Phase 3 — Build**

8. **Component behaviour** — define every state before writing the happy path.
9. **Implement** — semantic HTML first, then layout, then type, then surface.

**Phase 4 — Verify**

10. **Verify actual behaviour** — real viewports, keyboard only, long and empty content, slow network. Not a screenshot at one width.
11. **Refine on evidence** — change what you observed failing, not what you imagine could be nicer.

## Consuming the design language

**Deriving the visual direction is `frontend-design`'s job, not this skill's.** Palette, typeface selection, aesthetic point of view, and visual composition come from there. Scribeo's standard — premium, modern, editorial, intentional, distinctive — is the quality bar that direction must clear; it is never a template, and a law firm and a surf brand both clear it without resembling each other.

What this skill needs from the direction before engineering anything:

- **Density** — how much can appear at once, per breakpoint
- **Measure and type scale** — the structural consequences of the chosen typefaces
- **Focal intent** — which element is the focal point, so hierarchy can be built to serve it
- **Motion budget** — how much state change may be animated (craft belongs to `scribeo-motion`)
- **Content shape** — how many items actually exist, and how long they run

If the direction has not been established, ask for it or hand the aesthetic question to `frontend-design` — do not invent a palette and typeface here to unblock yourself, and do not stall structural work that the direction does not affect (semantics, states, responsive strategy, form behaviour) while waiting.

See `references/design-language.md` for the UX consequences of a direction and the structural causes of a generic result.

## Non-negotiable engineering defaults

These hold on every Scribeo build, regardless of design direction.

**Structure**
- Semantic HTML first: `<header> <nav> <main> <section> <article> <footer>`, one `<h1>`, no skipped heading levels. `<button>` for actions, `<a href>` for navigation — never a `<div>` with a click handler.
- Heading outline must make sense read alone, with CSS disabled.
- Landmarks and labels over ARIA. Add ARIA only when no native element expresses the semantics.

**Layout**
- No horizontal overflow at any width from 320px up. Verify at 320px, not just at a breakpoint.
- Constrain measure: body copy 60–75 characters. Full-width text in a 1440px container is a defect, not a style.
- Spacing comes from one scale. Arbitrary one-off values are how a design loses coherence.

**Type**
- Fluid type via `clamp()` with a `rem` floor and ceiling. Never size text in `vw` alone — it defeats user zoom (WCAG 1.4.4).
- Minimum 16px effective body size on mobile. Anything smaller triggers input zoom on iOS and fails real readers.

**Interaction**
- Every interactive element has visible `:focus-visible` — never `outline: none` without a replacement of equal or better visibility.
- Primary touch targets ≥44×44px; absolute floor 24×24px with adequate spacing (WCAG 2.2 SC 2.5.8). Adjacent targets need separation, not just size.
- Hover is an enhancement, never the only path to information. Anything hover-revealed must be reachable by keyboard and touch.
- Contrast: 4.5:1 body text, 3:1 large text (≥24px, or ≥18.66px bold), 3:1 for control boundaries and focus indicators.

**States**
- Every async surface defines **loading, empty, error, and success** before the happy path is considered done. An interface that only handles success is unfinished.
- Reserve space for content that loads. Layout that jumps is a UX failure (and a Core Web Vitals failure — profiling belongs to `scribeo-performance`).
- Meaning must survive `prefers-reduced-motion: reduce` and must never depend on animation to be understood.

## Reference routing

`SKILL.md` is the decision core. Load the reference when the work reaches that surface — do not load all of them.

| Load | When |
| --- | --- |
| `references/design-language.md` | Applying an established direction; diagnosing a structurally generic result |
| `references/layout.md` | Containers, grids, spacing rhythm, section composition, asymmetry, editorial layout |
| `references/typography.md` | Type scale, hierarchy, measure, line-height, tracking, responsive scaling |
| `references/responsive.md` | Desktop/tablet/mobile strategy, breakpoints, navigation adaptation, density |
| `references/components.md` | Buttons and CTAs, cards, sections, navigation, forms |
| `references/states.md` | Interaction, validation, loading, empty, and error states |
| `references/conversion.md` | Business-goal structure — lead gen, e-commerce, real estate, luxury, local |

## The generic test

Premium is not a visual effect. It is evidence of decisions. Whether the *aesthetic* reads as generic is `frontend-design`'s call; these are the **structural** checks that this skill owns. Before shipping, run each one — every failure has a specific structural cause:

1. **Swap test** — replace the logo and copy with a different company's. Does the page still work? If yes, it expresses no brand. *Cause: structure built before the design direction was established.*
2. **Section test** — could any section be lifted into an unrelated site unchanged? *Cause: template sections instead of content-driven composition.*
3. **Reason test** — point at three visual choices at random and state the reason for each. Hesitation means decoration.
4. **Hierarchy test** — squint until the type is illegible. Does the eye still land on the primary action first?
5. **Outline test** — read only the headings. Is that the argument the business needs to make?
6. **Content test** — does it survive real content: a 9-word headline, a 40-item list, one testimonial instead of three, a missing image?
7. **Proof test** — is every number, quote, logo, and badge traceable to something the client supplied?

Structurally, the "AI-generated" look is the absence of these answers: even density everywhere, no focal hierarchy, three feature cards because three fits, and sections that could belong to any company. The visual tells — default palettes, template chrome, decorative gradients — are `frontend-design`'s to catch.

## Quality gate

Do not report UX work complete until every line holds.

**Purpose**
- [ ] The primary action is obvious within seconds, without reading the page
- [ ] Hierarchy is unambiguous — one clear first thing per viewport
- [ ] The heading outline reads as a coherent argument on its own
- [ ] Nothing decorative remains whose reason you cannot state
- [ ] No section feels liftable from a template
- [ ] The interface communicates trust — and every proof element is real

**Responsive**
- [ ] Works at 320px through ultrawide, no horizontal overflow
- [ ] Mobile is a deliberate composition, not a narrowed desktop
- [ ] Content priority and density are correct per breakpoint
- [ ] Primary CTA reachable on mobile without hunting
- [ ] Type and measure readable at every width

**Interaction**
- [ ] Every interactive element: hover, focus-visible, active, disabled as applicable
- [ ] Loading, empty, and error states exist for every async surface
- [ ] Keyboard-only path completes the primary action
- [ ] Touch targets adequate and adequately separated
- [ ] Nothing essential is hover-only

**Robustness**
- [ ] Fully usable with `prefers-reduced-motion: reduce`
- [ ] Understandable with animation removed entirely
- [ ] Survives long, short, and missing content
- [ ] Semantics correct; ARIA used only where native elements fall short

**Verified, not assumed** — checked in a real browser at real viewports with real keyboard input. If you could not verify a line, say so explicitly rather than implying it passed.
