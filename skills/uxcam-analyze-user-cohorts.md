---
name: Analyze user cohorts in UXCam
description: Segment and aggregate UXCam users by custom properties, engagement, and
  device attributes using the Data Access API.
api: openapi/uxcam-data-access-openapi.yml
operations:
  - listUsers
  - analyzeUsers
generated: '2026-07-21'
method: generated
---

# Analyze user cohorts in UXCam

## Auth
Headers `X-App-Id` + `X-Api-Key` (UXCam Dashboard). Base URL
`https://api.uxcam.com/v2`.

## Steps
1. **Define the cohort** — call `listUsers` (`GET /user`) with a `filters` array,
   e.g. custom properties:
   `[{"attribute":"user_custom_property","operator":"in","property_name":"age_group","value":["20-29"]}]`
   or engagement: `[{"attribute":"user_gesture_count","operator":"greater_than","value":10}]`.
2. **Aggregate it** — call `analyzeUsers` (`GET /user/analytics`) with `group_by`
   (e.g. `[{"attribute":"device_country","max_group_number":10}]`) and `aggregation`
   (e.g. `[{"attribute":"user_session_duration","operator":"avg"}]`).
3. **Compare periods** — add `comparison=1` to compare the current window against
   the previous one.

## Rules
- Read-only GET surface; responses arrive in the `{success, data[]}` envelope.
- `page_size` default 50, max 1000; rate limits cap 500 records per request,
  500 requests/hour, 5 concurrent req/s.
- Users are keyed by `uxcamuserid`; sessions and events join to users via that id
  (see data-model/uxcam-data-model.yml).
