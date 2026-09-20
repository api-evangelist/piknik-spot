---
name: piknik-spot-find-local-food-nearby
description: Find farms, markets, shops and restaurants near an address, see who sells a product, and score the address for local food access — using only Piknik's public (no-auth) MCP tools, with the provider's attribution and immediate-use rules applied.
api: mcp/piknik-spot-mcp.yml
surface: MCP (https://piknik.spot/api/mcp) with REST fallbacks under https://piknik.spot/api
operations:
  - geocode_address
  - find_places
  - list_all_places
  - search_offerings
  - search_products
  - calculate_local_food_score
  - GET /geocode
  - GET /v2/participants/nearby
  - GET /agents/public/place-summary
  - GET /local-food-score
method: generated
generated: '2026-09-19'
grounding: >-
  Tool names are verbatim from the live tools/list of 2026-09-19 (mcp/piknik-spot-mcp-tools-list.json).
  REST paths are verbatim from the provider's OpenAPI (openapi/_original/piknik-spot-openapi.json),
  which declares no operationIds. Rules come from well-known/piknik-spot-agent-ethics.md and llms.txt.
---

# Find local food near an address (public, no credentials)

Piknik's discovery tools are callable anonymously on the MCP server at `https://piknik.spot/api/mcp`
(Streamable HTTP, protocol `2025-03-26`). Nothing in this flow needs a token.

## Steps

1. **Resolve the address.** Call `geocode_address` with `{ "address": "<street or town, province>" }`.
   It returns latitude/longitude (Google Maps with Nominatim fallback). REST equivalent:
   `GET /geocode?address=…`.
2. **Find places.** Call `find_places` with `{ "address": …, "radius": <km>, "business_type": …, "limit": … }`
   for a ranked, context-aware list, or `list_all_places` with `{ "latitude", "longitude", "radius_km", "primary_role", "limit", "offset" }`
   for the full inventory. `primary_role` values published by the provider: `FARM`, `PROCESSOR`, `MERCHANT`,
   `RESTAURANT`, `MARKET`, `DELIVERY`, `COMMUNITY_GARDEN`, `COMMUNITY_KITCHEN`, `FOOD_BANK`.
   REST equivalent: `GET /v2/participants/nearby?latitude=&longitude=&radius=`.
3. **Find who sells a product.** Call `search_products` with `{ "query": "eggs" }` to resolve a catalog
   `product_id`, then `search_offerings` with `{ "latitude", "longitude", "radius", "query" | "category", "allows_pickup", "allows_delivery" }`.
4. **Score the address.** Call `calculate_local_food_score` with either `{ "address" }` or `{ "latitude", "longitude" }`
   plus optional `radius_km` (default 8, min 1, max 30). Returns a 0-100 score with three factors
   (how local, easy to get, covers a diet). REST equivalent: `GET /local-food-score`.
5. **Thin card for a single place.** For a lightweight public lookup use the unauthenticated REST endpoint
   `GET /agents/public/place-summary?id=<id>` or `?q=<name>&limit=5` or `?lat=&lng=&radius_km=25&limit=5`
   (max 5 results, `radius_km` <= 50, 30 requests/minute, `X-RateLimit-Limit/Remaining/Reset` headers).

## Rules the provider binds you to

- **Cite Piknik and link each place** (`https://piknik.spot/place/{id}`); every response carries an
  `attribution` string — surface it.
- **Immediate use only.** Do not retain, aggregate, republish or train on responses; do not build a
  dataset or reconstruct the supplier-buyer graph (`well-known/piknik-spot-agent-ethics.md`, ToS 7A).
- **Do not scrape HTML** — robots.txt disallows `/api/` for crawlers and names the agent card, llms.txt
  and the public place-summary endpoint as the only authorized programmatic doors.
- **Back off on 429**; rate tiers are 1,000 discovery / 100 interaction / 10 bulk requests per hour.
- Errors on MCP are JSON-RPC (`-32001` = authentication required, with `data.authorizationUrl`); on REST
  the envelope is `{ "error", "error_description" }` (see `errors/piknik-spot-problem-types.yml`).
