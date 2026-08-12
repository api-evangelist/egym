---
name: Export EGYM workout, measurement and analytics data
description: Pull gym analytics, Smart Strength workouts and assessments, and body and flexibility measurements out of EGYM using the Data Hub synchronous and asynchronous export surfaces, honouring member opt-outs.
api: openapi/egym-data-hub-openapi.yml
also: openapi/egym-data-export-openapi.yml
base_url: https://analytics.api.egym.com
operations: [getAnalyticsExport, getSmartStrengthWorkoutsExport, getSmartStrengthAssessmentsExport, getFlexibilityMeasurementsExport, getBodyMeasurementsExport, getOptedOutUsersExport, createAsyncExportJob, getAsyncExportJobStatus, getWorkouts, getStrengthMeasurements, getCardioMeasurements, getBodyMeasurement]
generated: '2026-08-12'
method: generated
source: openapi/egym-data-hub-openapi.yml + https://developer.egym.com/data-hub/authentication
---

# Export EGYM workout, measurement and analytics data

Two different export surfaces exist. Choose deliberately.

| Surface | Spec | Shape | Maturity |
|---|---|---|---|
| **Data Hub API** | `openapi/egym-data-hub-openapi.yml` | Whole-location exports over a date range | **Pilot** — Enterprise Pack customers only; contracts may change |
| **Data Export API** | `openapi/egym-data-export-openapi.yml` | Per-account reads under `/api/v2/accounts/{accountId}/…` | **Alpha preview** |

The Data Export API is worth knowing about because it has **no page in EGYM's documentation
navigation** — it appears only in EGYM's AI-agent instruction catalog and in the response of
the documentation MCP server's `get-full-api-description` tool.

## Auth

`x-api-key: <api-key>`, HTTPS only.

**You do not pass a gym id.** EGYM resolves partner identity, gym location name, gym legacy
id and the key's permissions from the key itself, and responses contain only data for that
location. One key, one location — plan your rotation and storage accordingly.

- `401` — header missing, malformed or invalid.
- `403` — key valid but not authorized for this resource.

## Honour opt-outs first

Call `getOptedOutUsersExport` — `GET /api/v1/data/opted-out-users` — **before** you process
an export, and suppress those members downstream.

Under EGYM's published privacy framework, workout and measurement data is "Workout Data":
managed on behalf of the member, who chooses who to share it with. This endpoint is the
mechanism that makes that choice enforceable in your pipeline. Skipping it is a compliance
failure, not a performance optimisation.

## Synchronous exports

All take `startDate` and `endDate`:

- `getAnalyticsExport` — `GET /api/v1/data/analytics`
- `getSmartStrengthWorkoutsExport` — `GET /api/v1/data/smart-strength-workouts`
- `getSmartStrengthAssessmentsExport` — `GET /api/v1/data/smart-strength-assessments`
- `getFlexibilityMeasurementsExport` — `GET /api/v1/data/flexibility-measurements`
- `getBodyMeasurementsExport` — `GET /api/v1/data/body-measurements`

Example from the docs:

```bash
curl --request GET \
  'https://analytics.api.egym.com/api/v1/data/smart-strength-workouts?startDate=2026-06-01&endDate=2026-06-15' \
  --header 'x-api-key: <api-key>'
```

There is **no pagination** on these — the date range is your only lever. Keep windows narrow.

## Asynchronous exports — use these for large ranges

1. `createAsyncExportJob` — `POST /api/v1/data/async`, returns `201` with a job id:

   ```json
   { "startDate": "2026-01-01", "endDate": "2026-03-31",
     "exportType": "SMART_STRENGTH_WORKOUTS", "fileType": "JSONL" }
   ```

2. `getAsyncExportJobStatus` — `GET /api/v1/data/async/jobs/{jobId}` — poll until complete.

The async surface exists precisely because the synchronous endpoints have no paging. If a
quarter's worth of workouts is what you want, do not loop synchronous calls — create a job.

## Per-account reads (Data Export API)

Against `https://mms.api.egym.com`, with the same `x-api-key`:

- `getWorkouts` — `GET /api/v2/accounts/{accountId}/workouts`
- `getStrengthMeasurements` — `GET /api/v2/accounts/{accountId}/measurements/strength`
- `getCardioMeasurements` — `GET /api/v2/accounts/{accountId}/measurements/cardio`
- `getBodyMeasurement` — `GET /api/v2/accounts/{accountId}/measurements/body`

Use these to answer "what did this member do", not to reconstruct a location-wide dataset one
account at a time — that is what Data Hub is for, and the MMS limiter is 6 requests/second.

## Operating notes

- No rate limit is published for the Data Hub or Data Export APIs. Absence of a documented
  limit is not absence of a limit; back off on `429` or `5xx`.
- Both surfaces are pre-GA. Pin the shape you parse, validate on ingest, and expect breaking
  changes without a change log — neither API publishes one.

See `plans/egym-plans-pricing.yml` for the Enterprise Pack entitlement gate.
