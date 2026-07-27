---
name: Run open homes and vendor feedback on a VaultRE listing
description: >-
  Work a live listing the way an agency does — schedule open homes, record buyer feedback against
  the property life, surface matching buyers, and read the offers that follow.
api: openapi/vaultre-api-v1-3-openapi.yml
generated: '2026-07-26'
method: generated
source: openapi/vaultre-api-v1-3-openapi.yml
operations:
  - getAccountOpenHomes
  - getPropertyOpenHomes
  - addOpenHome
  - updateOpenHome
  - attachFeedbackToPropertyLife
  - getPropertyFeedbackLife
  - getPropertyFeedbackLifeSummary
  - getPropertyMatchingContacts
  - getPropertyOffers
  - getPropertyLifeOfferConditions
---

# Run open homes and vendor feedback on a VaultRE listing

Everything in this skill hangs off a **property life**, not a property. The address shape is
`/properties/{propertyid}/{salelease}/{lifeid}/…` where `{salelease}` is the literal string `sale`
or `lease` and `{lifeid}` is the `saleLifeId` or `leaseLifeId` from the property record. If you pass
the property id where a life id belongs, you will get a 404 or silently touch the wrong listing.

## Open homes

- `getAccountOpenHomes` (`GET /openHomes`) — every open home across the account, for a diary view.
- `getPropertyOpenHomes` (`GET /properties/{id}/{salelease}/{lifeid}/openHomes`) — the schedule for
  one listing.
- `addOpenHome` (`POST /properties/{id}/{salelease}/{lifeid}/openHomes`) — create one.
- `updateOpenHome` (`PUT /properties/{propertyid}/{salelease}/{lifeid}/openHomes/{id}`) — change
  the time or details.

Datetimes are `YYYY-MM-DDThh:mm:ss+z`; you may submit any offset, and reads generally come back
UTC. Confirm the agency's local timezone rather than assuming +10:00 — this platform runs across
Australian states with different DST behaviour and New Zealand.

## Feedback

- `attachFeedbackToPropertyLife` (`POST /properties/{id}/{salelease}/{lifeid}/feedback`) — record
  what a buyer said. Feedback belongs to a contact; the external-feedback surface can create the
  contact inline instead of requiring a `contactId` first.
- `getPropertyFeedbackLife` (`GET /properties/{id}/{salelease}/{lifeid}/feedback`) — the list.
- `getPropertyFeedbackLifeSummary` (`GET /properties/{id}/{salelease}/{lifeid}/feedback/summary`) —
  the rollup an agent takes to a vendor conversation.

Since 2026-05-29 external feedback also carries a **source of enquiry** (`source.id`) — populate it
from `GET /account/enquirySources` so the agency's reporting on where buyers came from stays honest.

## Matching buyers to the listing

`getPropertyMatchingContacts` (`GET /properties/{id}/{salelease}/{lifeid}/matchingContacts`) returns
contacts whose stored buying/renting requirements match this listing. This is the payoff for
recording requirements at enquiry time — see the enquiry-capture skill.

## Offers

- `getPropertyOffers` (`GET /propertyOffers`) — offers across the account, each carrying a
  `contact`, a `saleLifeId` or `leaseLifeId`, an `offerDate`, an `offerPrice` and a `status`.
- `getPropertyLifeOfferConditions` (`GET /properties/{id}/sale/{lifeid}/offerConditions`) — the
  conditions attached to a sale life, with `complete` / `uncomplete` transitions available on the
  individual condition.

## Write safety

There is no idempotency key on any of these writes. If `addOpenHome` or
`attachFeedbackToPropertyLife` times out, re-read the collection before retrying — a blind retry
creates a duplicate open home or a duplicate feedback record that an agent has to clean up.

Errors are `{"success": false, "msg": "...", "code": "..."}`. A `403` here often carries a specific
permission reason, e.g. "User does not have access to property financials" — surface `msg` to the
operator rather than a generic failure.
