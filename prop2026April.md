## Development Recap

Bison Wallet is shipping the [v1.1.0 release](https://github.com/decred/dcrdex/blob/master/docs/release-notes/release-notes-1.1.0.md), a major update with 400+ commits from 60+ contributors. 958 files were changed with 169,000+ insertions and 40,000+ deletions. All initiatives from the [previous proposal](https://proposals.decred.org/record/963f9fe) were completed, and the release is expected by the end of March or early April. Highlights include:

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

## What This Proposal Covers

This proposal funds the ongoing engineering work required to keep Bison Wallet functional, secure, and releasable. Bison Wallet is a large, multi-platform application with 377,000+ lines of Go and 56,000+ lines of TypeScript/JavaScript across 689 Go source files. It integrates with 13 independent blockchains (BTC, DCR, LTC, BCH, ETH, Polygon, XMR, ZEC, FIRO, DASH, DOGE, DGB, ZCL) plus ERC-20 tokens and OP Stack L2 networks. It ships on Linux, macOS, and Windows, with an Android companion app for remote access to a running Bison Wallet instance.

This is not speculative feature work. This is the base layer of engineering without which the product stops working. The scope includes:

### Dependency Management

The project has 82 direct Go dependencies and 199 total (including transitive), spread across 3 Go modules. These include upstream blockchain libraries (`btcd`, `go-ethereum`, `ltcd`, `bchd`, `go-monero`), cryptographic libraries, networking stacks, and database engines. Frontend dependencies include Electron 38, React 18, Webpack 5, and Hardhat for smart contract tooling.

Each upstream blockchain library follows its own release cycle. When Bitcoin Core, Geth, or any other upstream node software releases updates, the corresponding Go libraries must be updated and tested against the Bison Wallet integration. Failing to keep up means wallet breakage, transaction failures, or security vulnerabilities. This is not optional work - it is a requirement for the software to remain operational.

Go itself releases new versions on a regular cadence. The v1.1.0 release required updating to Go 1.24. Future Go releases will require the same attention, including adapting to deprecations, API changes, and toolchain updates.

NPM dependencies require regular audits and updates to address security advisories.

### Release Management

Cutting a release for Bison Wallet is a significant effort. The v1.1.0 release involves a protocol-breaking upgrade where v1.0.x clients cannot trade on v1.1.0 servers. Active swaps must be revoked, server operators must coordinate upgrades, and EVM smart contracts must be migrated from v0 to v1. Each release requires:

- Release candidate builds and testing across all supported platforms.
- Simnet integration testing across 24 trading pair configurations and 11 test scenarios using dedicated test harnesses for each supported asset.
- Patch releases for issues found post-release (e.g., the Electrum wallet regression in v1.1.0 that requires a follow-up patch).
- Coordinating with server operators on upgrade timing.

### Blockchain Maintenance

Each of the 13 blockchain integrations is effectively a standalone wallet implementation with its own transaction construction, fee estimation, address handling, and swap contract logic. When any of these blockchains undergo consensus changes, hard forks, or network upgrades, the corresponding Bison Wallet integration must be updated and tested. This includes:

- Monitoring upstream chain changes and adapting wallet code.
- Updating fee estimation as network conditions change.
- Maintaining SPV (lightweight) wallet support where applicable.
- Keeping testnet and simnet harnesses operational for each chain.

EVM chains add additional complexity. Bison Wallet deploys and interacts with on-chain swap contracts on Ethereum, Polygon, and Base. Gas estimation, L1 fee handling for L2 networks, and contract versioning all require ongoing attention.

### Smart Contract Operations

Bison Wallet has deployed swap contracts on multiple EVM networks. The gasless redeem system uses EIP-712 signed relay transactions, requiring a relay service that must be maintained and monitored. Contract upgrades (like the v0 to v1 migration in this release) require careful coordination, testing with Hardhat, and deployment across multiple networks.

### Security and Bug Fixes

The codebase handles real money across multiple blockchains. Security issues in swap logic, wallet handling, or network communication can result in lost funds. This proposal covers:

- Responding to security disclosures and vulnerability reports.
- Fixing bugs as they are discovered in production use.
- Addressing the 325+ TODO/FIXME items across 131 files, many of which flag known edge cases in swap settlement, wallet recovery, and error handling.
- Maintaining and expanding the 191 test files and CI pipeline (5 GitHub Actions workflows testing across Go 1.24/1.25 and Node 20/22).

### Platform Support

Bison Wallet ships as a desktop application (Electron on macOS, WebView on Linux/Windows) and a web application, with an Android companion app for remotely viewing and managing a running instance. Each platform has its own build pipeline, packaging requirements (deb, rpm, dmg, snap), and platform-specific bugs. The recent migration from WebView to Electron on macOS was driven by Apple breaking WebView functionality - this kind of platform maintenance is unpredictable but unavoidable.

### CI/CD and Testing Infrastructure

The project maintains 5 CI/CD workflows, 16 asset-specific test harnesses, and a simnet trade testing suite that exercises real atomic swaps across all supported trading pairs. Keeping this infrastructure working as dependencies and platforms evolve is ongoing work.

## Planned Work

The project [issue tracker](https://github.com/decred/dcrdex/issues) has 154 open issues with 1,033 closed to date. Open issues range from bug reports to feature requests and UX improvements. In addition to working through these, the following items are planned targets for this period:

- **Patch release for v1.1.0** - Fix the Electrum wallet regression (BTC, LTC, FIRO currently disabled) and address issues found during RC testing.
- **Monero improvements** - Continued development of the XMR integration, including ARM64 support that was recently added and further stability work.
- **Market making reliability** - Ongoing fixes for CEX adapter sync issues (Binance, Bitget, MEXC, Coinbase) and balance tracking.
- **Translation and localization** - Completing missing translations and maintaining localization infrastructure.
- **Documentation updates** - Keeping wiki, API docs, and contributor guides current.
- **Companion app hardening** - Security improvements for the Android companion app.

## What This Proposal Does Not Cover

This proposal does not cover Tatanka Mesh development or other major feature initiatives. Those are separate efforts with their own proposals. This proposal covers the base engineering work that keeps the existing product functional and secure - work that is required regardless of what new features are being developed on top.

## Team

Work will be performed by Bisoncraft LLC under the leadership of Joe, Marton, and Buck.

## Summary

We are requesting a maximum of $50,000 USD for six months of continued core development and operational costs. This covers dependency management, release engineering, blockchain maintenance, security response, bug fixes, platform support, and CI/CD upkeep for a 433,000+ line codebase integrating 13 blockchains across 3 desktop platforms plus a mobile companion app. Historically, not all allocated funds have been utilized, and any unused funds will remain in the treasury.
