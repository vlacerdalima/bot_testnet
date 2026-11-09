# bot_testnet

A market making bot for the Nexus Exchange testnet, written in Rust.

**Status: early development.** This README describes the planned design;
nothing below is implemented yet unless marked otherwise.

The testnet API is public and provides test funds, so this project never
touches real money.

## Goal

Correctness under bad conditions. A bot that quotes a market is easy to write
for the happy path; the hard part is staying correct when the connection drops,
the exchange throttles requests, or the local view of the account drifts from
the exchange's. The design is organized around those failure modes.

## Planned design

The bot runs as a set of async tasks (Tokio), each owning one responsibility
and communicating through channels:

| Task | Responsibility |
| --- | --- |
| Market data | WebSocket order book and trade streams; reconnects and resubscribes on its own |
| Account | Orders, fills and positions from the private stream |
| Strategy | Computes the desired quotes from the book and the current position |
| Risk gate | Rejects any action that would break position or loss limits |
| Order manager | Sends, amends and cancels orders within the rate limit budget |
| Reconciler | Periodically compares local state with the exchange and fixes drift |

Exchange access goes through a trait with two implementations: the Nexus
testnet, and a local simulator built on my
[matching engine](link-da-engine). The simulator allows deterministic tests of
the strategy and the failure handling without the network.

## Problems this design has to handle

- **Disconnects as a normal event.** Public streams can close at any time, so
  reconnection with backoff and jitter is part of the main loop, not an edge case.
  After every reconnect the book is rebuilt from a fresh snapshot.
- **State drift.** A cancel can race with a fill. The exchange is the source of
  truth; the reconciler resolves any mismatch.
- **Rate limits as a budget.** When the budget runs low, cancels take priority
  over new orders.
- **Clock skew.** Signed requests carry a timestamp that must be close to the
  server's time.
- **Shutdown.** A graceful stop cancels all open orders before exiting.

## Strategy

Quotes a bid and an ask at a fixed spread around the mid price, using post only
orders so the bot always provides liquidity. Quotes are skewed by the current
position, and quoting stops when the maximum position is reached.

## Risk controls

- Maximum position per market
- Maximum loss per session
- Kill switch: cancels everything and stops quoting

## Measurement

The bot will record what it actually experiences on the testnet: order
round-trip latency, fill rate, reconnect count and time spent disconnected.
Results will be published here together with the conditions they were measured in.

## Configuration (planned)

Settings through environment variables:

| Variable | Description |
| --- | --- |
| `NEXUS_API_URL` | Testnet REST endpoint |
| `NEXUS_WS_URL` | Testnet WebSocket endpoint |
| `NEXUS_API_KEY` / `NEXUS_API_SECRET` | Testnet credentials |
| `SPREAD_BPS` | Spread in basis points |
| `ORDER_SIZE` | Size of each quote |
| `MAX_POSITION` | Position limit |

## Disclaimer

For testing and learning on the testnet only. Not financial advice.
