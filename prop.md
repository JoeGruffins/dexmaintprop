Bison Wallet Maintenance Budget Oct 2025 - March 2026

## Objectives completed with the [last prop](https://proposals.decred.org/record/0d23788)

This last year, the DCRDEX project has created, with the help of Tivra, a one page, up-to-date website with info about bison wallet and dcrdex at bisonwallet.org. We have furthered development of mesh. We have created a custom database interface to speed up both mesh and other aspects of the dex. It is now being used for transaction history in a few wallets. We have created a system for fee oracles that can be extended to price as well, which will be used in mesh trading. We have new eth and pol contracts that use less gas and can be used without having any eth to start trading (not yet reviewed or deployed). Until now a user would need to have some tiny amount of eth to make their first trade, but not anymore. For btc, the groundwork for private swaps has been done. They are not yet usable but will be SOON. We have a working “companion app” where a user can run bison wallet on their home network and then connect to it over the web from anywhere through tor. The android app must currently be built from source. There have been multiple upgrades to the market making bots. The UI has been progressively updated, pol and numerous tokens have been added, and mulit-hop arbitrage has been enabled. Base has been added and is in a testing phase. And of course we have been doing our best to squash all the bugs.

## Core development

Bison Wallet / DCRDEX has been six years in development and the code base has grown substantially. There are a lot of things to update and bugs to fix. This proposal's main purpose is to keep devolopers fixing issues, squashing bugs, and updating modules.

Some specific updates we are looking at are:
- finalize the 1.0.4 patch
- create a 1.1.0 release
- incorperate react in our UI stack
- collab with the Creative Refresh prop to update [bisonwallet.org](https://bisonwallet.org/)
- solidify updated eth contracts (this includes outside review)
- update the eth simnet network so that base and contracts can be better tested

Bugs and other [issues](https://github.com/decred/dcrdex/issues) will be fixed in priority of severity to the best of our ability.

We expect costs in this area to not exceed $60k usd.

## New development

On top of continuing development we want to get a couple new things added that users have been vocal about.

The fist of these is a monero rpc wallet. JUST A USEABLE WALLET NO TRADING YET. We have decided not to implement xmr trading until mesh is in beta testing. This is because the current server would require too many changes that would eat up time we could be using getting the mesh ready. The xmr wallet will require the user to run a monero node, just like a btc or dcr wallet that connects through rpc.

The second initiative we would like to get done is dcr politetia voting. Currently we can vote for on chain changes but are unable to use staked dcr with politetia. This should be a priorty for us so that bison wallet users can join the decred community and vote.

We also expect costs in this area to not exceed $40k usd.

## Who

Joe, Marton, Buck, Philemon, Warrior, and any other capable devs.

## Summary

We are asking $100k usd max for six more months of core development. Historically, all funds are not used. Unused funds will be left in the treasury.
