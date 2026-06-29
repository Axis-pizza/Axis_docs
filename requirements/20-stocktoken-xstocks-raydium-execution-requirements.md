# 20. StockToken / xStocks Execution Requirements

## 20.1 Purpose

This document defines the Axis v1 requirements for supporting StockToken-style assets, with an initial focus on xStocks on Solana.

The goal is to decide how xStocks may be included in Axis DTF markets, under what conditions they can be enabled for mint/redeem execution, and what technical gates must be passed before any xStock becomes launch-ready.

This document extends the Asset Universe Requirements for the StockToken / xStocks category.

General asset registry fields, readiness states, pricing tiers, route readiness, and execution flags are defined in Asset Universe Requirements.

This document only defines StockToken / xStocks-specific execution requirements, including Token-2022, Scaled UI amount handling, venue-specific execution readiness, pricing/NAV policy, and xStocks launch readiness gates.

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

A StockToken may exist in the Axis asset universe while still being marked route-required, pricing-required, mint-required, simulation-required, review-required, or disabled.

Only a smaller manually reviewed subset may become execution-enabled after passing the gates in this document.

## 20.3 Non-Goals

Axis v1 will not aim for full xStocks coverage.

The following are out of scope for initial launch readiness:

* Enabling all xStocks by default
* Supporting 50+ xStocks immediately at launch
* Treating Jupiter quote availability as launch readiness
* Treating Titan route visibility as launch readiness
* Treating Backed / xStocks `supportsAtomicSwaps` as Axis execution readiness
* Supporting issuer redemption flows inside Axis Core
* Supporting off-chain market maker execution inside Axis mint/redeem
* Supporting CEX execution inside Axis mint/redeem
* Supporting RFQ execution inside Axis Core
* Supporting DFlow-routed execution inside Axis Core
* Supporting Jupiter-routed multi-hop execution inside Axis Core
* Supporting routes through non-official intermediate mints
* Marking any xStock as launch-ready without local/fork execution tests

## 20.4 Confirmed Research Findings

Initial xStocks research confirmed the following:

* xStocks public asset discovery is available.
* 100 Solana xStock deployment candidates were discovered through the xStocks public assets API.
* xStocks on Solana use Token-2022.
* Observed xStocks use Scaled UI amount configuration.
* Observed xStocks use 8 decimals for official provider-listed Solana mints.
* Axis accounting must use raw token amounts, not scaled display amounts.
* Frontend, backend, NAV display, and analytics must support Scaled UI conversion.
* Being present in the xStocks API does not imply Axis execution readiness.
* `supportsAtomicSwaps` is a provider capability signal, not Axis execution readiness.
* Jupiter route visibility is useful for route discovery but is not protocol execution readiness.
* xStocks route readiness is venue-specific and direction-specific.
* Raydium CLMM is a P0 venue candidate.
* Byreal Router AMM is also a P0 venue candidate.
* Raydium CLMM must not be assumed to be the only xStocks execution path.
* Multi-hop routes through intermediate mints must be rejected for v1 unless separately approved.

## 20.5 Core Execution Principle

Axis v1 StockToken execution must only allow direct routes between USDC and the official provider-listed Solana xStock mint.

The following rule is mandatory:

```text
Only direct USDC <-> official provider-listed xStock mint routes may become Axis v1 ApprovedRoute candidates.
```

Routes that pass through the following must be rejected unless separately approved through a dedicated risk review:

* non-official intermediate mints
* wrapper mints
* synthetic representations
* venue-specific representations
* similar ticker mints
* Jupiter multi-hop routes
* RFQ routes
* DFlow routes
* CEX/off-chain market maker routes

Protocol accounting, reserve balance delta verification, NAV calculation, and DTF mint/redeem accounting must use the official provider-listed xStock mint only.

Ticker similarity is not sufficient.

The mint address is the source of truth.

## 20.6 Readiness Model

StockToken readiness must not be tracked only at the asset level.

Axis v1 must track execution readiness by:

* asset
* direction
* venue
* route type
* official mint directness

Example:

```text
SPCXx / USDC -> SPCXx / BYREAL_ROUTER_AMM / DIRECT
SPCXx / SPCXx -> USDC / BYREAL_ROUTER_AMM / DIRECT
SPCXx / SPCXx -> intermediate mint -> USDC / MULTI_HOP / REJECTED
STRCx / USDC -> STRCx / BYREAL_ROUTER_AMM / DIRECT
STRCx / STRCx -> USDC / RAYDIUM_CLMM / DIRECT
```

