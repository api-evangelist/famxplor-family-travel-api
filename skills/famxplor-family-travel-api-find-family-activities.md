---
name: Find family activities near a place
description: >-
  Use the Famxplor Family Travel API to find kid-tested activities within a radius of a
  coordinate, then pull the full record for the ones worth recommending — including what real
  families did there and what they enjoyed.
api: openapi/famxplor-family-travel-api-openapi.yml
operations:
  - nearest_activities_v1_nearest_activities_post
  - activity_details_v1_activities_details__activity_id__get
generated: '2026-09-07'
method: generated
source: openapi/famxplor-family-travel-api-openapi.yml
---

# Find family activities near a place

## Before you start

- Base URL: `https://api.famxplor.com`
- Auth: send `api-key: <key>` as a request **header** on every call. There is no anonymous
  access; a missing key returns **403** with `{"detail":"An API key must be passed as header"}`
  — note 403, not 401, and no `WWW-Authenticate` header.
- Language: add `Accept-Language: fr` (two-letter ISO 639) to get localized content. Default
  is `en`, and an unavailable language silently falls back to English.
- Ignore response fields you do not recognise; the provider adds properties over time.

## Step 1 — geocode first

Every operation on this API takes a coordinate. There is **no** search-by-city, by-country or
by-name operation, so you must resolve "Montpellier" to a lat/lon yourself before calling
Famxplor. Do not attempt a text query — the API has no text parameter.

## Step 2 — `nearest_activities_v1_nearest_activities_post`

```
POST /v1/nearest-activities
Content-Type: application/json
api-key: <key>

{"lat": 43.60834, "lon": 3.88015, "max_distance": 20000}
```

- `max_distance` is in **metres**, minimum 0, **maximum 100000**, default 1000. A request
  above 100000 fails validation with 422 — clamp before sending.
- The response is `{"activities": [...]}`, capped at **100** results. There is no pagination
  and no total count, so if a dense city returns 100 you are seeing a truncated set: shrink
  `max_distance` and issue several calls around different centres rather than asking for more.
- Each activity carries `id`, `title`, `url`, `img_url`, `lat`, `lon`, `tags[]`.

## Step 3 — `activity_details_v1_activities_details__activity_id__get`

```
GET /v1/activities/details/{activity_id}
api-key: <key>
```

Pass the `id` from step 2 as `{activity_id}`. **Watch the field rename**: the search response
calls it `id` and returns `title`; the detail response returns `activity_id` and `name` for
the same entity. Map them or you will drop the record.

The detail response is where the product's real value sits: `description`, `country`,
`region`, `city`, `type`, and `posts[]` — each post carrying `what_did` and `enjoyed`, written
by a family who actually went. Quote those rather than the generic description when you
recommend a place.

`posts` is nullable. A 404 with `{"detail": "..."}` means the id is unknown — do not retry it.

## Rules

- **Budget your calls.** Plans are metered on a MONTHLY call quota (200 / 1,000 / 5,000 by
  tier). There are no rate-limit response headers and no documented 429, so nothing at runtime
  tells you how much quota is left — count your own calls. A naive loop calling details on all
  100 search results burns half a Starter month in one query.
- Do not retry a 403 or a 404; both are terminal. Retry a 5xx at most twice with backoff.
- On 422 read `detail[].loc` to find the offending field — that array is the only structured
  error information this API returns.
- There is no idempotency key and none is needed: all four operations are queries and none
  changes state.
