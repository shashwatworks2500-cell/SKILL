---
name: scribeo-seo
description: Use when working on technical SEO and search discoverability — crawlability, indexability, robots.txt and robots directives, XML sitemaps, canonical URLs, redirects and status codes, title and description metadata, Open Graph and social metadata, JSON-LD structured data and Schema.org, rich-result eligibility, internal linking for discovery, URL architecture, hreflang, rendering and JavaScript indexing concerns, and Search Console diagnostics. Triggers on requests like "SEO audit", "technical SEO", "SEO check", "fix SEO metadata", "add structured data", "fix the sitemap", "check the robots.txt", "canonical is wrong", "Google is choosing the wrong canonical", "Google isn't indexing this page", "why isn't this page showing in search", "our pages aren't being discovered", "indexing issue", "add JSON-LD", "set up hreflang", "fix redirects", "add noindex", "sitemap submission", and on mentions of Googlebot, robots meta, rich results, search snippets, breadcrumbs, or Search Console. Visual design belongs to frontend-design, UX architecture to scribeo-ux-engineering, animation to scribeo-motion, rendered verification to scribeo-visual-qa, performance measurement and budgets to scribeo-performance, accessibility conformance to scribeo-accessibility, and test architecture to scribeo-testing. Not triggered by general website building, design, UX, performance, accessibility or copywriting work unless SEO is specifically involved.
---

# Scribeo SEO

Scribeo Studio's technical SEO and search-discoverability standard. One loop, in order:

**UNDERSTAND → AUDIT → IMPLEMENT → VERIFY**

Apply this whenever the question is *can search engines discover, crawl, understand and appropriately index this content, and is the implementation eligible for documented search features.*

## The law

**SEO engineering makes content discoverable, understandable, crawlable, indexable, and eligible for documented search features. It does not control search rankings.**

This skill operates on the part of search that is **engineering-controlled**. It does not operate on — and must never claim to control — search-engine ranking decisions, competitive position, content quality, domain authority, user behaviour signals, external links, or algorithmic ranking systems.

### Four states, never conflated

This distinction decides most SEO conversations, and collapsing it is the commonest error in the field:

| State | Means | Controlled by |
| --- | --- | --- |
| **Crawlable** | A crawler can reach and fetch the URL | **Engineering — this skill** |
| **Indexable** | Nothing instructs the engine to exclude it, and it is eligible for the index | **Engineering — this skill** |
| **Indexed** | The engine has actually chosen to store it | **The search engine.** Not guaranteed by anything here |
| **Ranked** | It appears, at some position, for some query | **The search engine.** Never promised, never predicted |

**Crawlable ≠ indexable ≠ indexed ≠ ranked.** This skill can make a page crawlable and indexable. It cannot make it indexed, and it cannot make it rank.

**Never claim a page is indexed without evidence** from Search Console or another appropriate first-party source. "We added a sitemap, so it will be indexed" is a false statement.

## Do not invent SEO rules

**Never invent a ranking factor, a search-engine requirement, or a guaranteed outcome.**

The field is saturated with folklore repeated until it sounds official. These claims are **rejected unless a current authoritative source says otherwise**:

- ❌ "Sitemap submission guarantees indexing." Google: submitting a sitemap *"is merely a hint: it doesn't guarantee that Google will download the sitemap or use the sitemap for crawling URLs."*
- ❌ "robots.txt removes a URL from Google." Google: robots.txt *"is not a mechanism for keeping a web page out of Google."* A disallowed URL can still be indexed if linked from elsewhere.
- ❌ "This schema guarantees rich results." Google: *"Google does not guarantee that your structured data will show up in search results, even if your page is marked up correctly."*
- ❌ Exact keyword-density targets.
- ❌ Arbitrary universal title or meta-description character limits. Google: *"there's no limit on how long a `<title>` element can be"* — it is truncated for display, typically to fit device width.
- ❌ "Google always does X."
- ❌ Guaranteed indexing timelines.
- ❌ "A PageSpeed score of X guarantees SEO."
- ❌ "More pages automatically means better SEO."
- ❌ "More keywords automatically means better ranking."