This means `creation_enabled`, `mint_enabled`, and `redeem_enabled` must be independently controlled.

A StockToken may be allowed for display or market creation while still being disabled for mint/redeem execution.

## 20.7 Asset Enablement Model

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
* `ROUTE_RESEARCH_REQUIRED`
* `DIRECTION_SPECIFIC_ROUTE_DISCOVERED`
* `BYREAL_ROUTER_AMM_APPROVED_ROUTE_CANDIDATE`
* `RAYDIUM_CLMM_APPROVED_ROUTE_CANDIDATE`
* `APPROVED_ROUTE_SCHEMA_REQUIRED`
* `SIMULATION_REQUIRED`
* `ONE_WAY_ROUTE_ONLY`
* `MIXED_VENUE_ROUTE_REQUIRED`
* `HIGH_PRICE_IMPACT`
* `MARKET_HOURS_POLICY_REQUIRED`
* `TOKEN_EXTENSION_REVIEW_REQUIRED`
* `INTERMEDIATE_MINT_REJECTED`

The general readiness status must remain separate from StockToken-specific sub-status.

Examples:

```text
launch_readiness = SIMULATION_REQUIRED
stocktoken_status = BYREAL_ROUTER_AMM_APPROVED_ROUTE_CANDIDATE

launch_readiness = ROUTE_REVIEW_REQUIRED
stocktoken_status = DIRECTION_SPECIFIC_ROUTE_DISCOVERED

launch_readiness = DISABLED
stocktoken_status = INTERMEDIATE_MINT_REJECTED
```

No discovery script may assign `LAUNCH_READY`.

`LAUNCH_READY` must require manual review and successful local/fork execution tests.

## 20.8 Token-2022 / Scaled UI Requirements

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

## 20.9 Venue Candidates

For normal crypto assets, Axis v1 may continue to treat Orca Whirlpool and Raydium CPMM as primary/fallback production venue candidates.

For StockToken / xStocks, real route discovery shows that execution may require separate venue adapters.

P0 StockToken venue candidates:

* `BYREAL_ROUTER_AMM`
* `RAYDIUM_CLMM`

Out of scope for v1 unless separately approved:

* `BYREAL_RFQ`
* `JUPITER_MULTI_HOP`
* `DFLOW`
* `GOONFI`
* `CEX`
* `OFF_CHAIN_MARKET_MAKER`

Raydium CLMM remains a high-priority venue candidate.

However, Raydium CLMM alone is not sufficient as a universal xStocks assumption.

Byreal Router AMM must be treated as a separate P0 venue candidate because real route discovery found direct official-mint xStock routes through Byreal AMM.

## 20.10 Byreal Router AMM Requirements

Byreal Router AMM is a P0 venue candidate for xStocks execution.

A Byreal route can become an ApprovedRoute candidate only when it is:

* AMM route
* direct route
* official provider-listed xStock mint route
* RFQ excluded
* Jupiter excluded
* DFlow excluded
* intermediate mints excluded

Byreal Router API may be used for:

* route discovery
* quote discovery
* pool address discovery
* transaction/account analysis
* simulation preparation

However, Byreal Router API must not be treated as protocol truth.

Axis Core must validate the route against on-chain ApprovedRoute data.

A Byreal Router AMM ApprovedRoute must include:

* `venue_type = BYREAL_ROUTER_AMM`
* `router_program_id`
* `inner_venue_program_id`
* `pool_id`
* `input_mint`
* `output_mint`
* `input_token_program`
* `output_token_program`
* `allowed_direction`
* `route_type = DIRECT`
* `rfq_allowed = false`
* `jupiter_allowed = false`
* `dflow_allowed = false`
* `intermediate_mints_allowed = false`
* `max_trade_usdc`
* `max_price_impact_bps`
* `max_slippage_bps`
* `enabled`

Required verification:

* Router program id matches ApprovedRoute.
* Inner venue program id matches ApprovedRoute.
* Pool account matches ApprovedRoute.
* Input mint matches expected mint.
* Output mint matches expected mint.
* Token program accounts match expected programs.
* Pool vaults exist.
* Pool vault mints match expected mints.
* Pool vault owner equals pool account.
* Route is direct.
* No intermediate mint is used.
* Min-out is enforced.
* Actual raw balance delta is verified after execution.

## 20.11 Raydium CLMM Requirements

