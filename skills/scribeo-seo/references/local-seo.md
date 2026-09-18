# Local SEO

Load when the site represents a business with a physical location or a defined service area.

**Scope:** the engineering-controlled parts of local search discoverability. **This is not a link-building, review-generation or listing-manipulation guide.** Never promise local rankings, map-pack appearance, or review outcomes.

**Verify current Google guidance before stating any requirement** — local search features and their documentation change.

## What engineering controls

| Controlled here | Not controlled here |
| --- | --- |
| Business identity present and consistent in the site's markup | Whether the business appears in local results |
| `LocalBusiness` or `Organization` structured data matching visible content | Map-pack position |
| Location pages existing, crawlable and distinct | Reviews, ratings, and their acquisition |
| Contact information reachable and machine-readable | Google Business Profile ranking |
| Correct, crawlable, indexable implementation | Competitive standing |

## Identity consistency (NAP)

**Name, address and phone** should be consistent wherever they appear — the visible page, the structured data, and the business's other public listings. Inconsistent representations make it harder for an engine to resolve the entity confidently.

**As an implementation concern**, this means:

- **One source of truth** in the codebase for business details. Duplicated hardcoded addresses drift.
- **Visible on the page** — in the footer, on a contact page, and on each location page. Structured data must not assert what the page does not show (see `structured-data.md`).
- **Consistent formatting** across pages.
- **Phone as a real `tel:` link** — a usability requirement (`scribeo-ux-engineering`) that also makes the number unambiguous.

**Not controlled here:** whether third-party directories carry matching details. That is an operational task for the client, and worth stating as a recommendation rather than performing.

## Structured data

Use **`LocalBusiness`** (or a more specific subtype where one genuinely fits) when the business has a physical location serving customers. Use **`Organization`** for site-wide identity where there is no customer-facing premises.

Values must match visible page content. Typically relevant properties include the business name, address, telephone, URL, opening hours and geo coordinates — **verify the current required and recommended properties from Google's documentation for the specific feature before implementing.**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Example Jewellers",
  "url": "https://example.com",
  "telephone": "+44 1632 960123",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "1 Example Street",
    "addressLocality": "Bristol",
    "postalCode": "BS1 1AA",
    "addressCountry": "GB"
  }
}
</script>
```

**Never fabricate opening hours, addresses, telephone numbers, ratings or reviews.** If the client has not supplied a value, omit the property. Inventing it breaches Google's structured-data policy and misrepresents a real business.

**Do not add `AggregateRating` or `Review` markup for reviews the site does not genuinely host and display.**

## Location pages

Warranted when a business has **genuinely distinct locations** with different addresses, hours, staff or services.

**Requirements when they exist:** one page per real location, each with a stable URL · genuinely distinct content — address, hours, contact, and something specific to that location · crawlable internal links from a locations index · self-referencing canonicals, since they are distinct pages, not duplicates · `LocalBusiness` data per page matching that page's visible content.

**Not warranted:** generating near-identical pages for towns the business does not have premises in. Those are thin, near-duplicate pages representing nothing real — and whether to create them is a business and content decision that this skill should flag rather than implement.

**The engineering finding to report** when asked for such pages: "these would be near-duplicate pages with no distinct content, which is a duplication and quality risk. If each location is real, the pages are justified; if not, they are not."

## Google Business Profile

A separate, client-owned product. Its content, verification and management are **not** website engineering, and this skill does not manage it.

**The relationship that matters here:** details on the website should be consistent with the profile, and the profile normally links to the site. Keeping them consistent is the engineering-adjacent part. **Never promise profile visibility or ranking.**

## Practical checklist

- [ ] Business identity visible on the site and consistent across pages
- [ ] Single source of truth for business details in the codebase
- [ ] `LocalBusiness` or `Organization` markup matching visible content
- [ ] No fabricated hours, addresses, numbers, ratings or reviews
- [ ] Phone as a `tel:` link
- [ ] Location pages only where locations are real and distinct
- [ ] Each location page crawlable, canonical to itself, and genuinely distinct
- [ ] No promises about local rankings or map results
