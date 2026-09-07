---
name: Plan a family route between stops
description: >-
  Build a family-friendly itinerary: find kid-tested activities near each stop with the
  Famxplor API, then cost the drive between them with the travel-time operation.
api: openapi/famxplor-family-travel-api-openapi.yml
operations:
  - nearest_activities_v1_nearest_activities_post
  - travel_time_v1_travel_time_post
generated: '2026-09-07'
method: generated
source: openapi/famxplor-family-travel-api-openapi.yml
---

# Plan a family route between stops

## Before you start

Same auth as every Famxplor call: `api-key: <key>` in the request header against
`https://api.famxplor.com`. Coordinates in, coordinates out — geocode place names yourself.

## Step 1 — gather candidate stops

For each place on the route, call `nearest_activities_v1_nearest_activities_post` with that
coordinate and a `max_distance` you can defend as "worth a detour" (metres, ≤ 100000). Keep
`lat`/`lon` from each activity you shortlist — you need them in step 2.

## Step 2 — `travel_time_v1_travel_time_post`

```
POST /v1/travel-time
Content-Type: application/json
api-key: <key>

{"locations": [
  {"lat": 48.85341, "lon": 2.3488},
  {"lat": 49.18433, "lon": -0.36118}
]}
```

- `locations` needs **at least two** entries. The first is the origin, the last is the
  destination, and everything between is a waypoint in order.
- `lat` must be within -90..90 and `lon` within -180..180, or the call fails with 422.
- The response is a single leg total: `{"time": <seconds>, "length_mi": …, "length_km": …}`.
  There is **no per-leg breakdown**. If you need the drive time between each consecutive pair,
  call the operation once per pair — and remember each call spends monthly quota.

## Step 3 — assemble

Convert `time` from seconds before showing it to a person. Order stops by the
travel-time totals you measured, not by straight-line distance from the search results —
`max_distance` is a radius, not a road distance, and the two diverge badly around water and
mountains.

## Rules

- One `/v1/travel-time` call per pair, so an N-stop route costs N-1 calls on top of the N
  search calls. On the Starter plan (200 calls/month) a 5-stop route is ~9 calls; plan for it.
- No pagination, no 429 contract, no rate-limit headers — see
  `conventions/famxplor-family-travel-api-conventions.yml`.
- Read-only surface: nothing you call here can be undone because nothing you call here
  changes anything.