**If current authoritative documentation does not support a claim: verify before citing it.** Priority of sources: **Google Search Central and Search Console documentation · Schema.org · Next.js official documentation · MDN** for web-platform behaviour. SEO blogs, agency articles and forum posts may generate hypotheses; **they never become a Scribeo requirement.**

**Preserve the distinction between documented behaviour and guaranteed outcome.** Where Google documents what it does without guaranteeing a result, say exactly that. And mark **Scribeo recommendations** as recommendations, distinct from documented requirements.

## Boundaries

**The seam is the question each skill asks:**

| Skill | Asks |
| --- | --- |
| `frontend-design` | What should this look and feel like? |
| `scribeo-ux-engineering` | What should the experience and interaction structure be? |
| `scribeo-motion` | How should movement and animation be implemented? |
| `scribeo-visual-qa` | Does the rendered interface look and behave as intended? |
| `scribeo-performance` | How fast and efficiently does it perform under measurement? |
| `scribeo-accessibility` | Can people with different access needs perceive, operate, understand and interact with it? |
| `scribeo-testing` | What observable behaviour must remain correct, and can we automate verifying it? |
| **this skill** | **Can search engines discover, crawl, understand and appropriately index the content — and is the implementation eligible for documented search features?** |

**This skill owns:** crawlability · indexability · robots directives · sitemap generation · canonical implementation · redirects and the search implications of status codes · metadata implementation · structured data implementation · internal linking for discoverability · search-oriented constraints on information architecture · hreflang where genuinely required · technical URL architecture · search-engine-facing rendering considerations · SEO verification and regression requirements · Search Console-oriented diagnostics.

**This skill does not own:** visual aesthetics · UX decisions · animation · performance diagnosis · accessibility conformance · test architecture · content strategy as a whole · guaranteed rankings · guaranteed indexing · link-building · paid advertising · social-media growth.

**SEO frequently identifies requirements that cross into another skill.** That is expected. State the requirement, cite the source, and route the decision — do not implement across the boundary.

### Reciprocal handoffs

- **`scribeo-ux-engineering` → here.** Supplies the information architecture and navigation structure this skill assesses for discoverability.
- **here → `scribeo-ux-engineering`.** Identifies crawlable-IA requirements, internal-link discoverability needs and URL-structure implications. **Never redesigns the interface for search engines.** A pattern that serves crawlers and harms users is rejected here, not proposed.
- **`scribeo-performance` → here.** Supplies measured performance facts where a search-relevant rendering or delivery concern exists.
- **here → `scribeo-performance`.** Identifies search-relevant rendering and discoverability concerns and routes measurement there. **Never invents a performance threshold.** LCP, INP, CLS and TTFB are not SEO thresholds in this skill.
- **`scribeo-accessibility` → here.** Owns accessibility requirements and conformance.
- **here → `scribeo-accessibility`.** Benefits from semantic, meaningful HTML but **never treats accessibility as an SEO shortcut.** Alt text exists for users first; it is not a keyword slot.
- **`scribeo-visual-qa` → here.** Verifies the rendering consequences of SEO-related implementation changes.
- **here → `scribeo-visual-qa`.** Routes any change that alters rendering — visible metadata-driven content, structured-data-backed UI, canonical or indexability changes with visible effect.
- **`scribeo-motion` → here.** Owns animation implementation.
- **here → `scribeo-motion`.** Identifies discoverability and rendering requirements when important content depends on client-side behaviour. **Never dictates animation aesthetics.**
- **`scribeo-testing` → here.** Owns test architecture, fixtures, CI and regression suites. **It must not invent SEO requirements.**
- **here → `scribeo-testing`.** Supplies deterministic SEO contracts suitable for automated regression — see the testable-contracts list below.

