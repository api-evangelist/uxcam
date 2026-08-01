---
name: Triage problem sessions in UXCam
description: Find sessions with rage gestures, crashes, or unusual behavior via the
  UXCam Data Access API so they can be reviewed and fixed.
api: openapi/uxcam-data-access-openapi.yml
operations:
  - listSessions
  - analyzeSessions
generated: '2026-07-21'
method: generated
---

# Triage problem sessions in UXCam

## Auth
Send both headers on every request (from the UXCam Dashboard):
`X-App-Id: <app id>` and `X-Api-Key: <data access api key>`.
Base URL: `https://api.uxcam.com/v2`.

## Steps
1. **Scope the damage** — call `analyzeSessions` (`GET /session/analytics`) with a
   `between_dates` filter on the review window and `group_by`
   `[{"attribute":"app_version"}]` to see which release the problems cluster in.
2. **Pull the bad sessions** — call `listSessions` (`GET /session`) with filters such
   as `[{"attribute":"session_rage_gesture_count","operator":"greater_than","value":0}]`
   or `[{"attribute":"session_crashed","operator":"equal","value":true}]`.
   Page with `page` / `page_size` (default 50, max 1000).
3. **Prioritize replays** — prefer records where `session_has_video` is true and sort
   by `session_rage_gesture_count`; the `session_id` links back to the replay in the
   UXCam dashboard.

## Rules
- The API is read-only GET; there is no idempotency-key contract to manage.
- Respect rate limits: 5 concurrent req/s, 500 req/hour (resets every 24h), max 500
  records per request — combine filters/groupings into one request rather than many.
- On `429`, back off and consolidate queries; on `401`, re-check the X-App-Id /
  X-Api-Key pair (see errors/uxcam-problem-types.yml).
- Dates in filters are ISO 8601 (`2024-01-01T00:00:00Z`).
