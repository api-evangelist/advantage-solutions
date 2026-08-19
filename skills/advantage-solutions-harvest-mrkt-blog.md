---
name: Harvest the MRKT blog from Advantage Solutions
description: >-
  Crawl MRKT, the Advantage Solutions publication, from either of the two WordPress
  hosts that serve it, and resolve bylines and taxonomy without over-fetching.
api: openapi/advantage-solutions-mrktblog-content-openapi.yml
operations:
  - getWpV2Posts
  - getWpV2PostsByid
  - getWpV2Categories
  - getWpV2Mrktblogposts   # served by the youradv.com spec
  - getWpV2Search
generated: '2026-08-13'
method: generated
source: >-
  openapi/advantage-solutions-mrktblog-content-openapi.yml,
  openapi/advantage-solutions-youradv-content-openapi.yml
---

# Harvest the MRKT blog from Advantage Solutions

MRKT is published twice. Pick the right host before you start.

| Surface | Base URL | Collection | Count at last check |
|---|---|---|---|
| MRKT standalone site | `https://mrktblog.com/wp-json` | `/wp/v2/posts` | 169 |
| Corporate site mirror | `https://youradv.com/wp-json` | `/wp/v2/mrktblog-posts` | 279 |

The corporate site carries the larger archive under a custom post type. Both are
anonymous reads; neither is authoritative over the other, so if you need everything,
harvest `youradv.com` and use `mrktblog.com` only to fill gaps.

## 1. Page the archive

`getWpV2Mrktblogposts` — `GET https://youradv.com/wp-json/wp/v2/mrktblog-posts`
`getWpV2Posts` — `GET https://mrktblog.com/wp-json/wp/v2/posts`

Both accept `page`, `per_page` (max 100), `search`, `after` / `before`,
`modified_after` / `modified_before`, `order`, `orderby`. For an incremental pull,
use `modified_after=<ISO8601>` and `orderby=modified` — that is the only reliable
change signal, because there is no changelog and no webhooks.

## 2. Resolve bylines and terms in one call

Add `_embed=true` to inline the author, featured media and terms. On `youradv.com`
the byline is the custom `blog-author` type — `/wp/v2/users` is **not routed** there
(it returns `404 rest_no_route`), so do not try to dereference `author` against it.

## 3. Taxonomy

`getWpV2Categories` on `mrktblog.com`; `/wp/v2/mrktblog-category` on `youradv.com`.

## 4. Search instead of crawling

`getWpV2Search` — `GET /wp/v2/search?search=<term>` returns a lightweight cross-type
result set. Use it when you want one article, not the archive.

## Rules

- **Feeds are the cheap alternative.** `https://mrktblog.com/feed/` returns RSS and is
  already wired into this repo's `blogs/` pull. Use the API when you need structured
  fields (taxonomy ids, modified timestamps, featured media), the feed when you just
  want the latest posts.
- **No rate limits are documented and no rate-limit headers come back.** Stay serial.
- **No events.** There is no AsyncAPI, no webhook, no push. Polling `modified_after`
  is the only way to detect change.
- **Errors** use `{"code","message","data":{"status"}}`. Treat `rest_no_route` as
  "this deployment does not enable that route", not as a transient failure.
