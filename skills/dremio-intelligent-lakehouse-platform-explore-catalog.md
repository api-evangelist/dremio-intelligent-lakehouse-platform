---
name: Explore the Dremio catalog and describe a dataset
description: Walk the catalog by id or by path, read a dataset's schema, and understand what an agent is allowed to see before it queries.
api: openapi/dremio-intelligent-lakehouse-platform-catalog-api-openapi.yml
operations: [listCatalog, getCatalogEntity, getCatalogByPath, deleteCatalogEntity]
generated: '2026-09-06'
method: generated
source: openapi/dremio-intelligent-lakehouse-platform-catalog-api-openapi.yml + https://docs.dremio.com/dremio-cloud/api/catalog/
---

# Explore the Dremio catalog

## Steps

1. **List the top level** — `listCatalog`. Returns the catalog roots (sources, spaces,
   home). Supports the `include` query parameter.
2. **Resolve a path** — `getCatalogByPath`. Use this when you have a human path like
   `Analytics.production.quarterly_revenue`. It is the agent-friendly entry point.
3. **Read the entity** — `getCatalogEntity` by id. Returns the entity and its `tag`, a
   UUID version identifier.
4. **Descend** — folders nest up to 8 levels. Child listings page with `pageToken`; the
   continuation token comes back as `nextPageToken` on Folder and as `pageToken` on Data
   Maintenance. Do not assume one field name.

## Rules

- **Keep the `tag`.** Any update must send back the `tag` you read, or Dremio rejects the
  write. It is optimistic concurrency, and it is how you avoid clobbering a concurrent edit.
- **`deleteCatalogEntity` has no undo.** No restore endpoint is documented for a deleted
  catalog entity. If the entity is a table, the underlying Iceberg data may still be
  recoverable through snapshots, but the catalog object is not. Ask a human first.
- A `404` may mean renamed rather than gone — re-resolve by path.
- A `403` means the RBAC policy on this user denies it. Row-access and column-masking
  policies can also silently narrow what a query returns without raising anything.
