# Route Builder Backend API Requirements

## 1. Overview

This document defines the requirements for the Axis v1 route builder backend API.

The route builder backend is responsible for preparing structured route and transaction plans for Axis v1 DTF mint and redeem flows.

This document exists because Axis v1 requires a backend-owned planning layer rather than relying on legacy frontend transaction planners.

Axis v1 assumes:

```txt
- backend route builder is canonical for route and transaction planning
- frontend planners are reference / legacy only
- Axis Core remains the source of accounting truth
- backend outputs are advisory and must be validated on-chain
- mint and redeem are atomic in v1
- multi-transaction mint/redeem is out of scope
```

This document focuses primarily on:

```txt
- Mint route planning
- Redeem route planning
- ApprovedRoute references
- production venue account assembly
- quote freshness
- execution protection inputs
- transaction plan lifecycle
```

CreateMarket and fee claim flows are treated as lightweight transaction preparation flows because they do not require venue route discovery, quote generation, or per-asset execution planning.

This document defines API requirements, lifecycle, data categories, trust boundaries, and acceptance criteria.

This document does not finalize exact endpoint paths, exact JSON schemas, final TypeScript types, or final implementation-level serialization.

Exact implementation schemas may be finalized in a later implementation spec after backend engineering review.

## 2. Purpose

The purpose of the route builder backend is to convert user intent into a structured transaction plan that the app/client can use to construct valid Axis v1 transactions.

For mint, the route builder should help convert:

```txt
User wants to mint DTF with USDC
```

into:

```txt
- quote estimate
- per-asset route plan
- ApprovedRoute references
- venue account requirements
- min_out per asset
- required account categories
- transaction input preparation data
```

For redeem, the route builder should help convert:

```txt
User wants to redeem DTF into USDC
```

into:

```txt
- redeem quote estimate
- per-asset unwind route plan
- ApprovedRoute references
- venue account requirements
- min_usdc_out
- required account categories
- transaction input preparation data
```

The route builder does not decide final accounting results.

Axis Core must verify final accounting using actual on-chain balance deltas.

## 3. Goals

The goals of this document are:

```txt
- define the route builder backend responsibility boundary
- define required route / quote / transaction plan data categories
- define the route planning lifecycle
- define how route plans reference ApprovedRoute
- define how production venue account assembly should be handled
- define how min_out and min_usdc_out should be produced
- define how frontend/client should consume backend plans
- define on-chain validation assumptions
- define observability and audit requirements at the API level
- define compatibility boundaries with legacy Axis systems
```

## 4. Non-Goals

This document does not define:

```txt
- exact endpoint paths
- exact request / response JSON schema
- exact TypeScript types
- exact database schema
- ER diagram
- AWS infrastructure architecture
- final auth/session implementation
- final unsigned transaction format
- smart contract instruction discriminators
- full frontend UX
- PFDA AMM design
- LVR recapture design
```

Database schema, ER diagrams, and production infrastructure should be defined in a separate data infrastructure document.

PFDA AMM and LVR reduction should be defined in a separate protocol design document.

## 5. Confirmed Design Decisions

### 5.1 Backend Route Builder Is Canonical

The backend route builder is the canonical route and transaction planning layer for Axis v1.

Frontend planners may exist as reference or legacy code, but they must not define the canonical DTF v1 execution model.

Requirement:

```txt
Route planning, account assembly, and transaction input preparation must be backend-owned for Axis v1.
```

### 5.2 Structured Transaction Plan Is Required

The first version of the route builder API must return a structured route / transaction plan.

The structured plan is the primary API output.

Unsigned `VersionedTransaction` output may be supported later, but it is not required for the first API version.

Requirement:

```txt
v1 first API must return structured route / transaction plans.
Unsigned VersionedTransaction is optional / later.
```

### 5.3 Mint and Redeem Must Be Atomic

DTF v1 mint and redeem must be atomic.

Multi-transaction mint/redeem is out of scope for v1.

Requirement:

```txt
A mint or redeem transaction must either fully succeed or fully fail.
Partial reserve movement or partial accounting state must not be created.
```

If a 5-asset mint or redeem route cannot be executed atomically within v1 account, compute, venue, or transaction constraints, that route or asset composition must be treated as unsupported for v1.

The backend must not fall back to split or multi-transaction mint/redeem execution.

### 5.4 Jupiter Is Not Protocol Execution Readiness

Jupiter quote availability must not be treated as Axis execution readiness.

Jupiter must not automatically create or imply ApprovedRoute availability.

Jupiter may be used as a reference or external data source where useful, but Axis v1 execution readiness is based on Axis-approved venue routes.

Requirement:

```txt
Jupiter quote availability must not bypass ApprovedRoute requirements.
```

### 5.5 ApprovedRoute Is Protocol/Admin Controlled

ApprovedRoute is controlled by Axis protocol/admin configuration.

The backend route builder may reference existing ApprovedRoute entries, but it must not dynamically create trust or approve routes on behalf of the protocol.

Requirement:

