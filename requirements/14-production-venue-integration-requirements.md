# Production Venue Integration Requirements

## 1. Overview

Axis v1 must support production-grade CPI execution paths for DTF mint and redeem.

This document defines the venue integration requirements required for mainnet readiness.

`05-swap-cpi-execution-requirements.md` defines the general swap/CPI execution model.

This document focuses specifically on production venue readiness.

Axis is not a general-purpose DEX aggregator.

Axis Core may rely on off-chain systems for route discovery, quote building, and account assembly, but Axis Core must validate and execute approved routes on-chain.

Core boundary:

```txt
Route discovery / quote / account assembly = off-chain
Execution / validation / accounting        = on-chain
```

Mainnet readiness requires at least two production venue integration paths:

```txt
1. Orca Whirlpool
2. Raydium CPMM fallback
```

Controlled adapters may be used for deterministic local tests, but controlled adapters are not sufficient for mainnet readiness.

Axis v1 should initially prefer direct USDC routes:

```txt
Mint:   USDC -> target asset
Redeem: target asset -> USDC
```

Split routing and arbitrary multi-hop routing are out of scope for v1 unless explicitly approved later.

PumpSwap is a high-priority future venue candidate because meme and long-tail assets may be important for Axis PMF.

However, the minimum production venue launch gate is:

```txt
- Orca Whirlpool
- Raydium CPMM fallback
```

## 2. Goals

The goals of production venue integration are:

```txt
- prove Axis can execute real CPI swaps against production venue programs
- prove mint and redeem can move real SPL tokens
- prove actual token balance deltas can be verified
- prove min_out / min_usdc_out can be enforced
- prove invalid venue accounts are rejected
- measure compute usage and account limits
- validate 5-asset mint execution before mainnet launch
- validate 5-asset redeem execution before mainnet launch
- document venue-specific risks before mainnet launch
- avoid dependence on a single execution venue
```

## 3. Non-Goals

This document does not require:

```txt
- becoming a general-purpose aggregator
- supporting arbitrary route graphs
- supporting split routing
- supporting unlimited multi-hop routing
- supporting every Solana DEX venue before launch
- supporting every long-tail asset at launch
- treating SDK quote availability as CPI readiness
- treating Jupiter route availability as Axis CPI readiness
```

## 4. Venue Readiness Model

A venue is Axis-ready only when Axis Core can validate and execute the route safely.

SDK quote availability is not sufficient.

Jupiter route availability is not sufficient.

A production venue path is considered ready only when Axis can:

```txt
- validate the venue program id
- validate the approved route
- validate pool / market / venue accounts
- validate input mint
- validate output mint
- validate execution direction
- execute CPI successfully
- move real tokens
- verify actual token balance deltas
- enforce min_out or min_usdc_out
- reject invalid venue accounts
- reject invalid route accounts
- measure compute usage
- measure account usage
- pass local integration tests
- pass mainnet-fork or cloned-account tests
- document venue-specific risks
```

## 5. Required Production Venue Paths

Axis v1 mainnet readiness requires at least two production venue paths.

Required paths:

```txt
1. Orca Whirlpool
2. Raydium CPMM fallback
```

The first production venue candidate is Orca Whirlpool.

The fallback production venue candidate is Raydium CPMM.

Both Orca Whirlpool and Raydium CPMM must validate mint and redeem paths.

The purpose of requiring two paths is to avoid making Axis launch readiness dependent on a single execution venue.

If one venue becomes temporarily unsuitable due to CPI, compute, account, liquidity, or integration constraints, the fallback path should preserve the ability to continue validation and launch preparation.

## 6. Controlled Adapter Policy

Controlled adapters are allowed for local and integration testing.

Controlled adapters may be used to validate Axis Core accounting invariants.

Controlled adapters are useful for testing:

```txt
- CPI dispatch flow
- real SPL token movement
- actual balance delta accounting
- min_out success and failure behavior
- all-or-nothing mint behavior
- all-or-nothing redeem behavior
- fee and reserve separation
```

However:

```txt
Controlled adapter success does not satisfy production venue readiness.
```

Controlled adapters validate Axis accounting invariants.

Production venue adapters validate launch execution readiness.

## 7. Route Scope Policy

Axis v1 should initially prefer direct USDC routes.

For mint:

```txt
USDC -> target asset
```

For redeem:

```txt
target asset -> USDC
```

