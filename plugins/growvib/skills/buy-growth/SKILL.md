---
name: buy-growth
description: Find, price and buy social media growth on GrowVib (followers, subscribers, members, views, likes, reactions for Instagram, TikTok, YouTube, Telegram, X and more) through the bundled GrowVib MCP tools, with no account. Hand the user a checkout link they pay in their own USDC wallet, or pay over x402 from the agent's wallet. Covers per-post subscriptions, which link each option takes, and how to read delivery.
---

# Buy social media growth on GrowVib

This plugin connects GrowVib's public MCP endpoint, `https://api.growvib.com/mcp-public`. It needs
no account, no API key and no sign-in. Its tools:

| Job | Tool |
|-----|------|
| Find a service | `search_catalog` (by platform or free text; pass `quantity` to see only the options that accept that amount), `get_service` (one service with every option; takes the id or the slug, e.g. `telegram-members`) |
| Pick an option for a goal | `recommend_service` (`quantity` is required; one recommended option plus the cheapest, highest-quality and fastest alternatives, with the trade-offs as numbers) |
| Exact price | `get_quote` (never charges) |
| The person pays in their own wallet | `create_checkout`, then `get_checkout_status` |
| You pay from your own x402 wallet | `create_paid_order`, then `get_paid_order_status` and `list_paid_orders` |

Every tool refuses an argument it does not take and names the ones it accepts. Read the error and
fix the name; do not retry blindly.

## The flow

1. **Find the service.** Use the platform and metric the user named. Prefer
   `recommend_service(platform, goal, quantity, audience?, priority?)` when they state a goal
   ("500 telegram members, fast"). Present the real options with their prices and trade-offs and
   let the user choose; do not default to the cheapest. For a small buyer, `search_catalog` with
   `quantity` lists only the options whose minimum and maximum fit, with `options_fit` beside
   `options_total`; each service shows at most 8 options (the recommended ones first), and
   `options_truncated` says when some were cut, so call `get_service` for the full list.
2. **Read what the option says about itself** before quoting. Every option carries:
   - `link_type`: the kind of link the order needs. `post` means that post's own link, `channel`
     the channel's link (for Telegram `https://t.me/<channel>`), `profile` the profile, `story` the
     story, `comment` the comment's own permalink. A link of the wrong kind is refused before any
     money moves.
   - `requires_posts`: a per-post subscription. Its quantity is the amount delivered to EACH of the
     target's next posts, it needs `posts` (how many posts to cover), its link is the channel or
     profile itself (never one post), and the price is posts times the quoted amount.
   - `quantity_meaning`: set when the quantity is not simply "this many units now", for example an
     option that spreads one order across the target's next N posts. Treat such an option as a
     different product from a single-post one when comparing prices.
   - `sensitive`: a deliberate product such as a negative reaction mix. `recommend_service` never
     leads with one while a plain option fits; offer it only when the user asked for it.
3. **Quote it.** `get_quote(service_id, quantity, service_option_id?, posts?)` returns the exact
   total. Show it to the user. Never compute a price yourself: the quote is authoritative. Money is
   quoted to six decimals, so a small order can cost less than a cent and still be exact. Pass the
   option's full id from `options[].id`; a shortened id is refused.
4. **Pay.** Pick the path by who pays:
   - **The user pays (the usual case here).** Call `create_checkout(service_id, quantity, link,
     service_option_id?)`. Give the user its `page_url`: a GrowVib page where they connect their
     own USDC wallet on Base or Solana and sign once. Tell them the amount (`amount_usd`) and that
     the price is held until `expires_at` (30 minutes by default). Creating a session charges
     nothing; an unpaid session simply expires.
   - **You hold an x402 wallet and the user wants you to pay.** Call `create_paid_order` without
     a `payment` argument to get the payment requirements, sign exactly what it names, and call
     it again with the signed payment; pass `posts` beside `quantity` for a per-post subscription.
     The order returns an `agent_token`: later orders paid from the balance that wallet holds pass
     it as `agent_token` with an `idempotency_key` (a tool argument here, a JSON body field on the
     REST endpoint, never a header) and no payment. Full handshake:
     https://growvib.com/.well-known/agent-skills/growvib-x402/SKILL.md.
5. **Track it.** After handing over a checkout link, poll `get_checkout_status(token)` every few
   seconds until it reports `paid` (it then carries the order id and tracking code) or
   `expired`. For an x402 order, `get_paid_order_status(order_id, agent_token)`.

## Reading delivery

The tracking read states what is known and nothing more:

- `remains` counts down as units arrive and reads 0 once the order is `COMPLETED`. Delivery is
  gradual after it starts, and GrowVib never estimates a completion time, so a falling `remains`
  is progress, not a stall.
- `delivery.listed_start` is the start window the option listed and `delivery.start_within_minutes`
  its bound. `delivery.started` says delivery has begun. `delivery.start_overdue` is the only stall
  signal: it is set while an active order has not started past that bound, and never once delivery
  has begun. Report an order as stuck only when it is true.
- On a per-post subscription the clock runs from the target's next post, not from the order:
  `delivery.waiting_for_post` is true until the first post is covered, `start_overdue` is never
  set, and the read carries `posts`, the count the order covers.

## The minimum payment

If `amount_usd` is higher than `quote_usd`, the order is priced below the minimum one payment can
settle for, and the session asks for that minimum. Say so before handing over the link, in plain
words: "the order costs $X; the minimum payment is $Y; the difference stays as balance on the
wallet that pays." Do not present it as a fee and do not call it a bug.

## Rules that keep the user safe

- Never guess a `link`. Use the exact profile, post, channel or video URL the user gave, and match
  it to the option's `link_type`.
- Show the total before handing over a checkout link or paying.
- One live order per service and link: a second order for the same service on a link that is
  still being delivered is refused as `duplicate_order` with the `existing_order_id`, and nothing
  is spent. Different services on the same link (views and reactions on one post) may run
  together.
- Checkout statuses: `open` is waiting for payment; `pending` means a payment is being confirmed,
  so tell the user to wait and never to pay again; `paid` means the order exists;
  `paid_no_order` means the money arrived as balance on the paying wallet but the order could not
  be placed, so send the user to GrowVib support with the checkout reference; `failed` and
  `expired` charged nothing, so create a new session if the user still wants the order.
- The checkout `token` and `page_url` let whoever holds them pay or read that one session. Give
  them to the user only; do not post them anywhere public. The same goes for an `agent_token`.
- Each `create_checkout` call opens a new session. Call it once per intended order and reuse the
  result.
- A failed tool call returns an error whose text says what to fix (an out-of-range quantity, an
  unknown service, a misnamed argument, a missing `posts`). Read it and act on it.

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
