# VaultRE (vaultre)

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
