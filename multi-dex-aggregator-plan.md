# Bison Wallet Multi-DEX Aggregator - High-Level Design

## Vision

Transform Bison Wallet from a dcrdex-only client into a universal DEX
client that can execute swaps through multiple backends: dcrdex servers
(atomic swaps), EVM AMMs (Uniswap, Curve), intent-based protocols (NEAR
Intents), and potentially others. Users get comparison pricing and can
choose their trust/cost tradeoff per trade.

---

## Current Architecture

```
  UI (webserver + websocket)
          |
        Core  (client/core/core.go)
          |
    +-----+------+
    |             |
  Wallets       DEX Connections
  (per-asset)   (per-server, websocket)
    |             |
  asset.Wallet   dexConnection -> trackedTrade -> matchTracker
  interface       (atomic swap state machine)
```

**What's already well-abstracted:**
- Wallet layer (`client/asset/interface.go`) - clean per-chain drivers,
  trait system for optional capabilities. Adding a new blockchain is
  already a solved problem.
- Fee estimation - each wallet provides its own estimates.
- Asset registration - `asset.Register()` plugin system.
- EVM infrastructure - ETH wallet already handles smart contract calls,
  token approvals, gas estimation, multi-RPC failover.

**What's tightly coupled to dcrdex:**
- Trade execution state machine (`core/trade.go`) - hardwired 5-step
  atomic swap flow (NewlyMatched -> MakerSwapCast -> TakerSwapCast ->
  MakerRedeemed -> MatchComplete).
- Epoch-based order matching with preimage commitment.
- WebSocket message protocol (`dex/msgjson/`) - dcrdex-specific routes
  (MatchRoute, InitRoute, AuditRoute, RedeemRoute).
- Core orchestrator - assumes all exchanges are dcrdex servers.
- Database schema - orders/matches structured around atomic swap fields.

---

## New Abstraction: Exchange Adapter

The central change is introducing an `ExchangeAdapter` interface that
sits between Core and the various exchange backends.

```
  UI (webserver + websocket)
          |
        Core
          |
    +-----+------+----------+
    |             |          |
  Wallets    ExchangeRouter  |
  (unchanged)    |           |
           +-----+-----+----+---+
           |           |         |
     DCRDEXAdapter  AMMAdapter  IntentAdapter
     (atomic swaps) (Uniswap)   (NEAR)
```

### ExchangeAdapter Interface (new: client/exchange/)

```go
// ExchangeAdapter represents any backend that can execute swaps.
type ExchangeAdapter interface {
    // Identity
    Name() string
    Type() AdapterType // AtomicSwap, AMM, Intent, RFQ

    // Connection lifecycle
    Connect(ctx context.Context) error
    Disconnect()

    // What can be traded
    SupportedMarkets() []*Market

    // Pricing
    Quote(ctx context.Context, req *QuoteRequest) (*Quote, error)

    // Execution
    Trade(ctx context.Context, req *TradeRequest) (*TradeStatus, error)
    TradeStatus(tradeID string) (*TradeStatus, error)
    CancelTrade(tradeID string) error

    // Ongoing state
    Trades() []*TradeStatus
    Notifications() <-chan *TradeNotification
}

type AdapterType int
const (
    AdapterAtomicSwap AdapterType = iota
    AdapterAMM
    AdapterIntent
)

type Quote struct {
    AdapterName   string
    Price         float64
    Fee           uint64
    EstSettlement time.Duration
    TrustModel    string   // "atomic-swap", "smart-contract", "solver"
    Risks         []string // ["bridge", "mev", "slippage", "impermanent-loss"]
}

type TradeStatus struct {
    ID       string
    Adapter  string
    State    TradeState // Pending, Executing, Settled, Failed, Refunded
    Details  any       // adapter-specific (swap txids, AMM receipt, etc.)
}
```

### ExchangeRouter (new: client/exchange/router.go)

Sits in Core, manages all adapters, provides unified quote comparison:

