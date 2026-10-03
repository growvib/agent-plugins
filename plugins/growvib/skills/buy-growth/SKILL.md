---
name: buy-growth
description: Find, price and buy social media growth on GrowVib (followers, subscribers, members, views, likes, reactions for Instagram, TikTok, YouTube, Telegram, X and more) through the bundled GrowVib MCP tools, with no GrowVib account. Use when the user asks to buy or price social growth for a profile, post, channel or video, or wants a link to pay for such an order themselves.
---

# Buy social media growth on GrowVib

This plugin connects GrowVib's public MCP endpoint, `https://api.growvib.com/mcp-public`. It needs
no account, no API key and no sign-in. Its tools:

| Job | Tool |
|-----|------|
| Find a service | `search_catalog` (by platform or free text), `get_service` (one service with every plan option) |
| Pick an option for a goal | `recommend_service` (one recommended option plus the cheapest, highest-quality and fastest alternatives, with the trade-offs as numbers) |
| Exact price | `get_quote` (never charges) |
| The person pays in their own wallet | `create_checkout`, then `get_checkout_status` |
| You pay from your own x402 wallet | `create_paid_order`, then `get_paid_order_status` and `list_paid_orders` |

## The flow

1. **Find the service.** Use the platform and metric the user named. Prefer
   `recommend_service(platform, goal, quantity, priority?)` when they state a goal ("500 telegram
   members, fast"). Present the real options with their prices and trade-offs and let the user
   choose; do not default to the cheapest.
2. **Quote it.** `get_quote(service_id, quantity, service_option_id?)` returns the exact total.
   Show it to the user. Never compute a price yourself: the quote is authoritative.
3. **Pay.** Pick the path by who pays:
   - **The user pays (the usual case here).** Call `create_checkout(service_id, quantity, link,
     service_option_id?)`. Give the user its `page_url`: a GrowVib page where they connect their
     own USDC wallet on Base or Solana and sign once. Tell them the amount (`amount_usd`) and that
     the price is held until `expires_at` (30 minutes by default). Creating a session charges
     nothing; an unpaid session simply expires.
   - **You hold an x402 wallet and the user wants you to pay.** Call `create_paid_order` without
     a `payment` argument to get the payment requirements, sign exactly what it names, and call
     it again with the signed payment. See
     https://growvib.com/.well-known/agent-skills/growvib-x402/SKILL.md for the full handshake.
4. **Track it.** After handing over a checkout link, poll `get_checkout_status(token)` every few
   seconds until it reports `paid` (it then carries the order id and tracking code) or
   `expired`. For an x402 order, `get_paid_order_status(order_id, agent_token)`.

## The minimum payment

If `amount_usd` is higher than `quote_usd`, the order is priced below the minimum one payment can
settle for, and the session asks for that minimum. Say so before handing over the link, in plain
words: "the order costs $X; the minimum payment is $Y; the difference stays as balance on the
wallet that pays." Do not present it as a fee and do not call it a bug.

## Rules that keep the user safe

- Never guess a `link`. Use the exact profile, post, channel or video URL the user gave.
- Show the total before handing over a checkout link or paying.
- Checkout statuses: `open` is waiting for payment; `pending` means a payment is being confirmed,
  so tell the user to wait and never to pay again; `paid` means the order exists;
  `paid_no_order` means the money arrived as balance on the paying wallet but the order could not
  be placed, so send the user to GrowVib support with the checkout reference; `failed` and
  `expired` charged nothing, so create a new session if the user still wants the order.
- The checkout `token` and `page_url` let whoever holds them pay or read that one session. Give
  them to the user only; do not post them anywhere public.
- Each `create_checkout` call opens a new session. Call it once per intended order and reuse the
  result.
- A failed tool call returns an error whose text says what to fix (an out-of-range quantity, an
  unknown service). Read it and act on it; do not retry blindly.

## Paying from a GrowVib account instead

Account tools (`place_order`, `get_balance`, `add_funds`) live on a different endpoint,
`https://api.growvib.com/mcp`, which signs in with OAuth and pays from the user's GrowVib wallet
balance. They are not on the endpoint this plugin bundles. A user who has an account and wants
that can add it alongside:

```bash
claude mcp add --transport http growvib-account https://api.growvib.com/mcp
```

then run `/mcp`, choose `growvib-account` and authenticate.

## Human pages

- MCP server and every tool: https://growvib.com/mcp
- Paying with USDC, checkout links and x402: https://growvib.com/x402
- Catalog: https://growvib.com/services
