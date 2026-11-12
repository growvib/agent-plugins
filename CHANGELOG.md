# Changelog

All notable changes to the plugins in this marketplace. Versions are the plugin's own
(`plugins/<name>/.claude-plugin/plugin.json`); installed copies pick a new version up on update.

## growvib 0.2.0 (2026-11-12)

The `buy-growth` skill now describes the public endpoint's tools as they are today:

- `search_catalog` takes `quantity` and reports `options_fit` beside `options_total`; `get_service`
  accepts the catalog slug as well as the id; `recommend_service` requires `quantity`.
- Before quoting, read what each option says about itself: `link_type` (which link the order
  needs), `requires_posts` (a per-post subscription, ordered on the channel or profile with a
  `posts` count), `quantity_meaning` (multi-post options) and `sensitive`.
- `get_quote` and `create_paid_order` take `posts`; money is quoted to six decimals; balance orders
  pass `idempotency_key` as a tool argument.
- A "Reading delivery" section: `remains` is 0 once an order is completed, no completion time is
  ever estimated, `start_overdue` is the only stall signal, `waiting_for_post` on a subscription.
- The duplicate rule as it is: one live order per service and link.

## growvib 0.1.0 (2026-10-03)

First release: the `growvib` plugin bundling the public MCP endpoint and the `buy-growth` skill
(find, quote, checkout link or x402, track).
