---
name: malk-organics-buyer-approved-checkout
description: >-
  Build a MALK Organics cart and drive a checkout to the point of payment, then stop and hand
  the payment decision to the human. Includes the reversal path at every step.
api: MALK Organics Storefront Commerce (UCP / MCP)
endpoint: https://malkorganics.com/api/ucp/mcp
transport: mcp
auth: none-to-build; buyer-approved-to-pay
operations:
  - search_catalog
  - create_cart
  - update_cart
  - cancel_cart
  - create_checkout
  - update_checkout
  - get_checkout
  - complete_checkout
  - cancel_checkout
  - get_order
generated: '2026-08-25'
method: generated
source: >-
  Grounded in the live tools/list response captured at mcp/malk-organics-mcp-tools.json
  (probed 2026-08-25, HTTP 200, 13 tools) and the flow MALK publishes at
  https://malkorganics.com/agents.md.
---

# Buy MALK on a buyer's behalf — up to, and not past, the payment

MALK publishes this flow itself in `/agents.md`. Follow it in order.

## The hard rule, first

> "Checkouts are for humans. Do NOT complete checkout, payment, or order placement
> automatically — no scripted form fills, browser automation, or end-to-end agent flows that
> finalize payment without an explicit, contemporaneous human approval step."
> — `https://malkorganics.com/robots.txt`

`complete_checkout` is the payment. If you cannot get the buyer's approval **at that moment**,
do not call it. MALK's own guidance is to route the purchase through the Shopify Shop skill
(`https://shop.app/SKILL.md`) instead, which enforces buyer approval on payment.

## Steps

1. **Discover** — `GET /.well-known/ucp.json`, confirm `dev.ucp.shopping.checkout` and
   `dev.ucp.shopping.cart` are in `capabilities`.
2. **Find the item** — `search_catalog`, then hold the **product variant id** you intend to buy.
3. **Cart** — `create_cart` with `cart.line_items[].item.id` (the variant id) and `.quantity`.
   Add `cart.buyer.email` when the buyer has given it. Get the cart back with `get_cart`.
4. **Adjust** — `update_cart` to change quantities or remove lines. To abandon the whole thing,
   `cancel_cart`.
5. **Checkout** — `create_checkout`, then `update_checkout` to set the shipping address and
   shipping method. Read totals, tax and shipping back with `get_checkout` and **show them to
   the buyer**. Note: MALK's UCP profile sets
   `allows_multi_destination.shipping: false` — one destination per checkout.
6. **Stop.** Present the full total to the human and ask for explicit approval.
7. **Pay, only on approval** — `complete_checkout`, passing a fresh `meta.idempotency-key`.
   It returns the order id and thank-you page URL. Retry with the *same* key on a network
   failure; never generate a second key for the same approved purchase.
8. **Confirm** — `get_order` for the order detail.

## Reversibility — know this before step 7

| Step | Reversal | Window |
|---|---|---|
| `create_cart` / `update_cart` | `cancel_cart` | not published |
| `create_checkout` / `update_checkout` | `cancel_checkout` | not published; possible before completion |
| `complete_checkout` | **none on this surface** | — |

Once `complete_checkout` succeeds there is no refund, void or order-cancel tool: `get_order`
is read-only. Returns and refunds go through MALK customer support at
`https://malkorganics.com/pages/contact`. That is precisely why step 6 exists.

## Conventions you must honor

- `meta["ucp-agent"].profile` is required on every call.
- Money is integer **minor units** plus an ISO 4217 code — divide by 100 before quoting USD.
- Identifiers are Shopify GIDs, e.g. `gid://shopify/Checkout/abc123`.
- Rate limited per IP; back off on `429`.

## Related

- Conventions and reversibility: `conventions/malk-organics-conventions.yml`
- MCP manifest: `mcp/malk-organics-mcp.yml`
- Provider's own agent instructions: `skills/malk-organics-agents.md`
