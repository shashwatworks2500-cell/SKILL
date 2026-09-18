# URLs, Status Codes & Redirects

Load when designing URL structure or handling redirects and error responses.

**Verify current Google guidance before asserting how a status code is treated in Search.** No status code produces a ranking outcome, and this file does not claim one.

## URL architecture

URLs are a long-lived commitment: they appear in links, bookmarks, print, and other people's content. Changing them later costs redirects and signal dilution forever.

**Scribeo recommendations** (not documented requirements):

- **Readable and descriptive.** `/collections/engagement-rings` over `/c/47?t=2`.
- **Lowercase, hyphen-separated.** Underscores and mixed case cause avoidable duplicate variants.
- **Stable.** Do not encode anything that changes — dates, prices, campaign names.
- **Shallow where the structure allows.** Deep nesting for its own sake adds nothing.
- **One canonical form** of host, scheme and trailing slash, applied everywhere.
- **No session IDs or tracking parameters** in linked URLs.

**Whether a URL structure reflects the right information architecture is a `scribeo-ux-engineering` question.** This skill states technical requirements — stability, consistency, single canonical form — and routes structural decisions.

## Status codes

| Code | Meaning | Implementation note |
| --- | --- | --- |
| **200** | Success | The normal indexable response |
| **301** | Moved permanently | Use when a URL has permanently moved |
| **302** | Found / temporary | Use only when the move genuinely is temporary |
| **404** | Not found | The correct response for something that does not exist |
| **410** | Gone | Signals deliberate, permanent removal |
| **5xx** | Server error | Must be fixed; unreliable responses harm crawling |

**Do not claim one status code automatically produces one ranking outcome.** What is defensible: a status code communicates intent, and sending the *wrong* one communicates the wrong thing — a removed page returning 200 with "not found" text, or a permanent move sent as temporary.

### 404 vs 410

Both indicate the resource is unavailable. **410 states the removal is deliberate and permanent**; 404 leaves it open. Use 410 when content has been intentionally retired for good, 404 otherwise. **Verify current Google treatment before describing any difference in crawl behaviour.**

### Soft 404s

A **soft 404** is a page that returns 200 while presenting "not found" content to the user. It is a genuine defect: the response contradicts the content, and the engine may treat the URL as a low-value success rather than an absence.

Common causes: a client-rendered app rendering a not-found view without changing the status · a catch-all route returning 200 for unmatched paths · an empty category page rendering "no results" at 200.

**Fix:** return the status that matches the content. In frameworks with an explicit not-found mechanism, use it so the status is correct as well as the view.

## Redirects

Redirects are, per Google, **the strongest canonicalization signal** — stronger than `rel="canonical"`. Where a URL should never be used again, redirect it rather than canonicalising.

**Requirements:**

- **Redirect to the equivalent page**, not the homepage. Mass-redirecting removed pages to `/` is a common and user-hostile pattern: the user asked for something specific and gets nothing related. Where there is no equivalent, 404 or 410 is the honest answer.
- **Avoid chains.** `A → B → C` wastes crawl and loses clarity. Point A directly at C.
- **Never create loops.** `A → B → A` is a hard failure.
- **Keep them.** Removing a redirect after a migration breaks every external link that still uses the old URL.
- **Update internal links** to point at the destination. Internal links that rely on redirects are pure overhead.

### Migration

When URLs change at scale:

1. Map every old URL to its closest equivalent.
2. Redirect permanently, one hop.
3. Update internal links, sitemap entries and canonicals to the new URLs.
4. Keep the sitemap listing only new canonical URLs.
5. Retain redirects indefinitely.
6. Monitor Search Console coverage after the change — **the client's property, not something verifiable here.**

**Never promise that a migration will preserve rankings or traffic.** Correct implementation is what engineering controls; the outcome is the search engine's.

## Auditing

```bash
curl -sI https://example.com/old-page              # status and Location
curl -sIL https://example.com/old-page | grep -iE '^HTTP|^location'   # full chain
```

- [ ] Important routes return 200
- [ ] Removed content returns 404 or 410, not 200
- [ ] No soft 404s — status matches the content shown
- [ ] Redirects are single-hop
- [ ] No loops
- [ ] Redirect targets return 200
- [ ] http → https and host normalisation in place
- [ ] Trailing-slash handling consistent
- [ ] Internal links point at final destinations
- [ ] No 5xx on crawlable routes
- [ ] Status contracts handed to `scribeo-testing`
