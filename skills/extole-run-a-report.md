---
name: Run an Extole report and download the results
description: Discover a report type, submit a report run, poll it to completion, and download the
  output — the flow behind Extole's own extole_reporting_agent MCP toolset, expressed against the
  public Management API so an agent can run it without the MCP server.
api: openapi/_original/extole-management-openapi.json
operations:
  - listReportTypes
  - listReportTypeRecommendations
  - createReport
  - getReport
  - downloadReport
  - getLatestReport
  - downloadLatestReport
  - cancelReport
  - retryReport
generated: '2026-08-13'
method: generated
source: openapi/_original/extole-management-openapi.json + https://docs.extole.com/reference/common-errors
---

# Run an Extole report

Reporting is asynchronous. You submit a run, you poll it, you download it. Nothing about this flow
returns data in one call, and a client that assumes it does will read an empty or partial result.

Extole ships this same flow as an MCP toolset (`extole_reporting_agent`, described by the provider as
"discover, submit, poll, and download reports"). Use this skill when you are calling the REST API
directly — for example from a backend job, or when the MCP server is not connected.

## Base URL and auth

- Host: `https://api.extole.io`
- Credential: a bearer access token created in the My.Extole Security Center.
- Send it as `Authorization: Bearer <token>`. Do not use the `?access_token=` query form — it writes
  the token into logs and browser history.
- Scope: reporting reads need at minimum a token that carries an appropriate scope; a token without it
  returns `403` with code `scopes_denied`. Do not reach for `CLIENT_ADMIN` if a narrower scope works.

See `authentication/extole-authentication.yml` and `scopes/extole-scopes.yml`.

## Steps

1. **Find the report type.** `listReportTypes` (`GET /v6/report-types`) returns the report types
   available to this client. If you are choosing on the user's behalf rather than being told which
   report to run, `listReportTypeRecommendations` (`GET /v6/report-types/recommendations`) returns
   Extole's own recommendations — prefer it over guessing a report name.

2. **Submit the run.** `createReport` (`POST /v4/reports`) with the report type and its parameters.
   The response identifies the report; keep its id.

   There is no idempotency key on this endpoint. If the call times out, do **not** blind-retry it —
   a report may already be running. Call `listReports` (`GET /v4/reports`) and look for the run you
   submitted before submitting again. See `conventions/extole-conventions.yml`.

3. **Poll to completion.** `getReport` (`GET /v4/reports/{reportId}`) until the report reaches a
   terminal state. Back off between polls: the account-wide limit is 100 requests per minute per IP or
   token, and a tight poll loop on a slow report will spend it. On `429` (`too_many_requests`) back off
   exponentially — see `rate-limits/extole-rate-limits.yml`.

4. **Download the output.** `downloadReport` (`GET /v4/reports/{reportId}/download{format}`) once the
   report is complete. `getReportInfo` (`GET /v4/reports/{reportId}/info`) describes the output before
   you fetch it.

5. **Shortcut for a repeated run.** If you only need the most recent run of a report rather than a
   fresh one, `getLatestReport` (`GET /v4/reports/latest`) and `downloadLatestReport`
   (`GET /v4/reports/latest/download{format}`) skip steps 2 and 3 entirely. Reach for this first when
   the user asks "how did X do last week" — it costs one or two calls instead of a poll loop.

## Failure handling

Extole returns a stable string enum in `code`. Branch on it, not on the message text.

| Situation | What you get | What to do |
|---|---|---|
| Token missing or wrong | `401` / `403` with `missing_access_token`, `invalid_access_token`, `expired_access_token` | Acquire or refresh the token. Not retryable as-is. |
| Token lacks the scope | `403` `scopes_denied` | Use a token with the required scope. Not retryable. |
| Report type not enabled for this client | `402` `payment_required` | The feature is not provisioned. Escalate to the Extole guide; retrying will not help. |
| Bad parameters | `400` `validation_error` / `invalid_parameter` | Read `parameters.description` — it names the field and the condition that failed. Fix and resubmit. |
| Page size too large | `400` `max_fetch_size_1000` | Request 1000 or fewer. |
| Rate limited | `429` `too_many_requests` | Exponential backoff. |
| Server error | `500` | Retry with backoff. Log `unique_id` and quote it if you escalate. |

Always log `unique_id` from the error envelope. It is the only value Extole support can correlate.

Full registry: `errors/extole-error-codes.yml`.

## Cleanup

- `cancelReport` (`POST /v4/reports/{reportId}/cancel`) stops a run you no longer need.
- `retryReport` (`POST /v4/reports/{reportId}/retry`) re-runs a failed one rather than creating a
  duplicate through `createReport`.
