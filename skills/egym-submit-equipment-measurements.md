---
name: Authenticate a member at equipment and submit measurements
description: Use the EGYM Equipment Vendor API to log a member in at a machine via RFID or NFC wallet, read their enriched profile, and post body, cardio or flexibility measurements back to EGYM Cloud.
api: openapi/egym-equipment-vendor-standalone-openapi.yml
also: openapi/egym-equipment-vendor-server-openapi.yml
operations: [token, createToken, getUser, getUserDetails, createBodyMeasurement, createCardioMeasurement, createFlexibilityMeasurement, createWorkout, createWorkout_1, create, assignRfid, initialiseTest, updateTest, wellKnown]
generated: '2026-08-12'
method: generated
source: openapi/egym-equipment-vendor-standalone-openapi.yml + https://developer.egym.com/equipment/01-introduction-to-egym-equipment-vendor-apis
---

# Authenticate a member at equipment and submit measurements

You are an equipment vendor writing training results into EGYM Cloud. Pick your integration
shape first, because the two APIs differ:

| Shape | API | Use when |
|---|---|---|
| **Device-to-server** (standalone clients) | `openapi/egym-equipment-vendor-standalone-openapi.yml` — 31 operations | The machine itself talks to EGYM: cardio, strength and measurement devices |
| **Server-to-server** | `openapi/egym-equipment-vendor-server-openapi.yml` — 6 operations | Your own cloud relays asynchronously. **Measurement devices only.** |

> **Base URL warning.** The production host declared in the standalone spec,
> `partner-api.api.egym.com`, did not resolve (NXDOMAIN) when probed on 2026-08-12, and the
> server-to-server spec declares only a `.test.co.egym.coffee` test host. Get your real base
> URL from EGYM at `integrations@egym.com` — do not trust `servers[]` here.

## Step 1 — authenticate

`POST /api/v1/oauth/token` — `token` (standalone) / `createToken` (server-to-server).

This is **not** a standard OAuth grant. You select a login method with a `grantType` field:

| grantType | Payload | Status |
|---|---|---|
| `RFID` | `{"grantType":"RFID","rfid":"AB1020CD","rfidFormat":"MIFARE","machineName":"scale"}` | Standard and recommended |
| `NFC` | `{"grantType":"NFC","machineName":"scale","gymId":130,"payload":"<nfc-token>"}` | Apple/Google Wallet login, added in v1.1.1 |
| `ENCRYPTED_USER_ID` | — | Special use case; **requires prior alignment with EGYM** |
| `OBFUSCATED_USER_ID` | — | Special use case; requires prior alignment |
| `REFRESH_TOKEN` | — | Special use case; requires prior alignment |

The call is authenticated as the **partner** (API key on the server-to-server API, oauth2
client credentials on the standalone API). It returns an access token that represents **the
member**.

Every subsequent call carries `Authorization: Bearer <accessToken>`.

Verify EGYM-issued JWTs against `wellKnown` — `GET /api/v1/oauth/.well-known/jwks.json`.
Note the path: it is namespaced under `/api/v1`, not at the host root, so an RFC 8615
well-known probe will not find it.

## Step 2 — read the member

- Standalone: `getUser` — `GET /api/v1/users`. The token identifies the member; there is no
  id parameter.
- Server-to-server: `getUserDetails` — `GET /api/v1/gyms/{gymId}/users`.

On a first-ever login also check terms: `getTermsAndConditions`
(`GET /api/v1/resources/terms-and-conditions`) and `acceptTermsAndConditions`
(`PUT /api/v1/users/terms-and-conditions`). This is **Workout Data** under EGYM's privacy
framework — the member, not the gym, controls it, so consent is real and must be collected.

## Step 3 — submit

Measurements (identical operationIds on both APIs; all return `204`):

- `createBodyMeasurement` — `POST /api/v1/measurements/body`
- `createCardioMeasurement` — `POST /api/v1/measurements/cardio`
- `createFlexibilityMeasurement` — `POST /api/v1/measurements/flexibility`

Workouts (standalone only):

- `createWorkout` — `POST /api/v1/strength/workouts`
- `createWorkout_1` — `POST /api/v1/cardio/workouts`
- `create` — `POST /api/v1/open-exercises/workouts`

Cardio fitness test: `initialiseTest` (`POST /api/v1/cardio/tests`, returns `201`) then
`updateTest` (`PUT /api/v1/cardio/tests/{cardioTestId}`) as the test progresses.

## Step 4 — read back (standalone only)

`getUserBodyMeasurements`, `getUserCardioMeasurements`, `getUserFlexibilityMeasurements` for
the latest, and the `/history` variants for a series. `findAllTrainingPlans` and
`findAllTrainingPlans_1` return the member's strength and cardio plans so the machine can
prescribe the next set. `getUserActivityInformation` and `getWorkoutPoints` drive
member-facing progress UI.

## Fleet health

`machineHeartbeat` — `POST /api/v1/machines/heartbeat` — is partner-authenticated, not
user-authenticated. Send it on a schedule independent of member sessions.

## Retry discipline

Submissions return `204` with no body and **no idempotency key exists on this API**. A
timeout you retry can double-post a measurement into a member's health record. Retry only on
a transport error where you have evidence the request never landed; otherwise read back with
the corresponding `getUser*Measurements` call and reconcile.

Errors are `400 / 401 / 404 / 500` per operation. See `errors/egym-problem-types.yml`.