This keeps v1 route validation, slippage protection, and balance delta accounting simpler.

Split routing and arbitrary multi-hop routing are out of scope for v1.

A non-direct route may only be supported if it is explicitly approved as an `ApprovedRoute` and documented with venue-specific risk notes.

Examples of non-direct routes that require explicit approval:

```txt
- asset -> SOL -> USDC
- USDC -> SOL -> asset
- multi-hop routes through intermediate tokens
- venue-specific routing paths with non-obvious intermediate assets
```

The default v1 route assumption is direct USDC liquidity.

## 8. Requirements

### VENUE-001: Mainnet readiness must require at least two production venue paths

Axis v1 must validate at least two production venue integration paths before mainnet launch.

Required paths:

```txt
1. Orca Whirlpool
2. Raydium CPMM fallback
```

Acceptance criteria:

```txt
- Orca Whirlpool CPI path is tested
- Raydium CPMM fallback CPI path is tested
- both paths execute real token movement
- both paths verify actual token balance deltas
- both paths enforce min_out or min_usdc_out
- both paths measure compute usage
- both paths measure account usage
- both paths pass local integration tests
- both paths pass mainnet-fork or cloned-account tests
- both paths document venue-specific risks
```

### VENUE-002: Orca Whirlpool must be the first production venue candidate

Orca Whirlpool is the first production venue candidate for Axis v1.

The Orca integration must validate SwapV2 CPI feasibility.

Acceptance criteria:

```txt
- Orca Whirlpool program id is validated
- Whirlpool accounts are validated
- token vault accounts are validated
- input mint is validated
- output mint is validated
- SwapV2 CPI succeeds in local tests
- SwapV2 CPI succeeds in mainnet-fork or cloned-account tests
- actual balance deltas are verified
- invalid Whirlpool accounts fail safely
```

### VENUE-003: Raydium CPMM must be the fallback production venue candidate

Raydium CPMM is the fallback production venue candidate for Axis v1.

The Raydium CPMM integration must validate CPI feasibility and fallback execution readiness.

Acceptance criteria:

```txt
- Raydium CPMM program id is validated
- CPMM pool accounts are validated
- pool vault accounts are validated
- input mint is validated
- output mint is validated
- Raydium CPMM CPI succeeds in local tests
- Raydium CPMM CPI succeeds in mainnet-fork or cloned-account tests
- actual balance deltas are verified
- invalid CPMM accounts fail safely
```

### VENUE-004: Both required venues must validate mint and redeem paths

Both Orca Whirlpool and Raydium CPMM must validate mint and redeem execution paths.

Acceptance criteria:

```txt
- Orca Whirlpool mint path is validated
- Orca Whirlpool redeem path is validated
- Raydium CPMM mint path is validated
- Raydium CPMM redeem path is validated
- mint path validates USDC -> asset execution
- redeem path validates asset -> USDC execution
- both mint and redeem paths enforce min_out / min_usdc_out
- both mint and redeem paths verify actual balance deltas
```

### VENUE-005: Controlled adapter may be used for local validation

Controlled adapters may be used for deterministic local tests.

Acceptance criteria:

```txt
- controlled adapter can be used in local tests
- controlled adapter can move real SPL tokens
- controlled adapter can validate actual balance delta accounting
- controlled adapter can test min_out failure behavior
- controlled adapter can test transaction rollback behavior
```

### VENUE-006: Controlled adapter must not satisfy production readiness

Controlled adapter tests must not be counted as production venue readiness.

Acceptance criteria:

```txt
- controlled adapter tests do not satisfy VENUE-001
- controlled adapter tests do not satisfy the production venue mainnet gate
- production venue CPI tests are required before mainnet launch
- venue-specific compute usage is measured separately
- venue-specific account usage is measured separately
- venue-specific risks are documented separately
```

### VENUE-007: Venue route must be explicitly approved

Axis must only execute routes that are explicitly approved.

Acceptance criteria:

```txt
- ApprovedRoute exists -> pass
- ApprovedRoute missing -> fail
- ApprovedRoute disabled -> fail
- route venue type matches expected venue
- route direction matches expected operation
- route input mint matches expected input mint
- route output mint matches expected output mint
```

### VENUE-008: V1 production routes should prefer direct USDC routes

Axis v1 production routes should prefer direct USDC routes.

For mint:

```txt
USDC -> target asset
```

For redeem:

```txt
target asset -> USDC
```