```txt
Backend can reference ApprovedRoute.
Backend cannot create protocol trust dynamically.
```

### 5.6 Canonical Market ID

The canonical DTF market identifier is the on-chain `DTFMarket` address.

The DTF mint is the position token mint issued by that market.

The off-chain strategy registry is for display, indexing, metadata, and discovery only.

Requirement:

```txt
canonical_market_id = DTFMarket address
dtf_mint = position token mint
strategy_registry = display/indexing only
```

### 5.7 Legacy Axis Vault / PFMM Isolation

Legacy Axis Vault and PFMM flows must be isolated from the DTF v1 API model.

Existing implementation lessons may be reused, but legacy assumptions must not define the new API.

Reusable lessons include:

```txt
- wallet signing
- v0 transaction construction
- ALT handling
- simulation and failure handling
- metadata display patterns
```

Legacy assumptions to avoid:

```txt
- frontend-only Jupiter route assembly as canonical
- client-submitted trade accounting as protocol truth
- ambiguous vault_address semantics
- Kagemusha strategy assumptions
```

### 5.8 Auction / LVR Authority Is Separate From Mint/Redeem Route Planning

Axis-controlled JIT liquidity and LVR mitigation are specified separately in `17-auction-and-lvr-design-research.md` and `18-secondary-market-and-clear-correction-requirements.md`.

This document continues to focus on DTF v1 mint/redeem route planning. It additionally defines read/indexing data needed for the launch-day secondary-market surface, but it does not give the route builder auction authority.

Requirement:

```txt
- the route builder remains canonical for mint/redeem route planning
- the route builder is not the LVR-mitigation mechanism
- the route builder cannot determine auction winners or grant correction rights
- on-chain Axis Core and, where activated, the Axis Auction Program remain the validation and authorization authorities
```

## 6. Current Implementation Context

The existing Axis backend and app architecture has the following constraints:

```txt
- backend is currently a REST-style API
- backend does not currently provide DTF route builder plans
- backend does not currently return unsigned transactions
- backend does not currently return venue account metas
- frontend currently owns much of transaction construction logic
- frontend/backend shared typed schema is not yet established
- route / quote / ApprovedRoute / venue account persistence models do not yet exist
```

These constraints imply that Axis v1 requires a new route builder backend layer rather than a minor extension of the existing frontend planner.

## 7. Actors and Responsibilities

### 7.1 App / Client

The app/client is responsible for:

```txt
- collecting user intent
- requesting route plans from backend
- presenting estimates to users
- constructing transactions from backend-provided structured plans
- requesting wallet signature
- submitting transactions
- refreshing state after confirmation
- distinguishing estimates from actual results
```

The app/client must not treat backend estimates as final protocol accounting.

### 7.2 Route Builder Backend

The route builder backend is responsible for:

```txt
- quote estimate generation
- route discovery among approved routes
- ApprovedRoute lookup
- production venue account assembly
- route plan construction
- min_out / min_usdc_out suggestion
- transaction input preparation
- route plan freshness metadata
- route plan observability metadata
- secondary-market read and indexing data, where the backend serves the Axis-operated surface
```

The backend must not bypass on-chain validation, determine auction winners, or grant correction rights.

### 7.3 Axis Core

Axis Core is responsible for:

```txt
- validating DTFMarket
- validating ProtocolConfig
- validating AssetRegistry
- validating AssetExecutionPolicy
- validating ApprovedRoute
- validating venue program id
- validating venue accounts
- enforcing min_out / min_usdc_out
- executing CPI swaps
- measuring actual balance deltas
- computing minted DTF amount
- computing actual_usdc_received
- enforcing fee accounting
- preserving reserve accounting correctness
```

### 7.4 Protocol/Admin

Protocol/admin configuration is responsible for:

```txt
- creating ApprovedRoute entries
- disabling ApprovedRoute entries
- managing supported venue registry
- managing asset policies
- managing protocol-level execution limits
```

The backend route builder can only reference approved protocol state.

### 7.5 Off-Chain Storage / Indexing Layer

Off-chain storage may be used for:

```txt
- display metadata
- user-facing state
- route plan audit records
- quote request records
- transaction signatures
- execution attempt history
- indexed events
- analytics
- debugging and observability
```

Off-chain storage must not be treated as protocol truth.

The exact production database architecture, AWS deployment model, database schema, and ER diagram are out of scope for this document.

## 8. Route Builder Lifecycle

### 8.1 Mint Route Plan Lifecycle

A mint route plan should follow this lifecycle:

```txt
1. App submits mint intent
2. Backend reads DTFMarket and market asset list
3. Backend reads ProtocolConfig and fee config
4. Backend reads AssetRegistry and AssetExecutionPolicy
5. Backend discovers available ApprovedRoute entries
6. Backend selects route per target asset
7. Backend obtains quote estimates
8. Backend assembles venue account requirements
9. Backend computes suggested per-asset min_out
10. Backend creates route_plan_id
11. Backend returns structured mint route / transaction plan
12. App constructs transaction from plan
13. Wallet signs transaction
14. App submits transaction
15. Transaction signature is linked to route_plan_id where available
16. Axis Core validates and executes transaction atomically
17. App/backend/indexer refresh actual result
```

