---
name: Keep a local copy of a VaultRE account in sync
description: >-
  Mirror an agency's contacts and properties into your own store and keep them current using the
  event stream or webhooks, inside VaultRE's 10/second and 10,000/day quota.
api: openapi/vaultre-api-v1-3-openapi.yml
generated: '2026-07-26'
method: generated
source: openapi/vaultre-api-v1-3-openapi.yml + https://docs.api.vaultre.com.au/guide.html
operations:
  - getEventStream
  - getContacts
  - getProperties
  - getPropertiesLifeSale
  - getPropertiesLifeLease
  - getUsage
  - getTokenScopes
---

# Keep a local copy of a VaultRE account in sync

VaultRE's own guidance is unambiguous: do not call the API live on each page load, "synchronise
data periodically and cache it locally". This skill is that sync.

## Budget first

10 requests/second and **10,000 requests/day per API key**, resetting 00:00 UTC. That quota is
shared across every agency you serve on that key. Check `getUsage` (`GET /integrator/usage`) before
and after a run and stop early rather than getting 429s mid-sync.

## Phase 1 — initial backfill

Walk each list surface with `page` / `pagesize` (default 50) until `page` exceeds `totalPages`:

- `getContacts` (`GET /contacts`)
- `getProperties` (`GET /properties`)
- `getPropertiesLifeSale` (`GET /properties/sale`) and `getPropertiesLifeLease`
  (`GET /properties/lease`) for the marketing episodes

Record the timestamp you started. Raise `pagesize` rather than making more calls — every page is a
request against the daily quota.

## Phase 2 — incremental catch-up

On every subsequent run, pass the shared date-window component parameters instead of re-walking:

- `modifiedSince` / `modifiedBefore` — records changed in a window
- `insertedSince` / `insertedBefore` — records created in a window

These are declared once in `components.parameters` and applied across the list operations; they are
the intended incremental-sync mechanism.

## Phase 3 — change notification

Two options, and they carry the same semantics:

1. **Poll the event stream.** `getEventStream` (`GET /eventStream`) with `eventsSince` (a datetime)
   or `cursor` (for pagination). Each item is `{id, timestamp, type, action, data}`. **Events
   expire after 30 days** — if your consumer is down longer than that, fall back to a
   `modifiedSince` sweep, because the gap is unrecoverable from the stream.
2. **Receive webhooks.** Register a URL with VaultRE out of band (api@vaultre.com.au). Payloads are
   `{event, data:{id}, accountid, timestamp, itemBodies[]}` where `event` is `<object>.<action>`
   (e.g. `user.update`, `property.update`, `contact.merge`) and each `itemBodies[].data` is the full
   object serialised as a JSON **string**. Verify `X-VaultRE-Signature` before trusting anything:
   HMAC-SHA512 over `<timestamp-ms>.<raw body>` keyed with your API key, compared byte-exactly
   against the raw body **before** you deserialise it. Reject stale timestamps to block replays.
   Webhook requests carry no other authentication.

Either way, resolve the changed record with a targeted GET rather than assuming the payload is
complete.

## Handling contact merges

`contact.merge` fires when an asynchronous merge submitted through `POST /contacts/merge` completes.
The losing contact id disappears from `getContacts`. Resolve it with
`GET /contacts/{contactid}/merged` (operationId `getContactMergedDuplicates`) and repoint your local
foreign keys — do not delete the row on a 404 alone.

## Modelling note that will bite you

Key your store on the property **life**, not the property. `/properties/{propertyid}/{salelease}/{lifeid}`
is the real listing address; a property can carry both a `saleLifeId` and a `leaseLifeId` at the
same time, and a webhook update on one does not invalidate the other.