```go
type ExchangeRouter struct {
    adapters map[string]ExchangeAdapter
}

func (r *ExchangeRouter) BestQuote(req *QuoteRequest) ([]*Quote, error)
func (r *ExchangeRouter) Trade(adapter string, req *TradeRequest) (*TradeStatus, error)
func (r *ExchangeRouter) AllTrades() []*TradeStatus
```

---

## What Changes In Existing Code

### Core (client/core/core.go)

- Add `ExchangeRouter` as a field on Core.
- Existing dcrdex server connections get wrapped in a `DCRDEXAdapter`
  that implements `ExchangeAdapter`. The existing atomic swap logic
  stays intact inside this adapter - it's a wrapper, not a rewrite.
- New API methods on Core:
  - `MultiQuote(from, to, amount)` - returns quotes from all adapters
  - `MultiTrade(adapter, from, to, amount)` - execute via specific adapter
- Existing `Trade()` / `TradeAsync()` continue to work for dcrdex, no
  breaking changes.

### Database (client/db/)

- Add a generic `ExternalTrade` table for non-dcrdex trades. These have
  simpler state (pending/settled/failed) compared to atomic swap matches.
- Existing order/match tables stay unchanged for dcrdex trades.

### Webserver (client/webserver/)

- New API routes:
  - `GET /api/multi/quote` - compare prices across adapters
  - `POST /api/multi/trade` - execute via chosen adapter
  - `GET /api/multi/trades` - unified trade history
- Existing routes unchanged.

### UI

- New "compare" view when placing a trade showing price, fees, settlement
  time, and trust model across available adapters.
- Trade history shows adapter source per trade.

---

## Phase 1: ExchangeAdapter Framework + DCRDEXAdapter

**Goal:** Introduce the abstraction without changing behavior. All
existing dcrdex functionality works through the new adapter layer.

**Work:**
1. Define `ExchangeAdapter` interface in `client/exchange/`.
2. Implement `DCRDEXAdapter` wrapping existing `dexConnection` +
   `trackedTrade` logic. This is mostly delegation - the adapter calls
   into the existing Core methods.
3. Add `ExchangeRouter` to Core.
4. Add `MultiQuote` / `MultiTrade` API endpoints that initially only
   return dcrdex results.
5. Add `ExternalTrade` to the DB schema.

**Risk:** Low. This is additive. Existing code paths stay intact.

**Deliverable:** Framework is in place, ready for new adapters.

---

## Phase 2: EVM AMM Adapter (Uniswap V3 / V4)

**Goal:** Trade ERC-20 tokens and ETH via Uniswap directly from Bison
Wallet.

**Why first:** The ETH wallet (`client/asset/eth/`) already has smart
contract interaction, token approvals, gas estimation, and multi-RPC
management. The plumbing is there.

**Work:**
1. Implement `AMMAdapter` in `client/exchange/amm/`.
2. Uniswap router contract interaction:
   - `exactInputSingle` / `exactOutputSingle` for simple swaps
   - Quote via Quoter contract (read-only, no gas)
   - Token approval flow (already supported in ETH wallet)
3. Slippage protection - user-configurable tolerance.
4. MEV protection options:
   - Use Flashbots Protect RPC as an option
   - Deadline parameter on swaps
5. Multi-pool routing for better prices (V3 multi-hop).
6. Support Uniswap on multiple EVM chains (Ethereum, Polygon, Arbitrum,
   Base) since the wallet already supports some of these.

**What the ETH wallet already provides:**
- Account/nonce management
- Gas estimation and dynamic fee adjustment
- ERC-20 token approval flow
- Multi-RPC with failover
- Transaction monitoring

**What's new:**
- Uniswap router ABI integration
- Price quoting from on-chain pools
- Slippage/deadline parameters
- Pool discovery

**Scope:** ~3-5K lines of new code, mostly in the adapter + contract
interaction layer.

**Fundable by:** Uniswap Grants, Ethereum Foundation, any EVM ecosystem
grants (Polygon, Base, Arbitrum).

---

## Phase 3: NEAR Intents Adapter

**Goal:** Submit swap intents to the NEAR intent network from Bison
Wallet.

