# Devnet Rollout Requirements

## 1. Overview

This document defines the Devnet rollout requirements for Axis v1.

The purpose of Devnet is not only to prove that the program can be deployed.

The purpose of Devnet is to validate that Axis behaves like a real reserve-backed DTF protocol before moving toward mainnet migration.

Devnet should validate the following:

```txt
- Axis Core DTF lifecycle
- reserve-backed mint and redeem accounting
- real token movement
- actual balance delta verification
- app-level usability
- partner-facing demo readiness
- mainnet readiness evidence
```

Devnet rollout is a product and protocol validation environment.

It should help the team align around the correct Axis v1 model before updating the current mainnet contracts and integrating the new protocol into the product.

## 2. Devnet Goal

Devnet must prove that Axis can create and operate a minimal DTF market end-to-end.

The core lifecycle is:

```txt
create DTF market
-> mint with USDC
-> execute controlled swap path
-> receive underlying reserve assets
-> verify actual reserve balance deltas
-> mint DTF token

redeem DTF token
-> calculate pro-rata reserve share
-> execute controlled unwind path
-> receive USDC
-> verify actual USDC balance delta
-> burn or remove redeemed DTF token
```

Devnet must validate the same accounting model required for production:

```txt
Target weights define intended composition.
Actual reserve balances define accounting value.
```

Devnet must not treat frontend quotes, target weights, or mocked accounting as final protocol truth.

## 3. Scope Boundary

This document defines Devnet rollout requirements.

It does not compare these requirements against the current contract repository or frontend repository.

Out of scope for this document:

```txt
- current contract repository gap analysis
- current frontend repository gap analysis
- file-level implementation planning
- UI component-level task breakdown
- final instruction ABI
- final account layout
- final venue adapter implementation
- final mainnet deployment checklist
```

Those workstreams should begin after the requirements are stable enough to serve as the source of truth.

## 4. Devnet Validation Model

Devnet rollout should be based on both a minimum validation window and explicit readiness criteria.

Devnet success must not be defined only by calendar time.

```txt
Devnet should run for a minimum validation window and must pass required readiness criteria.
```

The exact minimum validation window is not fixed at this stage.

Candidate validation windows:

```txt
- 7 days
- 14 days
- 21 to 30 days
```

The validation window should be selected based on:

```txt
- launch timing
- implementation confidence
- partner demo needs
- amount of risk discovered during testing
- venue integration progress
- app integration readiness
```

A Devnet rollout is considered ready for mainnet candidate review only when the required readiness criteria are satisfied.

## 5. Controlled Liquidity Policy

Functional Devnet must not depend on existing public Devnet DEX liquidity.

Existing public Devnet pools may be unavailable, unstable, insufficient, or hard to discover.

For Functional Devnet, Axis may use:

```txt
- controlled test assets
- controlled test USDC
- controlled liquidity
- controlled approved routes
- ManualFixedPrice pricing
- controlled CPI adapter
```

The purpose of controlled liquidity is to validate Axis Core without making public Devnet DEX liquidity a blocker.

Public DEX venue integrations should be validated separately in Venue Integration Devnet.

## 6. Controlled CPI Adapter Policy

Functional Devnet may use a controlled CPI adapter.

The purpose of the controlled CPI adapter is to validate the Axis Core mint and redeem lifecycle without depending on existing public Devnet DEX liquidity.

However, mock accounting is not acceptable as Functional Devnet success.

The controlled CPI adapter must move real SPL tokens and allow Axis Core to verify actual token balance deltas.

Required properties:

```txt
- Axis Core must invoke the adapter through a real CPI call
- the adapter must move real SPL tokens
- mint execution must increase reserve asset balances
- redeem execution must produce USDC or test USDC output
- Axis Core must measure pre/post token balances
- actual balance delta must be used as accounting truth
- min_out checks must be enforced
- failed execution must revert the full mint or redeem transaction
```

A controlled CPI adapter is acceptable for Functional Devnet only if it validates the same accounting invariants required for production.

## 7. Devnet Rollout Phases

### Phase 1: Core Functional Devnet

Core Functional Devnet validates the core DTF lifecycle.

This phase may use controlled test assets, controlled liquidity, ManualFixedPrice pricing, and a controlled CPI adapter.

Required capabilities:

