# Next.js (App Router)

Load when implementing SEO in a Next.js project.

**APIs below were verified against the official Next.js documentation, version 16.3.5 (September 2026).** Next.js is **not installed in this environment** — a project pins its own. **Verify against the version a project actually uses before relying on any of this**, and never invent an API name.

Pages Router patterns are deliberately excluded; the current Scribeo stack is the App Router.

## Metadata: two forms

Export either a **`metadata` object** (static) or a **`generateMetadata` function** (dynamic), from `layout.js` or `page.js`.

```tsx
// app/page.tsx — static
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Handmade engagement rings — Scribeo Jewellery',
  description: '…',
}
```

```tsx
// app/products/[slug]/page.tsx — dynamic
import type { Metadata } from 'next'

export async function generateMetadata({ params }): Promise<Metadata> {
  const product = await getProduct((await params).slug)
  return { title: `${product.name} — Scribeo Jewellery`, description: product.summary }
}
```

**Documented constraints — all four matter:**

1. **Both are only supported in Server Components.** The docs state this explicitly: metadata must be resolved on the server before the page renders, so it can be included in the initial HTML.
2. **You cannot export both from the same route segment.**
3. **File-based metadata has higher priority** and will override the `metadata` object and `generateMetadata`.
4. Metadata can be added to `layout.js` and `page.js`; Next.js resolves it and creates the `<head>` tags.

**The Server Component constraint is the one that bites.** If a page is marked `"use client"`, it cannot export metadata at all. The documented pattern is to keep `page.tsx` a Server Component and move client logic into a separate component file. A page converted to a client component "to fix a hook error" silently loses its metadata — check for this when metadata mysteriously disappears.

## metadataBase

`metadataBase` sets a base URL prefix for metadata fields that require a fully qualified URL, letting fields in **that route segment and below** use relative paths which are composed into absolute URLs.

```tsx
// app/layout.tsx
export const metadata: Metadata = {
  metadataBase: new URL('https://example.com'),
  alternates: { canonical: '/' },
  openGraph: { images: '/og-image.png' },
}
```

Documented notes: it is **typically set in the root `app/layout.js`** to apply across all routes; it may contain a subdomain or a base path. When `generateMetadata` uses `'use cache'`, the return value must be serializable and `URL` instances are not supported by cache functions — return a string (for example `url.toString()`) in that case.

**Scribeo rule:** set `metadataBase` from an environment variable per deployment, so a preview build cannot emit production URLs — or production emit preview ones. Absolute-URL leakage between environments is a recurring canonical and Open Graph defect.

## Canonical and alternates

```tsx
export const metadata: Metadata = {
  metadataBase: new URL('https://example.com'),
  alternates: {
    canonical: '/collections/rings',
    languages: { 'en-GB': '/en-GB', 'de-DE': '/de-DE' },
  },
}
```

`alternates.canonical` emits the canonical link; `alternates.languages` emits language alternates. Documented sibling keys include `media` and `types`.

## Robots metadata

```tsx
export const metadata: Metadata = {
  robots: {
    index: true,
    follow: true,
    nocache: false,
    googleBot: {
      index: true,
      follow: true,
      noimageindex: false,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
}
```

## Open Graph

```tsx
export const metadata: Metadata = {
  openGraph: {
    title: '…', description: '…', url: '/collections/rings',
    images: '/og/rings.jpg', type: 'website',
  },
}
```

With `metadataBase` set, relative image and URL values resolve to absolute.

## sitemap file convention

`sitemap.(xml|js|ts)` in the root of `app`. A static `app/sitemap.xml` is supported for small sites; the code form exports a default function returning `MetadataRoute.Sitemap`.

```ts
// app/sitemap.ts
import type { MetadataRoute } from 'next'

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    { url: 'https://example.com', lastModified: new Date(), changeFrequency: 'yearly', priority: 1 },
    { url: 'https://example.com/about', lastModified: new Date(), changeFrequency: 'monthly', priority: 0.8 },
  ]
}
```

**Documented return type:**

```ts
type Sitemap = Array<{
  url: string
  lastModified?: string | Date
  changeFrequency?: 'always' | 'hourly' | 'daily' | 'weekly' | 'monthly' | 'yearly' | 'never'
  priority?: number
  alternates?: { languages?: Languages<string> }
  images?: string[]
  videos?: Videos[]
}>
```

`images` and `videos` produce image and video sitemap entries. For multiple sitemaps: nest `sitemap.(xml|js|ts)` in route segments, or use **`generateSitemaps`**, returning an array of objects with an `id` — generated sitemaps are served at `/…/sitemap/[id].xml`. In v16 the `id` is a promise resolving to a string.

`sitemap.js` is a Route Handler cached by default unless it uses a request-time API or dynamic config.

**Scribeo rule:** generate entries from real routes and real content-modification times. A hand-maintained list drifts; `lastModified: new Date()` at build time marks everything modified on every deploy, which is neither consistent nor verifiable — see `robots-sitemap.md`.

## robots file convention

`robots.(txt|js|ts)` in the root of `app`. Static form:

```txt
User-Agent: *
Allow: /
Disallow: /private/

Sitemap: https://example.com/sitemap.xml
```

Code form returns `MetadataRoute.Robots`:

```ts
// app/robots.ts
import type { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: '*', allow: '/', disallow: '/private/' },
    sitemap: 'https://example.com/sitemap.xml',
  }
}
```

**Documented shape:** `rules` accepts an object or an array of objects with `userAgent`, `allow`, `disallow`, `crawlDelay` and `other`; top-level `sitemap` and `host` are also supported. The `other` field (added in v16.3.0) passes non-standard per-agent directives through verbatim without validation.

**Scribeo rule:** drive the robots output from the deployment environment, so preview deployments cannot serve a production-permissive robots.txt — and protect non-production with authentication regardless.

## JSON-LD

Next.js has no dedicated structured-data API. The documented pattern is a script tag rendered from a Server Component:

```tsx
export default async function Page({ params }) {
  const product = await getProduct((await params).slug)
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    description: product.description,
  }
  return (
    <>
      <script type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }} />
      {/* … */}
    </>
  )
}
```

Render it from a **Server Component** so it is present in the initial HTML. Content requirements are in `structured-data.md` — including that the markup must represent content visible on the page.

## Server and client boundaries

- Metadata and JSON-LD belong on the **server** side of the boundary.
- Content that must be discoverable should be server-rendered rather than fetched client-side after hydration. See `javascript-rendering.md`.
- A `"use client"` boundary placed high in the tree pushes content client-side — a `scribeo-performance` concern as well as an SEO one.

## Audit checklist

- [ ] Every indexable route exports metadata (object or `generateMetadata`)
- [ ] No route needing metadata is a client component
- [ ] Both forms not exported from one segment
- [ ] `metadataBase` set from environment, in the root layout
- [ ] `alternates.canonical` correct per template
- [ ] `robots` metadata deliberate; no unintended `noindex` in production
- [ ] `sitemap` generated from real routes; `lastModified` meaningful
- [ ] `robots` output environment-aware
- [ ] JSON-LD rendered server-side and matching visible content
- [ ] Served HTML verified — not just the source
