---
name: Harvest the Envisics company profile and brand assets
description: Build a structured profile of Envisics — products, technology, company history, corporate status and imagery — from the static pages, media library and SEO metadata exposed by envisics.com.
api: openapi/envisics-pages-api-openapi.yml
operations: [listPages, getPage, search, listMedia, getMediaItem, getSeoHead, getOEmbed]
generated: '2026-08-12'
method: generated
---

# Harvest the Envisics company profile and brand assets

Everything Envisics publishes about itself is reachable as JSON. This skill assembles a company
profile without scraping HTML. Every operation named here exists in `openapi/` — verified against the
specs, not invented.

**Material fact you must surface in any output.** Envisics Ltd. entered administration on
2026-04-22 under the Insolvency Act 1986, with Geoff Rowley and Simon Carvill-Biggs of FRP Advisory
Trading Limited appointed as Joint Administrators. The notice is published on the homepage and is
readable through this API. Any profile that omits it is misleading
(`lifecycle/envisics-lifecycle.yml`).

## Authentication

None. Anonymous read-only. Base URL: `https://envisics.com/wp-json`

## Steps

1. **Enumerate the page set.** Call `listPages` with
   `_fields=id,slug,link,title,date,modified&per_page=50`. At capture there were 11 published
   pages: `envisics` (home), `products`, `technology`, `company`, `newsroom`, `multi-media`,
   `careers`, `contact`, `terms-of-use`, `supplier-terms-conditions`, `privacy-policy`.
2. **Read the substantive pages.** Call `getPage` for `products`, `technology` and `company`. The
   narrative lives in `content.rendered` as HTML — strip tags before summarizing.
3. **Read the homepage for corporate status.** Call `getPage` with `id=2`. The Notice to Creditors
   sits at the top of `content.rendered`. Extract the administration date, the administrators and
   the creditor contact address verbatim; do not paraphrase a legal notice.
4. **Pull structured metadata rather than parsing HTML.** For any page URL call `getSeoHead`. The
   `json.schema` property returns the schema.org JSON-LD graph — organization, WebPage and
   BreadcrumbList nodes — which is a cleaner source for entity extraction than the rendered body.
5. **Collect brand and product imagery.** Call `listMedia` with `per_page=100` and page through
   using `X-WP-TotalPages` (219 attachments at capture). Each item carries `source_url`, `alt_text`,
   `mime_type` and `media_details.sizes` with every generated variant. Call `getMediaItem` when you
   need one attachment's full detail.
6. **Cross-check with search.** Call `search` with a term such as `waveguide` or `LYRIQ` to find
   every post and page mentioning it in one request; the response gives `id`, `title`, `url`, `type`
   and `subtype`. Use `subtype` to pick the right operation to dereference with.
7. **Generate embeds if you need them.** Call `getOEmbed` with a page URL for oEmbed 1.0 rich
   metadata and ready-made iframe HTML.

## Conventions to respect

- `_fields` keeps payloads small; `_embed` inlines `author` and `featured_media` and saves round
  trips (`conventions/envisics-conventions.yml`).
- Pagination totals are in `X-WP-Total` / `X-WP-TotalPages` headers.
- Errors are the WordPress envelope, not RFC 9457 (`errors/envisics-problem-types.yml`).
- No rate-limit headers are returned — self-throttle and cache
  (`rate-limits/envisics-rate-limits.yml`).

## Do not

- Do not expect a company-history timeline through the API. The `timeline-awesome` custom post type
  appears in the sitemap but is **not** registered on the REST surface
  (`data-model/envisics-data-model.yml`); it is HTML-only.
- Do not present Envisics as an operating going concern without the administration notice.
- Do not attempt writes, and do not call `/wp/v2/settings` (HTTP 401).