```txt
- initialize protocol config
- register assets
- set asset execution policies
- set pricing sources
- register approved routes
- create a 2-asset DTF market
- mint DTF tokens with USDC or test USDC
- execute a real CPI path with controlled liquidity
- receive underlying assets into reserve accounts
- measure actual reserve balance deltas
- calculate actual added value
- mint DTF tokens based on pre-trade NAV
- redeem DTF tokens
- unwind reserve assets back to USDC or test USDC
- enforce min_out
- burn or remove redeemed DTF tokens from circulating supply
```

### Phase 2: App Integrated Devnet

App Integrated Devnet connects the Axis frontend or client to the Core Functional Devnet deployment.

The purpose is to make the DTF lifecycle usable from the product interface.

Detailed app requirements are defined separately in `13-app-integration-requirements.md`.

At minimum, App Integrated Devnet should allow users to:

```txt
- create or load a Devnet DTF market
- mint DTF tokens
- redeem DTF tokens
- view reserve balances
- view NAV
- view transaction results
- understand that they are using Devnet
```

### Phase 3: Partner Demo Devnet

Partner Demo Devnet is the environment used for BD conversations with external projects.

The purpose is to demonstrate how a partner ecosystem, asset, or narrative can become a DTF market on Axis.

Detailed partner demo requirements are defined separately in `14-partner-bd-demo-requirements.md`.

At minimum, Partner Demo Devnet should support:

```txt
- at least one end-to-end demo DTF
- create transaction proof
- mint transaction proof
- redeem transaction proof
- reserve backing display
- clear explanation that the DTF is not a frontend-only basket
```

### Phase 4: Venue Integration Devnet

Venue Integration Devnet validates external DEX venue integrations separately from the core DTF lifecycle.

This phase should investigate whether Axis can safely integrate with external venues such as:

```txt
- Orca Whirlpool
- Raydium CPMM
- PumpSwap
- Meteora DLMM
```

For each venue, the team should verify:

```txt
- Devnet program availability
- pool creation support
- liquidity provisioning support
- swap instruction construction
- CPI compatibility
- account requirements
- compute usage
- Token / Token-2022 compatibility
- route validation requirements
- failure modes
```

A venue must not be considered Axis-ready only because a route exists off-chain or through an SDK.

A venue is Axis-ready only when Axis Core can:

```txt
- validate the approved route
- execute the venue CPI
- verify actual token balance deltas
- enforce min_out
- measure compute and account usage
```

### Phase 5: Mainnet Candidate Review

Mainnet Candidate Review uses Devnet evidence to decide whether Axis is ready to move toward mainnet contract migration and product launch.

Detailed mainnet readiness requirements are defined separately in `15-mainnet-readiness-requirements.md`.

At minimum, mainnet candidate review should reference:

```txt
- Devnet validation results
- known issues
- unresolved blockers
- partner demo feedback
- app integration readiness
- venue integration status
- security and failure test results
```

## 8. Requirements

### DEVNET-001: Devnet must validate the core DTF lifecycle

Devnet must prove that Axis Core can create and operate a minimal reserve-backed DTF lifecycle.

Acceptance criteria:

```txt
- a 2-asset DTF market can be created
- a user can mint DTF tokens with USDC or test USDC
- the execution path moves real SPL tokens
- reserve balances increase based on actual received assets
- minted DTF amount is based on actual added value
- a user can redeem DTF tokens
- reserve assets are unwound back to USDC or test USDC
- redeemed DTF tokens are burned or otherwise removed from circulating supply
```

### DEVNET-002: Devnet accounting must use actual balance deltas

Devnet must validate the same accounting model required for production.

Acceptance criteria:

```txt
- initial NAV is 1 USDC when total supply is zero
- pre-trade NAV is used for mint calculation
- actual reserve balance deltas are used as accounting truth
- actual USDC balance deltas are used for redeem output accounting
- quotes are not used as final accounting truth
- target weights are not treated as actual reserve balances
```

### DEVNET-003: Devnet must not depend on existing public Devnet DEX liquidity

Functional Devnet must not require existing public Devnet DEX pools or liquidity to be available.

Acceptance criteria:

```txt
- controlled test assets may be used
- controlled test USDC may be used
- controlled liquidity may be used
- controlled approved routes may be used
- public Devnet DEX pool availability is not a blocker for Core Functional Devnet
```

### DEVNET-004: Controlled CPI adapter may be used under strict accounting requirements

Functional Devnet may use a controlled CPI adapter if it preserves the same accounting invariants required for production.

Acceptance criteria:

```txt
- Axis Core invokes the adapter through real CPI
- the adapter moves real SPL tokens
- reserve balances change through real token transfers
- USDC or test USDC output changes through real token transfers
- Axis Core verifies pre/post token balances
- mock accounting is not accepted
```

