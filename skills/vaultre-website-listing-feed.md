---
name: Publish VaultRE listings on an agency website
description: >-
  Pull an agency's currently available sale and lease listings out of VaultRE, with photos, and
  render them on the agency's own website — the single most common VaultRE integration.
api: openapi/vaultre-api-v1-3-openapi.yml
generated: '2026-07-26'
method: generated
source: openapi/vaultre-api-v1-3-openapi.yml + https://docs.api.vaultre.com.au/guide.html
operations:
  - getAvailableResidentialSaleProperties
  - getAvailableResidentialLeaseProperties
  - getResidentialSaleProperty
  - getPropertyPhotos
  - getResidentialSoldProperties
  - getTokenScopes
  - getUsage
---

# Publish VaultRE listings on an agency website

## Before you start

Every request needs both headers:

```
X-Api-Key: <integrator API key>
Authorization: Bearer <the agency's access token>
```

A missing or invalid `X-Api-Key` returns **403**. A missing or expired bearer token returns **401**.

Call `getTokenScopes` (`GET /scopes`) first and store what comes back. Scopes are chosen by the
agency, not requested by you, and the agency can revoke the token at any time — so a 403 on a
specific operation usually means "this token was never granted that", not "you are broken".

## Do NOT call this API per page load

VaultRE says so explicitly: 10 requests/second and 10,000 requests/day per API key, quota reset at
00:00 UTC, HTTP 429 on exceed. Synchronise on a schedule into your own store and serve the website
from that store. Check remaining quota with `getUsage` (`GET /integrator/usage`).

## Steps

1. **Pull available sale listings.** `getAvailableResidentialSaleProperties`
   (`GET /properties/residential/sale/available`). Page with `page` and `pagesize` (default 50) and
   read `totalItems` / `totalPages` from the response envelope; follow `urls.next` until exhausted.
2. **Pull available lease listings.** `getAvailableResidentialLeaseProperties`
   (`GET /properties/residential/lease/available`), same pagination.
3. **Fetch detail only where you need it.** `getResidentialSaleProperty`
   (`GET /properties/residential/sale/{id}`) returns the full record for a single listing. Prefer
   the list payload — it already carries heading, description, price display and address.
4. **Fetch photos and DOWNLOAD them.** `getPropertyPhotos` (`GET /properties/{id}/photos`).
   Hotlinking VaultRE image URLs is prohibited and will get the feed disabled. Store the files
   yourself and use each photo's `modtime` to decide whether to re-download on the next sync.
5. **Optionally publish recent sales.** `getResidentialSoldProperties`
   (`GET /properties/residential/sale/sold`) for a "recently sold" page.
6. **Re-sync incrementally.** On subsequent runs pass `modifiedSince` (a shared component
   parameter available on the list operations) with the timestamp of your last successful run,
   rather than re-walking every page.

## Understanding what you got back

A **property** is the physical asset. A **life** is one marketing episode of that asset on one side
of the market. The property carries `saleLifeId` and `leaseLifeId`, and anything listing-level —
feedback, open homes, offers, files, owners — is addressed as
`/properties/{propertyid}/{salelease}/{lifeid}/…` where `{salelease}` is the literal string `sale`
or `lease`. Key your website records on the **life**, not the property, or a house that is both for
sale and for lease will collapse into one listing.

Dates come back as `YYYY-MM-DD`; datetimes as `YYYY-MM-DDThh:mm:ss+z`, usually UTC.

## Errors

| Status | Meaning | What to do |
|---|---|---|
| 400 | Invalid data — the dominant error in this API | Fix the request; do not retry unchanged |
| 401 | Missing/expired bearer token | Re-obtain the agency token |
| 403 | Missing API key, or the token lacks this permission | Check `getTokenScopes`; ask the agency to re-issue |
| 429 | Rate limited | Back off; you are calling too often — cache harder |

The error body is `{"success": false, "msg": "...", "code": "..."}` — not RFC 9457 problem+json.

## Things this API does not give you

There is no `Idempotency-Key`, no field expansion or sparse fieldsets (use the sub-resource paths),
and no sandbox — you cannot exercise any of this before an agency grants you a real token.