### 8.2 Redeem Route Plan Lifecycle

A redeem route plan should follow this lifecycle:

```txt
1. App submits redeem intent
2. Backend reads DTFMarket and reserve asset list
3. Backend reads DTF supply and user redeem amount
4. Backend estimates pro-rata asset unwind requirements
5. Backend reads AssetRegistry and AssetExecutionPolicy
6. Backend discovers available ApprovedRoute entries
7. Backend selects unwind route per asset
8. Backend obtains quote estimates
9. Backend assembles venue account requirements
10. Backend computes suggested min_usdc_out
11. Backend creates route_plan_id
12. Backend returns structured redeem route / transaction plan
13. App constructs transaction from plan
14. Wallet signs transaction
15. App submits transaction
16. Transaction signature is linked to route_plan_id where available
17. Axis Core validates and executes transaction atomically
18. App/backend/indexer refresh actual result
```

### 8.3 CreateMarket Preparation Lifecycle

CreateMarket is a lightweight transaction preparation flow.

It does not require venue quote generation or per-asset execution route planning.

A CreateMarket preparation flow may include:

```txt
- asset validation
- weight validation
- creator fee destination validation
- market metadata preparation
- reserve account derivation inputs
- DTF mint creation inputs where applicable
```

### 8.4 Fee Claim Preparation Lifecycle

Fee claim flows are lightweight transaction preparation flows.

They do not require venue quote generation or production venue account assembly.

A fee claim preparation flow may include:

```txt
- market lookup
- fee state lookup
- authorized recipient validation
- claimable amount display
- claim transaction input preparation
```

## 9. Required API Data Categories

This section defines required data categories.

It does not define final field names or exact JSON schema.

### 9.1 Common Route Plan Data

A route / transaction plan should contain:

```txt
- route_plan_id
- plan_version
- operation
- market_address
- dtf_mint
- user_authority
- created_at
- expires_at or ttl
- quote_context_slot
- ttl_slots where applicable
- network / cluster
- plan_status
```

`operation` should support:

```txt
- create_market
- mint
- redeem
- claim_creator_fee
- claim_protocol_fee
```

For v1 route execution, the primary focus is:

```txt
- mint
- redeem
```

### 9.2 Market Data

A route plan should reference:

```txt
- market_address
- dtf_mint
- creator
- market asset list
- target weights
- reserve token accounts
- fee custody account or derivation inputs
```

The canonical market ID must be the on-chain `DTFMarket` address.

### 9.3 Asset Data

A route plan should include or reference:

```txt
- asset_mint
- asset_decimals
- asset_index
- target_weight_bps
- asset policy reference
- pricing source reference where applicable
```

### 9.4 Route Data

Each route leg should include:

```txt
- route_leg_id
- approved_route reference
- venue_type
- route_direction
- input_mint
- output_mint
- estimated_input_amount
- estimated_output_amount
- min_output_amount
- price_impact_estimate where available
- quote_source
- quote_context_slot
- quote_expiry
```

For mint:

```txt
input_mint = USDC
output_mint = target asset
```

For redeem:

```txt
input_mint = reserve asset
output_mint = USDC
```

### 9.5 ApprovedRoute Reference

Each route leg must reference an ApprovedRoute.

A route plan must not treat an external quote as sufficient.

Required category:

```txt
- approved_route_id or equivalent reference
- approved_route account address where applicable
- approved_route enabled status
- approved_route direction
- approved venue type
```

Acceptance criteria:

```txt
- route leg without ApprovedRoute is invalid
- disabled ApprovedRoute is invalid
- route direction mismatch is invalid
- input/output mint mismatch is invalid
```

### 9.6 Venue Account Data

The backend route builder must provide the venue account data required for the app/client to construct the transaction.

Top-level route plan should be venue-agnostic.

Venue-specific payloads may differ by adapter.

Required category:

```txt
- venue_type
- venue_program_id
- pool / market / venue account reference
- token vault accounts
- tick / observation / oracle accounts where required
- token program accounts
- remaining accounts where required
```

Initial required venues:

```txt
- Orca Whirlpool
- Raydium CPMM fallback
```

High-priority future venue:

```txt
- PumpSwap
```

The route builder abstraction must not assume only one venue forever.

### 9.7 Execution Protection Data

For mint, the backend must provide:

```txt
- min_out per asset
```

For redeem, the backend must provide:

```txt
- min_usdc_out
```

These values are suggested by the backend but enforced by Axis Core.

Acceptance criteria:

```txt
- missing min_out must fail for mint
- missing min_usdc_out must fail for redeem
- min_out is checked against actual balance delta
- min_usdc_out is checked against actual USDC received
```

### 9.8 Transaction Input Preparation Data

The backend should provide enough structured data for the app/client to construct transactions.

Required category:

```txt
- instruction type
- market accounts
- user accounts
- reserve accounts
- fee custody accounts
- route accounts
- venue accounts
- token program accounts
- pricing accounts where applicable
- compute budget recommendation where applicable
- address lookup table references where applicable
```

The backend may later support unsigned `VersionedTransaction`, but this is optional for the first API version.

### 9.9 Secondary-Market Read and Indexing Data

Where the backend serves the Axis-operated secondary-market surface, it must provide read/indexing data separately from mint/redeem route plans.

Known external DTF/USDC pool references must include:

```txt
- venue
- pool address
- DTF mint
- USDC mint
- verification status
- external-liquidity classification
```

Market-level secondary-liquidity data must expose:

```txt
- market secondary-liquidity status
- whether the market is architecturally compatible but inactive for Axis-native auction/JIT liquidity
- auction/JIT activation readiness status, including unmet readiness categories where available
- active Axis Auction Program / ClearCorrection status only when it is actually available
- a clear distinction between external liquidity and Axis-native auction-enabled liquidity
```

This read model may include a canonical market URL or share URL and partner-facing metadata. It must not label an external pool as Axis-native or LVR-mitigated.

## 10. Quote Estimate Requirements

### RBA-001: Quote estimates must be advisory

Quote estimates are advisory only.

They must not be treated as final protocol accounting.

Acceptance criteria:

```txt
- app displays estimates as estimates
- backend does not claim final minted DTF amount
- backend does not claim final redeem USDC output
- Axis Core determines final accounting from actual balance deltas
```

### RBA-002: Quote estimates must include time-based freshness metadata

Quote estimates must include time-based freshness metadata.

Required category:

```txt
- quote_created_at
- quote_expires_at or ttl
- quote_source
```

Acceptance criteria:

```txt
- expired quote should not be used by the app/client
- stale quote can be rejected before transaction submission
- backend can explain quote freshness in logs or responses
```

### RBA-003: Quote estimates must include slot-based freshness metadata

Quote estimates must include slot-based freshness metadata where applicable.

Required category:

```txt
- quote_context_slot
- ttl_slots where applicable
- observed_slot where applicable
```

Acceptance criteria:

```txt
- route plan can be evaluated against Solana slot freshness
- stale slot-based quote can be rejected or marked stale
- time-based freshness and slot-based freshness can both be used
```

### RBA-004: Quote source must not imply route approval

A quote source must not imply protocol approval.

Acceptance criteria:

```txt
- Jupiter quote does not imply ApprovedRoute
- venue SDK quote does not imply ApprovedRoute
- external price availability does not imply execution readiness
- ApprovedRoute reference is separately required
```

## 11. Route Plan Requirements

### RBA-005: Backend must return a structured route / transaction plan

The backend route builder must return a structured plan.

Acceptance criteria:

```txt
- route plan has stable plan version
- route plan has unique route_plan_id
- route plan specifies operation
- route plan references market_address
- route plan includes route legs where applicable
- route plan includes required account data categories
- route plan includes execution protection data
```

### RBA-006: route_plan_id must be created at plan generation

A `route_plan_id` must be created when the backend generates a route plan.

If the user later submits a transaction, the resulting transaction signature should be linked back to the original `route_plan_id`.

Acceptance criteria:

```txt
- every generated mint route plan has route_plan_id
- every generated redeem route plan has route_plan_id
- route_plan_id can be logged or persisted
- tx_signature can be linked after transaction submission where available
- route_plan_id supports debugging, observability, and audit
```

### RBA-007: Route plan must be deterministic enough for transaction construction

The app/client must be able to construct a transaction from the route plan without redoing route discovery.

Acceptance criteria:

```txt
- app/client does not need to discover routes independently
- app/client does not need to choose venues independently
- app/client does not need to look up venue accounts independently
- app/client can construct transaction using plan data and current wallet state
```

### RBA-008: Route plan must not bypass Axis Core validation

The route plan must be validated by Axis Core during transaction execution.

Acceptance criteria:

```txt
- invalid ApprovedRoute fails on-chain
- invalid venue program id fails on-chain
- invalid venue accounts fail on-chain
- invalid input/output mint fails on-chain
- min_out failure fails on-chain
- min_usdc_out failure fails on-chain
```

### RBA-009: Route plan must support direct USDC routes for v1

For v1, route builder should prefer direct USDC routes.

For mint:

```txt
USDC -> target asset
```

For redeem:

```txt
target asset -> USDC
```

Acceptance criteria:

```txt
- direct USDC routes are prioritized
- assets without direct routes are unsupported in v1 unless explicit non-direct ApprovedRoute exists
- split routes are out of scope unless explicitly approved
- arbitrary multi-hop routes are out of scope unless explicitly approved
- non-direct routes require explicit ApprovedRoute support
```

## 12. Mint API Requirements

### RBA-010: Mint route plan must support USDC input

Mint route plans must start from user USDC input.

Required category:

```txt
- gross_user_usdc_in
- estimated_mint_fee_usdc
- estimated_net_usdc_for_composition
- per-asset estimated allocation
- per-asset route leg
- per-asset min_out
```