### DEVNET-005: Devnet must test required failure cases

Devnet must prove that unsafe or invalid actions fail.

Acceptance criteria:

```txt
- min_out failure reverts
- disabled asset mint fails
- disabled route fails
- wrong input mint fails
- wrong output mint fails
- wrong reserve account fails
- paused market blocks mint
- exit-only configuration can block new mint while preserving redeem when safe
```

### DEVNET-006: Devnet must enforce all-or-nothing execution

Mint and redeem must not partially succeed.

Acceptance criteria:

```txt
- failed mint validation reverts the entire mint transaction
- failed redeem validation reverts the entire redeem transaction
- failed CPI execution reverts the entire transaction
- failed min_out check reverts the entire transaction
- DTF tokens are not minted if reserve accounting fails
- DTF tokens are not burned if redeem accounting fails
```

### DEVNET-007: Devnet must support app-level validation before partner demos

Devnet must be usable from the product interface before partner-facing demos.

Acceptance criteria:

```txt
- the app can create or load a Devnet DTF market
- the app can submit mint transactions
- the app can submit redeem transactions
- the app can display reserve balances
- the app can display NAV
- the app clearly indicates the Devnet environment
```

Detailed app requirements are defined in `13-app-integration-requirements.md`.

### DEVNET-008: Devnet must support partner-facing demo readiness

Devnet must support BD conversations with external projects.

Acceptance criteria:

```txt
- at least one demo DTF can be shown end-to-end
- create, mint, and redeem transaction links can be shared
- underlying reserve backing can be shown
- the demo clearly communicates that the DTF is not a frontend-only basket
```

Detailed partner demo requirements are defined in `14-partner-bd-demo-requirements.md`.

### DEVNET-009: Venue Integration Devnet must be tracked separately

External venue integration must be validated separately from Core Functional Devnet.

Acceptance criteria:

```txt
- Core Functional Devnet is not blocked by Orca, Raydium, PumpSwap, or Meteora integration
- venue-specific CPI research is tracked separately
- venue readiness requires actual CPI execution and balance delta verification
- venue readiness cannot be inferred only from off-chain route availability
```

### DEVNET-010: Devnet must produce mainnet readiness evidence

Devnet results must be useful for deciding whether Axis is ready to move toward mainnet.

Acceptance criteria:

```txt
- validation results are documented
- known issues are listed
- unresolved blockers are tracked
- partner demo feedback is recorded where relevant
- app integration status is recorded
- venue integration status is recorded
- mainnet candidate review can reference Devnet evidence
```

## 9. Out of Scope

The following are out of scope for Core Functional Devnet:

```txt
- dependency on existing public Devnet DEX liquidity
- production venue adapter requirement
- split routing
- routed SOL intermediate paths
- automatic rebalance
- Titan integration
- production fee model
- full 500 asset universe
- production oracle integration
- StockToken production pricing
- current repository gap analysis
```

The following may be addressed in later phases:

```txt
- Orca Whirlpool CPI integration
- Raydium CPMM CPI integration
- PumpSwap CPI integration
- Meteora DLMM CPI integration
- 3 to 5 asset execution feasibility
- public/internal partner testing
- mainnet migration checklist
```

## 10. Open Questions

The following decisions remain open:

```txt
- exact minimum Devnet validation window
- first production venue adapter
- whether 3 to 5 asset DTF execution must be supported before mainnet
- whether Venue Integration Devnet must pass before general launch
- which test assets should be used for Core Functional Devnet
- whether test USDC or canonical Devnet USDC should be used
- partner demo asset selection
- final mainnet migration timeline
```

Candidate minimum validation windows:

```txt
- 7 days
- 14 days
- 21 to 30 days
```

## 11. Issue Candidates

```txt
- Define Devnet rollout phases
- Implement controlled CPI adapter for Functional Devnet
- Define controlled test asset setup
- Define controlled test USDC setup
- Define Devnet approved route setup
- Implement Devnet create/mint/redeem end-to-end test
- Implement Devnet failure case tests
- Document Devnet validation results
- Research public Devnet DEX pool availability
- Research Orca Whirlpool Devnet CPI feasibility
- Research Raydium CPMM Devnet CPI feasibility
- Research PumpSwap Devnet CPI feasibility
- Research Meteora DLMM Devnet CPI feasibility
- Define App Integrated Devnet requirements
- Define Partner Demo Devnet requirements
- Define Mainnet Candidate Review requirements
```