Split routing and arbitrary multi-hop routing are out of scope for v1.

A non-direct route may only be supported if it is explicitly approved as an `ApprovedRoute` and documented with venue-specific risk notes.

Acceptance criteria:

```txt
- direct USDC -> asset mint route passes if approved
- direct asset -> USDC redeem route passes if approved
- split route fails
- arbitrary multi-hop route fails
- unsupported SOL intermediate route fails
- any non-direct route must be explicitly approved before use
- any non-direct route must include venue-specific risk documentation
```

### VENUE-009: Venue program id must be validated

Axis must validate the venue program id before CPI execution.

Acceptance criteria:

```txt
- approved venue program id passes
- unknown program id fails
- disabled venue program id fails
- program id must match route venue type
```

### VENUE-010: Venue accounts must be validated

Axis must validate venue-specific accounts before CPI execution.

Acceptance criteria:

```txt
- required venue accounts are present
- pool or market account matches ApprovedRoute
- token vault accounts match expected pool
- input token account is validated
- output token account is validated
- wrong pool account fails
- wrong vault account fails
- wrong token account fails
```

### VENUE-011: Venue execution must validate input and output mints

Axis must verify input and output mints for each venue execution.

Acceptance criteria:

```txt
- mint route supports USDC -> target asset
- redeem route supports target asset -> USDC
- input mint mismatch fails
- output mint mismatch fails
- Token / Token-2022 compatibility is validated where applicable
```

### VENUE-012: Venue execution must enforce min_out

Venue execution must enforce minimum output thresholds.

For mint:

```txt
actual_received_i >= min_out_i
```

For redeem:

```txt
actual_usdc_received >= min_usdc_out
```

Acceptance criteria:

```txt
- actual output >= min_out passes
- actual output < min_out fails
- quote output alone cannot satisfy min_out
- missing min_out fails unless explicitly allowed in test-only environment
```

### VENUE-013: Venue accounting must use actual balance deltas

Venue execution must return actual output based on balance deltas.

Acceptance criteria:

```txt
- pre-execution balances are captured
- post-execution balances are captured
- actual_received is computed from balance delta
- quote output is not used as accounting truth
- actual_received is passed into mint or redeem accounting
```

### VENUE-014: Venue execution must be all-or-nothing

Venue execution must not partially succeed in a way that breaks DTF accounting.

Acceptance criteria:

```txt
- failed CPI execution reverts the parent transaction
- failed min_out check reverts the parent transaction
- failed venue validation reverts the parent transaction
- failed mint does not mint DTF tokens
- failed redeem does not permanently burn DTF tokens
- failed execution does not leave accounting state inconsistent
```

### VENUE-015: Compute usage must be measured per venue

Compute usage must be measured for each production venue path.

Acceptance criteria:

```txt
- Orca Whirlpool compute usage is measured
- Raydium CPMM compute usage is measured
- compute usage is measured for single-asset execution
- compute usage is measured for multi-asset DTF execution
- compute usage is measured for 5-asset mint
- compute usage is measured for 5-asset redeem
- compute risk is documented before mainnet launch
```

### VENUE-016: Account usage must be measured per venue

Account usage must be measured for each production venue path.

Acceptance criteria:

```txt
- Orca Whirlpool account usage is measured
- Raydium CPMM account usage is measured
- account count is measured for single-asset execution
- account count is measured for multi-asset DTF execution
- account count is measured for 5-asset mint
- account count is measured for 5-asset redeem
- account limit risk is documented before mainnet launch
```

### VENUE-017: Five-asset mint and redeem execution must be validated before mainnet launch

Axis v1 supports DTF markets with up to 5 underlying assets.

Therefore, production venue readiness must validate 5-asset mint and redeem execution before mainnet launch.

Acceptance criteria:

```txt
- 2-asset DTF mint execution is tested
- 5-asset DTF mint execution is tested
- 2-asset DTF redeem execution is tested
- 5-asset DTF redeem execution is tested
- 5-asset mint verifies actual balance deltas for all reserve assets
- 5-asset redeem verifies actual USDC balance delta
- 5-asset mint enforces min_out per asset
- 5-asset redeem enforces min_usdc_out
- compute usage is measured for 5-asset mint
- compute usage is measured for 5-asset redeem
- account usage is measured for 5-asset mint
- account usage is measured for 5-asset redeem
- any compute or account limitation must be resolved before mainnet launch
```