Acceptance criteria:

```txt
- gross_user_usdc_in is provided
- estimated fee is separated from net composition amount
- route legs are generated from net composition amount
- per-asset min_out is provided
- final accounting is still computed on-chain
```

### RBA-011: Mint route plan must support 2 to 5 assets

Mint route plans must support DTF markets with 2 to 5 reserve assets.

Acceptance criteria:

```txt
- route plan can include 2 asset legs
- route plan can include 5 asset legs
- every asset leg maps to ApprovedRoute
- missing route for any required asset makes route plan unavailable
```

### RBA-012: Mint route plan must be atomic

A mint route plan must assume atomic execution.

Acceptance criteria:

```txt
- backend does not return multi-transaction mint plan for v1
- app/client does not attempt partial mint execution
- failed route leg means entire mint fails
- failed mint must not create partial reserve state
```

### RBA-013: Unsupported atomic mint must be marked unsupported

If a mint route cannot be executed atomically within v1 constraints, the backend must mark the route or asset composition as unsupported.

Acceptance criteria:

```txt
- backend does not split mint into multiple transactions
- backend does not return partial mint plans
- route too large is returned as unsupported or transaction_too_large
- compute risk can be returned as unsupported or compute_limit_risk
- unsupported asset composition is clearly communicated
```

## 13. Redeem API Requirements

### RBA-014: Redeem route plan must support DTF input and USDC output

Redeem route plans must start from DTF amount input and estimate USDC output.

Required category:

```txt
- dtf_amount_in
- estimated redeem share
- estimated per-asset unwind amount
- per-asset unwind route leg
- estimated_usdc_out
- min_usdc_out
```

Acceptance criteria:

```txt
- dtf_amount_in is provided
- redeem share can be estimated
- per-asset unwind routes are provided
- min_usdc_out is provided
- final USDC output is measured on-chain
```

### RBA-015: Redeem route plan must assume redeem_fee_bps = 0 for v1

For v1, redeem fee is zero.

Acceptance criteria:

```txt
- backend does not estimate explicit Axis redeem fee
- backend may estimate execution spread / slippage
- user_usdc_out is based on actual_usdc_received
- execution spread is not represented as protocol fee
```

### RBA-016: Redeem route plan must be atomic

A redeem route plan must assume atomic execution.

Acceptance criteria:

```txt
- backend does not return multi-transaction redeem plan for v1
- app/client does not attempt partial redeem execution
- failed unwind route means entire redeem fails
- failed redeem must not create partial burn or reserve state
```

### RBA-017: Unsupported atomic redeem must be marked unsupported

If a redeem route cannot be executed atomically within v1 constraints, the backend must mark the route or asset composition as unsupported.

Acceptance criteria:

```txt
- backend does not split redeem into multiple transactions
- backend does not return partial redeem plans
- route too large is returned as unsupported or transaction_too_large
- compute risk can be returned as unsupported or compute_limit_risk
- unsupported asset composition is clearly communicated
```

## 14. CreateMarket and Fee Claim Preparation Requirements

### RBA-018: CreateMarket should be treated as lightweight transaction preparation

CreateMarket does not require route discovery or production venue account assembly.

Acceptance criteria:

```txt
- CreateMarket preparation validates required market input categories
- CreateMarket preparation does not require quote generation
- CreateMarket preparation does not require venue route planning
- CreateMarket preparation does not require ApprovedRoute lookup
```

### RBA-019: Fee claim should be treated as lightweight transaction preparation

Fee claim flows do not require route discovery or production venue account assembly.

Acceptance criteria:

```txt
- fee claim preparation can read claimable fee state
- fee claim preparation can identify claim recipient
- fee claim preparation does not require quote generation
- fee claim preparation does not require venue route planning
- fee claim preparation does not affect reserve accounting
```

## 15. Venue Adapter Requirements

### RBA-020: Route builder must support venue adapter abstraction

The route builder must support venue-specific account assembly behind a common route plan interface.

Required initial venue adapters:

```txt
- Orca Whirlpool
- Raydium CPMM
```

Future high-priority venue adapter:

```txt
- PumpSwap
```

Acceptance criteria:

```txt
- top-level RoutePlan is venue-agnostic
- venue-specific payloads are supported
- Orca Whirlpool account data can be assembled
- Raydium CPMM account data can be assembled
- PumpSwap can be added without redesigning the whole API
```

### RBA-021: Orca Whirlpool must be supported as first production venue candidate

The route builder must support Orca Whirlpool account assembly for approved routes.

Acceptance criteria:

```txt
- Orca Whirlpool route account data can be returned
- Orca Whirlpool venue program id can be referenced
- Orca-specific accounts can be represented in venue payload
- contract can validate Orca route accounts on-chain
```

### RBA-022: Raydium CPMM must be supported as fallback production venue

The route builder must support Raydium CPMM account assembly as fallback.

Acceptance criteria:

```txt
- Raydium CPMM route account data can be returned
- Raydium CPMM venue program id can be referenced
- Raydium-specific accounts can be represented in venue payload
- contract can validate Raydium route accounts on-chain
```

