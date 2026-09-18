# Ecommerce SEO

Load when working on a product catalogue — including Scribeo's jewellery projects.

**There are no universal rules for every store architecture.** The right handling of facets, pagination and variants depends on catalogue size, how customers browse, and what the business needs found. This file gives the decisions to make, not a template to apply.

**Verify current Google guidance for product structured data and any ecommerce feature before implementing.**

## Product URLs

- **Stable and readable.** `/products/rose-gold-solitaire-ring` over `/p?id=4471`.
- **Do not encode volatile attributes** — price, stock, campaign — into the path.
- **Decide variant handling deliberately.** Colour, size or metal variants can be: one URL with in-page selection; separate URLs per variant; or a canonical variant with the others canonicalising to it. Each is defensible; the choice depends on whether customers search for the variant specifically and whether variants have genuinely distinct content.
- **Products in multiple categories** should have **one canonical URL**, not one per category path. Category-scoped duplicates are a classic source of duplication.

## Product pages

**Content that must be in the served HTML** to be reliably discoverable: name, description, price, availability, images, and key specifications. If any of these are fetched client-side after hydration, they are less reliably available — see `javascript-rendering.md`.

**Product structured data** typically covers name, description, image, offers with price and availability, and identifiers where they exist. **Verify current required and recommended properties from Google's documentation before implementing** — this is a frequently-updated feature.

Two non-negotiables from `structured-data.md`, restated because ecommerce is where they are most often broken:

- **Markup must match what the page displays.** A price or availability in markup that differs from the visible page violates policy.
- **Never fabricate ratings or reviews.** Only mark up reviews the site genuinely hosts and shows.

**Price and availability must stay accurate.** Stale structured data — a price the page no longer shows, `InStock` on a sold-out product — is a correctness defect, not a minor one.

## Out-of-stock products

A business and content decision with technical options. **Route the decision; implement the choice.**

| Option | When | Technical note |
| --- | --- | --- |
| Keep the page, mark unavailable | Returning stock; the product still has search and link value | Availability must reflect reality in markup and on the page |
| Keep, with alternatives shown | Permanently gone, but relevant products exist | Avoid a dead end for the user |
| Redirect to a close equivalent | A genuine successor exists | Redirect to the equivalent, never to the homepage |
| 404 or 410 | Genuinely gone with no successor | 410 for deliberate permanent removal |

**Never mass-redirect discontinued products to the homepage.** The user asked for something specific.

## Category and collection pages

- **Crawlable navigation** to every category that should be found — real `<a href>` links, not script-driven menus.
- **Distinct content per category.** Near-identical category pages differing only in a heading are duplication.
- **Empty categories** should not render "no results" at a 200 status — that is a soft 404. See `urls-status-redirects.md`.

## Pagination

**Provide real, crawlable paginated URLs** for long listings. If "load more" is the only way to reach page 2 onward, those products are discoverable only through the sitemap or other links — a genuine gap.

Each paginated URL is a distinct page and generally should **canonicalise to itself**, not to page 1: page 3 is not a duplicate of page 1, and canonicalising it away can cost the discoverability of everything on it.

**Verify current Google guidance on pagination handling** before adopting a scheme — this area's recommendations have changed historically, and older advice circulates widely.

## Faceted navigation

The largest source of URL explosion in ecommerce. Filters combine multiplicatively, generating enormous numbers of near-duplicate URLs.

**Decide, per facet, which of three things it is:**

| Category | Handling |
| --- | --- |
| **Genuinely valuable as a landing page** — people search for it | Real URL, indexable, distinct content, self-canonical, in the sitemap |
| **Useful to users, not worth indexing** | Keep usable; canonical to the unfiltered page, or `noindex` |
| **Combinatorial noise** — multi-select and sort permutations | Not crawlable as links; excluded from sitemap |

A jewellery store might reasonably treat "rose gold engagement rings" as the first category and "price ascending, 2 filters selected, page 4" as the third.

**Do not leave this undecided.** The default — every combination reachable and indexable — produces thousands of thin near-duplicate URLs.

**Which facets are valuable is a business and content question.** State the technical consequences of each option and route the decision.

## Duplicate parameters

- **Tracking parameters** (`?utm_*`) should canonicalise to the clean URL.
- **Sort order** does not usually justify a separate indexable URL.
- **Session identifiers** should never appear in URLs.
- **Consistent parameter order** avoids the same selection producing multiple URL strings.

## Sitemap

Include **canonical, indexable product and category URLs**. Exclude filtered permutations, sort variants, out-of-stock products that now redirect, and anything carrying `noindex`. On large catalogues, split by type and use a sitemap index — see `robots-sitemap.md`.

## Images

Product imagery is a discovery surface of its own for a jewellery site — see `images-video.md`. Requirements: reachable at stable URLs · not blocked in robots.txt · real `<img>` elements rather than CSS backgrounds for product photography · alt text that describes the product (the accessibility requirement belongs to `scribeo-accessibility`).

## Checklist

- [ ] Product URLs stable, readable, one canonical per product
- [ ] Variant strategy decided and consistent
- [ ] Product content server-rendered
- [ ] Product structured data matches visible content; properties verified against current docs
- [ ] Price and availability accurate in markup
- [ ] No fabricated ratings or reviews
- [ ] Out-of-stock handling decided per case; no blanket homepage redirects
- [ ] Categories crawlable and distinct; no soft 404s on empty ones
- [ ] Real paginated URLs; pagination canonical strategy verified against current guidance
- [ ] Facets triaged into indexable / usable-not-indexed / not-crawlable
- [ ] Sitemap contains only canonical indexable URLs
- [ ] Product images crawlable at stable URLs