### VENUE-018: Production venue tests must include invalid account cases

Venue tests must include invalid account and spoofing cases.

Acceptance criteria:

```txt
- wrong venue program id fails
- wrong pool account fails
- wrong token vault account fails
- wrong input token account fails
- wrong output token account fails
- wrong input mint fails
- wrong output mint fails
- wrong route direction fails
```

### VENUE-019: Venue-specific risks must be documented

Each production venue integration must document venue-specific risks.

Risk areas should include:

```txt
- CPI account requirements
- compute usage
- account usage
- token program compatibility
- liquidity availability
- route direction constraints
- pool state assumptions
- failure behavior
- upgrade or program risk
```

Acceptance criteria:

```txt
- Orca Whirlpool risk notes are documented
- Raydium CPMM risk notes are documented
- unresolved risks are listed before mainnet launch
```

### VENUE-020: SDK quote availability must not imply Axis readiness

SDK or aggregator quote availability must not be treated as production venue readiness.

Acceptance criteria:

```txt
- Jupiter route availability does not auto-register ApprovedRoute
- SDK quote success does not auto-register ApprovedRoute
- quote success does not prove CPI feasibility
- quote success does not prove Axis account validation correctness
- quote success does not prove balance delta accounting correctness
```

### VENUE-021: Execution spread must not be recorded as Axis fee

Execution spread, slippage, and price impact must remain separate from Axis creator/protocol fees.

Acceptance criteria:

```txt
- venue spread is not recorded as creator fee
- slippage is not recorded as protocol fee
- price impact is not recorded as Axis fee
- mint fee is the only explicit creator/protocol fee in v1
- redeem has no explicit Axis exit fee
```

### VENUE-022: PumpSwap should be prioritized for meme and long-tail asset coverage

PumpSwap should be treated as a high-priority venue candidate after the required Orca Whirlpool and Raydium CPMM production paths.

Memecoin and long-tail asset coverage may be important for Axis PMF.

PumpSwap is not part of the minimum two-path mainnet readiness gate unless explicitly upgraded later, but the design should avoid blocking future PumpSwap integration.

Acceptance criteria:

```txt
- PumpSwap is tracked as a high-priority venue candidate
- PumpSwap readiness must use the same production venue criteria before activation
- PumpSwap integration must validate program id
- PumpSwap integration must validate pool or venue accounts
- PumpSwap integration must validate input and output mints
- PumpSwap integration must execute CPI successfully before production use
- PumpSwap integration must verify actual balance deltas
- PumpSwap integration must enforce min_out
- PumpSwap integration must document venue-specific risks
```

### VENUE-023: Other future venues must meet the same readiness criteria

Raydium CLMM, Meteora DLMM, and other future venues may be evaluated later.

Future venues must satisfy the same readiness criteria before being considered production-ready.

Acceptance criteria:

```txt
- future venue has explicit ApprovedRoute support
- future venue validates program id
- future venue validates venue accounts
- future venue executes CPI successfully
- future venue verifies actual balance deltas
- future venue enforces min_out
- future venue measures compute usage
- future venue measures account usage
- future venue documents venue-specific risks
```

## 9. Venue Priority

Current production venue priority:

```txt
1. Orca Whirlpool
2. Raydium CPMM fallback
3. PumpSwap high-priority candidate for meme / long-tail coverage
4. Raydium CLMM
5. Meteora DLMM
```

Mainnet readiness requires:

```txt
- Orca Whirlpool production path validation
- Raydium CPMM fallback path validation
- both venues validating mint execution
- both venues validating redeem execution
- 5-asset mint execution success
- 5-asset redeem execution success
- local integration test success
- mainnet-fork or cloned-account test success
```

PumpSwap is strategically important for PMF, but is not required as part of the minimum two-path mainnet launch gate unless later explicitly decided.

## 10. Required Test Scenarios

### 10.1 Controlled Adapter Tests

```txt
- controlled adapter moves real SPL tokens
- controlled adapter verifies actual balance deltas
- controlled adapter enforces min_out
- controlled adapter min_out failure reverts
- controlled adapter failed execution reverts parent transaction
```

### 10.2 Orca Whirlpool Tests

```txt
- verify Orca Whirlpool program id
- validate Whirlpool account
- validate token vault accounts
- execute SwapV2 CPI in local test
- execute SwapV2 CPI in mainnet-fork or cloned-account test
- verify output balance delta
- enforce min_out
- measure compute usage
- measure account usage
- invalid Whirlpool account fails
- wrong input mint fails
- wrong output mint fails
- wrong route direction fails
```

