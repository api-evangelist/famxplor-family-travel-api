---
name: Source real family testimony for a destination
description: >-
  Pull first-hand family travel writing near a location from the Famxplor API — both the
  blog-post index and the embedded "what we did / what we enjoyed" testimony attached to an
  activity — and cite it correctly.
api: openapi/famxplor-family-travel-api-openapi.yml
operations:
  - nearest_posts_v1_nearest_posts_post
  - nearest_activities_v1_nearest_activities_post
  - activity_details_v1_activities_details__activity_id__get
generated: '2026-09-07'
method: generated
source: openapi/famxplor-family-travel-api-openapi.yml
---

# Source real family testimony for a destination

Famxplor's differentiator is that a human family went and wrote about it. This skill gets you
to that writing without hallucinating any of it.

## Two different things called "post"

1. **`nearest_posts_v1_nearest_posts_post`** (`POST /v1/nearest-posts`, body
   `{lat, lon, max_distance}`) returns an index of family travel blog posts near a
   coordinate: `{id, title, url, img_url, lat, lon}`, capped at 100. It gives you the
   **link**, not the text.
2. **The `posts[]` array inside an activity detail** carries the actual testimony:
   `name`, `url`, `what_did`, `enjoyed`, `lang`, plus thumbnail/favicon fields.

## The trap

The `id` returned by `/v1/nearest-posts` is a 32-character hex digest. **There is no operation
that accepts it.** No `getPost` exists, and it does not match the slug-form `activity_id`
used everywhere else, so you cannot join a nearby post to an activity. If you need post text,
you must go through an activity:

```
POST /v1/nearest-activities  ->  take activity.id
GET  /v1/activities/details/{activity_id}  ->  read posts[].what_did / posts[].enjoyed
```

## Citing it

- Always attribute to `posts[].url` — this is third-party family blog writing, not Famxplor's
  own copy, and the `url` is the author's site.
- `posts[].lang` tells you the language the testimony was written in; it is often not English
  even when your request said so. Say which language you are translating from.
- Quote `what_did` and `enjoyed` as written. Do not paraphrase them into generic travel copy —
  the specificity is the entire product.
- If `posts` is `null` or absent for an activity, say there is no family write-up for it.
  Do not fill the gap from the `description` field, which is generated encyclopedic text.

## Rules

- Auth: `api-key` header. 403 on a missing key, 404 on an unknown activity id, 422 with
  `detail[].loc` on a bad field.
- `Accept-Language` steers the response language and falls back to English silently.
- Every call spends monthly quota (200–5,000 by plan) and no header reports the balance.