**Work:**
1. NEAR wallet driver (`client/asset/near/`) - new chain integration:
   - Account model (named accounts)
   - Key management (ed25519)
   - RPC interaction with NEAR nodes
   - Transaction construction and signing
   - NEP-141 token support (NEAR's token standard)
2. `IntentAdapter` in `client/exchange/intent/`:
   - Intent construction and signing
   - Submission to solver network
   - Status polling / WebSocket subscription for fulfillment
   - Settlement verification
3. Cross-chain intent support - if NEAR Intents supports cross-chain
   (e.g., intent to swap ETH for NEAR), coordinate with existing ETH
   wallet for the source side.

**Larger scope** than Phase 2 because it requires a new chain wallet
driver plus the exchange adapter. Two pieces of fundable work.

**Fundable by:** NEAR Foundation grants.

---

## Phase 4: Additional Integrations (future)

Each of these follows the same pattern - implement `ExchangeAdapter`:

- **THORChain** - cross-chain AMM, would need adapter for their
  swap memo protocol. Interesting because it does native cross-chain
  without bridges (uses its own validator set).
- **0x / CoW Protocol** - RFQ/batch auction on EVM. Similar to NEAR
  intents model but Ethereum-native.
- **Osmosis** - Cosmos ecosystem AMM. Would need Cosmos SDK wallet
  driver.
- **Serum/OpenBook (Solana)** - on-chain order book. Would need Solana
  wallet driver.

---

## What This Looks Like For Users

When a user wants to swap ETH for USDC, the UI shows:

**Swap 1.0 ETH -> USDC**

|                | dcrdex (atomic swap)  | Uniswap V3 (AMM)              |
|----------------|-----------------------|--------------------------------|
| **Price**      | 3,842.50 USDC         | 3,840.10 USDC                  |
| **Fees**       | ~$1.20                | ~$4.80 (gas + 0.3% pool fee)  |
| **Settlement** | ~30 min               | ~15 sec                        |
| **Trust**      | None (atomic swap)    | Smart contract                 |
| **Risk**       | -                     | MEV, slippage                  |

Users see the tradeoff: dcrdex is cheaper and trustless but slower.
Uniswap is fast but has fees and MEV risk. Their choice.

---

## Funding Pitch Per Phase

| Phase | Deliverable | Pitch To | Ask |
|-------|------------|----------|-----|
| 1 | Adapter framework | OpenSats, HRF | Infrastructure grant |
| 2 | Uniswap integration | Uniswap Grants, EVM ecosystem | "Your AMM in a non-custodial desktop wallet" |
| 3 | NEAR Intents | NEAR Foundation | "Your intent protocol in a multi-DEX wallet" |
| 4a | THORChain | THORChain treasury | "Native cross-chain swaps in Bison Wallet" |
| 4b | Solana DEX | Solana Foundation | "Solana DEX access from desktop wallet" |

Each phase is independently fundable and delivers standalone value.
Phase 1 is the prerequisite but is small enough to self-fund or bundle
with Phase 2.

---

## Technical Risks

1. **Trust model mismatch in the UI.** Atomic swaps are fundamentally
   different from AMM swaps. The UI must clearly communicate what "trust:
   smart contract" means vs "trust: none". Don't paper over this.

2. **State complexity.** dcrdex trades have a multi-step state machine.
   AMM trades are single-tx. Intent trades are somewhere in between. The
   `TradeStatus` abstraction needs to handle all of these without
   becoming a leaky abstraction.

3. **Wallet reuse.** An AMM adapter on Ethereum reuses the same ETH
   wallet as dcrdex atomic swaps. Need to handle shared wallet state
   (nonce management, balance accounting) correctly when two adapters
   use the same wallet concurrently.

4. **Scope creep.** Each "simple" AMM integration tends to grow:
   multi-hop routing, LP positions, yield farming. Keep scope tight -
   swaps only, no LP management.

5. **Regulatory.** Aggregating across DEX types might attract more
   scrutiny than a single-protocol client. Worth considering but
   probably not a blocker given everything is non-custodial.