### 10.3 Raydium CPMM Tests

```txt
- verify Raydium CPMM program id
- validate CPMM pool account
- validate pool vault accounts
- execute Raydium CPMM CPI in local test
- execute Raydium CPMM CPI in mainnet-fork or cloned-account test
- verify output balance delta
- enforce min_out
- measure compute usage
- measure account usage
- invalid pool account fails
- wrong input mint fails
- wrong output mint fails
- wrong route direction fails
```

### 10.4 Five-Asset Execution Tests

```txt
- 5-asset mint using approved production venue paths
- 5-asset redeem using approved production venue paths
- 5-asset mint verifies balance deltas for all reserve assets
- 5-asset redeem verifies actual USDC balance delta
- 5-asset mint enforces min_out per asset
- 5-asset redeem enforces min_usdc_out
- 5-asset mint measures compute usage
- 5-asset redeem measures compute usage
- 5-asset mint measures account usage
- 5-asset redeem measures account usage
- 5-asset execution preserves all-or-nothing behavior
```

### 10.5 Route Validation Tests

```txt
- missing ApprovedRoute fails
- disabled ApprovedRoute fails
- wrong venue fails
- wrong pool fails
- wrong vault fails
- wrong direction fails
- wrong input mint fails
- wrong output mint fails
- unsupported split route fails
- unsupported arbitrary multi-hop route fails
- unsupported SOL intermediate route fails
```

### 10.6 Failure Tests

```txt
- wrong venue program id fails
- wrong pool account fails
- wrong vault account fails
- wrong direction fails
- wrong input mint fails
- wrong output mint fails
- actual output below min_out fails
- quote output is ignored as accounting truth
- execution spread is not recorded as Axis fee
- failed venue execution reverts parent mint or redeem
```

## 11. Mainnet Readiness Gate

Production venue integration is a mainnet readiness gate.

Axis v1 must not be considered mainnet-ready unless:

```txt
- Orca Whirlpool production path is validated
- Raydium CPMM fallback path is validated
- both venues validate mint execution
- both venues validate redeem execution
- both venues execute real token movement
- both venues verify actual balance deltas
- both venues enforce min_out / min_usdc_out
- both venues measure compute usage
- both venues measure account usage
- both venues pass local integration tests
- both venues pass mainnet-fork or cloned-account tests
- 5-asset mint execution succeeds before mainnet launch
- 5-asset redeem execution succeeds before mainnet launch
- venue-specific risks are documented
```

Controlled adapter tests are useful but insufficient for this gate.

## 12. Out of Scope

The following are out of scope for the initial production venue readiness gate unless later explicitly decided:

```txt
- arbitrary route graphs
- split routing
- automatic Jupiter route registration
- automatic SDK quote-based route approval
- all Solana DEX venues
- all long-tail assets
- unsupported SOL intermediate routes
- venue integrations without balance delta accounting
```

## 13. Open Questions

```txt
- Exact Orca Whirlpool account validation checklist
- Exact Raydium CPMM account validation checklist
- Whether SOL intermediate routes should be supported after launch
- Whether PumpSwap should be upgraded from high-priority candidate to required launch gate
- Exact PumpSwap integration timeline
```

## 14. Issue Candidates

```txt
- Define production venue readiness criteria
- Define controlled adapter test scope
- Implement controlled adapter for local tests
- Implement Orca Whirlpool CPI spike
- Implement Orca Whirlpool account validation
- Implement Orca Whirlpool invalid account tests
- Measure Orca Whirlpool compute usage
- Measure Orca Whirlpool account usage
- Implement Raydium CPMM fallback CPI spike
- Implement Raydium CPMM account validation
- Implement Raydium CPMM invalid account tests
- Measure Raydium CPMM compute usage
- Measure Raydium CPMM account usage
- Add local production venue integration tests
- Add mainnet-fork production venue integration tests
- Add 5-asset mint execution tests
- Add 5-asset redeem execution tests
- Add all-or-nothing venue execution tests
- Track PumpSwap as high-priority venue candidate
- Document Orca Whirlpool venue-specific risks
- Document Raydium CPMM venue-specific risks
- Document SDK quote vs Axis CPI readiness distinction
- Document execution spread vs Axis fee distinction
```
