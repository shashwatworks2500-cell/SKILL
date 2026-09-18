# Structured Data

Load when implementing or auditing JSON-LD and Schema.org markup.

Google facts verified against Google Search Central (September 2026). Schema.org is the vocabulary; Google's documentation governs search-feature eligibility.

## Three things that are not the same

| | Means |
| --- | --- |
| **Schema.org vocabulary** | A type or property exists in the vocabulary |
| **Google rich-result eligibility** | Google documents support for that feature, and the required properties are present |
| **A rich result appearing** | Google decided to show it, for that query, that user, that device |

**Google states directly: it does not guarantee that structured data will show up in search results, even if the page is marked up correctly.** Appearance depends on variables including search history, location and device type.

**So:** implement to become *eligible*. **Never promise a rich result.** "This markup makes the page eligible for the documented product rich result" is honest; "this will show star ratings in Google" is not.

## Do not mark up what is not there

Google's policy is explicit: do not mark up **content that is not visible to readers of the page**, and do not use structured data to deceive or mislead.

This makes one rule absolute: **structured data describes what the page actually shows.** Not what you wish it showed, not data held elsewhere in the system. A `Product` with a price the page does not display, a `Review` the site does not host, an `Organization` address that appears nowhere — all violate the policy and risk a manual action.

**Never fabricate ratings, reviews, prices, availability or business details in markup.** This is the same rule the other Scribeo skills apply to invented social proof, and here it also breaches a documented policy.

## Format

Google supports **JSON-LD, Microdata and RDFa**, and **recommends JSON-LD**. Use JSON-LD.

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Rose gold solitaire ring",
  "description": "…",
  "image": "https://example.com/products/rose-gold-solitaire.jpg"
}
</script>
```

Render it server-side so it is present in the initial HTML — see `nextjs.md`.

## Choosing a type

**Only use a type that genuinely matches the page.** Schema.org contains hundreds of types; the existence of one is not a reason to use it.

Types commonly appropriate on Scribeo builds — **each only where it genuinely applies**:

| Type | Use when |
| --- | --- |
| `Organization` | Site-wide identity for the business |
| `LocalBusiness` (or a subtype) | The business has a physical location serving customers. See `local-seo.md` |
| `WebSite` | Site-level; among the sources Google may use for the title link |
| `BreadcrumbList` | Breadcrumbs **that exist on the page** |
| `Product` | A product page. See `ecommerce.md` |
| `Article` / `BlogPosting` | Editorial content |
| `FAQPage` | A page with genuine question-and-answer content |
| `Person` | A real individual the page is about |
| `Event` | A real event |

**Do not stack types speculatively.** Three loosely-applicable types are worse than one accurate one, and each carries its own policy obligations.

**Before implementing any type, check Google's current documentation for that feature** — required versus recommended properties, and whether the feature is still supported. Supported features change; this is exactly the area where cached knowledge goes stale.

## Required vs recommended properties

Google documents required and recommended properties per feature. **Missing a required property means the markup is not eligible** for that rich result, regardless of how complete the rest is.

**Verify current requirements from Google's documentation for the specific feature** rather than recalling them — and prefer completing required properties accurately over adding recommended ones speculatively.

## Entity consistency

Where the same entity appears in several places — business name, address, phone, URL — the values should agree across structured data, visible page content, and other public sources. Inconsistency weakens the engine's ability to resolve the entity confidently.

Use `@id` to link related entities within a page's markup where appropriate, rather than repeating slightly different descriptions of the same thing.

## Validation

**Available here:** JSON syntax validity, presence of `@context` and `@type`, and whether the values match visible content.

```bash
curl -s https://example.com/product/x \
  | grep -o '<script type="application/ld+json">.*</script>' \
  | sed 's/<[^>]*>//g' | python3 -m json.tool > /dev/null && echo "valid JSON"
```

**Not available here:** rich-result eligibility. Google's Rich Results Test and the Search Console enhancement reports are the authoritative checks, and neither is reachable from this environment. **Say so** rather than implying eligibility was verified.

Syntax validity is a legitimate regression contract for `scribeo-testing`; eligibility is not.

## Audit checklist

- [ ] JSON-LD used; parses as valid JSON
- [ ] `@context` is `https://schema.org`, `@type` present
- [ ] Type genuinely matches the page
- [ ] Every value corresponds to content visible on the page
- [ ] No fabricated ratings, reviews, prices or business details
- [ ] Required properties for the intended feature present — verified against current Google docs
- [ ] Entity values consistent across the site
- [ ] Rendered server-side, present in initial HTML
- [ ] Eligibility described as eligibility, never as a guaranteed result
- [ ] Syntax contract handed to `scribeo-testing`
