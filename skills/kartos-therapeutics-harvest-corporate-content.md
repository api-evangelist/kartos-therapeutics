---
name: Harvest Kartos Therapeutics corporate pages, media and discovery metadata
description: >-
  Enumerate the published pages, the media library and the registered types of kartosthera.com
  over the WordPress REST API, and get an embeddable representation of any page.
api: openapi/kartos-therapeutics-content-openapi.yml
operations: [getRouteIndex, listPages, getPage, listMedia, listTypes, listTaxonomies, listStatuses, getOembed]
method: generated
generated: '2026-08-04'
---

# Harvest Kartos Therapeutics corporate pages, media and discovery metadata

## Before you start

- Base URL: `https://kartosthera.com/wp-json`
- No authentication required for any step below.
- The site advertises this API on every HTML response with
  `Link: <https://kartosthera.com/wp-json/>; rel="https://api.w.org/"`.

## Steps

1. **Start at the route index** — `getRouteIndex`

   ```
   GET /
   ```

   Returns the site's own description of the API: name, home URL, the six registered namespaces
   (`oembed/1.0`, `yoast/v1`, `wp/v2`, `wp-site-health/v1`, `wp-block-editor/v1`,
   `wp-abilities/v1`), 183 routes with their allowed methods and arguments, and the
   `authentication` block. Always read this first — it is the only discovery document the site
   publishes (`/.well-known/*`, `/robots.txt` and `/llms.txt` all 404).

2. **List the pages** — `listPages`

   ```
   GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,modified
   ```

   Eight pages: `home`, `science`, `research`, `presentations`, `advocacy`, `about`, `contact`,
   `privacy-policy`. Use `_fields` to keep responses small — the full page objects run to ~75 KB
   for the collection because `content.rendered` carries the whole page builder markup.

3. **Fetch full page content** — `getPage`

   ```
   GET /wp/v2/pages/{id}
   ```

   `content.rendered` is HTML produced by a page builder, so it contains layout markup as well as
   prose. Extract text rather than treating it as clean content.

4. **Enumerate the media library** — `listMedia`

   ```
   GET /wp/v2/media?per_page=100&_fields=id,slug,title,mime_type,source_url,alt_text
   ```

   62 items: 21 `image/svg+xml`, 18 `image/png`, 17 `image/jpeg`, 6 `application/pdf`. The SVG and
   PNG set includes investor and partner marks and the scientific figures used on `/science/`.

5. **Read the type and taxonomy registry** — `listTypes`, `listTaxonomies`, `listStatuses`

   ```
   GET /wp/v2/types
   GET /wp/v2/taxonomies
   GET /wp/v2/statuses
   ```

   13 post types, of which two are Kartos-specific (`presentation`, `team`); 4 taxonomies, none
   attached to those two types; statuses `publish` and `acf-disabled`.

6. **Get an embeddable representation** — `getOembed`

   ```
   GET /oembed/1.0/embed?url=https://kartosthera.com/science/
   ```

   Returns oEmbed 1.0 with `provider_name`, `title`, and embeddable `html`. Omitting `url`
   returns `400 rest_missing_callback_param`; a URL outside kartosthera.com returns
   `404 oembed_invalid_url`.

## Pagination contract

`page` (1-based) and `per_page` (default 10, max 100). Read the total from the `X-WP-Total`
response header and the page count from `X-WP-TotalPages`; `Link` carries RFC 8288 `next`/`prev`.
All three are exposed cross-origin via `Access-Control-Expose-Headers`.

## Do not attempt

These return `401` anonymously and there is no public credential that would change that — do not
retry, and do not try to authenticate:

`/wp/v2/settings`, `/wp/v2/menus`, `/wp/v2/menu-items`, `/wp/v2/menu-locations`, `/wp/v2/themes`,
`/wp/v2/plugins`, `/wp/v2/block-types`, `/wp/v2/sidebars`, `/wp/v2/templates`,
`/wp/v2/font-collections`, `/wp/v2/icons`, `/wp-block-editor/v1/url-details`,
`/wp-site-health/v1/*`, `/yoast/v1/*`, and the whole `wp-abilities/v1` capability registry beyond
its namespace index.

All write methods (POST/PUT/PATCH/DELETE) are registered but require credentials. This is a
read-only surface for any third party.
