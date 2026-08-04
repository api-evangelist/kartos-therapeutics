---
name: Read the Kartos Therapeutics leadership, board and advisor roster
description: >-
  Retrieve the structured executive, board-director and advisor records Kartos Therapeutics
  publishes, including job title, credentials and biography.
api: openapi/kartos-therapeutics-content-openapi.yml
operations: [listTeamMembers, getTeamMember, listTypes, getType]
method: generated
generated: '2026-08-04'
---

# Read the Kartos Therapeutics leadership, board and advisor roster

Kartos registers a `team` custom post type carrying the only genuinely structured data on this
surface: each person has Advanced Custom Fields for role, credentials and biography.

## Before you start

- Base URL: `https://kartosthera.com/wp-json`
- No authentication required.
- 15 records as of 2026-08-04, spanning the executive team, the board of directors and advisors.

## Steps

1. **Confirm the post type is registered** — `getType`

   ```
   GET /wp/v2/types/team
   ```

   Returns `rest_base: team`, `name: Team Members`, and an empty `taxonomies` array — there is no
   category separating executives from board members, so you must infer grouping from
   `acf.team-job-title`.

2. **List the roster** — `listTeamMembers`

   ```
   GET /wp/v2/team?per_page=100&_fields=id,slug,link,title,acf
   ```

   Each record returns:

   | Field | Meaning |
   |---|---|
   | `title.rendered` | Person's name |
   | `acf.team-job-title` | Role, e.g. `CEO`, `CMO`, `SVP, Translational Medicine`, or an external affiliation such as `Managing Partner at OrbiMed Advisors` for board and advisor entries |
   | `acf.team-credentials` | Post-nominals, e.g. `MD`, `PhD`, `MD, PhD`, `PhD, CFA` — empty string when none |
   | `acf.team-bio` | Biography as HTML |

3. **Separate employees from board and advisors.** There is no flag for this. The reliable signal
   is `acf.team-job-title`: an internal role (`CEO`, `CMO`, `CSO`, `COO`, `SVP, …`) means an
   operating executive; a title naming another firm (`Managing Partner at OrbiMed Advisors`,
   `President at Quogue Capital`, `Independent Board Director`) means a director or advisor.
   Treat this as inference, not as a field the provider asserts.

4. **Fetch a single record** — `getTeamMember`

   ```
   GET /wp/v2/team/{id}
   ```

## Cautions

- `acf.team-bio` is **HTML**, not plain text. Strip or render it; do not treat it as a string
  literal.
- The `wp/v2/users` collection is not the roster — it holds one record, `admin`, and no named
  people. Do not use it to build a people graph.
- One record still carries a `guid` on `kartoscorp.azurewebsites.net` from before the custom
  domain was in place. `guid` is an identifier, not a resolvable URL — use `link` instead.

## Error handling

Errors use the WordPress envelope `{code, message, data:{status}}`. `400 rest_invalid_param` for
`per_page` over 100; `404 rest_post_invalid_id` for an unknown id. Full list in
`errors/kartos-therapeutics-problem-types.yml`.
