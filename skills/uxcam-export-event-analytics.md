---
name: Export event analytics from UXCam
description: Pull raw custom-event records and aggregated event metrics out of UXCam
  for warehouses, BI, or conversion tracking.
api: openapi/uxcam-data-access-openapi.yml
operations:
  - listEvents
  - analyzeEvents
generated: '2026-07-21'
method: generated
---

# Export event analytics from UXCam

## Auth
Headers `X-App-Id` + `X-Api-Key` (UXCam Dashboard). Base URL
`https://api.uxcam.com/v2`.

## Steps
1. **Pull raw events** — call `listEvents` (`GET /event`) filtered to the export
   window: `[{"attribute":"date_range","operator":"between_dates","value":{"lower":"2026-07-01","upper":"2026-07-21"}}]`,
   optionally narrowed by `event_name` or `event_screen_name`. Iterate `page`
   until the `data` array comes back short.
2. **Aggregate conversions** — call `analyzeEvents` (`GET /event/analytics`) with
   `group_by` `[{"attribute":"event_uploadedon_day"}]` to get `event_count`,
   `event_unique_user_count`, and `event_unique_session_count` per day; use
   `aggregation` for custom-property math (e.g. avg of a numeric event property).
3. **Join downstream** — each event carries `session_id` and `uxcamuserid`, so
   exports join cleanly to session and user pulls (see
   data-model/uxcam-data-model.yml).

## Rules
- Budget requests: 500/hour with a 24h reset and 500 records per request — batch
  the export into as few, well-filtered calls as possible.
- Handle `429` by pausing the export loop; never parallelize past 5 concurrent
  requests.
- Timestamps are ISO 8601; the response envelope is `{success, data[]}`.
