---
name: Server-side reward lookup and fulfillment
description: Server-side reward lookup and fulfillment using the Extole API.
api: openapi/extole-integration-server-to-extole-openapi.json
operations: [listRewards, getReward, getRewardStateSummary, getRewardFulfillments]
---

# Server-side reward lookup and fulfillment

Guide a server integration through inspecting and reconciling rewards on Extole. Base host: `https://api.extole.io`.

## Auth
Use a server-side Extole access token in the `Authorization` header (Bearer).

## Steps
1. `listRewards` (GET) — list rewards, paging with `offset`/`limit`, filtered to the relevant person/program.
2. `getRewardStateSummary` — get the aggregate state summary to see counts by reward state.
3. `getReward` (GET /rewards/{id}) — fetch a specific reward's detail.
4. `getRewardFulfillments` — inspect fulfillment records for the reward to reconcile delivery.

## Conventions
- Errors: JSON envelope; 402/403 indicate account/authorization problems, 429 is rate limiting.
- Pagination: `offset`/`limit`.
