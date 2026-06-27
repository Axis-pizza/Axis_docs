# 20. StockToken / xStocks / Raydium Execution Requirements

## 20.1 Purpose

This document defines the Axis v1 requirements for supporting StockToken-style assets, with an initial focus on xStocks on Solana.

The goal is to decide how xStocks may be included in Axis DTF markets, under what conditions they can be enabled for mint/redeem execution, and what technical gates must be passed before any xStock becomes launch-ready.

This document is based on real-data route discovery and on-chain verification research.

This document extends the Asset Universe Requirements for the StockToken / xStocks category.

General asset registry fields, readiness states, pricing tiers, route readiness, and execution flags are defined in Asset Universe Requirements.

This document only defines StockToken / xStocks-specific execution requirements, including Token-2022, Scaled UI amount handling, Raydium CLMM execution, pricing/NAV policy, and xStocks launch readiness gates.

## 20.2 Scope

Axis v1 includes xStocks as a P0 candidate asset category.

However, Axis v1 must not automatically support all xStocks.

Only explicitly enabled xStocks may be used for:

* DTF market creation
* Mint execution
* Redeem execution
* Reserve accounting
* NAV display
* Secondary market display

Being listed in the official xStocks asset universe does not imply Axis readiness.

StockToken allocation in the Asset Universe represents candidate coverage, not launch-ready execution coverage.

A StockToken may exist in the Axis asset universe while still being marked route-required, pricing-required, mint-required, or disabled.

Only a smaller manually reviewed subset may become execution-enabled after passing the gates in this document.

## 20.3 Non-Goals

Axis v1 will not aim for full xStocks coverage.

The following are out of scope for initial launch readiness:

* Enabling all xStocks by default
* Supporting 50+ xStocks immediately at launch
* Treating Jupiter quote availability as launch readiness
* Treating Titan route visibility as launch readiness
* Treating Backed / xStocks `supportsAtomicSwaps` as Axis execution readiness
* Supporting off-chain market maker execution inside Axis mint/redeem
* Supporting CEX execution inside Axis mint/redeem
* Supporting issuer redemption flows inside Axis Core
* Marking any xStock as launch-ready without local/fork execution tests

## 20.4 Confirmed Research Findings

Initial xStocks research confirmed the following:

* xStocks on Solana use Token-2022.
* xStocks use Scaled UI amount configuration.
* Observed xStocks use 8 decimals.
* Observed xStocks did not expose TransferFee configuration in the research dataset.
* Axis accounting must use raw token amounts, not scaled display amounts.
* Frontend, backend, NAV display, and analytics must support Scaled UI conversion.
* Direct Raydium USDC pools exist only for a small subset of xStocks.
* The first confirmed high-priority executable candidates are SPCXx and STRCx.
* The confirmed Raydium pools are Concentrated / CLMM pools, not simple CPMM pools.
* Therefore, xStocks execution support requires Raydium CLMM research and testing.

## 20.5 Initial Spike Candidates

### 20.5.1 SPCXx / SpaceX xStock

SPCXx is the first high-priority xStocks execution spike candidate.

Current status:

* Official Solana mint verified
* Token-2022 verified
* Scaled UI verified
* Direct Raydium USDC CLMM pool found
* Pool account verified on-chain
* Pool owner matches Raydium CLMM program
* Vaults verified
* Vault mints match expected pool mints
* Vault balances are non-zero
* Observation account verified
* Pool config account verified
* Tick arrays obtainable
* Status: `CLMM_SPIKE_READY`

SPCXx must not be marked launch-ready until swap simulation and Axis Core integration gates pass.

### 20.5.2 STRCx / Strategy PP Variable xStock

STRCx is the second xStocks execution spike candidate.

Current status:

* Official Solana mint verified
* Token-2022 verified
* Scaled UI verified
* Direct Raydium USDC CLMM pool found
* Pool account verified on-chain
* Pool owner matches Raydium CLMM program
* Vaults verified
* Vault mints match expected pool mints
* Vault balances are non-zero
* Observation account verified
* Pool config account verified
* Tick arrays obtainable
* Status: `CLMM_SPIKE_READY`

STRCx must not be marked launch-ready until swap simulation and Axis Core integration gates pass.

## 20.6 Asset Enablement Model

Each xStock must have independent enablement flags.

Required flags:

* `creation_enabled`
* `mint_enabled`
* `redeem_enabled`
* `secondary_display_enabled`
* `route_discovery_enabled`

General launch readiness is defined in Asset Universe Requirements.

StockToken / xStocks may use the following category-specific sub-statuses:

* `TOKEN_2022_REVIEW_REQUIRED`
* `SCALED_UI_REVIEW_REQUIRED`
* `CLMM_ROUTE_RESEARCH_REQUIRED`
* `CLMM_SPIKE_CANDIDATE`
* `CLMM_SPIKE_READY`
* `ONE_WAY_ROUTE_ONLY`
* `HIGH_PRICE_IMPACT`
* `MARKET_HOURS_POLICY_REQUIRED`
* `TOKEN_EXTENSION_REVIEW_REQUIRED`

