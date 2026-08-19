---
name: Read Advantage360 research from Advantage Solutions
description: >-
  Pull the Advantage Solutions Advantage360 shopper and market research reports, and
  the solutions catalog, from the corporate site's publicly readable content API.
api: openapi/advantage-solutions-youradv-content-openapi.yml
operations:
  - getWpV2Advantage360reports
  - getWpV2Advantage360reportsByid
  - getWpV2Advantage360category
  - getWpV2Solution
generated: '2026-08-13'
method: generated
source: openapi/advantage-solutions-youradv-content-openapi.yml
---

# Read Advantage360 research from Advantage Solutions

Advantage Solutions has no developer program and no API keys. Its corporate site
(`youradv.com`) runs WordPress and serves the standard REST API anonymously for
published content. That is the only machine-readable surface the company offers,
and it is where the Advantage360 research lives.

**Base URL:** `https://youradv.com/wp-json`
**Auth:** none for reads. Do not send credentials.

## 1. List the reports

`getWpV2Advantage360reports` — `GET /wp/v2/advantage360-reports`

Use `per_page` (max 100) and `page`. Read `X-WP-Total` and `X-WP-TotalPages` from the
response headers to size the crawl; there were **52** reports at last check. Follow
the `Link: <...>; rel="next"` header rather than guessing page numbers.

Keep payloads small with `_fields=id,slug,title,link,date,modified` and add
`_embed=true` only when you actually need the featured image or terms inline.

```
GET /wp/v2/advantage360-reports?per_page=100&_fields=id,slug,title,link,date,modified
```

## 2. Fetch one report

`getWpV2Advantage360reportsByid` — `GET /wp/v2/advantage360-reports/{id}`

`content.rendered` is HTML, not markdown. `context=view` (the default) is all an
anonymous caller may request; `context=edit` will return `401 rest_forbidden`.

## 3. Narrow by category

`getWpV2Advantage360category` — `GET /wp/v2/advantage360-category` lists the taxonomy.
Filter reports with `?advantage360-category=<term_id>`.

## 4. Read the solutions catalog

`getWpV2Solution` — `GET /wp/v2/solution` returns the five service lines the company
sells. This is the closest thing to a product catalog Advantage Solutions publishes.

## Rules

- **Errors.** The envelope is `{"code","message","data":{"status"}}` — not RFC 9457.
  `rest_no_route` (404) means the route is not enabled on this deployment;
  `rest_forbidden` (401) means you asked for something that needs a capability.
- **Not idempotent.** There is no idempotency key. Do not write to this API at all —
  writes need a WordPress Application Password issued by their web team.
- **No rate limits are published** and no rate-limit headers are returned. Cloudflare
  fronts the origin, so throttle yourself: stay serial, use `per_page=100`, and honour
  `Cache-Control: max-age=600` rather than re-fetching.
- **Do not trust `llms.txt`.** `https://youradv.com/llms.txt` is served, but every URL
  inside it points at the internal staging host `youradv-prod.local` and will not
  resolve. Use the sitemap at `https://youradv.com/sitemap_index.xml` or this API.
