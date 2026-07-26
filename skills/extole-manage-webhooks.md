---
name: Create and manage event webhooks
description: Create and manage event webhooks using the Extole API.
api: openapi/extole-management-openapi.json
operations: [listWebhooks, createWebhook, getWebhook, updateWebhook, archiveWebhook]
---

# Create and manage event webhooks

Guide a management integration through subscribing to Extole events via webhooks. Base host: `https://api.extole.io`.

## Auth
Use a management-scoped Extole access token in the `Authorization` header (Bearer).

## Steps
1. `listWebhooks` (GET /v6/webhooks) — review existing webhook subscriptions.
2. `createWebhook` (POST /v6/webhooks) — register a new webhook endpoint and event filter.
3. `getWebhook` (GET /v6/webhooks/{webhook_id}) — verify the created subscription.
4. `updateWebhook` (PUT /v6/webhooks/{webhook_id}) — adjust the endpoint/filter.
5. `archiveWebhook` (DELETE /v6/webhooks/{webhook_id}) — retire a subscription.

## Conventions
- Errors return a JSON envelope; handle 401/403/429.
- Pagination: `offset`/`limit`.
