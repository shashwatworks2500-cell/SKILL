# JavaScript & Rendering

Load when content depends on client-side JavaScript.

Google facts verified against Google Search Central (September 2026).

## The claim to avoid

**"Google can't index JavaScript" is false.** Google renders pages using **an evergreen version of Chromium** and executes JavaScript.

The real concern is different and more specific: **important search-visible content should be reliably available in a way current search rendering and indexing guidance supports.** Reliability, not capability, is the question.

## The three phases

Google documents processing in three sequential phases:

1. **Crawling** — Googlebot fetches URLs and parses the HTML for links.
2. **Rendering** — pages are queued for JavaScript execution in headless Chromium.
3. **Indexing** — the rendered HTML is indexed and links extracted.

**Rendering is queued and deferred.** Google states the page *"may stay on this queue for a few seconds, but it can take longer than that."* Pages returning a 200 status are queued for rendering unless excluded by a robots meta tag.

Two consequences:

- **Content in the initial HTML is available at crawl time.** Content requiring rendering is available later, after an unspecified delay.
- **Links discovered only after rendering are discovered later**, which slows the discovery of anything reachable only through client-side navigation.

For content that changes frequently, or for a new site being discovered, that delay matters. **Scribeo recommendation:** server-render anything that must be discoverable and understood — content, links, metadata and structured data.

## What actually prevents indexing

**Blocked resources.** Google states it *"won't render JavaScript from blocked files or blocked pages."* Disallowing a script bundle, stylesheet or data route in robots.txt can leave the rendered page incomplete. Check that robots.txt does not block resources the page needs.

**Non-success status codes.** Pages not returning 200 are not queued for rendering.

**Robots meta exclusion.** A `noindex` prevents the page being indexed regardless of rendering.

**Content requiring user interaction.** Google's JavaScript basics page does not enumerate this as a failure mode, so **do not assert that Google never sees click-revealed content.** What is defensible, and what Scribeo requires: content that only exists in the DOM after a user action is **not reliably available**, and should not be the only home of important information. Put it in the initial HTML and use interaction to reveal, not to create.

That distinction matters — revealing hidden content with a click is fine; *fetching and inserting* it on click is not, for content that must be discoverable.

## Client-side navigation

A single-page application that changes views without real URLs is a discoverability problem, not a rendering one.

Requirements:

- **Every discoverable view has a real, fetchable URL** returning content directly.
- **Navigation uses real `<a href>` links** — see `crawl-index.md`.
- **Use the History API** rather than fragment-based routing for distinct pages. Fragments are not distinct URLs for indexing purposes.
- **Status codes must be meaningful** — a client-rendered "not found" view at 200 is a soft 404; see `urls-status-redirects.md`.

## Hydration

Hydration mismatches are a rendering-correctness defect with SEO consequences: if server output differs from what the client produces, the indexed content may not match what users see.

- Do not vary server-rendered content by viewport, capability, `localStorage` or randomness — those differ between server and client. Apply them in an effect.
- Metadata and structured data belong on the server side of the boundary — see `nextjs.md`.
- Hydration errors are a `scribeo-testing` regression target (console-error assertions) and a `scribeo-visual-qa` observable.

## Checking rendered output

Compare what is served with what is rendered — the gap is the finding.

```bash
# What the crawler receives at crawl time
curl -s https://example.com/page > served.html
grep -c '<h1' served.html
```

Then load the same URL in the browser via Playwright MCP and inspect the rendered DOM. **Content present after rendering but absent from the served HTML depends on JavaScript** — which is not fatal, but is a reliability and timing consideration worth reporting.

Also check the console and network for errors that could interfere with rendering.

**What cannot be checked here:** how Google actually rendered and indexed the page. That requires the URL Inspection tool in the client's Search Console property. **Say so** rather than implying the rendered check answered it.

## Checklist

- [ ] Important content present in the served HTML
- [ ] Navigation uses real `<a href>` links
- [ ] Every discoverable view has a real fetchable URL
- [ ] No content that must be discoverable is fetched only on interaction
- [ ] robots.txt does not block scripts, styles or data the page needs to render
- [ ] Status codes meaningful, including for client-rendered not-found views
- [ ] Metadata and structured data server-rendered
- [ ] No hydration mismatch on indexable routes
- [ ] Served vs rendered compared, and the difference reported
- [ ] Rendering verdicts attributed to Search Console, not inferred
