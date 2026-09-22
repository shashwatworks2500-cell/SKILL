# Scribeo Skills — Authoring Contract

Every directory here is one skill. This document is the contract each must satisfy. Read [`../CLAUDE.md`](../CLAUDE.md) first for repository-wide standards.

## Current state

All seven skills are **authored and validated**. Every directory contains a real `SKILL.md` plus a `references/` tree; no `.gitkeep` placeholders remain.

Every `SKILL.md` sits well under the ~500-line target, with depth pushed into `references/`. To check the current shape of the library rather than trusting a number written here:

```bash
for d in skills/*/; do
  printf '%-28s %4s lines  %2s refs\n' \
    "$(basename "$d")" "$(wc -l < "$d/SKILL.md")" "$(ls "$d/references" 2>/dev/null | wc -l)"
done
```

When a **new** domain directory is scaffolded, it carries a `.gitkeep` — git cannot track an empty directory, so removing the placeholder removes the directory. Delete it only when replacing it with a real `SKILL.md`.

## Minimum viable skill

A skill is complete when it has this and passes validation:

```
skills/scribeo-<domain>/
└── SKILL.md
```

```markdown
---
name: scribeo-<domain>
description: Use when <concrete trigger situation> — covers <specific capabilities>.
---

# Scribeo <Domain>

<what this does, in one line>

## When this applies
<the situations that should invoke it, and the adjacent ones that should not>

## Procedure
1. <ordered decision steps>

## Standards
<the rules Claude enforces in this domain>

## Patterns
<correct pattern, contrasted with the common mistake>
```

## Full skill layout

Add these directories only when the skill genuinely needs them:

| Path | Purpose | Loading |
| --- | --- | --- |
| `SKILL.md` | Decision procedure and core standards | Always, in full |
| `references/*.md` | Deep reference material | On demand, when the body links to it |
| `assets/*` | Templates, configs, baselines to copy or fill | Read or copied when used |
| `scripts/*` | Executable helpers | Executed, not read |

`SKILL.md` costs tokens every time the skill fires. Keep it to the core; push depth outward.

## The seven domains and their boundaries

Overlapping descriptions are the primary failure mode in a multi-skill collection — when two skills could claim the same task, neither fires reliably. Each description must state its boundary.

| Skill | Owns | Explicitly does **not** own |
| --- | --- | --- |
| `scribeo-ux-engineering` | Component architecture, state, layout, design-token implementation | Animation (→ motion), verification (→ visual-qa) |
| `scribeo-motion` | Timing, easing, choreography, reduced-motion, animation performance | Static layout (→ ux-engineering), general profiling (→ performance) |
| `scribeo-visual-qa` | Visual regression, design-fidelity review, cross-viewport/browser checks | Functional assertions (→ testing), a11y audits (→ accessibility) |
| `scribeo-performance` | Core Web Vitals, bundles, render/runtime profiling, assets and fonts | Animation-specific jank (→ motion) |
| `scribeo-accessibility` | WCAG conformance, semantics, keyboard/focus, assistive tech | Visual diffing (→ visual-qa) |
| `scribeo-seo` | Crawlability, metadata, structured data, canonicalisation, rendering strategy | Page speed as a metric (→ performance) |
| `scribeo-testing` | Unit, integration, e2e strategy and implementation | Visual snapshots (→ visual-qa) |

## External skill dependency

Every skill in this collection routes aesthetic judgement — palette, typeface, visual composition, distinctive styling — to **`frontend-design`**, an Anthropic-provided skill that is **not part of this marketplace**.

Two rules follow:

- **Declare it.** A skill that hands work to `frontend-design` must say so in its boundary table, marked `(Anthropic)` so no reader looks for it here.
- **Degrade gracefully.** Every skill carries an *"If `frontend-design` is unavailable"* clause in its `## Boundaries` section, stating what it still completes without the visual direction. A skill that simply blocks when an uninstalled skill is missing is broken. Match the existing wording when adding a new skill.

Do not add a new dependency on a skill outside this marketplace without both.

## Definition of done

A skill ships only when all of these hold:

- [ ] `name` matches the directory exactly and carries the `scribeo-` prefix
- [ ] `description` is third person and names concrete triggers, not self-description
- [ ] `description` states its boundary against the adjacent skills above
- [ ] Any dependency on a skill outside this marketplace is marked `(Anthropic)` and carries an *"If `<skill>` is unavailable"* clause naming what still completes without it
- [ ] Body is imperative, decision-first, and free of filler
- [ ] At least one correct/incorrect pattern contrast
- [ ] `SKILL.md` under ~500 lines; depth pushed to `references/`
- [ ] No secrets, client names, or private URLs anywhere in the skill
- [ ] `claude plugin validate .` passes from the repository root
- [ ] Plugin `version` bumped in `.claude-plugin/marketplace.json`
