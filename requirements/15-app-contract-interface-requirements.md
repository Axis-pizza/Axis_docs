# App Contract Interface Requirements

## 1. Overview

This document defines the integration boundary between the Axis app/client/backend layer and Axis Core contracts.

This is not a frontend UX specification.

This document does not define layout, visual design, copywriting, product UX polish, or a full app redesign.

The purpose of this document is to define what the app, client, backend, and route builder must provide in order to construct valid Axis v1 transactions and read contract state correctly.

Axis v1 assumes that Axis operates a backend / route builder service.

The backend / route builder may support:

```txt id="1c8z2q"
- quote retrieval
- route discovery
- route plan construction
- required account assembly
- transaction input preparation
- app-facing estimates
- integration smoke tests
```

However, Axis Core must not trust backend-provided values as protocol truth.

Axis Core remains the source of truth for:

```txt id="sxq36l"
- market validation
- asset policy validation
- approved route validation
- venue account validation
- CPI execution
- min_out / min_usdc_out enforcement
- actual balance delta verification
- mint accounting
- redeem accounting
- fee accounting
- reserve custody validation
```

Exact backend API endpoints, request/response schemas, route builder API shape, transaction plan serialization, and frontend/backend data contracts are out of scope for this document.

Those details should be defined separately in:

```txt id="y95g9u"
16-route-builder-backend-api-requirements.md
```

after reviewing the existing Axis backend implementation and current API/schema structure.

This document defines required responsibilities and data categories, not final API schemas.

## 2. Goals

The goals of app contract interface requirements are:

```txt id="h6j8f5"
- define the app/backend/contract responsibility boundary
- define required read accounts for the app and backend
- define required data categories for CreateMarket transaction construction
- define required data categories for Mint transaction construction
- define required data categories for Redeem transaction construction
- define required data categories for fee claim transaction construction
- define what backend-provided values must be revalidated on-chain
- define integration events/logs needed for indexing and debugging
- define local and mainnet-fork integration validation scope
```

## 3. Non-Goals

This document does not require:

```txt id="1i5xvn"
- full frontend redesign
- visual design specification
- exact UI copy
- creator discovery UX
- social/feed UX
- portfolio analytics UX
- mobile-native UX
- public Devnet UX
- route builder API schema definition
- exact endpoint naming
- exact request / response schema
- exact transaction plan serialization
- app-side accounting as protocol truth
- backend-side accounting as protocol truth
```

Frontend display details may be specified in separate UI/product documents.

Backend API details should be specified in `16-route-builder-backend-api-requirements.md`.

## 4. Integration Boundary

### 4.1 App / Client Responsibilities

The app or client may be responsible for:

```txt id="o2slrv"
- collecting user intent
- selecting market creation inputs
- requesting quotes from backend
- requesting route plans from backend
- requesting required account lists from backend
- constructing transaction requests
- sending transactions through the user's wallet
- reading contract state
- reading indexed events
- refreshing balances and market data after confirmation
```

### 4.2 Backend / Route Builder Responsibilities

The backend / route builder may be responsible for:

```txt id="91v1sg"
- route discovery
- quote retrieval
- venue account lookup
- ApprovedRoute lookup
- route plan construction
- required account assembly
- min_out / min_usdc_out suggestion
- transaction input preparation
- simulation support
- app-facing estimate generation
```

Backend outputs are advisory inputs.

Backend outputs must not bypass on-chain validation.

The exact backend API shape, endpoint names, and request/response schemas must be defined separately after reviewing the existing Axis backend.

### 4.3 Axis Core Responsibilities

Axis Core must be responsible for:

```txt id="c6c3cz"
- validating DTFMarket state
- validating ProtocolConfig
- validating MarketFeeState
- validating AssetRegistry entries
- validating AssetExecutionPolicy
- validating ApprovedRoute
- validating venue program id
- validating venue-specific accounts
- validating input and output mints
- enforcing min_out / min_usdc_out
- executing CPI swaps
- measuring actual token balance deltas
- computing actual_added_value_usdc
- computing minted DTF amount
- computing actual_usdc_received on redeem
- enforcing fee accounting
- preserving reserve and fee custody separation
```