**If `frontend-design` is unavailable.** It is an Anthropic-provided skill, not part of the `scribeo-skills` marketplace, so it may not be installed. Never invent the aesthetic here to unblock yourself, and never stall work the aesthetic does not gate. Say plainly that the visual direction is unset, ask for it, and proceed with the full audit and implementation. Nothing this skill owns depends on the visual direction; route only the rendering-visible consequences of a change once the direction is known.

## Workflow

**UNDERSTAND**

1. **Establish the site's purpose and the pages that matter.** Discoverability work on pages nobody needs found is wasted.
2. **Establish what is actually claimed.** Is this a technical audit, a fix, or a question about search performance? The last is often not an engineering question at all.
3. **Gather first-party evidence** where available — Search Console coverage and URL inspection data beat inference every time.

**AUDIT** — inner loop: **DISCOVER → CRAWL → INSPECT → CLASSIFY → FIX → VERIFY → REGRESSION-PROTECT**

4. **Discover** the URL surface — sitemap, navigation, internal links, known routes.
5. **Crawl** what is reachable; note what is not.
6. **Inspect** each surface: HTTP status, robots.txt, robots directives, canonical, title, description, headings, internal links, structured data, rendering, duplicate variants, redirects, environment leakage, social metadata.
7. **Classify** each finding by severity (below) and by owner.
8. **Fix** what this skill owns; route the rest.
9. **Verify** the implementation — re-fetch, re-inspect, confirm the served output.
10. **Regression-protect** by handing deterministic contracts to `scribeo-testing`.

**Full procedure in `references/audit.md`.**

## Tooling in this environment

Verified this session, not assumed:

| Capability | Status |
| --- | --- |
| `curl` / `wget` | ✅ Fetch headers, status codes, robots.txt, sitemaps, served HTML |
| Playwright MCP | ✅ Rendered-DOM inspection, console, network — useful for JS-rendering questions |
| Chromium 141 | ✅ headless |
| **Next.js** | ❌ **Not installed and not resolvable.** Latest on npm is **16.3.5** (MIT). Project APIs documented in `references/nextjs.md` were verified against the official 16.3.5 docs |
| **Search Console** | ❌ **No access.** Indexing status, coverage and URL inspection cannot be checked here — they require the client's verified property |
| Rich Results Test / validators | ❌ Not available offline; structured data can be checked for syntax locally, **not for rich-result eligibility** |
| Lighthouse, SEO crawlers | ❌ Not installed |

**No dependencies were added, and this repository has no `package.json`.**

**The Search Console gap is the important one.** Without it, this skill can verify *implementation*, never *outcome*. Say so explicitly rather than implying indexing was confirmed.

## Severity

Severity describes **engineering and discoverability risk** — never predicted ranking loss.

| Severity | Definition |
| --- | --- |
| **BLOCKER** | Prevents crawling or indexing of important content — a stray `noindex` in production, a `Disallow: /` shipped live, important routes returning errors |
| **HIGH** | Materially impairs discovery or understanding — missing or wrong canonicals across a template, a broken sitemap, important content unavailable without interaction |
| **MEDIUM** | A correctness defect with limited blast radius — a duplicated title across a few pages, an incomplete structured-data type, a redirect chain |
| **LOW** | Minor implementation inconsistency with little practical consequence |
| **INFORMATIONAL** | An observation, or a recommendation requiring a business decision |

**No severity implies a ranking outcome.** "BLOCKER" means the engineering prevents indexing, not that traffic will fall by a number.

## SEO report format

```
Scope:        scribeo-example.com — 6 templates, 240 URLs from sitemap
Environment:  Production, fetched via curl + rendered check in Chromium 141
Checked:      HTTP status, robots.txt, robots meta, canonical, title,
              description, headings, internal links, JSON-LD, sitemap,
              duplicate variants, OG metadata
Finding:      All /collections/* pages emit <meta name="robots" content="noindex">
Severity:     BLOCKER
Evidence:     curl -sI + served HTML for 12 sampled URLs; consistent
Source:       Google documents noindex as the supported way to exclude a
              page from Search — this is doing so unintentionally
Fix:          Remove the noindex from the collections template; it appears to
              be leaking from a staging default
Verified:     Re-fetched 12 URLs after fix — directive absent, 200 OK
Uncertainty:  Whether these URLs were previously indexed and will return to
              the index is NOT verifiable here — requires Search Console
Next action:  Client to check Coverage in Search Console; regression contract
              handed to scribeo-testing
```

