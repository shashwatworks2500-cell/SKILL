# CLAUDE.md — Scribeo Skills Repository

Standards for working in this repository. This is a **skills library and plugin marketplace** — not an application. There is no app to run, no server to start, no dependencies to install.

## What this repository is

`SKILL` is Scribeo's canonical, permanent home for reusable Claude Code skills. Skills are defined here exactly once and installed everywhere else from this marketplace. A skill that lives anywhere else is a fork, not a skill.

## Non-negotiables

1. **This repository is public.** Never commit secrets, API keys, tokens, credentials, client names, client URLs, internal hostnames, or private project data. Not in skill content, not in examples, not in test fixtures. Use obvious placeholders: `example.com`, `<API_KEY>`, `acme-corp`.
2. **One skill, one directory.** `skills/<skill-name>/SKILL.md`. Never nest a skill inside another skill.
3. **Never invent a skill directory.** The seven skill domains are fixed. A genuinely new domain is a discussion, not a commit.
4. **Validate before every commit:** `claude plugin validate .` must pass.
5. **Never remove a `.gitkeep` unless that directory now contains a real `SKILL.md`.** Git cannot track empty directories; deleting the placeholder deletes the directory.
6. **Skills are documentation, not code.** Prefer prose that instructs over scripts that execute. Add executable helpers only when a task genuinely cannot be described.

## Skill authoring standard

Every skill is a directory containing `SKILL.md` with YAML frontmatter:

```markdown
---
name: scribeo-motion
description: Use when implementing or reviewing animation, transitions, or motion choreography — covers timing, easing, reduced-motion behaviour, and performance-safe technique.
---

# Scribeo Motion

<body: the actual instructions>
```

### Frontmatter rules

- `name` — **required.** Kebab-case, must exactly match the directory name, must carry the `scribeo-` prefix.
- `description` — **required.** Third person, states *when to use this skill*, not what it is.
- Keep frontmatter to these two fields unless a feature genuinely requires more.

### The `description` field is the whole product

The `description` is the **only** text the model sees when deciding whether to load a skill. A skill with a brilliant body and a vague description never fires — it is dead weight that costs tokens and delivers nothing.

Write descriptions that name concrete triggers:

- ❌ `"Helps with performance."` — no trigger, no scope, will not fire reliably.
- ❌ `"This skill is for performance optimisation work."` — describes itself, not the situation.
- ✅ `"Use when diagnosing or improving Core Web Vitals, bundle size, render performance, or asset loading — includes LCP/CLS/INP triage and profiling workflow."`

Include the vocabulary a person would actually use: tool names, metric names, symptoms, file types. Prefer over-specific to over-general.

### Body rules

- Open with a one-line statement of what the skill does and when it applies.
- Write imperatively. The reader is Claude mid-task, not a student.
- Lead with the decision procedure — the order of operations — before reference detail.
- Show the correct pattern *and* the common wrong one. Contrast teaches faster than assertion.
- Be concrete and framework-honest: name the actual API, flag, or metric.
- No filler. No "in today's fast-paced world". Every line must change what Claude does.

### Progressive disclosure

`SKILL.md` is loaded in full whenever the skill fires, so it pays for its own length. Keep it to the decision-making core — target **under ~500 lines**. Push depth into sibling files the body links to, loaded only when needed:

```
skills/scribeo-performance/
├── SKILL.md                  # Decision procedure, always loaded
├── references/
│   └── web-vitals.md         # Deep reference, loaded on demand
├── assets/
│   └── budget-template.json  # Templates the skill copies from
└── scripts/
    └── audit.mjs             # Executable helpers, run not read
```

Use `references/` for prose Claude should read, `assets/` for files it should copy or fill in, and `scripts/` for code it should execute.

## Conventions

- **Naming:** all skills carry the `scribeo-` prefix; directories and `name` fields are kebab-case and identical.
- **Markdown:** ATX headings (`##`), fenced code blocks with a language tag, relative links between files in this repo.
- **Spelling:** consistent within a file; don't mix `colour`/`color` in the same skill.
- **Versioning:** bump the `scribeo-skills` `version` in `.claude-plugin/marketplace.json` on every substantive skill change, or installed copies will not detect an update. Semver: patch for wording, minor for new skills or capabilities, major for breaking reorganisation.
- **Commits:** imperative subject, scoped where useful — `add scribeo-motion skill`, `fix trigger description in scribeo-seo`.

## Validation

```bash
claude plugin validate .        # Validates the marketplace manifest and all skills
claude plugin eval ./           # Runs bundled eval suites, when a skill has them
```

`claude plugin validate .` is the gate. Do not commit a repository state that fails it.

## Testing a skill's trigger accuracy

A skill that does not fire is broken regardless of its content. Before considering a skill done, verify the `description` actually triggers on realistic phrasings — and does *not* fire on adjacent work that belongs to a different skill. Overlapping descriptions across the seven domains are the most likely failure mode here; when two skills could both claim a task, make each description state its boundary explicitly.

## Out of scope

- Do not add application code, build tooling, or dependencies to this repository.
- Do not install plugins, skills, or MCP servers as part of work here.
- Do not modify any other repository — in particular, the separate `Skills` repository is intentionally untouched.