## 5. Trust Boundary

The app and backend may provide estimates.

The app and backend must not determine final protocol accounting.

The following values may be estimated off-chain, but must not be trusted as final truth:

```txt id="zkpamw"
- expected output amount
- expected minted DTF amount
- expected redeem USDC output
- expected price impact
- expected NAV
- expected route success
- expected actual_added_value_usdc
```

The following values must be determined or verified by Axis Core:

```txt id="ckotgk"
- actual_received per asset
- actual reserve balance deltas
- actual_usdc_received on redeem
- minted DTF amount
- min_out / min_usdc_out satisfaction
- fee accrual amounts
- reserve custody validity
- fee custody separation
```

## 6. Required Read Model

The app/backend must be able to read the following state for integration.

This section defines required read categories, not exact API response schemas.

### 6.1 Protocol-Level State

```txt id="r0fdhr"
- ProtocolConfig
- protocol fee config
- protocol fee caps
- USDC mint
- supported venue registry
- global pause or emergency state where applicable
```

### 6.2 Market-Level State

```txt id="m8cspt"
- DTFMarket
- DTF mint
- creator
- creator_fee_destination
- market status
- market asset list
- target weights
- reserve token accounts
- market fee config snapshot
- accrued creator fee amount
- accrued protocol fee amount
- fee custody reference
```

### 6.3 Asset-Level State

```txt id="y5tmx6"
- AssetRegistry entry
- asset mint
- asset decimals
- creation_enabled
- mint_enabled
- redeem_enabled
- max_weight_bps
- max_trade_usdc
- max_price_impact_bps
- pricing source reference
```

### 6.4 Route-Level State

```txt id="a90dr7"
- ApprovedRoute
- route id
- venue type
- venue program id
- pool / market / venue account id
- input mint
- output mint
- route direction
- route enabled flag
- route risk or policy metadata where applicable
```

### 6.5 Reserve / NAV State

```txt id="wxr8bd"
- reserve token account balances
- DTF total supply
- approved prices
- NAV-related state or inputs
- pricing freshness data
```

Fee vault balances must not be treated as reserve balances.

Accrued creator/protocol fees must not be treated as DTF backing.

## 7. Required Transaction Flows

Axis v1 app/backend integration must support the following contract interaction categories:

```txt id="ujka61"
1. CreateMarket
2. Mint
3. Redeem
4. ClaimCreatorFee
5. ClaimProtocolFee
6. Read Market / Reserve / NAV State
7. Read Route Readiness
```

Exact instruction names may be finalized in the implementation spec.

If instruction names differ, the behavior and integration requirements must remain equivalent.

## 8. Requirements

### APPIF-001: App/backend must treat Axis Core as accounting source of truth

The app and backend must not treat their own calculations as final accounting truth.

Acceptance criteria:

```txt id="mpu1ch"
- backend quote is not used as final minted DTF amount
- backend quote is not used as final redeem USDC output
- app estimates are separate from contract results
- Axis Core computes or verifies final accounting values
```

### APPIF-002: Backend route builder may prepare route plans

Axis v1 may use a backend route builder service to prepare route plans and required accounts.

Acceptance criteria:

```txt id="3xygcz"
- backend can provide quote data
- backend can provide route plan data
- backend can provide required route account data
- backend can suggest min_out / min_usdc_out
- backend can prepare transaction input data
- contract still validates route and execution safety on-chain
```

### APPIF-003: Backend route plan must map to ApprovedRoute

Every backend-provided route plan must map to an enabled ApprovedRoute.

Acceptance criteria:

```txt id="5ph7dk"
- route plan includes route id or equivalent route reference
- route reference maps to ApprovedRoute
- ApprovedRoute missing fails on-chain
- ApprovedRoute disabled fails on-chain
- backend cannot route through unapproved venue
```

### APPIF-004: Backend-provided venue accounts must be revalidated on-chain

Venue accounts assembled by the backend must be validated by Axis Core.

Acceptance criteria:

```txt id="xttqqe"
- venue program id is validated on-chain
- pool / market / venue account is validated on-chain
- token vault accounts are validated on-chain
- input token account is validated on-chain
- output token account is validated on-chain
- wrong venue account fails
```

### APPIF-005: CreateMarket integration must provide required market data categories

The app/backend must support constructing CreateMarket transactions.

Required data categories include:

```txt id="qrt4ui"
- creator
- asset mints
- target weights
- creator_fee_destination
- required reserve account references or derivation inputs
- DTF mint creation inputs where applicable
```

Acceptance criteria:

```txt id="9w480n"
- 2 to 5 assets can be submitted
- target weights sum to 10000 bps
- duplicate assets are rejected or prevented
- creator_fee_destination is provided
- creator cannot customize fee bps
- market fee config is derived from protocol config or preset
```

### APPIF-006: Mint integration must provide required mint data categories

The app/backend must support constructing Mint transactions.

Required data categories include:

```txt id="k6gbrn"
- market
- user authority
- user USDC source account
- user DTF destination account
- gross_user_usdc_in
- per-asset route plan
- per-asset min_out
- reserve token accounts
- fee custody account
- required venue accounts
- required token program accounts
- required asset policy accounts
- required pricing accounts where applicable
```

Acceptance criteria:

```txt id="z0hmx0"
- mint input uses USDC
- gross_user_usdc_in is provided
- min_out is provided per asset
- route account data is provided per asset
- reserve account data is provided per asset
- fee custody account is provided or derivable
- contract verifies all critical accounts on-chain
```

### APPIF-007: Mint integration must not provide final minted DTF amount as truth

The app/backend may estimate minted DTF amount, but Axis Core must compute the actual minted DTF amount.

Mint accounting flow:

```txt id="0w1sox"
gross_user_usdc_in
-> mint_fee_usdc
-> net_usdc_for_composition
-> CPI execution
-> actual reserve balance deltas
-> actual_added_value_usdc
-> minted_dtf = actual_added_value_usdc / pre_trade_nav
```

Acceptance criteria:

```txt id="czzpom"
- backend may estimate minted DTF amount
- contract does not trust estimated minted DTF amount
- contract measures actual reserve balance deltas
- contract computes actual_added_value_usdc
- contract mints DTF based on actual_added_value_usdc
```

### APPIF-008: Redeem integration must provide required redeem data categories

The app/backend must support constructing Redeem transactions.

Required data categories include:

```txt id="jovjyu"
- market
- user authority
- user DTF source account
- user USDC destination account
- dtf_amount_in
- per-asset unwind route plan
- min_usdc_out
- reserve token accounts
- required venue accounts
- required token program accounts
- required asset policy accounts
- required pricing accounts where applicable
```

Acceptance criteria:

```txt id="fbcruh"
- redeem input uses market DTF token
- dtf_amount_in is provided
- min_usdc_out is provided
- unwind route account data is provided
- reserve account data is provided per asset
- contract verifies all critical accounts on-chain
```

### APPIF-009: Redeem integration must not provide final USDC output as truth

The app/backend may estimate redeem output, but Axis Core must measure actual USDC received.

Redeem accounting flow:

```txt id="c68y5i"
redeem_fee_bps = 0
actual_usdc_received = post_usdc_balance - pre_usdc_balance
user_usdc_out = actual_usdc_received
```

Acceptance criteria:

```txt id="3a6ze4"
- backend may estimate USDC output
- contract does not trust estimated USDC output
- contract measures actual USDC balance delta
- contract enforces min_usdc_out
- user_usdc_out equals actual_usdc_received for v1
```

### APPIF-010: Integration must support fee claim transaction construction

The app/backend must support constructing fee claim transactions.

Fee claim flows should include:

```txt id="b1bonj"
- ClaimCreatorFee
- ClaimProtocolFee
```

Exact instruction discriminators and implementation names may be finalized in the implementation spec.

Acceptance criteria:

```txt id="iaf9rw"
- creator fee claim can identify market
- creator fee claim can identify authorized recipient
- protocol fee claim can identify authorized recipient
- claim amount is derived from accrued fee state
- claim flow does not use reserve custody
- claim flow does not affect NAV
- creator and protocol authorization paths remain distinct
```

