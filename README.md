# bot_testnet

A trading bot that runs on the **Nexus Exchange testnet**. The testnet API is public and provides test funds, so nothing here touches real money.

The goal of this project is **robustness**: a bot that keeps running correctly when the network, the exchange, or the market misbehaves.

## Features

- **Market data via WebSocket** – consumes live order book / trade streams.
- **Automatic reconnection** – recovers from dropped connections with backoff and resubscribes on its own.
- **Rate limit awareness** – throttles requests to stay within the exchange's limits.
- **Position and PnL tracking** – keeps track of current position, realized and unrealized PnL.
- **Simple strategy** – market making with a fixed spread around the mid price.

## Stack

The Nexus SDKs are available in Rust, Python and TypeScript at [github.com/nexus-xyz](https://github.com/nexus-xyz). The language used here is: _TBD_.

## Getting started

```bash
git clone <this-repo>
cd bot_testnet
cp .env.example .env   # add your testnet credentials
```

Then install dependencies and run the bot (commands depend on the chosen language).

## Configuration

Settings are provided through environment variables (see `.env.example`):

| Variable | Description |
| --- | --- |
| `NEXUS_API_URL` | Testnet REST endpoint |
| `NEXUS_WS_URL` | Testnet WebSocket endpoint |
| `NEXUS_API_KEY` / `NEXUS_API_SECRET` | Testnet credentials |
| `SPREAD_BPS` | Fixed spread in basis points |
| `ORDER_SIZE` | Size of each quote |

## Strategy

The bot quotes a bid and an ask at a fixed spread around the mid price, refreshing quotes as the market moves, and limits its exposure through a maximum position size.

## Disclaimer

For testing and learning on the testnet only. Not financial advice.
