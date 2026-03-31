## Development Recap

Bison Wallet [v1.1.0](https://github.com/decred/dcrdex/blob/master/docs/release-notes/release-notes-1.1.0.md) is at release candidate 2, a major update with 400+ commits from 60+ contributors. 958 files were changed with 169,000+ insertions and 40,000+ deletions. All initiatives from the [previous proposal](https://proposals.decred.org/record/963f9fe) were completed. Highlights include:

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

This proposal funds ongoing maintenance and new development for Bison Wallet. Bison Wallet is a large, multi-platform application with a 433,000+ line codebase (377,000+ lines of Go and 56,000+ lines of TypeScript/JavaScript) across 689 Go source files. It integrates with 13 independent blockchains (BTC, DCR, LTC, BCH, ETH, Polygon, XMR, ZEC, FIRO, DASH, DOGE, DGB, ZCL) plus ERC-20 tokens and OP Stack L2 networks. It ships on Linux, macOS, and Windows, with an Android companion app for remote access to a running Bison Wallet instance.

The maintenance scope includes:

### Dependency Management

The project has 82 direct Go dependencies and 199 total (including transitive), spread across 3 Go modules. These include upstream blockchain libraries (`btcd`, `go-ethereum`, `ltcd`, `bchd`, `go-monero`), cryptographic libraries, networking stacks, and database engines. Frontend dependencies include Electron 38, React 18, Webpack 5, and Hardhat for smart contract tooling.

Each upstream blockchain library follows its own release cycle. When Bitcoin Core, Geth, or any other upstream node software releases updates, the corresponding Go libraries must be updated and tested against the Bison Wallet integration. Failing to keep up means wallet breakage, transaction failures, or security vulnerabilities. This is not optional work - it is a requirement for the software to remain operational.

Go itself releases new versions on a regular cadence. The v1.1.0 release required updating to Go 1.24. Future Go releases will require the same attention, including adapting to deprecations, API changes, and toolchain updates.

NPM dependencies require regular audits and updates to address security advisories.

### Release Management

Cutting a release for Bison Wallet is a significant effort. The v1.1.0 release involves a protocol-breaking upgrade where v1.0.x clients cannot trade on v1.1.0 servers. Active swaps must be revoked, server operators must coordinate upgrades, and EVM smart contracts must be migrated from v0 to v1. Each release requires:

- Release candidate builds and testing across all supported platforms.
- Simnet integration testing across 24 trading pair configurations and 11 test scenarios using dedicated test harnesses for each supported asset.
- Patch releases for issues found post-release.
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

## Planned Maintenance

The project [issue tracker](https://github.com/decred/dcrdex/issues) has 154 open issues with 1,033 closed to date. Open issues range from bug reports to feature requests and UX improvements. In addition to working through these, the following items are planned targets for this period:

- **Final release of v1.1.0** - Remaining work includes addressing issues found during RC testing.
- **Bug fixes** - Open bugs include an invalid passphrase error on startup ([#3574](https://github.com/decred/dcrdex/issues/3574)), a gasless relay race condition ([#3572](https://github.com/decred/dcrdex/issues/3572)), fund availability issues when posting trades ([#3512](https://github.com/decred/dcrdex/issues/3512)), and USDC/Polygon wallet irregularities ([#3457](https://github.com/decred/dcrdex/issues/3457)).
- **Companion app** - The companion app currently cannot use some endpoints ([#3583](https://github.com/decred/dcrdex/issues/3583)).
- **Market making reliability** - Ongoing fixes for CEX adapter sync issues including MEXC arb errors ([#3466](https://github.com/decred/dcrdex/issues/3466)) and balance tracking.
- **Monero improvements** - Continued development of the XMR integration, including ARM64 support that was recently added and further stability work.
- **Translation and localization** - Completing missing translations and maintaining localization infrastructure.
- **Documentation updates** - Keeping wiki, API docs, and contributor guides current.

We anticipate maintenance costs will not exceed $100,000 USD.

This proposal does not cover Tatanka Mesh development. That is a separate effort with its own proposal.

## New Development Initiatives

- **Split Ticket Staking:** Implement split ticket purchasing directly in the Bison Wallet DCR wallet. Split tickets allow multiple users to pool funds to purchase a staking ticket, making staking accessible to users who cannot afford the full ticket price. This was previously available through external tooling but will now be a native wallet feature. The initial implementation will use a coordination server to manage sessions, match participants, build the multi-participant ticket transaction, and collect signatures. We had planned on migrating to use the decentralized Mesh network after it was finished, removing the need for a centralized server, but the final Mesh product may not be suitable depending on development decisions there. The coordination server will be open source so anyone can run one. One will be run on mainnet as part of this and ongoing maintenance proposals.

We anticipate costs in this area will not exceed $25,000 USD.

- **Fiat On-Ramp:** Integrate a third-party fiat on-ramp service (such as Alchemy Pay) into Bison Wallet, allowing users to purchase cryptocurrency directly within the wallet using credit cards, bank transfers, and other traditional payment methods. The on-ramp provider handles all KYC/AML compliance and payment processing. Our integration work covers the UI, API integration, supported asset and region filtering, and testing.

We anticipate costs in this area will not exceed $10,000 USD.

## Code Signing

Bison Wallet releases are currently signed by the Decred organization, but this adds delays when shipping patches and updates. By obtaining our own code signing certificates for Windows and macOS, we can release updates independently and get fixes to users faster. An Apple Developer membership is required for macOS signing and notarization, and a code signing certificate is required for Windows.

We anticipate ongoing annual costs in this area will not exceed $1,000 USD.

## Team

Work will be performed by Bisoncraft LLC under the leadership of Joe, Marton, and Buck.

## Summary

We are requesting a maximum of $136,000 USD for one year of continued maintenance, new development, and operational costs. Historically, not all allocated funds have been utilized, and any unused funds will remain in the treasury.
