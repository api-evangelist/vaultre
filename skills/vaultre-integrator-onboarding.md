---
name: Onboard agencies as a multi-tenant VaultRE integrator
description: >-
  Operate at the integrator level — mint HS512 JWTs, enumerate the accounts that granted you a
  token, drive the OAuth flow so agencies self-serve their token, and audit scopes and quota.
api: openapi/vaultre-api-v1-3-openapi.yml
generated: '2026-07-26'
method: generated
source: >-
  openapi/vaultre-api-v1-3-openapi.yml + https://docs.api.vaultre.com.au/integrator.html
  + https://docs.api.vaultre.com.au/oauth.html
operations:
  - getIntegratorAccounts
  - getIntegratorAccount
  - getIntegratorAccountUsers
  - getIntegratorAccountUser
  - getIntegratorTokens
  - getIntegratorScopes
  - getTokenScopes
  - validateIntegratorUser
  - getIntegratorMergeFields
  - getUsage
---

# Onboard agencies as a multi-tenant VaultRE integrator

An integrator serving many agencies holds **one API key and many access tokens** — one per customer
account. These endpoints are how you manage that fleet.

## Authentication is different here

Integrator endpoints do **not** use a customer bearer token. You sign your own:

```
JWT algorithm: HS512
Payload: {"apiKey": "<your API key>", "timestamp": <current epoch seconds>}
Secret:  <secret key provided by VaultRE>
TTL:     300 seconds
```

Send it as `Authorization: Bearer <jwt>` alongside the usual `X-Api-Key`. Mint a fresh JWT per
request batch — five minutes is the whole lifetime. VaultRE publishes sample code at
`github.com/VaultGroup/api-samples/blob/master/python/create_jwt.py`.

(The Aggregator API uses the same pattern with a different payload —
`{"crmKey": ..., "timestamp": ...}` — and a 120-second TTL. It is a separate registration and a
separate host.)

## Getting an agency connected

Two paths, and you should support the second:

1. **Manual.** The agency goes to Office Integrations > Third-Party Access, selects you from the
   dropdown, clicks Create Token, and sends you the token. They choose the scopes. They can delete
   the token at any time.
2. **OAuth.** Send the agency to
   `https://login.vaultre.com.au/cgi-bin/clientvault/oauth-authorize.cgi?client_id=<client_id>&redirect_uri=<redirect_uri>&response_type=code`
   for an office/account-level token, or `oauth-authorize-user.cgi` for a user-level token. On
   confirm they return to your `redirect_uri` with `?reason=success&code=<code>`; on cancel with
   `?reason=User%20denied%20request`. Exchange the code **server-side only** (CORS blocks the
   browser) at `https://login.vaultre.com.au/cgi-bin/clientvault/integrations/oauthexchange.cgi`
   with `client_id` and `code`. You get `{"token": "...", "message": "Token generated successfully"}`.

   Two traps: the code is valid for **60 seconds**, and redirect URIs are matched **literally,
   including every query-string variant** — register each one explicitly, prefix matching does not
   exist. Your `client_id` is a different value from your API key and is issued by emailing
   api@vaultre.com.au. There is no dynamic client registration.

## Running the fleet

- `getIntegratorAccounts` (`GET /integrator/accounts`) — every account that has granted you a
  token. This is your tenant list; reconcile it against your own database on a schedule to catch
  revocations.
- `getIntegratorAccount` (`GET /integrator/accounts/{id}`) — one account.
- `getIntegratorTokens` (`GET /integrator/tokens`) — the tokens themselves, with their accounts.
- `getIntegratorAccountUsers` / `getIntegratorAccountUser` — staff on an account.
- `validateIntegratorUser` (`POST /integrator/validateUser`) — validate a user's credentials, for
  integrations that sign agency staff in.
- `getIntegratorMergeFields` (`GET /integrator/merge-fields`) — merge fields scoped to an account,
  the integrator-authenticated equivalent of `GET /merge-fields`.

## Scopes and quota

- `getIntegratorScopes` (`GET /integrator/scopes`) returns the scopes **possible** for your API key.
- `getTokenScopes` (`GET /scopes`, customer bearer token) returns the scopes **granted** on one
  agency's token.

Run the second at the start of every tenant session and cache it. The scope vocabulary is not
published anywhere in VaultRE's documentation — these two endpoints are the only place it exists,
so treat their output as the source of truth and degrade features gracefully rather than
hard-coding scope names.

`getUsage` (`GET /integrator/usage`) reports your remaining daily quota. Remember it is **per API
key, not per tenant**: 10,000 requests a day is shared across every agency you serve, so a
badly-behaved sync for one customer starves the rest.

## Errors specific to this surface

- `403` on every call usually means your JWT expired (300 seconds) or was signed with the wrong
  secret — not that the account lacks permission.
- `404 Account not found or not enabled for this integrator` means the token was revoked.
- `400 account_id is required, or an invalid parameter value was supplied` on
  `getIntegratorMergeFields` — that endpoint needs an explicit account.