The general readiness status must remain separate from StockToken-specific sub-status.

Examples:

```text
launch_readiness = SPIKE_CANDIDATE
stocktoken_status = CLMM_SPIKE_READY

launch_readiness = PRICING_REQUIRED
stocktoken_status = MARKET_HOURS_POLICY_REQUIRED
```

No discovery script may assign `LAUNCH_READY`.

`LAUNCH_READY` must require manual review and successful local/fork execution tests.

## 20.7 Token-2022 / Scaled UI Requirements

Axis Core must treat xStocks as Token-2022 assets.

Protocol accounting requirements:

* Reserve accounting must use raw token amounts.
* Mint/redeem balance delta verification must use raw token amounts.
* DTF supply calculation must not use scaled display amounts.
* Fee accounting must not use scaled display amounts.
* NAV accounting must separate protocol truth from display conversion.

Backend/UI requirements:

* Backend must store raw token amount.
* Backend may derive scaled UI amount for display.
* Frontend must display scaled UI amount when appropriate.
* Analytics must clearly separate raw amount and display amount.

The following rule is mandatory:

```text
Raw amount is protocol truth.
Scaled UI amount is display representation only.
```

## 20.8 Execution Venue Requirements

For normal crypto assets, Axis v1 may continue to treat Orca Whirlpool and Raydium CPMM as primary/fallback production venue candidates.

For xStocks, the real-data research shows that initial executable candidates use Raydium Concentrated / CLMM pools.

Therefore, xStocks require a separate Raydium CLMM execution path.

Raydium CPMM support alone is not sufficient for xStocks if available xStock liquidity is primarily in Raydium CLMM pools.

## 20.9 Raydium CLMM Requirements

Before any xStock is execution-enabled, Axis must verify the full Raydium CLMM account set.

Required CLMM fields:

* `venue_type = RAYDIUM_CLMM`
* `venue_program_id`
* `pool_id`
* `pool_config`
* `mint_a`
* `mint_b`
* `token_program_a`
* `token_program_b`
* `vault_a`
* `vault_b`
* `observation_account`
* `tick_spacing`
* `tick_arrays`
* `allowed_direction`
* `max_trade_usdc`
* `max_price_impact_bps`
* `max_slippage_bps`
* `enabled`

Required verification:

* Pool account exists.
* Pool owner matches Raydium CLMM program.
* Pool mints match expected xStock/USDC pair.
* xStock mint is Token-2022.
* USDC mint is legacy SPL Token.
* Vault A exists.
* Vault B exists.
* Vault A mint matches pool mint A.
* Vault B mint matches pool mint B.
* Vault balances are non-zero.
* Observation account exists.
* Pool config exists.
* Tick arrays can be assembled for both directions.
* Swap instruction can be constructed.
* Local/fork swap simulation succeeds.

## 20.10 Route Discovery vs ApprovedRoute

Jupiter, Titan, and other aggregators may be used for route discovery and market intelligence.

However, aggregator route visibility does not automatically create an Axis ApprovedRoute.

For Axis Core execution, each route must be explicitly approved and validated.

Jupiter/Titan route discovery may answer:

* Which venues currently provide liquidity
* Whether buy/sell routes exist
* Whether routes are one-way or two-way
* Price impact quality
* Whether venues such as HumidiFi, Byreal, FluxBeam, Whirlpool, or Raydium appear in routes

But these are not sufficient for execution enablement.

An Axis ApprovedRoute requires:

* Known venue program
* Known pool/market accounts
* Known token programs
* Known vaults/accounts
* On-chain validation
* Min-out enforcement
* Actual balance delta verification
* Route-specific risk controls

## 20.11 Pricing / NAV Requirements

xStocks pricing must be handled carefully because they represent stock-like assets with market-hours and potential stale-price concerns.

Before any xStock becomes launch-ready, Axis must define:

* Primary pricing source
* Fallback pricing source
* Market-hours behavior
* Stale price threshold
* Halted asset behavior
* Price deviation tolerance
* Secondary market price vs NAV separation
* Display price vs accounting price distinction

Raydium CLMM pool price may be useful for route execution and sanity checks, but it must not automatically become NAV truth without an explicit pricing policy.

## 20.12 Launch Readiness Gates

An xStock can become `LAUNCH_READY` only if all gates pass.

Required gates:

1. Official Solana mint verified.
2. Token-2022 ownership verified.
3. Scaled UI configuration verified.
4. Raw amount accounting supported.
5. Frontend/backend scaled display supported.
6. Direct or approved executable USDC <-> xStock route confirmed.
7. Venue adapter implemented.
8. Pool account verified.
9. Venue program owner verified.
10. Vaults verified.
11. Observation account verified if CLMM.
12. Tick arrays assembled if CLMM.
13. Liquidity screen passed.
14. Price impact threshold passed.
15. Pricing/NAV source defined.
16. Market-hours/stale-price policy defined.
17. Local or fork swap simulation passed.
18. Actual reserve balance delta verification passed.
19. Min-out and slippage protection passed.
20. Compute unit usage measured.
21. Account count measured.
22. Transaction size feasibility measured.
23. 1-asset mint test passed.
24. 1-asset redeem test passed.
25. Multi-asset atomic feasibility reviewed.
26. Emergency disable controls exist.
27. ApprovedRoute registered and enabled for the target route.
28. Manual launch approval recorded.