Raydium CLMM remains a P0 venue candidate for xStocks.

However, Raydium CLMM readiness must be proven per asset and direction.

A Raydium CLMM route may become an ApprovedRoute candidate only when it is a direct route between USDC and the official provider-listed xStock mint.

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
* Tick arrays can be assembled for the target direction.
* Swap instruction can be constructed.
* Local/fork swap simulation succeeds.

## 20.12 Route Discovery vs ApprovedRoute

Jupiter, Titan, Byreal Router API, and other aggregators or router APIs may be used for route discovery and market intelligence.

However, route visibility does not automatically create an Axis ApprovedRoute.

Discovery may answer:

* Which venues currently provide liquidity
* Whether buy/sell routes exist
* Whether routes are one-way or two-way
* Whether routes are direct or multi-hop
* Whether intermediate mints appear
* Price impact quality
* Which venue labels appear in route discovery

But these are not sufficient for execution enablement.

An Axis ApprovedRoute requires:

* Known venue program
* Known router/venue program when applicable
* Known pool/market accounts
* Known token programs
* Known vaults/accounts
* On-chain validation
* Direction-specific approval
* Route-type approval
* Min-out enforcement
* Actual raw balance delta verification
* Route-specific risk controls

## 20.13 Pricing / NAV Requirements

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

Pool price may be useful for route execution and sanity checks.

Pool price must not automatically become NAV truth without an explicit pricing policy.

## 20.14 Updated Initial Spike Candidates

### 20.14.1 SPCXx / SpaceX xStock

SPCXx is the first high-priority xStocks execution spike candidate.

Current status:

* Official Solana mint verified
* On-chain mint exists
* Token-2022 verified
* 8 decimals verified
* Direct Byreal AMM quote verified for USDC -> SPCXx
* Direct Byreal AMM quote verified for SPCXx -> USDC
* Byreal AMM transaction build verified for both directions
* Same Byreal pool observed for both directions
* Byreal Router is the outer execution program
* Byreal CLMM program is present in the route account list
* Pool vaults verified on-chain
* USDC vault mint verified
* SPCXx vault mint verified
* Vault owner equals pool account
* RFQ not required for the observed route
* Intermediate mint route discovered separately and rejected for v1

Status:

```text
stocktoken_status = BYREAL_ROUTER_AMM_APPROVED_ROUTE_CANDIDATE
launch_readiness = SIMULATION_REQUIRED
```

SPCXx must not be marked launch-ready until simulation, min-out verification, compute/account measurement, transaction feasibility review, and Axis Core balance delta validation pass.

### 20.14.2 STRCx / Strategy PP Variable xStock

STRCx is the second xStocks execution spike candidate.

Current status:

* Official Solana mint verified
* On-chain mint exists
* Token-2022 verified
* 8 decimals verified
* Byreal buy route discovered
* Raydium CLMM sell route discovered
* Route readiness is direction-specific and mixed-venue
* Same-venue two-way ApprovedRoute readiness not yet verified

Status:

```text
stocktoken_status = DIRECTION_SPECIFIC_ROUTE_DISCOVERED
launch_readiness = ROUTE_REVIEW_REQUIRED
```

STRCx must not be marked launch-ready until both mint and redeem directions have approved direct official-mint routes and simulation passes.

## 20.15 Launch Readiness Gates

An xStock can become `LAUNCH_READY` only if all gates pass.

Required gates:

1. Official Solana mint verified.
2. Token-2022 ownership verified.
3. Scaled UI configuration verified.
4. Raw amount accounting supported.
5. Frontend/backend scaled display supported.
6. Direct USDC <-> official xStock route confirmed.
7. Route is direct.
8. No intermediate mint is used.
9. Venue adapter selected.
10. ApprovedRoute schema defined.
11. Direction-specific ApprovedRoute registered.
12. Pool account verified.
13. Venue/router program verified.
14. Token programs verified.
15. Vaults verified.
16. Vault mints verified.
17. Vault owner verified.
18. Liquidity screen passed.
19. Price impact threshold passed.
20. Pricing/NAV source defined.
21. Market-hours/stale-price policy defined.
22. Local or fork swap simulation passed.
23. Min-out and slippage protection passed.
24. Actual raw balance delta verification passed.
25. Compute unit usage measured.
26. Account count measured.
27. Transaction size feasibility measured.
28. 1-asset mint test passed.
29. 1-asset redeem test passed.
30. Multi-asset atomic feasibility reviewed.
31. Emergency disable controls exist.
32. Manual launch approval recorded.

