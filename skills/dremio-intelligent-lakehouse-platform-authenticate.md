---
name: Authenticate an agent to Dremio
description: Choose the right credential for the surface you are calling — hosted MCP, self-hosted MCP, CLI, or REST — and mint it correctly.
api: openapi/dremio-intelligent-lakehouse-platform-authentication-api-openapi.yml
operations: [login, listUserTokens, createUserToken]
generated: '2026-09-06'
method: generated
source: authentication/dremio-intelligent-lakehouse-platform-authentication.yml + https://docs.dremio.com/dremio-cloud/api/oauth-token
---

# Authenticate an agent to Dremio

Three credentials, three surfaces, and they are **not** interchangeable.

| Surface | Credential |
|---|---|
| Hosted MCP server (`https://mcp.dremio.cloud/mcp/{project_id}`) | OAuth only |
| Self-hosted MCP server, Dremio Developer CLI, JDBC/ODBC | Personal access token |
| REST from a machine-to-machine service | Service user, `client_credentials` |

## Machine-to-machine (recommended for agents)

1. Create a **service user** in the Dremio console and get its `client_id` and
   `client_secret`.
2. `POST https://login.dremio.cloud/oauth/token` with
   `grant_type=client_credentials`, `client_id`, `client_secret`, `scope=dremio.all`.
3. Use the returned `access_token` as `Authorization: Bearer <token>`. It lives one hour.
   Request `offline_access` to get a `refresh_token` and renew without re-consent.

## Other paths

- **Exchange an external JWT** — `grant_type=urn:ietf:params:oauth:grant-type:token-exchange`
  with `subject_token_type=urn:ietf:params:oauth:token-type:jwt`, for callers already
  authenticated by Microsoft Entra ID or another OIDC provider.
- **Exchange a PAT** — Dremio recommends trading a PAT for an OAuth access token before
  going to production.
- **Mint a PAT** — `createUserToken` (`listUserTokens` to see existing ones). PATs are
  long-lived; `millisecondsToExpire` sets the lifespan.
- **Legacy username/password** — the `login` operation on self-managed Dremio only.
  Rate limited to 45 requests/second per IP.

## The rule that matters most

**The OAuth scope is always `dremio.all`.** There is no read-only token. Least privilege
has to be expressed as Dremio RBAC on the service user — roles, grants, row-access and
column-masking policies. If you are giving an agent a Dremio token, scope the *user*,
because the scope string will not scope anything.
