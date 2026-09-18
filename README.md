# SKILL — Scribeo's Central Skills Repository

The single source of truth for Scribeo's reusable [Claude Code](https://code.claude.com/docs) skills and workflows.

This repository is an **installable Claude Code plugin marketplace**. Skills live here once, are versioned here, and are installed from here — on any machine, in any project, without copying directories by hand.

---

## Why this repository exists

Skills kept in a project's `.claude/` directory die with that project. Skills copied into `~/.claude/skills/` on one laptop don't exist on the next one. Both approaches quietly fork: the same skill drifts into three versions and none of them is canonical.

This repository fixes that with one rule: **a Scribeo skill is defined exactly once, here.** Everything else — every machine, every project, every session — installs from this marketplace and updates from it.

That gives us:

- **One canonical definition.** No drift, no "which copy is current?"
- **Real version control.** Skills are reviewed, diffed, and versioned like the rest of our engineering work.
- **Portable setup.** A new machine is two commands away from the full Scribeo toolkit.
- **Composability.** Skills are built to be combined — a UX engineering pass that hands off to visual QA, which hands off to accessibility.

## Installation

```bash
# 1. Register this repository as a marketplace (once per machine)
/plugin marketplace add shashwatworks2500-cell/SKILL

# 2. Install the Scribeo skill collection
/plugin install scribeo-skills@scribeo
```

To pull newer skills later:

```bash
/plugin marketplace update scribeo
/plugin update scribeo-skills
```

> **Note:** five skills are authored (`scribeo-ux-engineering`, `scribeo-motion`, `scribeo-visual-qa`, `scribeo-performance`, `scribeo-accessibility`); the remaining two are scaffolding. See [Status](#status).

## Repository structure

```
SKILL/
├── README.md                   # This file
├── CLAUDE.md                   # Standards Claude follows when working in this repo
├── .gitignore                  # Secret-safe ignore rules (this repo is public)
├── .claude-plugin/
│   └── marketplace.json        # Plugin marketplace manifest
└── skills/
    ├── README.md               # The skill-authoring contract
    ├── scribeo-ux-engineering/ # Interface implementation & component architecture
    ├── scribeo-motion/         # Animation, transitions, choreography
    ├── scribeo-visual-qa/      # Visual regression & design-fidelity review
    ├── scribeo-performance/    # Core Web Vitals, bundle & runtime performance
    ├── scribeo-accessibility/  # WCAG conformance, semantics, assistive tech
    ├── scribeo-seo/            # Technical SEO, metadata, structured data
    └── scribeo-testing/        # Unit, integration & end-to-end testing
```

## The skill domains

| Skill | Domain |
| --- | --- |
| `scribeo-ux-engineering` | Translating design intent into production interfaces — component architecture, state, layout, design-token discipline. |
| `scribeo-motion` | Animation and transition work: timing, easing, choreography, reduced-motion behaviour, and performance-safe technique. |
| `scribeo-visual-qa` | Verifying built UI against design intent — visual regression, cross-viewport and cross-browser review, pixel-fidelity audits. |
| `scribeo-performance` | Core Web Vitals, bundle analysis, render and runtime profiling, asset and font strategy. |
| `scribeo-accessibility` | WCAG conformance, semantic markup, keyboard and focus behaviour, assistive-technology verification. |
| `scribeo-seo` | Technical SEO: crawlability, metadata, structured data, canonicalisation, rendering strategy. |
| `scribeo-testing` | Test strategy and implementation across unit, integration, and end-to-end layers. |

## Status

**5 of 7 skills authored.**

Remaining directories under `skills/` are intentional placeholders holding only a `.gitkeep` (git cannot track empty directories). Skills are authored one at a time, each following the contract in [`skills/README.md`](skills/README.md).

| Skill | Status |
| --- | --- |
| `scribeo-ux-engineering` | **Authored** — v0.2.0 |
| `scribeo-motion` | **Authored** — v0.3.0 |
| `scribeo-visual-qa` | **Authored** — v0.4.0 |
| `scribeo-performance` | **Authored** — v0.5.0 |
| `scribeo-accessibility` | **Authored** — v0.6.0 |
| `scribeo-seo` | Not started |
| `scribeo-testing` | Not started |

## Contributing a skill

1. Read [`skills/README.md`](skills/README.md) — it is the authoring contract, not a suggestion.
2. Read [`CLAUDE.md`](CLAUDE.md) for the repository's standards and conventions.
3. Author `skills/<skill-name>/SKILL.md` with valid frontmatter.
4. Validate before committing:
   ```bash
   claude plugin validate .
   ```
5. Bump the plugin `version` in `.claude-plugin/marketplace.json` so installed copies detect the update.

## Security

This repository is **public**. Never commit credentials, API keys, tokens, client identifiers, or private project data. `.gitignore` blocks the common cases, but it is a safety net — not a substitute for reading your own diff. Anything pushed to a public repository is permanently recoverable from history, forks, and third-party caches, even after deletion.

---

Maintained by Scribeo.
