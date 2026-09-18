# SEO Audit

Load when running a full audit.

**DISCOVER → CRAWL → INSPECT → CLASSIFY → FIX → VERIFY → REGRESSION-PROTECT**

## Before starting

State four things, or the audit's conclusions cannot be read correctly:

1. **Scope** — which routes and templates.
2. **Environment** — production, staging, or local. Auditing the wrong environment is a wasted pass, and staging often carries deliberate exclusions.
3. **Access** — is Search Console available? **It is not available from this environment**, which bounds every conclusion about indexing.
4. **What is being asked** — an implementation audit, a specific defect, or a question about search performance. The last is frequently not an engineering question.

## 1. Discover

Build the URL surface: the XML sitemap · the navigation · internal links from key pages · known important routes · anything the client names.

Note immediately anything **in the sitemap but not linked**, or **linked but not in the sitemap** — both are inconsistencies worth reporting.

## 2. Crawl

Fetch each URL and record status, redirects and response headers.

```bash
curl -sI https://example.com/page | grep -iE '^HTTP|^location|^x-robots-tag|^content-type'
curl -s  https://example.com/robots.txt
curl -sI https://example.com/sitemap.xml
```

Record what could **not** be fetched and why — that is often the finding.

## 3. Inspect

Per template (not per URL — sample representative URLs from each template):

| Surface | Check | Reference |
| --- | --- | --- |
| **Status** | 200 on indexable routes; correct codes elsewhere | `urls-status-redirects.md` |
| **robots.txt** | Correct, not over-blocking, sitemap declared | `robots-sitemap.md` |
| **Robots directives** | No unintended `noindex` in meta or header | `metadata.md` |
| **Canonical** | Present, absolute, correct, consistent | `canonicals.md` |
| **Title** | Present, unique, specific | `metadata.md` |
| **Description** | Present where used, unique, accurate | `metadata.md` |
| **Headings** | One `h1`; outline reflects structure | `crawl-index.md` |
| **Internal links** | Real `<a href>`; important pages linked | `crawl-index.md` |
| **Structured data** | Valid JSON; type fits; matches visible content | `structured-data.md` |
| **Rendering** | Important content in served HTML | `javascript-rendering.md` |
| **Duplicates** | Host, scheme, slash, case, parameter variants | `canonicals.md` |
| **Redirects** | Single-hop, no loops, targets 200 | `urls-status-redirects.md` |
| **Sitemap** | Reachable, valid, canonical indexable URLs only | `robots-sitemap.md` |
| **Social metadata** | Complete, absolute, image reachable | `metadata.md` |
| **Environment leakage** | No staging hosts in canonicals, sitemap, OG | `crawl-index.md` |

A fast first pass on one URL:

```bash
curl -s https://example.com/page \
  | grep -iE '<title>|name="description"|name="robots"|rel="canonical"|property="og:|application/ld\+json|<h1'
```

## 4. Classify

Assign **severity** — BLOCKER / HIGH / MEDIUM / LOW / INFORMATIONAL, as defined in `SKILL.md` — and an **owner**.

Severity describes **engineering and discoverability risk, never a predicted ranking outcome.**

Many findings are not this skill's to fix: IA and navigation → `scribeo-ux-engineering` · rendering performance → `scribeo-performance` · alt text and semantics as requirements → `scribeo-accessibility` · visible rendering → `scribeo-visual-qa` · content decisions → the client. **Route them; do not implement across the boundary.**

## 5. Fix

Fix what this skill owns, **one change at a time where possible**, so verification is unambiguous.

**Order by severity.** A production `noindex` or a `Disallow: /` outranks a duplicate meta description by a wide margin.

## 6. Verify

**Re-fetch the served output.** Not the source, not the local build.

```bash
curl -s https://example.com/page | grep -i 'name="robots"'
curl -sI https://example.com/page | grep -iE '^HTTP|^x-robots-tag'
```

Where content depends on JavaScript, also check the **rendered** DOM via Playwright MCP, and compare with the served HTML.

**What verification can and cannot establish:**

| Can verify here | Cannot verify here |
| --- | --- |
| The served HTML and headers are correct | Whether Google crawled it |
| Status codes and redirect chains | Whether Google indexed it |
| Canonical, metadata, structured-data syntax | Which canonical Google chose |
| Content present in served or rendered output | Whether a rich result will appear |
| Sitemap and robots.txt validity | Any ranking outcome |

**Everything in the right-hand column requires Search Console or is the search engine's decision.** Say so explicitly.

## 7. Regression-protect

Hand deterministic contracts to `scribeo-testing` — the list is in `SKILL.md`. The highest-value ones: **no production route emits `noindex`** · canonicals point where expected · key routes return 200 · the sitemap route is valid · JSON-LD parses.

These are exactly the defects that reappear silently during unrelated work.

## Reporting

Use the report format in `SKILL.md`. Every finding carries scope, environment, evidence, severity, source where relevant, fix, verification result, remaining uncertainty and next action.

**Three things never to write:**

- That a page is indexed, without first-party evidence.
- That a change will improve rankings or traffic.
- That structured data will produce a rich result.

**One thing always to write:** what could not be verified in this environment, and what would be needed to verify it.