### APPIF-011: App/backend must read fee state separately from reserve state

Fee state and reserve state must be read and handled separately.

Acceptance criteria:

```txt id="e5r39l"
- fee custody account is not treated as reserve account
- accrued_creator_fee_usdc is not included in NAV
- accrued_protocol_fee_usdc is not included in NAV
- fee balances are not treated as DTF backing
- fee claim does not affect reserve display or reserve accounting
```

### APPIF-012: Integration must support route readiness checks

The app/backend must be able to determine whether a market has route support for mint and redeem.

Acceptance criteria:

```txt id="ei9mon"
- app/backend can query ApprovedRoute availability
- app/backend can detect disabled routes
- app/backend can detect unsupported venues
- app/backend can distinguish quote availability from ApprovedRoute readiness
- route readiness does not bypass on-chain validation
```

### APPIF-013: Integration must support production venue account assembly boundary

The backend route builder must support required production venue account assembly at the integration boundary.

Initial required venues:

```txt id="3jnldc"
- Orca Whirlpool
- Raydium CPMM fallback
```

High-priority future venue:

```txt id="mdn90a"
- PumpSwap
```

Acceptance criteria:

```txt id="lo4tod"
- backend can provide account data needed for Orca Whirlpool routes
- backend can provide account data needed for Raydium CPMM routes
- backend abstraction does not block future PumpSwap support
- backend does not assume a single venue forever
- contract still validates venue accounts on-chain
```

### APPIF-014: Integration must support direct USDC route assumption for v1

The app/backend route builder should initially prefer direct USDC routes.

For mint:

```txt id="zuy5ct"
USDC -> target asset
```

For redeem:

```txt id="v45vqf"
target asset -> USDC
```

Acceptance criteria:

```txt id="g2g40s"
- backend prioritizes direct USDC -> asset routes for mint
- backend prioritizes direct asset -> USDC routes for redeem
- split routes are not constructed for v1 unless explicitly approved
- arbitrary multi-hop routes are not constructed for v1 unless explicitly approved
- non-direct routes require explicit ApprovedRoute support
```

### APPIF-015: Integration must support min_out and min_usdc_out data

The app/backend must provide execution protection data categories.

For mint:

```txt id="fcx3u1"
min_out_i per asset
```

For redeem:

```txt id="zlxz7w"
min_usdc_out
```

Acceptance criteria:

```txt id="fdp6yz"
- mint transaction includes min_out per asset
- redeem transaction includes min_usdc_out
- missing min_out fails on-chain
- missing min_usdc_out fails on-chain
- min_out / min_usdc_out are enforced against actual balance deltas
```

### APPIF-016: Integration must support actual result indexing

The app/backend/indexer should be able to read actual execution results from logs, events, or refreshed account state.

Mint result data should include:

```txt id="1o159s"
- market id
- user
- gross_user_usdc_in
- mint_fee_usdc
- creator_fee_usdc
- protocol_fee_usdc
- net_usdc_for_composition
- actual_received per asset
- actual_added_value_usdc
- minted DTF amount
```

Redeem result data should include:

```txt id="jh8wo2"
- market id
- user
- dtf_amount_in
- redeem_share
- actual_usdc_received
- user_usdc_out
```

Acceptance criteria:

```txt id="l6pf67"
- integration can observe successful mint result
- integration can observe successful redeem result
- integration can distinguish estimate from actual result
- integration can refresh state after transaction confirmation
```

### APPIF-017: Integration must support failure observability

The app/backend/indexer should be able to detect and classify transaction failures where possible.

Failure examples:

```txt id="7idvpz"
- market paused
- asset disabled
- route missing
- route disabled
- wrong venue account
- min_out failure
- min_usdc_out failure
- price impact failure
- stale pricing
- wallet rejection
- transaction simulation failure
```

Acceptance criteria:

```txt id="cmm4xz"
- failed mint is not treated as successful
- failed redeem is not treated as successful
- validation failure reason can be surfaced where available
- backend can use simulation or logs for debugging
- failed transaction does not update indexed state as success
```

