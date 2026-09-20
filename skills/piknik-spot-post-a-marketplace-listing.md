---
name: piknik-spot-post-a-marketplace-listing
description: Authenticate, discover which place you manage, and post a surplus / wanted / service listing on Piknik's marketplace, then fulfil or close it — via MCP or REST.
api: openapi/piknik-spot-openapi.yml
surface: MCP (https://piknik.spot/api/mcp) or REST under https://piknik.spot/api
operations:
  - get_my_places
  - search_listings
  - create_listing
  - GET /v2/participants/my-businesses
  - GET /listings
  - POST /listings
  - GET /listings/my
  - PUT /listings/{id}
  - POST /listings/{id}/fulfill
  - POST /listings/{id}/reactivate
  - DELETE /listings/{id}
scopes: [marketplace:write]
method: generated
generated: '2026-09-19'
grounding: >-
  Tool names from the live tools/list (2026-09-19); REST paths and the CreateListingRequest schema
  verbatim from openapi/_original/piknik-spot-openapi.json (no operationIds in the spec); scope names
  from well-known/piknik-spot-oauth-authorization-server.json; the flow order mirrors the provider's
  own skills/piknik-spot-provider-rest-api-SKILL.md.
---

# Post a marketplace listing

## Authenticate

Either mint a personal access token at `https://piknik.spot/settings/developer` (prefix `pik_pat_`)
or run OAuth 2.0 authorization code + PKCE against `https://piknik.spot/api/oauth/authorize` /
`/api/oauth/token` (dynamic client registration at `/api/oauth/register`). Request the
`marketplace:write` scope. Send `Authorization: Bearer <token>` on every call.

## Steps

1. **Find the place you act for.** MCP `get_my_places` (no arguments) or REST
   `GET /v2/participants/my-businesses`. Save the `participant_id`; every listing is owned by a place.
2. **Check for an existing listing** before creating one. MCP `search_listings`
   `{ "query", "listing_type", "address" | "location", "radius_km", "limit" }` or REST `GET /listings?q=&participantId=`.
3. **Create.** MCP `create_listing` or REST `POST /listings` with the `CreateListingRequest` body:
   required `participant_id`, `listing_type` (`wanted_to_sell` | `wanted_to_buy` | `service_offering`),
   `title` (<= 200 chars), `description`, `contact_method` (`email` | `phone` | `message`), `end_date`
   (`YYYY-MM-DD`); optional `quantity`, `radius_km` (default 50), `wanted_in_return`, `association_ids[]`.
   The MCP tool additionally accepts `offering_id` to tie the listing to a product.
4. **Manage.** `GET /listings/my` lists yours; `PUT /listings/{id}` edits; `POST /listings/{id}/fulfill`
   marks it fulfilled; `POST /listings/{id}/reactivate` re-opens an expired one; `DELETE /listings/{id}` removes it.
   Counterparty actions exist as `POST /listings/{id}/accept`, `/decline`, `/confirm-acceptance`,
   `/decline-acceptance`, `/confirm-direct-pickup`.

## Cautions

- **No idempotency key exists on this surface.** A retried `POST /listings` after a timeout can create a
  duplicate — run step 2 again before retrying (see `conventions/piknik-spot-conventions.yml`).
- **Reversal:** a created listing can be deleted or fulfilled at any time by its owner; the provider
  publishes no time window, so the reversal is `documented`, not `verified`.
- 401 with `{ "error", "error_description" }` means the token is missing/expired — refresh via the
  token endpoint (`refresh_token` grant) and retry; 403 means the place is not yours.
- Listings are public and indexable once posted (privacy policy) — do not post personal data.
