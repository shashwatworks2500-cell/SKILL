# robots.txt & XML Sitemaps

Load when implementing or auditing robots.txt or a sitemap.

All Google facts verified against Google Search Central (September 2026). Re-verify before quoting.

## robots.txt — what it actually does

**Purpose, per Google:** a robots.txt file tells crawlers which URLs they can access, *"used mainly to avoid overloading your site with requests."*

**What it is not, in Google's own words:** *"it is not a mechanism for keeping a web page out of Google."*

This is the single most misunderstood file in SEO. Google states plainly that a page disallowed in robots.txt **can still be indexed if linked to from other sites** — the URL and publicly available information such as anchor text may still appear in results, though the listing will lack a description.

### Three consequences that matter

1. **robots.txt is not security.** It is a public file listing paths you would rather were not crawled — which is closer to an index of interesting URLs than a protection mechanism. **Never use it to hide sensitive content.** Google's stated methods for keeping a page out of Search are: password-protect the files, use `noindex` (meta tag or response header), or remove the page.
2. **Disallow ≠ noindex.** If a URL is disallowed, the crawler cannot fetch it — so it cannot see a `noindex` on it either. To exclude a page from the index, it must be **crawlable** and carry `noindex`. Blocking a page you want deindexed is self-defeating.
3. **Blocking resources affects rendering.** Google will not render JavaScript from blocked files or blocked pages, which can change what is indexed. See `javascript-rendering.md`.

### Placement and syntax

The file must be at the **root of the site** — `https://example.com/robots.txt`. A robots.txt elsewhere is not read as one.

```txt
User-agent: *
Allow: /
Disallow: /account/
Disallow: /cart/

Sitemap: https://example.com/sitemap.xml
```

`Sitemap:` declares sitemap locations; Google supports discovery this way, alongside Search Console submission.

### Common defects

| Defect | Consequence |
| --- | --- |
| `Disallow: /` shipped to production | Nothing crawlable. **BLOCKER** |
| Disallowing a page you want deindexed | The `noindex` is never seen |
| Blocking CSS, JS or API routes needed to render | Rendering incomplete |
| Relying on it to hide private content | Content still reachable; URL may still surface |
| Staging robots.txt inherited by production | Either over- or under-blocking |
| Sitemap URL wrong or unreachable | Declared sitemap useless |

**Always verify the production file directly:** `curl -s https://example.com/robots.txt`. Do not audit the repository copy.

## XML sitemaps

A sitemap tells engines which URLs you consider worth crawling. Google is explicit about its weight: **submitting a sitemap** *"is merely a hint: it doesn't guarantee that Google will download the sitemap or use the sitemap for crawling URLs on the site."*

**Never tell a client a sitemap guarantees indexing.**

### Documented limits

Google documents a single sitemap as limited to **50MB uncompressed or 50,000 URLs**. Beyond either, split across multiple sitemaps and optionally create a **sitemap index file** to submit as one entry.

*(Verified September 2026 — re-verify before citing in client deliverables.)*

### What belongs in a sitemap

**Include only URLs you want crawled and indexed** — canonical, indexable, success-returning pages.

**Exclude:**

| Do not include | Why |
| --- | --- |
| Non-canonical duplicates | Contradicts your canonical signal |
| `noindex` pages | Contradictory instruction |
| Redirecting URLs | The destination belongs there instead |
| Error or gone URLs | Nothing to index |
| robots.txt-disallowed URLs | Cannot be crawled |
| Login, cart, account pages | Not intended for search |
| Paginated duplicates, filter permutations | See `ecommerce.md` |

**Do not blindly put every URL in the sitemap.** A sitemap containing non-canonical, noindexed and redirecting URLs sends contradictory signals and makes the file less useful as a hint.

### lastmod

Google's position is conditional: it uses `<lastmod>` *"if it's consistently and verifiably accurate"* — for example by comparison with the page's actual last modification. It should reflect **the last significant update** — content, structured data or links — **not** trivial changes such as a copyright year.

**Scribeo recommendation:** derive `lastmod` from real content modification time. A build-time `new Date()` on every deploy makes every URL look modified on every deploy — which is neither consistent nor verifiable, and devalues the field.

### Discovery

1. A `Sitemap:` line in robots.txt.
2. Submission in Search Console's Sitemaps report.

Both are appropriate; neither guarantees anything downstream.

### Audit checks

```bash
curl -sI https://example.com/sitemap.xml          # status and content type
curl -s  https://example.com/sitemap.xml | head   # well-formed XML?
```

- [ ] Returns a success status and XML content type
- [ ] Well-formed; parses
- [ ] URLs are absolute and on the canonical host and scheme
- [ ] All entries are canonical, indexable, success-returning
- [ ] No staging hostnames
- [ ] Within documented size and URL limits, or split with an index
- [ ] `lastmod` accurate, or omitted
- [ ] Declared in robots.txt
- [ ] Regression contract handed to `scribeo-testing`
