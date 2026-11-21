# Architecture

A high-level map of how Zuz is put together. Component level only — this is not
an index of the source.

## Shape

Zuz is two independent Node.js services over one Postgres database, plus a set
of small signal watchers.

```
   Telegram                      Telegram Mini App
   @ZuzTradeBot                  app.zuztrade.com
        │                               │
        ▼                               ▼
  ┌───────────┐                   ┌───────────┐
  │    Bot    │                   │ Mini App  │
  │  service  │                   │    API    │
  └─────┬─────┘                   └─────┬─────┘
        │                               │
        │   shared wallet + trade code  │
        └───────────────┬───────────────┘
                        ▼
        ┌───────────────────────────────┐
        │  Postgres   ·   Redis   ·  KMS │
        └───────────────┬───────────────┘
                        ▼
  Robinhood · Ethereum · Base · Arbitrum · Solana · Igra · Kaspa
```

**Bot service.** The Telegram bot: wallets, trading, exit orders, limit buys,
signal delivery and withdrawals. It also runs the position monitor that watches
open trades and fires exits, and the dispatcher behind automated actions.

**Mini App API.** The HTTP API behind the Telegram Mini App. It is a **separate
process**. It imports the shared wallet and trade modules but deliberately never
runs the monitor, the dispatcher or Telegram polling, so there is exactly one
process responsible for firing exits.

**Signal watchers.** Small independent services, one per chain, that watch for
newly created pools and filter them on age, liquidity, volume and buy/sell
counts before anything is published. Community calls arrive on a separate path
and are republished credited to their author.

## Stack

| Layer | Choice |
|---|---|
| Language | TypeScript on Node.js |
| Telegram | grammY |
| Database | Postgres |
| Cache, locks, queues | Redis |
| EVM | ethers v6 |
| Solana | @solana/web3.js |
| Key wrapping | Google Cloud KMS |
| Mini App | Expo / React Native Web |
| Hosting | Google Cloud |

## Wallets

Each user gets one BIP-39 recovery phrase, generated from a CSPRNG. Every chain
wallet is derived from it:

- **Solana** — SLIP-0010
- **EVM** — BIP-44, one key shared across Robinhood Chain, Ethereum, Base,
  Arbitrum and Igra
- **Kaspa L1** — its own derivation

Phrases and derived keys are held only as encrypted blobs. Envelope encryption
is used: each secret gets a fresh random data key, and that data key is wrapped
by a Google Cloud KMS key whose material never leaves Google. Secrets are
decrypted in memory only at the moment of signing or export.

See [`SECURITY.md`](SECURITY.md) for the disclosure policy and the custody
model.

## Trading

Each chain routes through its own venue — Uniswap on Robinhood Chain, Ethereum,
Base and Arbitrum, Jupiter on Solana, Zealous on Igra. Above that sits one
common layer that every chain shares:

**Exit orders.** Take-profit ladders, stop-loss ladders, a trailing
take-profit, a time exit and an anti-rug exit. Each rung sells a percentage of
the remaining position, so ladders compose rather than conflict. Take-profit
targets are evaluated against a quoted sell net of fees, not against raw price.

**One action per position.** A database-level advisory lock per position covers
manual sells from either surface and automatic exits, so the monitor and a user
cannot sell the same position twice. Limit buys are claimed atomically and fire
once.

**Paper mode.** A full parallel execution path against real live prices with
simulated balances, running the same exit ladder as real trades.

## Bridges

- **Between Ethereum, Base, Arbitrum, Robinhood Chain and Solana** via Relay
- **Between Kaspa and its L2s** (Igra, Kasplex)

Bridges carry no Zuz fee.

Funds only ever move between the user's own wallets: the destination is always
the user's own address on the other chain, and every quote is checked —
sender, recipient, amount and both chains — before anything is signed.

## Data

Postgres holds users, wallets (encrypted), positions, orders, trades, fee rows
and the signal register. Redis carries caches, rate limits and short-lived
locks. Schema changes ship as ordered migrations.

Settings are environment variables; no credential is held in the repository or
in the image.
