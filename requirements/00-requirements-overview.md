# Requirements Overview

## 1. Purpose

This document set translates the Axis v1 Open Design into implementation-grade requirements.

The goal is to make each protocol decision explicit enough that it can be turned into GitHub issues for engineering work.

## 2. Product Goal

Axis v1 should let users create, mint, and redeem DTFs on Solana.

A DTF is a tradable position token backed by a basket of underlying reserve assets.

Users should be able to:

```txt
1. Create a DTF with 2 to 5 assets
2. Define target weights
3. Mint the DTF with USDC
4. Have Axis compose underlying assets through CPI swaps
5. Hold the DTF token
6. Trade, distribute, or use the DTF in partner or sponsor campaigns through a secondary-market surface
7. Redeem the DTF back to USDC
```

## 3. Protocol Goal

Axis v1 must be:

```txt
- open
- reserve-backed
- deterministic in accounting
- controlled in execution
- safe under per-asset and per-transaction limits
- independent from external routers
- compatible with future router integrations
```

## 4. Core Architecture Requirement

Axis Core is responsible for:

```txt
- DTF market creation
- DTF mint lifecycle
- reserve accounting
- mint execution
- redeem execution
- pricing source validation
- NAV calculation
- execution policy validation
- approved CPI route validation
```

Axis Auction Program, when enabled for a market, is a separate program responsible for Axis-controlled correction-auction state and settlement. It does not own DTF reserve custody, NAV, or mint/redeem accounting. See `18-secondary-market-and-clear-correction-requirements.md`.

External systems may help with:

```txt
- route discovery
- quote building
- account assembly
- UI display prices
- distribution / routing into Axis markets
- public DTF/USDC liquidity on third-party venues
- secondary-market indexing and display
```

But external systems must not be required for Axis Core accounting.

Public third-party DTF/USDC pools are external liquidity. Axis may index or display them, but must not represent them as Axis-native liquidity or claim that they receive Axis LVR mitigation.

## 5. Requirement Naming

Requirements use the following IDs:

```txt
AXIS-CORE-*   global protocol and architecture requirements
DTF-*         DTF market requirements
MINT-*        mint flow requirements
REDEEM-*      redeem flow requirements
EXEC-*        swap CPI execution requirements
PRICE-*       pricing and NAV requirements
POLICY-*      execution policy and risk control requirements
ASSET-*       asset universe requirements
ADMIN-*       admin, safety, and emergency requirements
FEE-*         creator fee and protocol fee requirements
VENUE-*       production venue integration requirements
APP-*         app contract integration requirements
BD-*          partner demo and business development requirements
PMV-*         pre-mainnet validation requirements
MAINNET-*     guarded mainnet launch requirements
NFR-*         non-functional requirements
TEST-*        testing requirements
SECONDARY-*   secondary-market and external-pool requirements
AUCTION-*     Axis Auction Program and ClearCorrection requirements
```

## 6. Implementation Phasing

### Phase 0: Docs and Design Freeze

```txt
- finalize requirements
- define Axis v1 contract scope
- define DTF market account model
- define mint and redeem instruction surfaces
- define reserve accounting model
- define creator fee requirements
- define venue integration requirements
- define pre-mainnet validation plan
- define launch-day secondary-market surface and external-pool policy
- define the Axis Auction Program boundary and ClearCorrection technical-spike plan
- turn requirements into GitHub issues
```

### Phase 1: Core Contract Validation

The first implementation phase must prove that Axis Core can create and operate a minimal reserve-backed DTF lifecycle.

This phase does not require public Devnet deployment.

The goal is to validate the core protocol model through deterministic local tests and integration tests.

Required capabilities:

```txt
- initialize protocol config
- register assets
- set asset execution policies
- set pricing sources
- register approved CPI routes
- create a 2-asset DTF market
- create DTF mint and reserve token accounts
- mint DTF tokens with USDC
- execute a real CPI execution path in a controlled test environment
- move real SPL tokens
- measure actual reserve balance deltas
- calculate actual added value
- mint DTF tokens based on pre-trade NAV
- accrue creator fees where applicable
- redeem DTF tokens
- execute unwind path from reserve assets back to USDC
- enforce min_out
- transfer actual USDC output to user
- verify actual token balance deltas after execution
- burn or remove redeemed DTF tokens from circulating supply
```

Phase 1 constraints:

```txt
- 2-asset DTF end-to-end is required
- 3-5 asset account structure should be supported
- 3-5 asset execution feasibility may be validated after the 2-asset lifecycle is stable
- controlled CPI adapter may be used for local validation
- mock accounting is not acceptable
- real token transfer is required
- actual balance delta verification is required
- direct USDC <-> asset routes are preferred
- no split routing for the first implementation milestone
- no routed SOL intermediate path for the first implementation milestone
- ManualFixedPrice pricing source may be used for local validation
- creator fee support must be implemented and tested before mainnet launch
- rebalance is out of scope for the first implementation milestone
- Titan integration is out of scope for Axis Core
```

### Phase 2: Local and Fork-Based Integration Validation

The second phase validates Axis v1 against more realistic execution and account conditions.

Public Devnet is not required.

Validation should use local validator tests, LiteSVM tests, and mainnet-fork or cloned-account tests where applicable.

Required capabilities:

```txt
- validate mint and redeem end-to-end with integration tests
- test actual balance delta accounting
- test creator fee accrual and accounting
- test mixed decimal assets
- test min_out failure behavior
- test disabled asset behavior
- test disabled route behavior
- test paused or emergency states
- test all-or-nothing execution
- measure compute usage
- measure account usage
- document known limitations
```

### Phase 3: Production Venue Integration

At least one production venue integration path must be validated before mainnet launch.

