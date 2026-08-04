---
name: Read the Kartos Therapeutics navtemadlin scientific record
description: >-
  Retrieve the congress abstracts, posters and publications Kartos Therapeutics publishes for
  navtemadlin (KRT-232), and resolve each one to its PDF in the media library.
api: openapi/kartos-therapeutics-content-openapi.yml
operations: [listPresentations, getPresentation, listMedia, getMediaItem, searchContent]
method: generated
generated: '2026-08-04'
---

# Read the Kartos Therapeutics navtemadlin scientific record

Kartos Therapeutics registers a WordPress custom post type, `presentation`, that holds its
published scientific record for navtemadlin. This is the highest-value collection on the surface
and it is readable anonymously.

## Before you start

- Base URL: `https://kartosthera.com/wp-json`
- No authentication. Do not send an `Authorization` header — every operation below is a public GET.
- No rate-limit headers are returned. Keep request volume low and cache what you fetch.

## Steps

1. **List the presentations** — `listPresentations`

   ```
   GET /wp/v2/presentation?per_page=100&_fields=id,slug,title,link,date,modified
   ```

   Read `X-WP-Total` for the count (5 as of 2026-08-04). Each item's `title.rendered` is the
   congress abstract or poster title; `link` is the human-readable page.

2. **Fetch one in full** — `getPresentation`

   ```
   GET /wp/v2/presentation/{id}
   ```

   Note: the `acf` field comes back as an **empty array** on this post type, so there are no
   structured metadata fields (congress name, date presented, authors) to read. The detail lives
   in the rendered page body and in `yoast_head_json`. Do not invent structured metadata that
   the API does not return.

3. **Find the matching PDF** — `listMedia`

   ```
   GET /wp/v2/media?mime_type=application/pdf&per_page=100&_fields=id,slug,title,source_url,date
   ```

   Six PDFs are in the library. Match a presentation to its PDF by slug/title similarity — the
   POIESIS Phase 3 poster, for example, is stored under a filename that echoes its title. This
   match is heuristic: the API exposes no `has_one` field binding a presentation to its file.

4. **Search across everything** — `searchContent`

   ```
   GET /wp/v2/search?search=navtemadlin&per_page=100
   ```

   Returns `{id, title, url, type, subtype}` across pages and presentations. A `navtemadlin` query
   returned 28 matches. Use `subtype=presentation` to constrain to the scientific record.

## Error handling

Errors use the WordPress envelope `{code, message, data:{status}}`, **not** RFC 9457 problem+json.

- `400 rest_invalid_param` — most often `per_page` above 100. Cap at 100 and paginate with `page`.
- `404 rest_post_invalid_id` — the id does not exist in the `presentation` collection.
- `404 rest_no_route` — you used a path that is not registered; fetch `/wp-json/` to see the routes.

See `errors/kartos-therapeutics-problem-types.yml` for the full observed catalog.

## What is not here

There is no blog, no press-release archive and no news feed — the `posts` collection is registered
but empty. Clinical trial detail (NCT numbers, sites, eligibility) is not in the API; it is on the
`/research/` page and in ClinicalTrials.gov.
