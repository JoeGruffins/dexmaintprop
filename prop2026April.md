## Development Recap

Bison Wallet is shipping the v1.1.0 release, a major update with 393 commits from 14+ contributors. All initiatives from the [previous proposal](https://proposals.decred.org/record/963f9fe) were completed, and the release is expected by the end of March or early April. Highlights include:

- **Monero Wallet:**
  - Added Monero wallet support using a CGO-based wallet integration that links directly to the Monero wallet library.

- **EVM Improvements:**
  - Added bridge UI for cross-chain transfers between EVM networks.
  - Improved wallet UI for tokens sharing the same parent wallet, making multi-network assets easier to manage.

- **Market Making:**
  - Added Bitget and MEXC exchange adapters, completing the MEXC integration from the previous proposal.
  - Added server-side epoch snapshots and a liquidity verification tool (`verifymm`) for proving market making activity.
  - Added epoch reporting for bot performance analysis with profit tracking per run.

- **Politeia Governance:**
  - Integrated Politeia proposal voting into the wallet, allowing users to vote on Politeia proposals with staked DCR directly from Bison Wallet.

- **Trade Reliability:**
  - Fixed infinite refund loops, added broadcast recovery for stuck transactions, SPV rescan stall detection, and mempool transaction recovery.
  - Added per-match swap addresses, a protocol-level change that improves swap isolation.
  - Added secret hash deduplication on the server to prevent replay attacks.

- **Infrastructure:**
  - Updated to Go 1.24, modernized codebase with `slices`, `maps`, `any`, and built-in `max`/`min`.
  - Migrated the macOS desktop app from WebView to Electron for improved stability.
  - Refactored and updated many testing tools to help catch bugs and improve performance.
  - Added wallet timeouts, active trade limits, slippage warnings for market orders, and fiat value display in trade forms.
  - Hardened WebSocket handling and comms rate limiting.
  - Fixed numerous bugs across wallets, market making, and UI.

## Core Development

Bison Wallet has a large and mature codebase that requires ongoing maintenance. This proposal ensures continued bug fixes, dependency updates, release management, and module upkeep.

We anticipate costs in this area will not exceed $50,000 USD.

## Team

Work will be performed by Bisoncraft LLC under the leadership of Joe, Marton, and Buck.

## Summary

We are requesting a maximum of $50,000 USD for six months of continued core development and operational costs. Historically, not all allocated funds have been utilized, and any unused funds will remain in the treasury.
