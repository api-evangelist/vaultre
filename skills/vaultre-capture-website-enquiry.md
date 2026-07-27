---
name: Capture a website enquiry into VaultRE
description: >-
  Take a form submission from an agency website (listing enquiry, appraisal request, newsletter
  signup) and land it in VaultRE as an enquiry and a contact, without creating duplicates.
api: openapi/vaultre-api-v1-3-openapi.yml
generated: '2026-07-26'
method: generated
source: openapi/vaultre-api-v1-3-openapi.yml + https://docs.api.vaultre.com.au/samples.html
operations:
  - searchContactsEmail
  - searchContactsPhone
  - getContact
  - addContact
  - updateContact
  - addEnquiry
  - getEnquiries
  - addContactNote
  - addContactRequirement
  - getTokenScopes
---

# Capture a website enquiry into VaultRE

This is the flow VaultRE names first in its own getting-started page: "You wish to capture
enquiries/newsletter signups from your office website as a contact in your VaultRE account."

## Before you start

Send both headers on every call — `X-Api-Key` (your integrator key) and
`Authorization: Bearer <agency access token>`. Call `getTokenScopes` (`GET /scopes`) once at start
of session and confirm the token was granted contact and enquiry access.

## Steps

1. **Look for an existing contact before creating one.** There is no upsert and no idempotency key
   in this API, so a naive create-on-every-submission produces duplicates that a human then has to
   merge. Try `searchContactsEmail` (`GET /search/contacts/email`, exact match) and then
   `searchContactsPhone` (`GET /search/contacts/phone`, exact match).
2. **Create the contact only if nothing matched.** `addContact` (`POST /contacts`). If you matched,
   use `updateContact` (`PUT /contacts/{id}`) to fill gaps — do not overwrite fields the agency has
   curated. Store your own system's identifier in the contact's `remoteId` field so the next
   submission resolves without a search.
3. **Submit the enquiry.** `addEnquiry` (`POST /enquiries`) lodges a listing or agent enquiry into
   the holding area, where agency staff process it. Attach it to the listing the visitor was
   looking at via `saleLifeId` or `leaseLifeId` — the LIFE id, not the property id.
4. **Record the buyer's criteria if the form collected them.** `addContactRequirement`
   (`POST /contacts/{id}/requirements`) stores buying/renting requirements, which is what powers
   VaultRE's own contact↔listing matching.
5. **Add context as a note if you have free text the enquiry shape does not carry.**
   `addContactNote` (`POST /contacts/{id}/notes`).
6. **Verify.** `getEnquiries` (`GET /enquiries`) lists the holding area; `getContact`
   (`GET /contacts/{id}`) confirms the contact.

## Consent

If you are capturing marketing consent, VaultRE exposes consent records at
`GET /contacts/{contactid}/gdpr` (operationId `getContactGdprOptins`), returning email, SMS and
phone opt-in flags. Read it before you add anyone to a marketing flow — do not assume a form tick
propagated.

## Retry safety — read this

**VaultRE has no idempotency contract.** No `Idempotency-Key` header or parameter exists in the
specification. If `addContact` or `addEnquiry` times out you cannot safely blind-retry: search
first (step 1) to see whether the write actually landed, and only then retry.

## Errors

`400 Invalid data` is by far the most common response and means your body failed validation — fix
it, do not retry. `403` means either no `X-Api-Key` header or the token lacks a permission such as
"delete contacts" or "send email". `429` means you exceeded 10 requests/second or 10,000/day.
Bodies are `{"success": false, "msg": "...", "code": "..."}`; the `code` vocabulary is readable at
runtime from `GET /responseCodes` (`getResponseCodes`).
