---
name: Subscribe to EGYM gym and equipment events
description: Register a webhook subscription on the MMS API V2, verify it with a TEST trigger, and consume EGYM's six event types correctly — including the fact that the payload is a notification, not a state transfer.
api: openapi/egym-mms-api-v2-openapi.yml
base_url: https://mms.api.egym.com
operations: [createWebhook, getAllWebhooks, getWebhookById, updateWebhook, deleteWebhook, triggerWebhook, retrieveAccount]
generated: '2026-08-12'
method: generated
source: openapi/egym-mms-api-v2-openapi.yml + https://developer.egym.com/general/webhooks
---

# Subscribe to EGYM gym and equipment events

EGYM pushes real-time notifications to an HTTPS URL you host. Subscriptions are **per gym**;
one gym can have several.

## Prepare the receiver first

Before you register anything, your endpoint must satisfy EGYM's stated requirements:

- **HTTPS only**, with a valid SSL certificate. Plain HTTP is rejected.
- Reachable by EGYM **without VPN or extra configuration**.
- **Responds within 5 seconds.** Past that, EGYM treats the delivery as unsuccessful.
  Acknowledge fast and process asynchronously — never do your business logic inline.
- Authenticates the caller on the `x-api-key` header, which carries the secret you supply at
  registration.

**There is no HMAC payload signature.** The shared secret tells you the call came from
someone who knows the secret; it does not attest that the body is unmodified. Treat the
payload as a trigger to go and read authoritative state, not as trusted data. EGYM also
documents no retry, backoff, dead-letter or ordering guarantees, so build for at-least-once
delivery and out-of-order arrival.

## Register

`createWebhook` — `POST /api/v2/webhooks` with your URL, your secret and the event types.

Expect `409 webhookUrlAlreadyRegistered` if that URL is already registered for the gym — call
`getAllWebhooks` (`GET /api/v2/webhooks`) and `updateWebhook`
(`PUT /api/v2/webhooks/{id}`) instead of creating a second one.

## Verify

`triggerWebhook` — `POST /api/v2/webhooks/{id}/trigger` — fires a `TEST` event at your
endpoint. Do this before you go live; it is the only self-service way to prove the path works.

## The payload

```json
{ "accountId": "[UUID]", "timestamp": 1667921312188, "type": "GYM_CHECKIN" }
```

Three fields. `timestamp` is epoch **milliseconds**, UTC.

This is a **notification, not a state transfer** — it does not tell you what changed. On
receipt, call back into the API (`retrieveAccount` — `GET /api/v2/accounts/{accountId}`) to
read the current state. Budget for that: a check-in spike produces a webhook spike which
produces a read spike against a 6 requests/second limiter.

## Event types

| type | Fires when |
|---|---|
| `GYM_CHECKIN` | A customer physically checked in at a gym |
| `GYM_CHECKOUT` | A customer physically checked out |
| `EQUIPMENT_CHECKIN` | A customer logged in to equipment in a gym |
| `MEMBERSHIP_DELETE` | Member data was deleted from EGYM |
| `GENIUS_ONBOARDING_COMPLETED` | A member completed EGYM Genius onboarding |
| `TEST` | Verification event from `triggerWebhook` |

EGYM states more types will be added, so **switch on `type` with a default branch that logs
and ignores** rather than one that throws.

## Consent filtering

Since 2026-06-24 EGYM filters events by member data-sharing consent. You will not receive
events for members who have not consented to share with you. A gap in your event stream is
therefore not necessarily a delivery failure — do not reconcile by assuming you should have
seen every member.

## Handle MEMBERSHIP_DELETE properly

It is a deletion instruction. Honour it in your own store; it is the mechanism by which a
member's erasure propagates out of EGYM to you.

## Teardown

`deleteWebhook` — `DELETE /api/v2/webhooks/{id}`.

Note: EGYM has stated these management endpoints will eventually be replaced by a UI-based
configuration interface, with migration guidance to follow. Keep the subscription id you
receive; do not hard-code assumptions about this API's permanence.

See `asyncapi/egym-events-webhooks.yml` and the derived
`asyncapi/egym-mms-events-asyncapi.yml`.