### RBA-023: PumpSwap must not be blocked by API design

PumpSwap is not a minimum first launch requirement, but the route builder API must not prevent future PumpSwap support.

Acceptance criteria:

```txt
- venue_type is extensible
- venue-specific payload can support new account layouts
- route plan format does not assume only Orca or Raydium
```

## 16. Simulation Requirements

### RBA-024: Backend simulation is optional for normal API responses

Normal user-facing route plan APIs are not required to simulate every route plan before returning a response.

Acceptance criteria:

```txt
- backend can return route plan without mandatory simulation
- route plan remains advisory
- app/client and Axis Core still enforce transaction safety
```

### RBA-025: Simulation support is required for mainnet readiness and smoke tests

Backend simulation support is required for mainnet readiness, local integration tests, mainnet-fork tests, and smoke tests.

Acceptance criteria:

```txt
- backend can simulate or test route plans in controlled environments
- Orca Whirlpool route plans can be validated in smoke tests
- Raydium CPMM route plans can be validated in smoke tests
- 5-asset mint route plan can be validated before launch
- 5-asset redeem route plan can be validated before launch
```

## 17. Off-Chain Storage and Observability

### RBA-026: Off-chain storage must not be protocol truth

Off-chain storage may be used for indexing, display, audit, and analytics.

It must not be used as final protocol accounting truth.

Acceptance criteria:

```txt
- reserve accounting truth comes from on-chain state
- minted DTF truth comes from on-chain state
- redeem output truth comes from on-chain state
- fee accounting truth comes from on-chain state
- off-chain DB is used for indexing and display only
```

### RBA-027: Route plans should be observable

The backend should make route plans observable for debugging and audit.

Recommended persisted or logged fields:

```txt
- route_plan_id
- operation
- market_address
- user_authority where appropriate
- selected routes
- ApprovedRoute references
- venue types
- quote source
- quote timestamp
- quote context slot
- expiry
- transaction signature when submitted
- execution result when indexed
- failure code where applicable
```

The exact database schema is out of scope.

### RBA-028: Production database architecture is deferred

The production DB architecture is not defined in this document.

Open mainnet may use AWS-backed infrastructure instead of Cloudflare D1.

The following should be defined separately:

```txt
- AWS architecture
- production database choice
- schema
- ER diagram
- indexing architecture
- user data model
- route/execution audit store model
```

### 17.1 Secondary-Market Read and Indexing Requirements

### RBA-037: Known external pools must be returned as read/indexing data

The backend may list known external DTF/USDC pools for a market. Each result must return venue, pool address, DTF mint, USDC mint, and verification status.

Acceptance criteria:

```txt
- an external pool result is explicitly classified as external liquidity
- pool identity metadata can be verified against the stated DTF and USDC mints
- unavailable or unverified data is not represented as a verified Axis-native pool
```

### RBA-038: Secondary-liquidity status must distinguish external and native liquidity

The read API must distinguish external public liquidity from an explicitly active Axis-native auction/JIT configuration.

Acceptance criteria:

```txt
- public Orca or Raydium DTF/USDC pools are returned as external unless separately activated as Axis-native
- external pools receive no Axis LVR-mitigation claim
- a market that is architecturally compatible but inactive is reported as inactive, not enabled
```

### RBA-039: Activation and ClearCorrection status must be read-only and evidence-based

The read API may expose auction/JIT activation readiness and active Auction Program / ClearCorrection status only from indexed events or validated on-chain state.

Acceptance criteria:

```txt
- readiness exposes relevant categories such as pool availability, pricing freshness, route support, Auction Program support, account/compute feasibility, and safety validation where known
- no active ClearCorrection status is returned for a market without an activated native configuration
- off-chain readiness data is not protocol authorization or accounting truth
```

### RBA-040: Route builder must not receive auction authority

The route builder may prepare advisory account or transaction data where separately authorized by the Auction Program design, but it must not select auction winners, grant correction rights, or bypass Axis Core or Auction Program validation.

Recommended event/indexing concepts:

```txt
- external_pool_discovered
- external_pool_verified
- auction_market_registered
- native_liquidity_activated
- native_liquidity_disabled
- clear_correction_executed
```

## 18. Authentication and Trust Requirements

### RBA-029: Read and quote endpoints may be public

Read-only and quote-style endpoints may be public if they do not mutate trusted state.

Examples:

```txt
- route estimate
- token metadata
- market display data
- route readiness check
```

### RBA-030: State-changing metadata endpoints must require authentication

Any endpoint that writes user-linked or creator-linked metadata must require wallet authentication, signature verification, or equivalent authorization.

Acceptance criteria:

```txt
- user-linked writes are not accepted blindly
- creator-linked metadata writes require authorization
- client-submitted TVL or execution values are not protocol truth
- backend avoids trusting unauthenticated accounting claims
```

Final authentication implementation is out of scope for this document.

## 19. Error and Failure Requirements

### RBA-031: Route builder must return clear failure reasons

