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

## New Development Initiative

**Split Ticket Staking:** Implement split ticket purchasing directly in the Bison Wallet DCR wallet. Split tickets allow multiple users to pool funds to purchase a staking ticket, making staking accessible to users who cannot afford the full ticket price. This was previously available through external tooling but will now be a native wallet feature. The initial implementation will use a coordination server to manage sessions, match participants, build the multi-participant ticket transaction, and collect signatures. Once Mesh is online, the coordination will be migrated to use the decentralized Mesh network, removing the need for a centralized server.

We anticipate costs in this area will not exceed $25,000 USD.

**Fiat On-Ramp:** Integrate a third-party fiat on-ramp service (such as Alchemy Pay) into Bison Wallet, allowing users to purchase cryptocurrency directly within the wallet using credit cards, bank transfers, and other traditional payment methods. The on-ramp provider handles all KYC/AML compliance and payment processing. Our integration work covers the UI, API integration, supported asset and region filtering, and testing.

We anticipate costs in this area will not exceed $10,000 USD.

## Code Signing

Bison Wallet releases are currently signed by the Decred organization, but this adds delays when shipping patches and updates. By obtaining our own code signing certificates for Windows and macOS, we can release updates independently and get fixes to users faster. An Apple Developer membership is required for macOS signing and notarization, and a code signing certificate is required for Windows.

We anticipate ongoing annual costs in this area will not exceed $1,000 USD.

## Team

Work will be performed by Bisoncraft LLC under the leadership of Joe, Marton, and Buck.

## Summary

We are requesting a maximum of $86,000 USD for six months of continued core development, new initiatives, and operational costs. Historically, not all allocated funds have been utilized, and any unused funds will remain in the treasury.
