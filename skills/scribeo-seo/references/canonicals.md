# Canonicals & International

Load when handling duplicate URL variants, canonical tags, or hreflang.

Google facts verified against Google Search Central (September 2026). Re-verify before quoting.

Both topics answer the same question — **which URL variant should represent this content?** — which is why they live together and why they interact.

## Canonical is a signal, not a command

Google describes `rel="canonical"` as **a strong signal** that the specified URL should become canonical — not a directive it must obey. Where no canonical is specified, Google *"will identify which version of the URL is objectively the best version to show to users."*

**Google may choose a different canonical than the one you declare.** That is documented behaviour, not a bug, and it is the correct answer when a client asks "why is Google showing the wrong URL?" — you can strengthen the signal; you cannot force the outcome.

### Signal strength, as documented

| Method | Strength |
| --- | --- |
| **Redirects** | The strongest signal |
| **`rel="canonical"`** (HTML element or HTTP header) | A strong signal |
| **Sitemap inclusion** | A weak signal |

The practical consequence: **if a URL should never be used, redirect it.** Canonical is for cases where both URLs must remain reachable.

## Implementation

```html
<link rel="canonical" href="https://example.com/collections/rings">
```

**Absolute URLs are required.** Google mandates absolute paths over relative ones for `rel="canonical"`, warning that relative paths *"can cause problems in the long run."*

**Self-referencing canonicals are recommended.** Google explicitly advises including a `rel="canonical"` on the canonical page itself.

**Scribeo rule:** be consistent, and be deliberate. "Always self-canonicalize everything" is a reasonable default for a simple site, but it is not an unconditional rule — on paginated sets, filtered views and syndicated content the correct canonical is a judgement about which URL should represent the content. Decide per template, and document the decision.

### Consistency requirements

Every canonical must agree with the rest of the site's signals. Mismatches weaken or contradict the hint:

- **One host and scheme.** Pick `https://example.com` or `https://www.example.com`, and make canonicals, internal links, sitemap entries, redirects and Open Graph URLs all agree.
- **Trailing-slash consistency.** `/rings` and `/rings/` are different URLs.
- **Case consistency.** Paths are case-sensitive on most servers.
- **No staging hostnames** in production canonicals — a frequent and damaging leak.
- **Canonical target must be indexable.** Pointing at a `noindex` or redirecting URL sends conflicting instructions.

## Canonical vs redirect vs noindex

Three different tools, frequently confused:

| Situation | Use |
| --- | --- |
| The URL should no longer be used at all | **Redirect** (strongest signal) |
| Both URLs must remain reachable; one should represent the content | **Canonical** |
| The page should not appear in Search at all | **`noindex`** |

Google specifically advises **against using `noindex` to manage canonical selection within a single site**, because it *"will completely block the page from Search."* Using `noindex` where you meant `canonical` removes the page rather than consolidating it.

## Common duplicate-variant sources

| Source | Handling |
| --- | --- |
| `http` and `https` | Redirect to `https` |
| `www` and non-`www` | Redirect to the chosen host |
| Trailing slash variants | Redirect to the chosen form |
| Tracking parameters (`?utm_*`) | Canonical to the clean URL |
| Sort and filter parameters | See `ecommerce.md` |
| Index variants (`/`, `/index.html`) | Redirect to the canonical form |
| Uppercase and lowercase paths | Redirect to lowercase |
| Print or AMP variants | Canonical to the primary version |

## Auditing

```bash
curl -s https://example.com/collections/rings | grep -i 'rel="canonical"'
```

- [ ] Present on every indexable page
- [ ] Absolute URL
- [ ] Correct host, scheme and trailing-slash form
- [ ] Target returns a success status and is indexable
- [ ] Consistent with sitemap entries and internal links
- [ ] Exactly one canonical element per page — multiples are ambiguous
- [ ] No staging hostname
- [ ] Regression contract handed to `scribeo-testing`

## hreflang — only when genuinely needed

hreflang signals language and regional variants of the same content.

**Do not implement hreflang on a single-language, single-region website.** It adds complexity and failure modes with no purpose. Most Scribeo sites do not need it.

**It is warranted when** the same content exists in genuinely different language versions, or in region-specific versions for different markets. It is **not** warranted for a single site that happens to have visitors abroad.

### Requirements when used

- **References must be reciprocal.** If A declares B as an alternate, B must declare A. One-sided declarations are commonly ignored.
- **Self-reference** is included in the set.
- **Use valid language and optional region codes** — a language code alone, or language plus region.
- **`x-default`** can designate the fallback for unmatched users.
- **hreflang and canonical must agree.** Each language version canonicalises to *itself*, not to another language. Canonicalising all versions to one URL tells the engine the others should not be used — which defeats the purpose.

**The commonest failures:** non-reciprocal references · canonical pointing at a different language version · invalid codes · hreflang pointing at redirecting or `noindex` URLs · implementing it for a site with no genuine language variants.

**Verify the current specifics against Google's documentation before implementing** — the details of supported formats and code validity are exactly the kind of thing worth re-checking rather than recalling.
