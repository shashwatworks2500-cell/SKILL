# Crawlability, Indexability & Internal Linking

Load when diagnosing whether content can be found and included in the index.

Facts below verified against Google Search Central (September 2026). Re-verify before quoting in client work.

## The four states

| State | Means | Who controls it |
| --- | --- | --- |
| **Crawlable** | A crawler can reach and fetch the URL | Engineering |
| **Indexable** | Nothing excludes it, and it is eligible | Engineering |
| **Indexed** | The engine has chosen to store it | **The search engine** |
| **Ranked** | It appears for a query, at some position | **The search engine** |

Engineering owns the first two. **Never claim the third without evidence, and never promise the fourth.**

The practical consequence: when someone asks "why isn't this page in Google?", the engineering question is *"is it crawlable and indexable?"* If both are yes, the remaining answer is the search engine's, and the honest response is to check Search Console rather than to speculate.

## Crawlability

A URL is crawlable when a crawler can **discover** it and **fetch** it.

**Discovery** happens through: links from other crawled pages · the XML sitemap · links from external sites · previously known URLs.

**Fetching** succeeds when: robots.txt does not disallow it · it returns a success status · it does not require authentication · the server responds reliably.

### Crawlable links

Google discovers links by parsing HTML. The reliable form is an `<a>` element with an `href` attribute resolving to a real URL.

```html
<!-- Discoverable -->
<a href="/collections/rings">Rings</a>

<!-- Not a link: no href, so nothing to discover -->
<span onclick="navigate('/collections/rings')">Rings</span>
<a onclick="navigate('/collections/rings')">Rings</a>
<button onclick="router.push('/collections/rings')">Rings</button>
```

**Scribeo rule:** every route that should be discoverable is reachable by at least one real `<a href>` in server-rendered HTML. A click handler is a UX mechanism, not a discovery mechanism — and it is also a `scribeo-ux-engineering` and `scribeo-accessibility` defect when it replaces a link.

### Blocked resources

Google states that **it will not render JavaScript from files or pages blocked in robots.txt**. Blocking a bundle, stylesheet or API route that a page needs to render can therefore affect what is indexed, even when the page itself is allowed. See `javascript-rendering.md`.

## Indexability

A crawlable page is indexable when nothing instructs the engine to exclude it.

**Exclusion mechanisms — engineering-controlled:**

| Mechanism | Effect |
| --- | --- |
| `<meta name="robots" content="noindex">` | Requests exclusion from the index |
| `X-Robots-Tag: noindex` HTTP header | Same, for non-HTML responses (PDFs, images) |
| Non-success status codes | An error or gone response is not indexed as content |
| Authentication | Content behind a login is not fetched |
| Canonical pointing elsewhere | A *signal* the other URL is preferred — not an exclusion |

**A critical asymmetry:** `noindex` only works **if the page can be crawled.** If robots.txt disallows the URL, the crawler never fetches it and never sees the `noindex`. Blocking and excluding are different operations, and combining them incorrectly is a common defect — see `robots-sitemap.md`.

### The accidental-noindex class

The highest-severity SEO defect in practice, because it is invisible without checking the served output:

- A staging default shipping to production.
- An environment variable controlling robots behaviour that is wrong in production.
- A template-level `noindex` inherited by pages that should be indexed.
- A CMS or framework flag toggled during development and never reverted.

**Audit the served HTML of production, per template.** Not the source, not the local build — the actual response.

```bash
curl -s https://example.com/collections/rings | grep -i 'name="robots"'
curl -sI https://example.com/collections/rings | grep -i 'x-robots-tag'
```

This check belongs in a regression test — hand it to `scribeo-testing`.

## Internal linking for discovery

Internal links are the primary discovery mechanism within a site, and they distribute crawl attention.

**Engineering requirements:**

- **Every important page is linked from at least one crawlable page.** A page in the sitemap but linked from nowhere is weakly discoverable — the sitemap is a hint, not a substitute for structure.
- **Important content is reachable in few clicks from an entry point.** Deeply buried pages are discovered later and less reliably.
- **No orphan pages** — reachable only by direct URL.
- **Link text describes the destination.** Google uses anchor text among its signals for understanding a page.
- **Pagination and archives are crawlable** — if "load more" is the only way to reach older items, those items are not discoverable. Provide real paginated URLs.

**The boundary:** *whether* a page deserves prominence in the navigation is a `scribeo-ux-engineering` decision. This skill states the discoverability requirement — "this page has no crawlable path to it" — and routes the IA decision.

### Breadcrumbs

Breadcrumbs aid both users and crawlers in understanding hierarchy, and can be marked up with `BreadcrumbList` structured data where they exist. **Do not add breadcrumbs to a flat site for SEO reasons** — that is an IA change for `scribeo-ux-engineering`, and marking up a hierarchy that does not exist misrepresents the page.

## Diagnostic sequence

When "this page isn't in search":

1. **Does it return a success status?** `curl -sI <url>`
2. **Is it disallowed in robots.txt?** Fetch and read the file.
3. **Does it emit `noindex`?** Check served HTML *and* response headers.
4. **Does a canonical point somewhere else?** If so, the engine is being told another URL is preferred.
5. **Is it linked from anywhere crawlable?**
6. **Is it in the sitemap** — and is the sitemap itself reachable?
7. **Does the content require JavaScript or interaction to appear?** See `javascript-rendering.md`.
8. **Is it a duplicate of another URL?** See `canonicals.md`.
9. **If all the above are correct:** the engineering is sound, and the rest is the search engine's decision. **Check Search Console; do not speculate.**

Steps 1–8 are this skill's. Step 9 is where honesty matters most.

## Environment leakage

Staging and development sites appearing in search is a real and embarrassing failure.

- Protect non-production environments with **authentication**, not robots.txt — Google is explicit that robots.txt is not a mechanism for keeping pages out of Search.
- Ensure production does not inherit staging's robots configuration, and that staging cannot inherit production's.
- Check for absolute URLs pointing at a staging host in canonicals, sitemaps and Open Graph tags.
