# VaultRE (vaultre)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

VaultRE - now marketed as MRI Vault CRM, after VaultRE's parent Vault Group / PropTech Group was absorbed into MRI Software - is an Australian cloud real estate CRM and transaction platform used by residential, commercial, rural, land, business and property-management agencies across Australia and New Zealand. It sits in the agency and CRM layer of the value chain rather than the portal or registry layer: it holds the agency's contacts, appraisals, listings, offers, open homes, feedback, tenancies, maintenance, invoicing and AML records, and feeds listings outward to Australia's portal duopoly (realestate.com.au and Domain) instead of being a portal, a land registry or a conveyancing rail itself. Unlike most of this sector, VaultRE publishes a genuinely open, versioned, machine-readable contract - a public MkDocs developer site at docs.api.vaultre.com.au carrying downloadable OpenAPI 3.0.1 documents for v1.1, v1.2 and v1.3 of the core API (324 paths and 453 operations in v1.3) plus a separate Aggregator API for franchise-group CRM data feeds. The API surface is documented openly but access is not self-serve: anyone can read the docs and download the specs, but API keys are issued only after an integrator-registration request is reviewed and approved, and each agency must then mint a scoped access token for that integrator from inside its own VaultRE account. RESO plays no part here - Australia has no RESO mandate and no RESO, OData or `$metadata` reference appears anywhere in VaultRE's documentation or specifications; the local listing-distribution seam is REAXML and portal-specific feeds instead.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/vaultre/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/vaultre/refs/heads/main/apis.yml)

## Tags

- Real Estate
- Australia
- New Zealand
- PropTech
- CRM
- Property Listings
- Property Management
- Rentals
- Commercial Real Estate
- Webhooks

## Timestamps

- **Created:** 2026-07-26
- **Modified:** 2026-07-26

## Access

