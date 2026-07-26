---
name: Advocate share and referral flow
description: Advocate share and referral flow using the Extole API.
api: openapi/extole-integration-consumer-to-extole-openapi.json
operations: [getMyProfile, getMeShareables, createMeShareable_2, shareEvent, getShares]
---

# Advocate share and referral flow

Guide an authenticated advocate (consumer) through creating a shareable and referring a friend on Extole.

## Auth
Authenticate the consumer with an Extole access token. Supply it as the `Authorization` header (Bearer), the `extole_token` cookie, or the `access_token` query parameter. Base host: `https://{brand}.extole.io`.

## Steps
1. `getMyProfile` (GET /api/v4/me) — read the advocate profile and confirm the identity of the caller.
2. `getMeShareables` (GET /api/v6/me/shareables) — list the advocate's existing shareables (share links / codes).
3. `createMeShareable_2` (POST /api/v6/me/shareables) — create a new shareable for the advocate if none is suitable.
4. `shareEvent` (POST /api/v4/events/share) — record a share to one or more channels/recipients.
5. `getShares` (GET /api/v4/me/shares) — confirm the share was recorded and track referral status.

## Conventions
- Pagination on list endpoints uses `offset` / `limit`.
- Errors return a JSON envelope (`error_code`, `message`, `parameters`); handle 401 (bad/expired token) and 429 (rate limited) explicitly.
- No idempotency-key header is supported; avoid blind retries on POST.
