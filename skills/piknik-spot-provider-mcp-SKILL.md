# Piknik Local Food Finder Skill

## Overview
This skill enables AI agents to discover, enrich, and connect local food businesses on Piknik using the MCP server at `https://piknik.spot/api/mcp`.

## When to Use
- Finding farms, markets, restaurants, and processors
- Importing vendors and documenting supply chains
- Linking catalog products to places
- Creating farm/processor → buyer connections
- Regional Local Food Score analysis

## Authentication
- **Public tools** work without auth (search, geocode, analyze).
- **Write tools** need a Personal Access Token (`pik_pat_…`) or OAuth in MCP config.
- Generate tokens at https://piknik.spot/settings/developer

---

## Import workflow (places + products + connections)

Use this sequence when adding businesses to Piknik:

### 1. Create the place
```
suggest_place(
  address, business_name, business_address, business_type,
  context, website, contact_phone
)
```
**Response includes `participant_id`** — save it; do not guess IDs from tracking IDs alone.

### 2. Activate and fix profile
```
update_place(participant_id, status="ACTIVE", description="...", metadata_notes="")
update_place(participant_id, primaryRole="PROCESSOR")   # admin: fix misclassified roles
geocode_place(participant_id)
```

Valid `primaryRole` values: `FARM`, `PROCESSOR`, `MERCHANT`, `RESTAURANT`, `MARKET`, `DELIVERY`, `COMMUNITY_GARDEN`, `COMMUNITY_KITCHEN`, `FOOD_BANK`

### 3. Link catalog products (do not invent duplicate free-text names)
```
list_products(participant_id)              # skip if already listed
search_products(query="eggs")              # get product_id from catalog
create_product(participant_id, product_id=...)   # idempotent — skips duplicates
```

Prefer **`search_products` → `product_id` → `create_product`**. Only pass `name` when no catalog match exists.

### 4. Connect supplier → buyer
```
apply_connection(supplier_id, buyer_id, products=["vegetables"])   # admin, live link
suggest_connection(supplier_id, buyer_id)                            # non-admin, pending review
```

- **Supplier roles:** `FARM` or `PROCESSOR`
- **Buyer roles:** `RESTAURANT`, `MERCHANT`, `MARKET`, or `FARM`
- Only **FARM** supplier links affect Local Food Score today; processor links show on profiles.

---

## Key MCP tools

### Public (no auth)
| Tool | Purpose |
|------|---------|
| `find_places` | Search farms, markets, restaurants by location |
| `list_all_places` | List places by GPS radius; use `status="DRAFT"` to find new suggestions |
| `search_products` | **Catalog lookup** — returns `product_id` for create_product |
| `search_offerings` | Find who sells a product nearby |
| `search_jobs` | Open jobs/volunteer postings (check before create_job) |
| `geocode_address` | Address → coordinates |
| `calculate_local_food_score` | Score an address for local food access |

### Authenticated (write)
| Tool | Purpose |
|------|---------|
| `suggest_place` | Create place → returns **participant_id** |
| `update_place` | Edit profile, **image** URL, status, **primaryRole** (admin) |
| `geocode_place` | Set GPS from address (admin) |
| `list_products` | Offerings already on a place |
| `create_product` | Link catalog product to place (idempotent) |
| `apply_connection` | Live supplier→buyer link (admin) |
| `suggest_connection` | Propose link for review |
| `get_my_places` | Places you manage |
| `create_job` | Post a job from a published opening (admin: any place) |
| `update_job` | Edit or expire a job (`CANCELLED` / `FILLED`) |
| `list_associations` | Find association id/slug |
| `create_association` | Create association (admin) |
| `list_place_tours` | Drafts + published tours you can manage |
| `create_place_tour` / `update_place_tour` / `set_place_tour_stops` | Create, publish, and order stops |

---

## Examples

### Add a farmers market vendor
1. `suggest_place` → note `participant_id`
2. `update_place` ACTIVE + description
3. `geocode_place`
4. `search_products("tomatoes")` → `create_product(product_id=…)`
5. `apply_connection(farm_id, market_id)`

### Fix a misclassified meat packer listed as a farm
```
update_place(participant_id, primaryRole="PROCESSOR", description="Processing plant, not a farm")
search_products("deli meats")
create_product(participant_id, product_id=...)
```

---

## Error handling
- **Duplicate product:** `create_product` returns "Already on This Place" — no action needed.
- **No catalog match:** try a simpler query in `search_products`, or use a vocabulary name (`eggs`, `honey`, `vegetables`).
- **Connection rejected:** check supplier is `FARM` or `PROCESSOR`, buyer is `MARKET`/`MERCHANT`/`RESTAURANT`/`FARM`.

## Resources
- Map: https://piknik.spot/map
- Developer tokens: https://piknik.spot/settings/developer
