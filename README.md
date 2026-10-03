# GrowVib agent plugins

Plugins that let AI agents find, price and buy social media growth on
[GrowVib](https://growvib.com): followers, subscribers, members, views, likes and reactions for
Instagram, TikTok, YouTube, Telegram, X and more.

This repository is a Claude Code plugin marketplace. It holds one plugin today, `growvib`.

## Install

In a terminal:

```bash
claude plugin marketplace add GrowVib/agent-plugins
claude plugin install growvib@growvib
```

Or inside a Claude Code session:

```text
/plugin marketplace add GrowVib/agent-plugins
/plugin install growvib@growvib
```

No account, API key or sign-in is needed. Run `/mcp` to confirm the `growvib` server shows as
connected.

## What you get

- **The GrowVib public MCP server** (`https://api.growvib.com/mcp-public`), with its tools:
  - find and price: `search_catalog`, `get_service`, `recommend_service`, `get_quote`
  - let the user pay in their own wallet: `create_checkout`, `get_checkout_status`
  - pay from the agent's own x402 wallet: `create_paid_order`, `get_paid_order_status`,
    `list_paid_orders`
- **The `buy-growth` skill**, which teaches Claude the flow: find the service, quote it, hand the
  user a checkout link (or pay over x402), and track the order. Claude uses it automatically when
  you ask to buy growth, or you can invoke it as `/growvib:buy-growth`.

Then just ask:

```text
Find 1,000 Instagram followers for https://instagram.com/yourprofile and give me a link to pay.
```

Claude searches the catalog, shows you the options and the exact price, and gives you a GrowVib
checkout page. You pay there in USDC on Base or Solana from your own wallet, and Claude reports
the order and its tracking code once it is paid.

## How payment stays safe

- Nothing is charged until you sign in your own wallet. Creating a checkout link moves no money,
  and an unpaid link simply expires.
- The price is computed by GrowVib and held for the session (30 minutes by default). The model
  never handles a key, a signature or a card.
- A link that is paid, or whose payment is still being confirmed, can never be paid twice.

## Already have a GrowVib account?

The account tools (`place_order`, `get_balance`, `add_funds`) pay from your GrowVib wallet
balance and live on a separate endpoint that signs in with OAuth. Add it alongside the plugin:

```bash
claude mcp add --transport http growvib-account https://api.growvib.com/mcp
```

then run `/mcp`, choose `growvib-account` and authenticate.

## Links

- MCP server and every tool: https://growvib.com/mcp
- Paying with USDC, checkout links and x402: https://growvib.com/x402
- MCP server card: https://api.growvib.com/.well-known/mcp/server-card.json
- Official MCP registry entry: `com.growvib/growvib`

## License

[MIT](LICENSE)