```
Bad:   "Fixed SEO — the pages will now rank."
Good:  "Removed an unintended noindex from the collections template;
        12 sampled URLs now return 200 with no robots exclusion. Whether
        Google re-indexes them, and when, is its decision — verify in
        Search Console."
```

**Never state that a page is indexed, will be indexed, or will rank.** Report what was implemented and verified; attribute outcomes to the search engine.

## Contracts worth regression-testing

Deterministic, application-controlled, and therefore suitable to hand to `scribeo-testing`:

- An expected `<title>` exists and is non-empty on key routes
- The canonical points at the expected absolute URL
- The robots directive is what it should be — and **no production route accidentally emits `noindex`**
- The sitemap route returns the expected status and content type
- JSON-LD parses as valid JSON and carries the expected `@type`
- Redirects resolve to the expected destination and status, without chains
- Critical routes return the expected HTTP status
- Internal links resolve rather than 404

**Never testable, never asserted:** that Google will index a page · that it will rank · that a schema will produce a rich result. Those are not application contracts.

## Reference routing

`SKILL.md` is the decision core. Load a reference when work reaches that surface.

| Load | When |
| --- | --- |
| `references/crawl-index.md` | Crawlability, indexability, internal linking, the four-state distinction |
| `references/robots-sitemap.md` | robots.txt and XML sitemaps |
| `references/canonicals.md` | Canonical URLs, duplicate variants, hreflang and international |
| `references/metadata.md` | Titles, descriptions, robots meta, Open Graph and social |
| `references/nextjs.md` | Next.js App Router metadata, sitemap and robots APIs |
| `references/structured-data.md` | JSON-LD, Schema.org, rich-result eligibility |
| `references/urls-status-redirects.md` | URL architecture, status codes, redirects, soft 404s |
| `references/javascript-rendering.md` | Client rendering, hydration, content availability to crawlers |
| `references/local-seo.md` | Local business identity, LocalBusiness data, location pages |
| `references/ecommerce.md` | Product and category URLs, facets, product structured data |
| `references/images-video.md` | Image and video discoverability, and the alt-text boundary |
| `references/audit.md` | Running a full audit end to end |

## Quality gate

Do not report SEO work complete until every line holds.

**Truthfulness**
- [ ] No ranking, traffic, indexing or visibility outcome promised
- [ ] No page claimed as indexed without first-party evidence
- [ ] No invented ranking factor or search-engine requirement
- [ ] Documented behaviour distinguished from guaranteed outcome
- [ ] Scribeo recommendations marked as recommendations, not requirements
- [ ] Every normative claim traceable to an authoritative source

**Implementation**
- [ ] Important content crawlable and indexable; nothing unintentionally excluded
- [ ] Canonicals absolute, consistent, and correct for the template
- [ ] robots.txt correct, and not relied on to hide anything
- [ ] Sitemap contains canonical, indexable URLs only
- [ ] Metadata present, unique per page, and representative of the content
- [ ] Structured data matches content visible on the page
- [ ] Status codes and redirects behave as intended
- [ ] No staging or environment leakage in production output

**Boundaries**
- [ ] No UX, design, motion, performance, accessibility or test-architecture decisions taken here
- [ ] Cross-boundary requirements stated and routed, not implemented
- [ ] Accessibility never used as an SEO shortcut
- [ ] No performance threshold invented

**Verification**
- [ ] Served output re-fetched and confirmed after each change
- [ ] Rendered output checked where content depends on JavaScript
- [ ] Deterministic contracts handed to `scribeo-testing`
- [ ] Remaining uncertainty stated explicitly, including anything needing Search Console

**Verified, not assumed** — if something could not be checked in this environment, say so plainly rather than implying it passed.
