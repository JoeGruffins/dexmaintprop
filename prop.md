# Bison Wallet Maintenance Budget: October 2025 – March 2026

## Achievements from the [Previous Proposal](https://proposals.decred.org/record/0d23788)

In our previous proposal, we requested $305,000 USD, of which only $220,000 USD was spent. We achieved the following milestones:

- **Market Making:**
  - Live configuration and balance updates.
  - Enabled multi-hop arbitrage.
  - Added rebalancing using bridges.
  - Made UI improvements, including a revamped market making configuration UI using React.

- **EVM Swaps:**
  - Developed a gas-efficient swap contract that supports both native assets and ERC-20 tokens within the same contract.
  - Incorporated account abstraction to enable redemptions without requiring the asset in the wallet to cover gas fees.

- Added bridging support for EVM assets using Polygon, CCTP, and Across bridges.

- Improved wallet UI by grouping EVM assets across various networks.

- **Private Swaps:**
  - Completed DCR and BTC wallet support for adaptor signature atomic swaps, but it is not integrated into the DCRDEX server. This will be utilized once Mesh is released.

- **Mesh:**
  - Advanced the development of the Mesh decentralized network.
  - Created a system for fee oracles, extensible to price oracles, for use in Mesh trading.

- Developed a "companion app" that allows users to run Bison Wallet on their home network and access it remotely via Tor over the web (Android app currently requires building from source).

- Created a custom Lexi database library to enhance performance in Mesh and other DEX components, now integrated for transaction history in select wallets.

- Integrated Base, currently in testing phase.

- Launched a streamlined, one-page website at [bisonwallet.org](https://bisonwallet.org/) with up-to-date information on Bison Wallet and DCRDEX.

- Fixed numerous bugs and issues in the codebase.

## Core Development

Bison Wallet / DCRDEX has been six years in development and the code base has grown substantially. There are a lot of things to update and bugs to fix. This proposal's main purpose is to keep devolopers fixing issues, squashing bugs, and updating modules.

Additionally, the features we have implemented during the previous proposal have not yet made it into a release. An urgent priority will be to get a 1.1.0 release out.

Furthermore, we will collaborate with the Creative Refresh proposal to update [bisonwallet.org](https://bisonwallet.org/). 

We anticipate costs in this area will not exceed $60,000 USD.

## New Development Initiatives

In addition to ongoing maintenance, we aim to introduce user-requested features to enhance Bison Wallet's functionality:

- Release a Monero (XMR) RPC wallet. **This will be a functional wallet only—no trading support at this stage.** XMR trading will only be supported on Mesh.

- Implement Decred Politeia voting integration. Currently, users can vote on on-chain proposals but not Politeia proposals using staked DCR. This feature will enable Bison Wallet users to fully participate in the Decred governance community.

We anticipate costs in this area will not exceed $40,000 USD.

## Team

The work will be led by Joe, Marton, Buck, Philemon, Warrior, and other qualified developers as needed.

## Summary

We are requesting a maximum of $100,000 USD for six months of continued core development and new initiatives. Historically, not all allocated funds have been utilized, and any unused funds will remain in the treasury.