# Scribeo Skills — Authoring Contract

Every directory here is one skill. This document is the contract each must satisfy. Read [`../CLAUDE.md`](../CLAUDE.md) first for repository-wide standards.

## Current state

All seven directories are **scaffolding**. Each contains only a `.gitkeep` placeholder — git cannot track an empty directory, so removing it removes the directory. Delete the `.gitkeep` only when replacing it with a real `SKILL.md`.

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

## Definition of done

A skill ships only when all of these hold:

- [ ] `name` matches the directory exactly and carries the `scribeo-` prefix
- [ ] `description` is third person and names concrete triggers, not self-description
- [ ] `description` states its boundary against the adjacent skills above
- [ ] Body is imperative, decision-first, and free of filler
- [ ] At least one correct/incorrect pattern contrast
- [ ] `SKILL.md` under ~500 lines; depth pushed to `references/`
- [ ] No secrets, client names, or private URLs anywhere in the skill
- [ ] `claude plugin validate .` passes from the repository root
- [ ] Plugin `version` bumped in `.claude-plugin/marketplace.json`