## 20.16 Initial v1 Decision

Axis v1 should proceed with the following xStocks strategy:

* Include xStocks as a P0 candidate asset category.
* Do not enable all xStocks by default.
* Use explicit allowlist / enablement controls.
* Treat xStocks readiness as asset-, direction-, venue-, and route-type-specific.
* Treat SPCXx as the first Byreal Router AMM execution spike candidate.
* Treat STRCx as a mixed-venue direction-specific route review candidate.
* Treat Byreal Router AMM and Raydium CLMM as P0 venue candidates.
* Treat Jupiter/Titan routes as discovery signals, not protocol execution readiness.
* Reject multi-hop routes through non-official intermediate mints for v1.
* Do not mark any xStock as launch-ready until local/fork execution tests pass.
* Keep all other xStocks in route research status unless a verified executable direct route is found.

## 20.17 Development Tasks

### P0-1: Direction-specific ApprovedRoute schema

Purpose:
Define ApprovedRoute so that StockToken execution is approved per asset, direction, venue, and route type.

Required:

* input mint
* output mint
* venue type
* router program id when applicable
* inner venue program id when applicable
* pool id
* vaults
* token programs
* direction
* route type
* intermediate mint policy
* RFQ/Jupiter/DFlow flags
* slippage / price impact limits
* enable flag

### P0-2: Byreal Router AMM SPCXx simulation

Purpose:
Validate whether SPCXx can be safely swapped through Byreal Router AMM before integrating the route into Axis Core mint/redeem.

Scope:
This task validates direct Byreal Router AMM swap feasibility.
It does not require full Axis Core mint/redeem integration.

Targets:

* SPCXx / USDC

Required test cases:

* 10 USDC -> SPCXx
* SPCXx -> USDC
* min_out success
* min_out failure
* actual raw balance delta verification
* Token-2022 SPCXx transfer
* legacy SPL USDC transfer
* compute unit measurement
* account count measurement
* serialized transaction size measurement

Pass criteria:

* swap simulation succeeds in both directions
* output amount is >= min_out
* transaction fails when min_out is intentionally too high
* input/output raw token balance deltas match actual transfer results
* SPCXx uses Token-2022
* USDC uses legacy SPL Token
* pool vault balances change in the expected direction
* compute units are recorded
* account count is recorded
* transaction size is recorded

### P0-3: Raydium CLMM parallel validation

Purpose:
Keep Raydium CLMM as a P0 venue candidate, but validate it per asset and direction rather than assuming Raydium-only execution.

Targets:

* SPCXx Raydium route candidates
* STRCx Raydium route candidates

Required:

* reject routes through non-official intermediate mints
* validate only direct official-mint routes
* verify pool/vault/mint/account layout
* run local/fork simulation before enablement

### P0-4: Axis Core balance delta validation

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

## 20.18 Open Questions

The following remain open:

1. Can Byreal Router AMM be safely called from Axis Core CPI, or is a direct Byreal CLMM adapter required?
2. Can Raydium CLMM direct routes be found and simulated for enough launch candidates?
3. How many xStock routes can fit in one atomic mint/redeem transaction?
4. What are the compute unit and account count limits for 1-asset, 2-asset, and 5-asset DTF mint/redeem?
5. Should xStocks use separate per-asset max trade sizes?
6. Should xStocks require stricter stale-price policies than crypto assets?
7. Should SPCXx be enabled for creation before mint/redeem execution is enabled?
8. Should xStocks with only aggregator-visible routes remain display-only until a direct ApprovedRoute exists?
9. Should additional venues such as GoonFi be considered later as separate venue adapters?
10. How should Axis handle xStocks whose buy and sell directions require different venues?

## 20.19 Current Status

Current status:

* xStocks: P0 candidate category
* Ondo: future/partner-track provider, not launch-day execution provider
* SPCXx: Byreal Router AMM ApprovedRoute Candidate / Simulation Required
* STRCx: Direction-Specific Route Discovered / Route Review Required
* Byreal Router AMM: P0 venue candidate
* Raydium CLMM: P0 venue candidate
* Multi-hop / intermediate mint routes: rejected for v1
* Launch readiness: not yet achieved

Next required step:

```text
Define direction-specific ApprovedRoute schema, then run local/fork simulation for SPCXx Byreal Router AMM buy and sell. Verify min-out behavior, actual raw balance delta, compute units, account count, and transaction size.
```