A venue is not Axis-ready only because an SDK can quote a route.

A venue is Axis-ready only when Axis Core can validate the route, execute the CPI, move real tokens, verify actual balance deltas, enforce min_out, and measure compute/account usage.

Candidate venue integrations:

```txt
- Orca Whirlpool
- Raydium CPMM
- PumpSwap
- Meteora DLMM
```

Required capabilities:

```txt
- verify production venue program ID
- construct venue CPI instruction
- validate required accounts
- execute CPI in a local or fork-based environment
- verify pre/post token balances
- enforce min_out
- reject invalid input mint
- reject invalid output mint
- reject invalid pool or venue accounts
- measure compute usage
- measure account usage
- document venue-specific risks
```

### Phase 4: Launch-Day Secondary Market Surface

Axis v1 must launch with a secondary-market surface so that created DTFs can be traded, distributed, and used in partner or sponsor campaigns from day one.

Required capabilities:

```txt
- make created DTF markets and their transferable DTF tokens discoverable in the Axis app or an equivalent Axis-operated surface
- expose market identity, reserve/NAV context, mint/redeem availability, and external DTF/USDC pool references where available
- provide a canonical market URL or share URL and stable partner-facing market metadata
- support clear routing or linking to public third-party DTF/USDC pools, including pools on Orca or Raydium
- distinguish external public pools from Axis-native auction-enabled liquidity in all status and risk presentation
- avoid any claim that external public pools receive Axis LVR mitigation
- support market metadata that partners and sponsors can use for campaign and distribution integrations
```

Public third parties may create DTF/USDC pools without Axis permission. Axis may index or display eligible public pools, subject to the external-pool policy in `18-secondary-market-and-clear-correction-requirements.md`.

### Phase 5: Axis-Controlled Auction Settlement Technical Spike

All DTF markets must be architecturally compatible with future Axis-controlled JIT liquidity and NAV Correction Auctions. This does not mean that every market is enabled for auction/JIT liquidity at launch.

The preferred research path is Axis-controlled JIT liquidity settled on Orca Whirlpool. ClearCorrection and auction state belong in a separate Axis Auction Program; Axis Core remains responsible for DTF creation, mint, redeem, reserve custody, NAV, and accounting.

Required technical-spike outcomes:

```txt
- validate whether Axis-controlled JIT liquidity can settle a bounded correction on Orca Whirlpool
- validate single-transaction ordered settlement as the preferred execution mode
- measure account and compute feasibility across representative 2-to-5 asset DTF configurations, and identify which market classes can safely activate
- validate NAV/pricing freshness, route support, winner-only authorization, and safe failure behavior
- evaluate Jito bundles only as a fallback when they demonstrate ordered atomic execution and prevent non-winner interception
- document activation gates and unresolved feasibility limits
```

This phase is not an August launch blocker. Production ClearCorrection / Auction Program activation remains gated by the technical-spike results and the per-market readiness gates in `18-secondary-market-and-clear-correction-requirements.md`.

### Phase 6: App Contract Integration

The app experience should not be treated as a full UX redesign in this requirements phase.

The existing index creation and product experience should be preserved where possible.

The app must be updated to connect to the new Axis v1 contract model.

Required capabilities:

```txt
- load DTF market state
- load target weights
- load reserve token accounts
- load actual reserve balances
- load NAV or approved reserve value
- construct mint transactions
- construct redeem transactions
- display reserve-backed DTF state
- display creator fee information where relevant
- display transaction results
```

Detailed UI and UX requirements should be defined in a separate app requirements document.

### Phase 7: Partner Demo Readiness

Partner demos should be backed by contract-level evidence.

The goal is to use Axis v1 as a real BD asset when speaking with ecosystem teams.

Partner demo readiness should support:

```txt
- sharing the contract GitHub repository
- showing how Axis-issued DTF markets work
- showing reserve-backed mint and redeem behavior
- showing transaction or test evidence
- explaining venue integration status
- explaining how external teams can route into Axis-issued DTF exposure
- showing the launch-day secondary-market surface and the distinction between external pools and any Axis-native auction-enabled liquidity
```

Partner positioning:

```txt
Axis handles DTF issuance.
Axis owns reserve-backed DTF markets.
External partners may become distribution or routing surfaces.
```

Example partner narratives:

```txt
- Titan can route or surface swaps into Axis-issued DTF exposure from Titan UI
- Orca can review Axis CPI usage and provide technical design feedback
- other Solana projects can collaborate around ecosystem-specific DTF markets
```

### Phase 8: Guarded Mainnet Candidate

Axis v1 should move toward mainnet only after required pre-mainnet validation evidence is available.

Public Devnet runtime is not required as a launch gate.

Mainnet candidate requirements:

```txt
- core create/mint/redeem lifecycle is validated
- actual balance delta accounting is validated
- reserve backing is observable
- creator fee behavior is implemented and tested
- at least one production venue integration path is validated
- app contract integration is tested
- launch-day secondary-market surface is available and accurately labels external liquidity
- local and fork-based tests pass
- failure cases are tested
- deployment source and binary traceability are documented
- guarded launch controls are configured
- known blockers are resolved or explicitly accepted
```

Production-grade Axis-controlled JIT liquidity and ClearCorrection are not required for this guarded launch unless the technical spike has completed and the relevant market activation gates are satisfied. Axis must not claim LVR mitigation for public external pools.

Guarded launch controls:

```txt
- limited initial asset universe
- approved route registry
- per-asset execution policy
- per-transaction limits
- creator fee limits
- protocol fee limits
- emergency pause controls
- route disable controls
- asset disable controls
- monitoring
- launch checklist
- mitigation procedures
```
