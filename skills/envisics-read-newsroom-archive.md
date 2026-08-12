---
name: Read the Envisics newsroom archive
description: Retrieve, filter and dereference Envisics press releases, news items and technical shorts from the WordPress REST content API behind envisics.com.
api: openapi/envisics-posts-api-openapi.yml
operations: [listPosts, getPost, listCategories, listTags, getUser, getMediaItem]
generated: '2026-08-12'
method: generated
---

# Read the Envisics newsroom archive

Envisics publishes no developer program. The newsroom is nonetheless readable as structured JSON
through the WordPress REST content API on the corporate website. This skill is grounded entirely in
operations that exist in `openapi/envisics-posts-api-openapi.yml` and
`openapi/envisics-taxonomy-api-openapi.yml`.

**Before you start.** Envisics Ltd. entered administration on 2026-04-22
(`lifecycle/envisics-lifecycle.yml`). This surface carries no service commitment and may be
withdrawn. Cache what you read.

## Authentication

None. Every operation below is anonymous and read-only. Send no `Authorization` header — there is no
credential to send (`authentication/envisics-authentication.yml`).

Base URL: `https://envisics.com/wp-json`

## Steps

1. **Discover the classification vocabulary first.** Call `listCategories`. At capture the live
   terms were `news` (24 posts), `press` (8), `tech-shorts` (8), `blogdid-you-know` (8),
   `did-you-know` (6) and `white-papers` (1). Keep the returned `id` for each slug you care about —
   the posts collection filters by term id, not slug.
2. **List the posts you want.** Call `listPosts`. Useful parameters, all declared in the spec:
   - `categories` — array of term ids from step 1.
   - `search` — free-text.
   - `after` / `before` — ISO 8601 publication-date bounds.
   - `orderby` + `order` — default is `date` descending.
   - `per_page` — 1 to 100 (default 10). Anything outside that range returns HTTP 400
     `rest_invalid_param`.
   - `_fields` — restrict the payload, e.g. `_fields=id,slug,link,title,date,categories`.
3. **Page to completion.** Read `X-WP-Total` and `X-WP-TotalPages` from the response headers, or
   follow the `Link` header's `rel="next"`. Do not guess the page count. At capture the archive held
   34 published posts.
4. **Fetch a single item when you need the body.** Call `getPost` with the numeric `id`. The
   `content.rendered` field is HTML; strip tags before handing it to a model.
5. **Dereference the relations you need.** A post carries `author` (integer), `featured_media`
   (integer, `0` when unset), `categories[]` and `tags[]`. Resolve them with `getUser`,
   `getMediaItem`, `getCategory` and `getTag` — or avoid the extra round trips entirely by adding
   `_embed` to the `listPosts`/`getPost` call, which inlines them under `_embedded`.

## Conventions to respect

- **Pagination** is page-number based (`page`, `per_page`, `offset`); totals arrive in headers, not
  in the body (`conventions/envisics-conventions.yml`).
- **Errors** use the WordPress envelope `{code, message, data.status}` — *not* RFC 9457
  `application/problem+json`. Branch on `code`, e.g. `rest_post_invalid_id` on a 404
  (`errors/envisics-problem-types.yml`).
- **No rate-limit headers exist.** The API returns no `RateLimit-*` and no `Retry-After`
  (`rate-limits/envisics-rate-limits.yml`). Self-throttle: a few requests per second at most, and
  cache — collection responses carry `s-maxage=2592000`.
- **No idempotency contract exists**, and none is needed: every operation here is a GET.

## Do not

- Do not attempt `POST`/`PUT`/`DELETE`. The collections advertise `Allow: GET` anonymously; writes
  require WordPress application passwords issued to site operators.
- Do not call `/wp/v2/settings` — it returns HTTP 401 `rest_forbidden` and is out of scope.
- Do not treat the newsroom as current. The most recent post is dated 2025-06-25.
