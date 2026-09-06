---
name: Run a SQL query on Dremio and read the results
description: Submit SQL as an asynchronous job, poll it to completion, and page through the result rows — the single most common Dremio API flow.
api: openapi/dremio-intelligent-lakehouse-platform-jobs-api-openapi.yml
operations: [submitJob, getJob, getJobResults, cancelJob]
generated: '2026-09-06'
method: generated
source: openapi/dremio-intelligent-lakehouse-platform-jobs-api-openapi.yml + https://docs.dremio.com/dremio-cloud/api/sql
---

# Run a SQL query on Dremio

Dremio's SQL API is **asynchronous**. A 200 on submit does not mean the query
succeeded — it means the job was accepted. This is the mistake to avoid.

## Authenticate

Send `Authorization: Bearer <token>` on every request. The token is a personal access
token or an OAuth access token. OAuth access tokens live one hour (`expires_in: 3599`),
so a long-running agent should expect a `401` on schedule and re-mint rather than treat
it as a failure.

- Dremio Cloud base: `https://api.dremio.cloud/v0/` (EU: `https://api.eu.dremio.cloud`)
- Self-managed base: `https://{hostname}/api/v3`

## Steps

1. **Submit** — `submitJob`. POST the SQL statement. Include a `requestId` (a fresh UUID)
   in the body: it is Dremio's idempotency key, so a retry after a timeout returns the
   original response instead of running the query twice. Keys are reusable after 24 hours.
2. **Poll** — `getJob` with the returned job id. Read the job state. A job that failed
   reports it here, not in the HTTP status of step 1.
3. **Read** — `getJobResults` once the job completed. This endpoint pages with `limit`
   and `offset`, not with `pageToken` — the catalog endpoints use cursors, job results do
   not. Loop until fewer rows come back than requested.
4. **Cancel** — `cancelJob` if the caller aborts or a budget is exceeded. Cancelling does
   not delete the job object; `getJob` still works afterwards.

## Rules

- **Rate limits are per IP address**, across all organizations and projects: 1,200 API
  calls/minute overall, 1,000/minute on `/job/{id}/results`, 100/minute on `/job/{id}`
  and 100/minute on `/job/{id}/cancel`. Poll `getJob` no faster than once per second per
  job, and stagger parallel agents — they share the budget if they share an egress IP.
- **There are no rate-limit response headers.** Nothing tells you how much budget is
  left. Budget from the published numbers, not from the response.
- `400` is a SQL or request error — fix it, do not retry. `403` is RBAC on the
  authenticated user, not a token-scope problem: the OAuth scope is always `dremio.all`.
  `500` is safe to retry with the same `requestId`.
- Arrow Flight SQL (`grpc+tls://data.dremio.cloud:443`) is the right transport for large
  result sets; the REST results endpoint is for modest ones. The Flight path caps at 10GB
  returned.
