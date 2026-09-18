# Metadata

Load when implementing or auditing titles, descriptions, robots meta or social metadata.

Google facts verified against Google Search Central (September 2026).

## Google does not have to use your title

This is the most important correction to common practice. **Google does not always use the HTML `<title>` element** for the title link in search results. Documented sources it may use include: the `<title>` content, the main visual title on the page, heading elements such as `<h1>`, `og:title`, large prominent text, page content, anchor text from links, and `WebSite` structured data.

Google may **rewrite** a title when it detects: incomplete titles · outdated titles no longer matching the page · inaccuracy · boilerplate repeated across pages · an unclear main heading · a language mismatch · redundant site-name repetition.

**So:** supply a good `<title>`, and never promise a client that the search result will display it verbatim. The correct framing is *"we supply the title; Google chooses what to display."*

### Length

Google states: *"While there's no limit on how long a `<title>` element can be, the title link is truncated in Google Search results as needed, typically to fit the device width."*

**There is no documented character limit.** Reject "titles must be under N characters" as folklore. What is true and useful: long titles are **truncated for display**, and the truncation point varies by device — so put the distinguishing words first. That is a **Scribeo recommendation** about display, not a documented requirement.

### Writing titles

Google's documented best practices: descriptive and concise text · avoid keyword stuffing · avoid boilerplate · brand concisely · ensure the main title stands out as the most prominent on the page.

**Scribeo pattern:** `Specific page subject — Brand`. Unique per page. Front-load the distinguishing term.

```html
<!-- Good: specific, front-loaded, concise brand -->
<title>Handmade engagement rings — Scribeo Jewellery</title>

<!-- Bad: boilerplate repeated site-wide, brand first, no specificity -->
<title>Scribeo Jewellery | Quality Jewellery | Rings, Necklaces, Bracelets, Gifts</title>
```

## Meta description

The meta description is **not** a documented ranking factor and should not be presented as one. Its function is to influence the snippet, and Google may generate a snippet from page content instead when that better matches the query.

**Scribeo recommendations** (not documented requirements): one per page, unique, accurately describing that page, written to be useful to a person deciding whether to click. Avoid keyword lists. As with titles, snippets are truncated for display — front-load the substance.

**A missing description is a minor issue, not a defect.** Google will generate a snippet. An *inaccurate* or duplicated-across-the-site description is worse than none.

## Robots meta

```html
<meta name="robots" content="noindex">
<meta name="robots" content="noindex, nofollow">
<meta name="googlebot" content="noindex">
```

And for non-HTML responses, the header form:

```
X-Robots-Tag: noindex
```

**`noindex` is Google's documented method for keeping a page out of Search** — but only works if the page is crawlable. See `robots-sitemap.md`.

**The production audit that matters most:** no route should emit `noindex` unintentionally. Check served output per template, and hand the check to `scribeo-testing` as a regression contract.

Pages that legitimately carry `noindex`: internal search results, thank-you and confirmation pages, cart and account pages, filtered duplicates, staging (though staging should be authenticated, not merely noindexed).

## Open Graph and social

Open Graph is **not** a Google ranking mechanism. It controls how a URL renders when shared on platforms that consume it — a real business requirement, and worth implementing correctly, but it belongs in the "social sharing" column, not the "SEO" one.

```html
<meta property="og:title" content="Handmade engagement rings">
<meta property="og:description" content="…">
<meta property="og:image" content="https://example.com/og/rings.jpg">
<meta property="og:url" content="https://example.com/collections/rings">
<meta property="og:type" content="website">
```

Requirements: **absolute URLs** for `og:image` and `og:url` · an image that exists and returns a success status · `og:url` agreeing with the canonical · no staging hostnames.

Note one real interaction with search: `og:title` is among the sources Google may draw on for the title link. An `og:title` that contradicts the `<title>` is a genuine inconsistency worth fixing.

## Other document metadata

- **`lang` on `<html>`** — required for accessibility (`scribeo-accessibility` owns the requirement) and it also declares document language. Set it correctly.
- **Viewport meta** — relevant here only because **disabling zoom is an accessibility failure**; `scribeo-accessibility` owns that rule.
- **Charset** — declare it.

## Auditing

```bash
curl -s https://example.com/page | grep -iE '<title>|name="description"|name="robots"|rel="canonical"|property="og:'
```

- [ ] `<title>` present, non-empty, unique per page, specific
- [ ] Description present and unique where used; accurate to the page
- [ ] No unintentional `noindex` anywhere in production
- [ ] Canonical present and absolute (see `canonicals.md`)
- [ ] Open Graph complete, absolute, with a reachable image
- [ ] `og:url` agrees with the canonical
- [ ] `html lang` set
- [ ] No staging hostnames in any metadata
- [ ] No duplicated boilerplate titles across templates
- [ ] Contracts handed to `scribeo-testing`

**Do not report metadata work as a ranking improvement.** It is an implementation improvement with documented effects on eligibility and display.
