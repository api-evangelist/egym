---
name: Sync gym members into EGYM Cloud
description: Push and reconcile member accounts and memberships from a member-management system into EGYM using the MMS API V2, including the full-vs-partial update decision and the 409 conflict paths that this API requires you to handle.
api: openapi/egym-mms-api-v2-openapi.yml
base_url: https://mms.api.egym.com
operations: [publishAccount, listAccounts, retrieveAccount, retrieveAccountByMembershipId, getByEmail, updateAccount, partialUpdate, eraseAccountMembershipData, uploadImage, deleteImage]
generated: '2026-08-12'
method: generated
source: openapi/egym-mms-api-v2-openapi.yml + https://developer.egym.com/mms-api-v2/tutorials/member-account-example
---

# Sync gym members into EGYM Cloud

You are integrating a gym management system (MMS) with EGYM Cloud. Your job is to keep EGYM's
copy of the gym's members correct.

## Before you start

- **Auth**: every request carries `x-api-key: <key>`. The key is issued by EGYM **per gym
  location** and carries granular permissions. HTTPS only — plain HTTP fails.
- **The key is the scope.** You do not pass a gym id. EGYM resolves partner identity, gym
  location and permissions from the key. Do not try to act across gyms with one key.
- **You are a data processor for the gym.** Member names, birthdays and contract data are
  "Gym Data" under EGYM's published privacy framework. Workout and health data is "Workout
  Data" and belongs to the member — do not attempt to read it through this API.

## Create a member

1. `publishAccount` — `POST /api/v2/accounts`. Returns `201`.
2. If the member is a corporate-fitness (EGYM Wellpass) member, set
   `membershipType: CORPORATE_FITNESS` and supply the 9-digit `verificationTAN`, which links
   the gym account to the member's existing EGYM account. Without it the link does not form.

## Read a member

Pick the lookup route that matches the identifier you hold — do not list and filter:

- `retrieveAccount` — `GET /api/v2/accounts/{accountId}`
- `retrieveAccountByMembershipId` — `GET /api/v2/accounts/membership/{membershipId}`
- `getByEmail` — `GET /api/v2/accounts/email/{email}`
- `getByRfid` / `getByNfc` for credential-based lookup
- `listAccounts` — `GET /api/v2/accounts` for a sweep. Parameters: `offset` (default 0),
  `limit` (default 10, **max 100**), `currentGymOnly` (default true), and `fromTimestamp` /
  `toTimestamp` for a delta window. Use the timestamps for incremental sync; do not re-walk
  the whole roster.

The response envelope carries `total`, `hasNext` and a `nextAfterId` cursor. **`nextAfterId`
has no matching request parameter** — page with `offset` + `limit` and use `hasNext` to stop.

## Update a member

- `updateAccount` — `PUT /api/v2/accounts/{accountId}` replaces the whole record. Use it only
  when you own every field.
- `partialUpdate` — `PATCH /api/v2/accounts/{accountId}` changes only what you send. Prefer
  this when the gym's trainers also edit member data in EGYM tooling, otherwise a `PUT` will
  silently clobber their work.

Validation to respect: `membership.endOfContract` is optional, but if present it must not be
before `membership.startOfContract`.

## Handle conflicts — this is required, not optional

Every write can return `409` with a typed `errorCode`. Branch on it:

| errorCode | What happened | Do |
|---|---|---|
| `userEmailConflict` | Email already used in this gym or chain | Resolve to the existing account instead of creating a second one |
| `userEmailUsedByAnotherAccountConflict` | Email belongs to a different account | Do not force; reconcile identity |
| `membershipExists` | Membership conflict | Reconcile the existing membership first |
| `tanAssociatedWithAnotherAccount` | Wellpass TAN belongs to someone else | Re-verify the TAN with the member |
| `forbiddenAccountMerge` | Conflicting personal data with an existing chain-profile account | Escalate; do not retry |
| `generalConflict` | Resource already exists | Treat as duplicate create; fetch the existing one |

**There is no idempotency key on this API.** Retrying a failed `POST /api/v2/accounts` blind
can create a duplicate. Retry only after you have read back the resource, or after a `409`
has told you it already exists.

## Profile pictures

- `uploadImage` — `PUT /api/v2/accounts/{accountId}/images`
- `deleteImage` — `DELETE /api/v2/accounts/{accountId}/images`

## Delete

`eraseAccountMembershipData` — `DELETE /api/v2/accounts/{accountId}`. This fires the
`MEMBERSHIP_DELETE` webhook to every subscribed partner.

## Errors and pacing

- `401` — `x-api-key` header missing.
- `403 missingPermission` — the key is real but lacks the permission. Contact EGYM support;
  do not retry.
- `404` — read `metadata.entity` to learn which resource is missing (`ACCOUNT`,
  `CHAIN_PROFILE`, `MEMBERSHIP`, `TAN`, …) and branch on it rather than guessing.
- `429` — you exceeded ~6 requests/second. Read `X-RateLimit-Remaining` on every response and
  slow down **before** you hit the wall. There is no `Retry-After`; the bucket refills
  continuously, so simply staying under 6/s is enough.
- `ErrorDTO.requestId` is present on error bodies — capture it for support escalation.

Full error registry: `errors/egym-error-codes.yml`. Conventions: `conventions/egym-conventions.yml`.
