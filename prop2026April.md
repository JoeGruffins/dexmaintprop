## Development Recap

Since [our last proposal](https://proposals.decred.org/record/963f9fe), Bison Wallet has shipped the v1.1.0 release, a major update with 393 commits from 14+ contributors. All initiatives from the previous proposal were completed and shipped. Highlights include...

- **Monero Wallet:**
  - Added Monero wallet support with automatic download of required client tools.
  - Reworked wallet creation flow with a new Opener interface to support Monero's CGO-based integration.

- **EVM Improvements:**
  - Deployed a new v1 swap contract with gasless redemptions using EIP-712 relayed transactions, eliminating the need for the redeemer to hold native tokens for gas.
  - Added Base network support (OP Stack L2) with L1 fee accounting.
  - Added cross-chain bridging via Across Protocol between Ethereum and Polygon PoS, with bridge UI and market making integration for cross-chain arbitrage.
  - Changed native Polygon asset from MATIC to POL.

- **Private Atomic Swaps:**
  - Implemented adaptor signature atomic swaps for BTC and DCR with bundled libsecp256k1 for DLEQ proofs. This is ready for use once Mesh is released.

- **Market Making:**
  - Added Bitget and MEXC exchange adapters, completing the MEXC integration from the previous proposal.
  - Added multi-hop arbitrage support, allowing market makers to operate on markets without an exact CEX match.
  - Added server-side epoch snapshots and a liquidity verification tool (`verifymm`) for proving market making activity.
  - Live config and balance updates for running bots without restart.
  - Epoch reporting for bot performance analysis with profit tracking per run.

- **Politeia Governance:**
  - Integrated Politeia proposal voting into the wallet, allowing users to vote on Politeia proposals with staked DCR directly from Bison Wallet.

- **Desktop and Mobile:**
  - Migrated the desktop app from WebView to Electron for improved stability and cross-platform support.
  - Developed a companion app with Tor routing for remote access to a home Bison Wallet instance.

- **Trade Reliability:**
  - Fixed infinite refund loops, added broadcast recovery for stuck transactions, SPV rescan stall detection, and mempool transaction recovery.
  - Added per-match swap addresses, a protocol-level change that improves swap isolation.
  - Added secret hash deduplication on the server to prevent replay attacks.

- **Infrastructure:**
  - Introduced Lexi, a new database package based on Badger DB, replacing BoltDB for transaction history.
  - Updated to Go 1.25, modernized codebase with `slices`, `maps`, `any`, and built-in `max`/`min`.
  - Forked btcwallet into utxowallet and neutrino for independent maintenance.
  - Created a pure-Go Electrum client for lightweight native wallets.

- **Other:**
  - Added wallet timeouts, active trade limits, slippage warnings for market orders, and fiat value display in trade forms.
  - Hardened WebSocket handling and comms rate limiting.
  - Fixed numerous bugs across wallets, market making, and UI.

## Core Development

Bison Wallet has a large and mature codebase that requires ongoing maintenance. This proposal ensures continued bug fixes, dependency updates, release management, and module upkeep.

We anticipate costs in this area will not exceed $50,000 USD.

## New Development Initiatives

- **Split Ticket Staking:** Implement split ticket purchasing directly in the Bison Wallet DCR wallet. Split tickets allow multiple users to pool funds to purchase a staking ticket, making staking accessible to users who cannot afford the full ticket price. This was previously available through external tooling but will now be a native wallet feature. The initial implementation will use a coordination server to manage sessions, match participants, build the multi-participant ticket transaction, and collect signatures. Once Mesh is online, the coordination will be migrated to use the decentralized Mesh network, removing the need for a centralized server.

We anticipate costs in this area will not exceed $10,000 USD.

## Team

Work will be performed by Bisoncraft LLC under the leadership of Joe, Marton, and Buck.

## Summary

We are requesting a maximum of $60,000 USD for six months of continued core development and new initiatives. Historically, not all allocated funds have been utilized, and any unused funds will remain in the treasury.