The backend should return clear failure categories when route planning fails.

Failure categories may include:

```txt
- market_not_found
- market_paused
- asset_disabled
- direct_route_missing
- route_missing
- route_disabled
- unsupported_non_direct_route
- unsupported_venue
- quote_unavailable
- quote_expired
- pricing_stale
- account_assembly_failed
- transaction_too_large
- compute_limit_risk
- unsupported_asset_composition
- unsupported_operation
```

Exact error codes may be finalized later.

### RBA-032: Failed planning must not imply failed protocol state

A failed route plan request is an off-chain failure.

It must not imply any on-chain state change.

Acceptance criteria:

```txt
- failed route planning does not mutate protocol state
- failed route planning does not create user position
- failed route planning does not update accounting truth
```

### RBA-033: Failed execution must be indexed separately from estimates

If a transaction fails after a route plan is created, the failure should be observable separately from the original quote estimate.

Acceptance criteria:

```txt
- route plan can be associated with submitted transaction where available
- failed transaction can be classified where possible
- failed transaction is not indexed as successful mint/redeem
```

## 20. Legacy Compatibility Requirements

### RBA-034: Legacy frontend planners must be isolated

Legacy frontend planners may be used as reference implementations only.

They must not define canonical DTF v1 route builder behavior.

Acceptance criteria:

```txt
- DTF v1 route planning is backend-owned
- legacy frontend transaction planner does not bypass backend route builder
- legacy Jupiter-based assumptions are not treated as DTF v1 execution policy
```

### RBA-035: Legacy strategy registry may be adapted only for display

Existing strategy registry patterns may be adapted for display and discovery.

They must not replace on-chain DTFMarket state.

Acceptance criteria:

```txt
- strategy registry is display/indexing only
- DTFMarket address is canonical
- dtf_mint is explicitly stored or indexed
- legacy vault_address semantics are not reused ambiguously
```

### RBA-036: Legacy client-submitted accounting must not be reused as truth

Legacy client-submitted trade, TVL, or investment updates must not be treated as protocol truth.

Acceptance criteria:

```txt
- user position state is derived from on-chain state or indexed events
- client-submitted investment data is not trusted for accounting
- display data is clearly separated from protocol truth
```

## 21. Test Requirements

### 21.1 Route Plan Construction Tests

```txt
- construct mint route plan for 2-asset DTF
- construct mint route plan for 5-asset DTF
- construct redeem route plan for 2-asset DTF
- construct redeem route plan for 5-asset DTF
- route plan includes route_plan_id
- route plan includes market_address
- route plan includes ApprovedRoute references
- route plan includes venue account data categories
- route plan includes min_out or min_usdc_out
```

### 21.2 ApprovedRoute Tests

```txt
- route plan fails when ApprovedRoute is missing
- route plan fails when ApprovedRoute is disabled
- route plan fails when route direction is unsupported
- route plan fails when input/output mints do not match ApprovedRoute
- backend cannot create dynamic protocol trust
```

### 21.3 Venue Adapter Tests

```txt
- Orca Whirlpool account assembly succeeds
- Raydium CPMM account assembly succeeds
- unsupported venue returns clear failure
- PumpSwap placeholder/extensibility path does not break route plan format
```

### 21.4 Quote Freshness Tests

```txt
- quote contains created_at
- quote contains expires_at or ttl
- quote contains context slot
- quote contains ttl_slots where applicable
- expired quote is rejected or marked expired
- stale quote is not presented as fresh route readiness
```

### 21.5 Atomic Execution Assumption Tests

```txt
- backend does not return multi-transaction mint plan
- backend does not return multi-transaction redeem plan
- failed route leg prevents full mint plan readiness
- failed unwind route prevents full redeem plan readiness
- unsupported 5-asset atomic mint is marked unsupported
- unsupported 5-asset atomic redeem is marked unsupported
```

### 21.6 Direct Route Tests

```txt
- direct USDC -> asset route is preferred for mint
- direct asset -> USDC route is preferred for redeem
- asset without direct route is unsupported by default
- non-direct route is allowed only with explicit ApprovedRoute
- Jupiter multi-hop availability does not imply route readiness
```

### 21.7 Authentication Tests

```txt
- read endpoint can be public
- quote endpoint can be public
- route readiness endpoint can be public
- user-linked write requires wallet authentication or signature verification
- creator metadata write requires wallet authentication or signature verification
```

### 21.8 Integration Tests

```txt
- app can request mint plan
- app can request redeem plan
- app can construct transaction from structured plan
- app can distinguish estimate from actual result
- route_plan_id can be linked to tx_signature
- local integration test succeeds
- mainnet-fork or cloned-account integration test succeeds where applicable
```

### 21.9 Simulation / Smoke Tests

```txt
- backend simulation support exists
- Orca Whirlpool route plan smoke test succeeds
- Raydium CPMM route plan smoke test succeeds
- 5-asset mint route plan smoke test succeeds
- 5-asset redeem route plan smoke test succeeds
```

