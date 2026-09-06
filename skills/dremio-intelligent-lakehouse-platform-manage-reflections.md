---
name: Manage Dremio Reflections safely
description: List, create, update and delete Reflections — the acceleration layer — including how to rehearse a refresh before running it.
api: openapi/dremio-intelligent-lakehouse-platform-reflections-api-openapi.yml
operations: [listReflections, createReflection, getReflection, updateReflection, deleteReflection]
generated: '2026-09-06'
method: generated
source: openapi/dremio-intelligent-lakehouse-platform-reflections-api-openapi.yml + https://docs.dremio.com/dremio-cloud/ai-integration/cli/
---

# Manage Dremio Reflections

A Reflection is a materialization that accelerates queries over an anchor dataset.
Creating and refreshing them costs compute — Dremio Cloud bills per DCU — so this is a
flow where an agent should rehearse before it acts.

## Steps

1. **Inventory** — `listReflections`, or `getReflection` for one. The Reflection Summary
   endpoint pages with `pageToken`/`nextPageToken`, filters with `filter` and sorts with
   `orderBy`.
2. **Rehearse a refresh** — the REST API has no dry-run. The Dremio Developer CLI does:
   `dremio reflection refresh <reflection-id> --dry-run` validates before executing. Use
   the CLI for this step if it is available.
3. **Create** — `createReflection`. Include a `requestId` UUID; it is a POST, so the
   idempotency key applies and a retry will not create a duplicate.
4. **Update** — `updateReflection`. Send the `tag` you read from `getReflection`.
5. **Delete** — `deleteReflection`. Returns 204.

## Rules

- **A deleted Reflection cannot be restored** — it has to be recreated from scratch.
  Treat delete as irreversible and confirm with a human.
- Quotas are hard: 500 Reflections per project, 100 of them autonomous. A create that
  would cross the ceiling fails.
- Reflection refresh frequency is capped at once per hour.
- Since 2026-08-19, Reflection changes take effect immediately for query planning rather
  than after a periodic sync — do not build in a wait.