- **Developer portal:** [https://docs.api.vaultre.com.au/](https://docs.api.vaultre.com.au/) — public, no login
- **Access gate:** application-approval — register as an approved integrator at [resources.vaultre.com.au/api-integrations](https://resources.vaultre.com.au/api-integrations), then each agency grants your integration a scoped token from Office Integrations → Third-Party Access
- **Auth:** `X-Api-Key` (integrator key) + `Authorization: Bearer` (per-agency access token); OAuth2-style authorization-code flow for token minting; HS512 JWT for integrator and aggregator endpoints
- **RESO:** no RESO reference found — no Web API or Data Dictionary certification, no OData `$metadata` (HTTP 404), no UPI
- **Open data:** none — every dataset is an individual agency's private CRM data

## APIs

### VaultRE API

The core VaultRE REST API — the open API a third-party developer integrates an agency website or application against. Version 1.3 documents 324 paths and 453 operations covering contacts, properties across every life type (residential, commercial, rural, land, business, holiday rental, livestock, clearing sales), listings and photos, enquiries, offers and offer conditions, open homes, feedback, appraisals, calendar and tasks, inspections, maintenance and suppliers, tenancies, invoices, deals, campaigns and advertising, templates, merge fields, SMS and email messaging, AML checks, keys, suburbs and precincts, an event-stream poll endpoint, CoreLogic property and AVM lookups, and REINZ sales reporting for New Zealand. Every request carries both an integrator `X-Api-Key` header and a per-account Bearer access token; the granted scopes on a token are readable from `GET /scopes`.

- **Human URL:** [https://docs.api.vaultre.com.au/swagger/index.html](https://docs.api.vaultre.com.au/swagger/index.html)
- **Base URL:** `https://ap-southeast-2.api.vaultre.com.au/api/v1.3`

#### Tags

- Contacts
- Properties
- Listings
- Property Management
- Enquiries
- AML
- Valuation

#### Properties

- [OpenAPI](openapi/vaultre-api-v1-3-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [OpenAPI](openapi/vaultre-api-v1-2-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [OpenAPI](openapi/vaultre-api-v1-1-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://docs.api.vaultre.com.au/swagger/index.html)
- [Documentation](https://docs.api.vaultre.com.au/guide.html)
- [Getting Started](https://docs.api.vaultre.com.au/basics.html)
- [Authentication](https://docs.api.vaultre.com.au/oauth.html)
- [Documentation](https://docs.api.vaultre.com.au/webhooks.html)
- [Code Examples](https://docs.api.vaultre.com.au/samples.html)
- [ChangeLog](https://docs.api.vaultre.com.au/changelog.html)
- [Source Code](https://github.com/VaultGroup/api-samples)

### VaultRE Integrator API

A distinct set of endpoints that operate at the integrator level rather than at an individual agency-account level, letting an approved integrator enumerate the accounts that have granted it an access token, list users on those accounts, validate a user, read granted scopes, list tokens, retrieve account-scoped merge fields, and read its own API usage. These endpoints replace the customer-supplied Bearer token with a short-lived HS512 JWT the integrator signs itself, carrying its API key and a current epoch timestamp and valid for 300 seconds, alongside the same `X-Api-Key` header.

- **Human URL:** [https://docs.api.vaultre.com.au/integrator.html](https://docs.api.vaultre.com.au/integrator.html)
- **Base URL:** `https://ap-southeast-2.api.vaultre.com.au/api/v1.3/integrator`

#### Tags

- Integrator
- Accounts
- Scopes
- Usage

#### Properties

- [OpenAPI](openapi/vaultre-api-v1-3-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://docs.api.vaultre.com.au/integrator.html)
- [API Reference](https://docs.api.vaultre.com.au/swagger/index.html)
- [Source Code](https://github.com/VaultGroup/api-samples/blob/master/python/create_jwt.py)

### VaultRE Aggregator API

A deliberately separate write-only ingestion API that lets other CRM systems feed property data into VaultRE on behalf of a franchise-group agency. Six documented operations accept staff records and property lifecycle events — appraisal, listing, unconditional sale, settlement and withdrawn listing. Submissions are queued rather than processed live, returning HTTP 202 on successful receipt, with validation and processing errors delivered asynchronously to a nominated webhook URL. Authentication uses a per-aggregator Secret Key in the `X-Api-Key` header plus a self-signed HS512 JWT bearing the aggregator's CRM Key, valid for 120 seconds.

- **Human URL:** [https://docs.api.vaultre.com.au/swagger/aggregator/index.html](https://docs.api.vaultre.com.au/swagger/aggregator/index.html)
- **Base URL:** `https://aggregator.api.vaultre.com.au/api/v1.0`

#### Tags

- Aggregator
- Property Data
- Franchise
- Ingestion

#### Properties

- [OpenAPI](openapi/vaultre-aggregator-api-v1-0-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://docs.api.vaultre.com.au/swagger/aggregator/index.html)
- [Documentation](https://docs.api.vaultre.com.au/aggregator.html)
- [Source Code](https://github.com/VaultGroup/api-samples/blob/master/aggregator/client.py)

## Common Properties

- [Website](https://www.mrisoftware.com/au/products/vault/)
- [Documentation](https://docs.api.vaultre.com.au/)
- [API Reference](https://docs.api.vaultre.com.au/swagger/index.html)
- [Getting Started](https://docs.api.vaultre.com.au/basics.html)
- [Authentication](https://docs.api.vaultre.com.au/oauth.html)
- [Rate Limits](https://docs.api.vaultre.com.au/guide.html)
- [ChangeLog](https://docs.api.vaultre.com.au/changelog.html)
- [Code Examples](https://docs.api.vaultre.com.au/samples.html)
- [Signup](https://resources.vaultre.com.au/api-integrations)
- [GitHub Organization](https://github.com/VaultGroup)
- [Source Code](https://github.com/VaultGroup/api-samples)
- [Login](https://login.vaultre.com.au/)

## Maintainers

- **Kin Lane** — kin@apievangelist.com