## 20.13 Initial v1 Decision

Axis v1 should proceed with the following xStocks strategy:

* Include xStocks as a P0 candidate asset category.
* Do not enable all xStocks by default.
* Use explicit allowlist / enablement controls.
* Treat SPCXx and STRCx as first CLMM execution spike candidates.
* Require Raydium CLMM support for initial xStocks execution.
* Treat Jupiter/Titan routes as discovery signals, not protocol execution readiness.
* Do not mark any xStock as launch-ready until local/fork execution tests pass.
* Keep all other xStocks in route research status unless a verified executable route is found.

## 20.14 Development Tasks

### P0-1: Raydium CLMM xStocks swap simulation

Purpose:
Validate whether SPCXx and STRCx can be safely swapped through Raydium CLMM before integrating the route into Axis Core mint/redeem.

Scope:
This task validates direct Raydium CLMM swap feasibility.
It does not require full Axis Core mint/redeem integration.

Targets:
- SPCXx / USDC
- STRCx / USDC

Required environment:
- local validator or mainnet-fork environment
- cloned Raydium CLMM pool accounts
- cloned pool config
- cloned observation account
- cloned tick arrays
- cloned vaults
- xStock Token-2022 mint
- USDC legacy SPL mint

Required test cases:
- 10 USDC -> xStock
- xStock -> USDC
- min_out success
- min_out failure
- actual raw balance delta verification
- Token-2022 xStock transfer
- legacy SPL USDC transfer
- compute unit measurement
- account count measurement
- serialized transaction size measurement

Pass criteria:
- swap simulation succeeds in both directions
- output amount is >= min_out
- transaction fails when min_out is intentionally too high
- input/output raw token balance deltas match actual transfer results
- xStock uses Token-2022
- USDC uses legacy SPL Token
- pool vault balances change in the expected direction
- compute units are recorded
- account count is recorded
- transaction size is recorded

Required output:
- asset symbol
- xStock mint
- USDC mint
- pool id
- pool config
- observation account
- tick arrays used
- direction
- input raw amount
- actual output raw amount
- min_out
- compute units
- account count
- transaction size
- pass/fail
- failure reason if failed

### P0-2: ApprovedRoute schema for Raydium CLMM

Implement or specify ApprovedRoute fields for Raydium CLMM routes.

Required:

* Pool id
* Venue program id
* Pool config
* Vaults
* Observation account
* Tick arrays
* Token programs
* Direction
* Slippage / price impact limits
* Enable flag

### P0-3: Axis Core balance delta validation

Mint side:

* User pays USDC.
* Mint fee is deducted from USDC input.
* Axis swaps net USDC into reserve xStock.
* Reserve xStock balance delta must equal actual received amount.
* DTF mint amount must be based on actual reserve value.

Redeem side:

* Axis burns or receives DTF.
* Axis unwinds reserve xStock back to USDC.
* User output equals actual USDC received.
* Redeem fee is zero.
* Min-out is enforced.

Scaled UI amount must never be used for protocol accounting, fee accounting, mint/redeem calculation, or reserve balance delta validation.

### P1: Scaled UI display support

Backend and frontend must support:

* Raw amount
* Scaled UI amount
* Decimals
* Display conversion
* Reserve display
* NAV display

### P1: Pricing and market-hours policy

Define:

* Price source
* Stale price behavior
* Halted asset behavior
* Market-hours behavior
* Secondary price vs NAV display

## 20.15 Open Questions

The following remain open:

1. Can Raydium CLMM swap fit safely inside Axis atomic multi-asset mint/redeem?
2. How many xStock CLMM routes can fit in one transaction?
3. What are the compute unit and account count limits for 1-asset, 2-asset, and 5-asset DTF mint/redeem?
4. Should xStocks use separate per-asset max trade sizes?
5. Should xStocks require stricter stale-price policies than crypto assets?
6. Should SPCXx be enabled for creation before mint/redeem execution is enabled?
7. Should xStocks with only Jupiter routes remain display-only until a direct ApprovedRoute exists?
8. Should HumidiFi, Byreal, FluxBeam, or other venues be considered later as separate venue adapters?

## 20.16 Current Status

Current status:

* xStocks: P0 candidate category
* SPCXx: CLMM spike ready
* STRCx: CLMM spike ready
* Other xStocks: route research required
* Raydium CLMM: required for initial xStocks execution
* Launch readiness: not yet achieved

Next required step:

```text
Run local/fork Raydium CLMM swap simulation for SPCXx and STRCx.
Verify actual balance delta, min_out behavior, compute units, account count, and transaction size.
```