### APPIF-018: Integration must support local contract testing

The app/backend integration must support local contract testing.

Acceptance criteria:

```txt id="roiq5q"
- backend can connect to local validator
- app/client can construct transactions for local deployment
- route builder can use controlled adapter in local tests
- local create market transaction can be constructed
- local mint transaction can be constructed
- local redeem transaction can be constructed
```

### APPIF-019: Integration must support mainnet-fork or cloned-account testing

The app/backend integration must support mainnet-fork or cloned-account testing where applicable.

Acceptance criteria:

```txt id="pme1wn"
- backend can connect to fork-based or cloned-account environment
- route builder can provide production venue account data in fork-based tests
- app/client can construct transactions for fork-based deployment
- Orca Whirlpool path can be tested in fork-based environment
- Raydium CPMM path can be tested in fork-based environment
- actual balance delta behavior can be validated in fork-based environment
```

### APPIF-020: Integration must not require public Devnet

Public Devnet is not mandatory for Axis v1 app contract integration.

Acceptance criteria:

```txt id="h87zzs"
- public Devnet is not a required integration gate
- test deployments may use local, fork-based, or internal environments
- main app UI does not require Devnet-specific indicator
- Devnet may be used opportunistically but is not required
```

### APPIF-021: Integration must avoid investment-advice assumptions

The app/backend should not encode assumptions that Axis recommends assets or portfolios.

Acceptance criteria:

```txt id="d273ha"
- market creation is treated as user/creator-driven
- route building does not imply asset recommendation
- backend estimates do not imply guaranteed performance
- DTFs are represented as tokenized basket markets, not recommendations
```

## 9. Event / Log Requirements

Axis Core should emit or expose enough information for app/backend/indexer integration.

Exact event/log encoding is implementation-specific.

However, the required fields must be documented and stable enough for app/backend/indexer consumption.

### 9.1 CreateMarket Event

Recommended fields:

```txt id="gc5ut0"
- market id
- creator
- DTF mint
- asset mints
- target weights
- creator_fee_destination
- market fee config snapshot
```

### 9.2 Mint Event

Recommended fields:

```txt id="dzgmw9"
- market id
- user
- gross_user_usdc_in
- mint_fee_usdc
- creator_fee_usdc
- protocol_fee_usdc
- net_usdc_for_composition
- actual_received per asset
- actual_added_value_usdc
- pre_trade_nav
- minted_dtf_amount
```

### 9.3 Redeem Event

Recommended fields:

```txt id="tolb4d"
- market id
- user
- dtf_amount_in
- redeem_share
- actual_usdc_received
- user_usdc_out
- min_usdc_out
```

### 9.4 Fee Claim Event

Recommended fields:

```txt id="pqpbtk"
- market id
- claimer
- recipient
- fee type
- claimed amount
```

## 10. Required Test Scenarios

### 10.1 Read Model Tests

```txt id="c6piww"
- read ProtocolConfig
- read DTFMarket
- read MarketFeeState
- read AssetRegistry entries
- read AssetExecutionPolicy
- read ApprovedRoute
- read reserve token accounts
- read fee custody separately from reserves
```

### 10.2 CreateMarket Integration Tests

```txt id="3dii2a"
- construct CreateMarket transaction from required data categories
- provide 2 assets
- provide 5 assets
- provide target weights
- provide creator_fee_destination
- ensure fee bps customization is not exposed as market input
- verify transaction fails with invalid assets or weights
```

### 10.3 Mint Integration Tests

```txt id="q7ujxh"
- construct Mint transaction from required data categories
- provide gross_user_usdc_in
- provide per-asset route plan
- provide per-asset min_out
- provide reserve accounts
- provide venue accounts
- verify contract computes actual minted DTF amount
- verify failed mint does not accrue fees
```

### 10.4 Redeem Integration Tests

```txt id="oqwqc4"
- construct Redeem transaction from required data categories
- provide dtf_amount_in
- provide unwind route plan
- provide min_usdc_out
- provide reserve accounts
- provide venue accounts
- verify contract measures actual_usdc_received
- verify no redeem fee accrues
```

