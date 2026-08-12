---
name: Onboard and admit an EGYM Wellpass corporate-fitness member
description: Link a Wellpass (corporate fitness) member to their existing EGYM account with a verification TAN, then validate their admission at every gym entry and translate EGYM's corporate-fitness rejection codes into the right door decision.
api: openapi/egym-mms-api-v2-openapi.yml
base_url: https://mms.api.egym.com
operations: [publishAccount, getCorporateFitnessMemberAccounts, checkAdmission, checkin, checkout, retrieveAccountByMembershipId, getByRfid, getByNfc]
generated: '2026-08-12'
method: generated
source: openapi/egym-mms-api-v2-openapi.yml + https://developer.egym.com/mms-api-v2/tutorials/wellpass-integration
---

# Onboard and admit an EGYM Wellpass member

EGYM Wellpass (formerly Qualitrain) is a corporate fitness benefit: an employer buys access
and the employee uses a network of partner gyms. Your gym is one of those venues. Wellpass
members **already have an EGYM account** — your job is to link yours to theirs, then check
them in correctly every visit.

## Auth

`x-api-key: <key>`, issued per gym location. HTTPS only.

## Step 1 — link the member on first visit

The member gives you a **TAN**: a 9-digit code from the Wellpass mobile app or account
settings that uniquely identifies them.

Call `publishAccount` — `POST /api/v2/accounts` — with:

- `membership.membershipType: CORPORATE_FITNESS`
- `membership.verificationTAN`: the 9-digit TAN. **Required** for this membership type.
- `membership.startOfContract`: still required; use the member's start date at your gym.

Store the returned `accountId` — it is the join key for every later call, and it is the only
field in EGYM's webhook payloads.

The TAN is short-lived and used only at account creation. If it fails:

- `404` with `metadata.entity: TAN` — the TAN was not found. Ask the member to re-read it.
- `409 tanAssociatedWithAnotherAccount` — the TAN belongs to a different account. Do not
  retry; re-verify with the member.

## Step 2 — read the membership window

`retrieveAccountByMembershipId` (or `retrieveAccount`) returns a **read-only**
`membership.corporateFitness` object with `startTimestamp` and `endTimestamp`. You do not set
these — EGYM owns them, because the employer owns the entitlement, not your gym.

To sweep the whole cohort at your venue, use `getCorporateFitnessMemberAccounts` —
`GET /api/v2/accounts/corporate-fitness`.

## Step 3 — validate every entry

**Do not open the door on your local record.** Call `checkAdmission` —
`POST /api/v2/accounts/{accountId}/admissions` — on every visit. EGYM is the authority on
whether this member is currently entitled at this venue.

Resolve the `accountId` from whatever the member presents at the door: `getByRfid`
(`GET /api/v2/accounts/rfid/{rfid}`) or `getByNfc` (`GET /api/v2/accounts/nfc/{nfc}`) for a
wallet pass.

A `403` from `checkAdmission` is a **door decision**, not a bug. Map it:

| errorCode | Meaning | Door |
|---|---|---|
| `corporateFitnessMembershipExpired` | The benefit has ended | Refuse — member renews with their employer |
| `corporateFitnessMembershipNotStarted` | Benefit starts in the future | Refuse until the start date |
| `corporateFitnessGymNotInNetwork` | Your gym is not a Wellpass network partner | Refuse — this is a contract issue at your venue, not a member issue |
| `corporateFitnessGymNotInNetworkPlus1` | Member holds a Plus1 (guest) membership but your gym does not run Plus1 | Refuse the guest |

Surface the distinction to staff. "Expired" is the member's problem; "gym not in network" is
yours, and telling a member their pass is invalid when your venue simply is not enrolled is
the worst outcome available.

## Step 4 — record the visit

- `checkin` — `POST /api/v2/accounts/{accountId}/checkins`
- `checkout` — `POST /api/v2/accounts/{accountId}/checkouts`

These are what EGYM bills and reports on, and they are what fire the `GYM_CHECKIN` /
`GYM_CHECKOUT` webhooks to other subscribed partners.

## Pacing

A busy door can burst. The limiter is a token bucket: capacity 60, refilling at 6/second, per
API key. Bursts up to 60 are fine; sustained traffic above 6/s is not. Read
`X-RateLimit-Remaining` and throttle before you get a `429` — there is no `Retry-After`.

See `errors/egym-error-codes.yml`, `rate-limits/egym-rate-limits.yml`.