### 21.10 Secondary-Market Read and Indexing Tests

```txt
- known external pool result includes venue, pool address, DTF mint, USDC mint, and verification status
- external pool is labelled external liquidity
- external and active Axis-native statuses are distinguishable
- architecturally compatible but inactive market is not labelled auction/JIT enabled
- active Auction Program / ClearCorrection status is absent when unavailable
- read/indexing API cannot grant correction rights or choose winners
```

## 22. Mainnet Readiness Gate

The route builder backend is part of Axis v1 mainnet readiness.

Axis v1 should not be considered route-builder-ready unless:

```txt
- backend route builder is canonical for DTF v1 planning
- structured route / transaction plan API exists
- frontend planner is not canonical for DTF v1
- 16 focuses primarily on mint/redeem route planning
- CreateMarket and ClaimFee are lightweight transaction preparation flows
- mint route plan works for 2 to 5 assets
- redeem route plan works for 2 to 5 assets
- route plans reference ApprovedRoute
- backend cannot route through unapproved venues
- Orca Whirlpool account assembly is supported
- Raydium CPMM account assembly is supported
- direct USDC route preference is implemented
- assets without direct routes are unsupported unless explicit non-direct ApprovedRoute exists
- min_out per asset is provided for mint
- min_usdc_out is provided for redeem
- quote freshness metadata includes time-based and slot-based data
- route_plan_id is created at plan generation
- tx_signature can be linked to route_plan_id
- app can construct transactions from structured plans
- Axis Core validates route and accounting on-chain
- backend does not return multi-transaction mint/redeem plans
- unsupported atomic routes are marked unsupported
- read / quote / route readiness endpoints can be public
- user-linked writes require wallet authentication or signature verification
- local integration tests pass
- mainnet-fork or cloned-account integration tests pass where applicable
- simulation / smoke tests pass for required production venues
- secondary-market read/indexing data accurately distinguishes external and active Axis-native liquidity where the backend serves that surface
```

## 23. Relationship to Other Documents

This document depends on:

```txt
- 02-dtf-market-requirements.md
- 03-mint-requirements.md
- 04-redeem-requirements.md
- 05-swap-cpi-execution-requirements.md
- 06-pricing-nav-requirements.md
- 07-execution-policy-risk-controls.md
- 08-asset-universe-requirements.md
- 13-fee-model-requirements.md
- 14-production-venue-integration-requirements.md
- 15-app-contract-interface-requirements.md
- 17-auction-and-lvr-design-research.md
- 18-secondary-market-and-clear-correction-requirements.md
```

This document should be followed by separate documents for:

```txt
- PFDA AMM and LVR requirements
- data infrastructure and indexing requirements
- partner demo and BD requirements
- mainnet readiness requirements
```

## 24. Deferred Implementation Questions

The following are intentionally deferred:

```txt
- exact endpoint names
- exact request / response JSON schema
- exact TypeScript types
- exact shared schema package layout
- exact unsigned transaction format
- exact authentication implementation
- exact production database schema
- exact AWS deployment architecture
- exact ER diagram
- exact event indexing schema
- exact PumpSwap account assembly payload
```

## 25. Open Questions

```txt
- Should unsigned VersionedTransaction output be added in v1.1 or later?
- What quote TTL should be used for each venue type?
- What ttl_slots should be used for each venue type?
- What minimum simulation support is required before transaction submission?
- What shared schema package should be introduced between backend and frontend?
- What auth model should be used for user-linked metadata writes?
- What exact failure code taxonomy should be exposed to frontend?
```

## 26. Issue Candidates

```txt
- Define route builder backend responsibility boundary
- Implement structured route plan API
- Implement mint route plan generation
- Implement redeem route plan generation
- Implement route_plan_id generation
- Implement tx_signature linkage to route_plan_id
- Implement ApprovedRoute lookup in backend
- Implement Orca Whirlpool account assembly
- Implement Raydium CPMM account assembly
- Prepare PumpSwap-compatible venue adapter abstraction
- Add time-based quote freshness metadata
- Add slot-based quote freshness metadata
- Add per-asset min_out generation for mint
- Add min_usdc_out generation for redeem
- Add unsupported atomic route handling
- Add direct-route-only v1 policy
- Add non-direct ApprovedRoute exception handling
- Add route planning failure codes
- Add route plan observability logs
- Add frontend route plan consumer
- Isolate legacy frontend planner from DTF v1 execution path
- Add public read / quote / route readiness endpoints
- Add wallet-authenticated user-linked write endpoints
- Add local route builder integration tests
- Add mainnet-fork route builder integration tests
- Add 5-asset mint route plan test
- Add 5-asset redeem route plan test
- Add ApprovedRoute negative tests
- Add venue account assembly tests
- Add simulation / smoke tests for Orca Whirlpool
- Add simulation / smoke tests for Raydium CPMM
- Add external DTF/USDC pool indexing and verification-status reads
- Add secondary-liquidity and auction/JIT activation-status reads
- Add secondary-market event indexing for pool and ClearCorrection lifecycle events
```