### 10.5 Route Builder Boundary Tests

```txt id="9gnylc"
- route builder output maps to ApprovedRoute
- route builder provides account data needed for Orca Whirlpool
- route builder provides account data needed for Raydium CPMM
- route builder prioritizes direct USDC routes
- route builder does not treat SDK quote as route approval
- route builder handles missing route as unsupported
- contract rejects invalid route builder output
```

### 10.6 Event / Indexing Tests

```txt id="09evfb"
- CreateMarket event can be indexed
- Mint event can be indexed
- Redeem event can be indexed
- Fee claim event can be indexed
- indexed actual result differs from estimate when execution differs
```

### 10.7 Environment Tests

```txt id="xabfim"
- local integration test succeeds
- controlled adapter local test succeeds
- mainnet-fork or cloned-account integration test succeeds where applicable
- Orca Whirlpool fork-based integration test succeeds
- Raydium CPMM fork-based integration test succeeds
```

## 11. Mainnet Readiness Gate

App contract interface integration is part of mainnet readiness.

Axis v1 should not be considered integration-ready unless:

```txt id="e2i35i"
- backend route builder can prepare required route data categories
- app/client can construct CreateMarket transactions
- app/client can construct Mint transactions
- app/client can construct Redeem transactions
- app/client can construct fee claim transactions where applicable
- app/backend can read DTFMarket state
- app/backend can read reserve state
- app/backend can read fee state separately from reserve state
- app/backend can read ApprovedRoute state
- app/backend can distinguish quote estimates from contract results
- app/backend supports Orca Whirlpool route account data
- app/backend supports Raydium CPMM route account data
- local integration tests pass
- mainnet-fork or cloned-account integration tests pass where applicable
```

## 12. Out of Scope

The following are out of scope for this document:

```txt id="wvbslv"
- frontend visual design
- full app UX redesign
- exact UI layout
- exact UI copy
- social feed
- creator discovery
- portfolio analytics
- mobile-native design
- public Devnet UX
- route builder API schema
- endpoint naming
- request / response schema
- transaction plan serialization format
- investment recommendation engine
- automated asset recommendation
```

## 13. Deferred API / Backend Schema Questions

The following items are intentionally deferred to:

```txt id="3srfus"
16-route-builder-backend-api-requirements.md
```

These should be finalized only after reviewing the existing Axis backend implementation, current API endpoints, current schemas, and current transaction construction flow.

Deferred items:

```txt id="2v2yoj"
- Exact route builder API shape
- Exact endpoint names
- Exact request / response schemas
- Exact transaction input serialization format
- Exact transaction plan object shape
- Exact frontend/backend data contract
- Exact route account serialization format
- Exact PumpSwap route account assembly design
```

## 14. Remaining Open Questions

```txt id="0g75ev"
- Exact event/log implementation format
- Exact local integration test harness
- Exact mainnet-fork / cloned-account test harness
- Whether fee claim instruction names should be finalized as ClaimCreatorFee and ClaimProtocolFee in the implementation spec
```

## 15. Issue Candidates

```txt id="zs2qfb"
- Define app/backend/contract integration boundary
- Define required read model for app/backend integration
- Define CreateMarket integration data categories
- Define Mint integration data categories
- Define Redeem integration data categories
- Define fee claim integration data categories
- Implement ProtocolConfig read support
- Implement DTFMarket read support
- Implement MarketFeeState read support
- Implement reserve account read support
- Implement ApprovedRoute read support
- Implement fee custody read support
- Implement backend route plan integration boundary
- Implement Orca Whirlpool account assembly support at integration boundary
- Implement Raydium CPMM account assembly support at integration boundary
- Ensure route builder outputs are revalidated on-chain
- Ensure backend estimates are not treated as protocol truth
- Implement local integration test harness
- Implement mainnet-fork integration test harness
- Implement CreateMarket integration test
- Implement Mint integration test
- Implement Redeem integration test
- Implement fee claim integration test
- Implement event/indexing integration tests
```
