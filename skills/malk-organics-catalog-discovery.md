---
name: malk-organics-catalog-discovery
description: >-
  Search and read the MALK Organics product catalog over the store's live UCP/MCP endpoint,
  without any credentials, and quote prices to a buyer correctly.
api: MALK Organics Storefront Commerce (UCP / MCP)
endpoint: https://malkorganics.com/api/ucp/mcp
transport: mcp
auth: none
operations:
  - search_catalog
  - lookup_catalog
  - get_product
generated: '2026-08-25'
method: generated
source: >-
  Grounded in the live tools/list response captured at mcp/malk-organics-mcp-tools.json
  (probed 2026-08-25, HTTP 200, 13 tools) and https://malkorganics.com/agents.md.
---

# Read the MALK Organics catalog

MALK Organics sells organic, clean-label plant-based milks and creamers. Its Shopify UCP/MCP
endpoint answers catalog reads anonymously — no key, no signup, no quota.

## Before you call anything

Every tool requires a `meta` object carrying your agent's UCP profile URI:

```json
{ "meta": { "ucp-agent": { "profile": "https://your-agent.example/ucp-profile" } } }
```

A call without `meta["ucp-agent"].profile` will not validate.

## Steps

1. **Confirm the surface.** `GET https://malkorganics.com/.well-known/ucp.json` and check
   `ucp.version` (currently `2026-04-08`) and that `services["dev.ucp.shopping"]` is present.
2. **Search.** Call `search_catalog` with `catalog.query` (natural language) and/or
   `catalog.filters`. At least one of the two is required. Pass
   `catalog.context.address_country`, `.currency` and `.language` so pricing and availability
   are localized.
3. **Page.** Results are truncated by default. Read `pagination.cursor` from the response and
   pass it back on the next `search_catalog` call — only when the buyer asks for more.
4. **Resolve identifiers.** Use `lookup_catalog` to resolve several product or variant
   identifiers at once, or `get_product` for full detail on a single one.

## Rules that will bite you

- **Money is in minor units.** `{"amount": 600, "currency": "USD"}` is **$6.00**, not $600.
  Divide by 100 for two-decimal currencies; JPY and other zero-decimal currencies are already
  whole units. Convert before you say a number to a buyer.
- **Do not scrape instead.** MALK's `robots.txt` disallows `/cart.js` and
  `/recommendations/products` and tells agents to use UCP/MCP for catalog, cart and checkout.
- **Do not invent product facts.** MALK's `/llms.txt` explicitly asks that AI systems not
  fabricate ingredient lists, nutrition claims or pricing, and not present seasonal or
  limited-edition items as permanently available. If the tool did not return it, do not say it.
- **Rate limits are per IP** and undocumented in size. Back off on `429`.

## Related

- Conventions: `conventions/malk-organics-conventions.yml`
- Rate limits: `rate-limits/malk-organics-rate-limits.yml`
- Provider's own agent instructions: `skills/malk-organics-agents.md`
